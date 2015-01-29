## Overview

Cupboard is a way to manage persistence in a sqlite instance for your app. It was written by [Hugo Visser] (https://twitter.com/botteaap). His talk on the library can be found [here](https://skillsmatter.com/skillscasts/4806-simple-persistence-with-cupboard). It's a small library, simple to use, and it was designed specifically for Android unlike ORMlite.

Using the Cupboard persistence library makes managing client-side models extremely easy in simple cases. For more advanced or custom cases, you can use [[SQLiteOpenHelper|Local-Databases-with-SQLiteOpenHelper]] to manage the database communication directly. However, keep in mind that Cupboard was written with the intention to abstract away a lot of boilerplate and reused code that would go into making SQLiteOpenHelper function. 

Cupboard works like an **Object Relational Mapper**(ORM) by mapping java classes to database tables and mapping java class member variables to the table columns. Through this process, **each table** maps to a **Java model** and **the columns** in the table represent the respective **data fields**. Similarly, each row in the database represents a particular object. This allows us to create, modify, delete and query our SQLite database using instantiated objects instead of writing SQL every time.

For example, a "Tweet" model would be mapped to a "Tweet" table in the database. The Tweet model might have a "body" field that maps to a body column in the table and a "timestamp" field that maps to a timestamp column. Through this process, each row would map to a particular tweet. 

However, it is not a true ORM as it does not support relating one object to a nested object inherently.

### Installation

To install manually, you can [download the latest JAR file](https://search.maven.org/remote_content?g=nl.qbusict&a=cupboard&v=LATEST)

To install Cupboard with Maven, simply add the line

```xml
<dependency>
    <groupId>nl.qbusict</groupId>
    <artifactId>cupboard</artifactId>
    <version>(insert latest version)</version>
</dependency>
```

To install Cupboard with Gradle, simply add the line

```groovy
compile 'nl.qbusict:cupboard:(insert latest version)'
```

to the dependencies section of your app's build.gradle file.

You should now have ahold of the files you need for Cupboard.


### Configuration

Next, we'll setup a custom SQLiteOpenHelper. This is a standard object in the Android framework that assists in dealing with SQLite databases. For now, we'll just create the object and register one Plain Old Java Object (POJO) in our database: `Bunny`
```java
import android.content.Context;
import android.database.sqlite.SQLiteDatabase;
import android.database.sqlite.SQLiteOpenHelper;
import static nl.qbusict.cupboard.CupboardFactory.cupboard;

public class PracticeDatabaseHelper extends SQLiteOpenHelper {

    private static final String DATABASE_NAME = "cupboardTest.db";
    private static final int DATABASE_VERSION = 1;

    public PracticeDatabaseHelper(Context context) {
        super(context, DATABASE_NAME, null, DATABASE_VERSION);
    }

    static {
        // register our models
        cupboard().register(Bunny.class);
    }

    @Override
    public void onCreate(SQLiteDatabase db) {
        // this will ensure that all tables are created
        cupboard().withDatabase(db).createTables();
        // add indexes and other database tweaks in this method if you want

    }

    @Override
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
        // this will upgrade tables, adding columns and new tables.
        // Note that existing columns will not be converted
        cupboard().withDatabase(db).upgradeTables();
        // do migration work if you have an alteration to make to your schema here

    }

}
```

After this, somewhere in your app(most likely your Application class or your Main Activity), you will have to instantiate your DatabaseHelper. 
```java
PracticeDatabaseHelper dbHelper = new PracticeDatabaseHelper(this);
db = dbHelper.getWritableDatabase();
```

Now, with your database instantiated, you are ready to use Cupboard.

### Usage

#### Defining Models

First, we define our models by creating POJO's to represent them. However, a POJO for cupboard must always contain a variable of type `Long` called `_id`. This will serve as the index for the pojo within the SQLite table. 

```java
public class Bunny {

    public Long _id; // for cupboard
    public String name; // bunny name
    public Integer cuteValue ; // bunny cuteness


    public Bunny() {
        this.name = "noName";
        this.cuteValue = 0;
    }

    public Bunny(String name, Integer cuteValue) {
        this.name = name;
        this.cuteValue = cuteValue;
    }
}
```

The name part of the class will be the name of the Table and the names of the variables will be those of the Columns, so make sure to use the SQLite naming conventions for those.

Then, as seen above in setting up the database, you must register the class in your DatabaseHelper's static constructor. 

```java
    static {
        // register our models
        cupboard().register(Bunny.class);
    }
```
##### Alternative names for columns and ignoring columns
There exists two annotations available to you by default to help with how cupboard writes field names to columns in your database. 

The first is `@Column()` and the second is `@Ignore()`

The column annotation allows you to choose an alternative name for a column, while the ignore annotation tells cupboard to ignore a field while creating the table. 
As an example, say I wanted to have the `cuteValue` field be underscore delineated and for there to be a non-persisted boolean `isAwake`. My Model code would now look like this:

```java
public class Bunny {

    public Long _id; // for cupboard
    public String name; // bunny name
    @Column("cute_value")
    public Integer cuteValue; // bunny cuteness
    @Ignore
    public Boolean isAwake;

    ...
```

#### CRUD Operations

Now we can create, modify and delete records for these models backed by SQLite:

#####Create items
```java
// Create a Bunny
Bunny bunny = ...
long id = cupboard().withDatabase(db).put(bunny);
```
If Bunny has it's _id field set, then any existing Bunny with that id will be replaced. If _id is null then a new Bunny will be created in the table. In both cases the corresponding id is returned from put().

#####Read items

We can query records with a simple query syntax `get` method.

```java
Bunny bunny = cupboard().withDatabase(db).get(Bunny.class, 12L);
```

This will return the bunny with _id = 12. If no such bunny exists, then it will return null. 

We can also use the `get` command.
```java
// get the first bunny in the result
Bunny bunny = cupboard().withDatabase(db).query(Bunny.class).get();
// Get the cursor for this query
Cursor bunnies = cupboard().withDatabase(db).query(Bunny.class).getCursor();
try {
  // Iterate Bunnys
  QueryResultIterable<Bunny> itr = cupboard().withDatabase(db).query(Bunny.class).iterate();
  for (Bunny bunny : itr) {
    // do something with bunny
  }
} finally {
  // close the cursor
  itr.close();
}
// Get the first matching Bunny with name Max
Bunny bunny = cupboard().withDatabase(db).query(Bunny.class).withSelection( "name = ?", "Max").get();
```

#####Update items

To update an item, simply retrieve it from the database, update it's accessible fields, and use `put()` to get it back into the database

```java
cupboard().withDatabase(db).put(b);
```

However, if you're doing a larger update of items, you can use the `update()` command. This is done with a [ContentValues](http://developer.android.com/reference/android/content/ContentValues.html) object.

```java
//Let's consider the problem in which we've forgotten to capitalize the 'M' in all bunnies named 'Max'
ContentValues values = new ContentValues(1);
values.put("name", "Max")
// update all books where the title is 'android'
cupboard().withDatabase(db).update(Book.class, values, "name = ?", "max");
```

#####Delete items
```java
// Delete an item, in this case the item where _id = 12
cupboard().withDatabase(db).delete(Bunny.class, 12L);

// by passing in the item
cupboard().withDatabase(db).delete(bunny);
// or by passing in a query. This will delete all bunnies named "Max"
cupboard().withDatabase(db).delete(Bunny.class, "name = ?", "Max");
```

That's Cupboard in a nutshell.


#### Migrations


If you need to add a field to your an existing model or need to add a new model to be tracked you have two options, you can either uninstall the app and then reinstall it with the new models and fields registered as you normally would, or you you'll need to write a migration to add the column to the table that represents your model. You'll need to write a migration if you've already deployed the app to production. It would be considered poor practice to ask users to uninstall your app before installing it once more. Here's how one might go about said migration:

1. Add a new field to your existing model:
  ```java
  public class Bunny {

    public Long _id; // for cupboard
    public String name; // bunny name
    public String furColor; // new String to store fur color.
    public Integer cuteValue ; // bunny cuteness
  
  ...
  ```

2. Change the database version the the AndroidManifest.xml's metadata. Increment by 1 from the last version:

  ```java
  public class PracticeDatabaseHelper extends SQLiteOpenHelper {
    private static final String DATABASE_NAME = "cupboardTest.db";
    private static final int DATABASE_VERSION = 2;
    ...
  ```

3. Write your migration script. You'll have to keep in mind all possibilities for migrations, e.g. are they migrating from version 1 to 3, 2 to 3, etc.

  ```java
  public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
        // This line adds new columns and tables from changes you've made in new or existing
        // registered models. 
        cupboard().withDatabase(db).upgradeTables();

        // After this, you can populate the new Columns or tables with default values if you choose
        if (newVersion == 2) {
            ContentValues cv = new ContentValues();
            cv.put("furColor", "black");
            cupboard().withDatabase(db).update(Bunny.class, cv);
        }
  }
  ```

## Sample Project

A sample project utilizing basic Cupboard CRUD functions and a migration schema has been constructed around this lesson. 
[It can be found here](https://github.com/koalahamlet/cupboardTestApp/tree/master/app/src/main).

Be sure to review the common questions below.

## Common Questions

> Question: How does Cupboard handle duplicate IDs? For example, I want to make sure no duplicate twitter IDs are inserted. Is there a way to specify a column is the primary key in the model?

The only way that cupboard knows if an object is the duplicate of another, by default, is if they share the same number in the _id field. 

However you can enable annotation support for `@Index` and `@CompositeIndex` like so

```java
Cupboard cupboard = new CupboardBuilder().useAnnotations().build();
```

For more information on the subject, refer to [the cupboard wiki entry on annotations](https://bitbucket.org/qbusict/cupboard/wiki/existingData)

> Question: How do you specify the data type (int, text)? Does Cupboard automatically know what the column type should be?

The type is inferred automatically from the type of the field.

> Question: How do I store dates into Cupboard?

Dates in Cupboard can be stored as Longs.

```java
long time = Calendar.getInstance().getTimeInMillis();
```
and then retrieved and displayed in whatever format you choose.

```java
String ISO_FORMAT = "yyyy-MM-dd'T'HH:mm:ss.SSS'Z'";
SimpleDateFormat sdf = new SimpleDateFormat(ISO_FORMAT);
dateString = sdf.format(time);
```

> Question: How do you represent a 1-1 relationship?

Cupboard is not a real ORM as it doesn't manage relations between objects, which keeps things simple.
You can write a system with corresponding ID's if you wish to have this functionality.

> Question: How do I delete all the records from a table?

```java
public static void clearAllBunnyData(db) {
    db.execSQL("DELETE FROM " + Bunny.class.getSimpleName());
}
```

## References

* [Cupboard bitbucket](https://bitbucket.org/qbusict/cupboard)
* [Cupboard Getting Started](https://bitbucket.org/qbusict/cupboard/wiki/GettingStarted)
* [Cupboard working with annotations and existing data](https://bitbucket.org/qbusict/cupboard/wiki/existingData)
* [Cupboard working with databases](https://bitbucket.org/qbusict/cupboard/wiki/withDatabase)
* [Cupboard talk] (https://skillsmatter.com/skillscasts/4806-simple-persistence-with-cupboard)