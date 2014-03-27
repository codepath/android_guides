## Overview

Using the ActiveAndroid ORM makes managing client-side models extremely easy in simple cases. For more advanced or custom cases, you can use [SQLiteOpenHelper](http://www.androidhive.info/2011/11/android-sqlite-database-tutorial/) to manage the database communication directly. But for simple model mapping from JSON, ActiveAndroid keeps things simple. 

### Installation

#### Get the Latest ActiveAndroid Library

1. Download the latest build of [ActiveAndroid](https://github.com/pardom/ActiveAndroid/archive/master.zip).

2. Unzip the `master.zip` file.

3. Open up the **Terminal** and navigate to the directory that was just unzipped.

4. Type `ant` to build the project and generate a project jar file.

5. Move the generated jar file from `dist/ActiveAndroid.jar` to the `libs` directory in your **project home**.

        mv dist/ActiveAndroid.jar <your_project_home>/libs/

#### Configure Your Application

Next, we need to configure the `application` node with the `name` property of `com.activeandroid.app.Application` to use the correct application class name within the `AndroidManifest.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    ...>
    <!- add the name property to your application node ->
    <application
        android:name="com.activeandroid.app.Application"
        android:icon="@drawable/ic_launcher"
        android:label="@string/app_name"
        android:theme="@style/AppTheme" >
        
        <!- add the following metadata for version and database name ->
        <meta-data
            android:name="AA_DB_NAME"
            android:value="RestClient.db" />
        <meta-data
            android:name="AA_DB_VERSION"
            android:value="1" />

        <activity
            android:name="com.codepath.apps.activities.MainActivity"
            android:noHistory="true"
            android:label="@string/app_name" >
            <!- ... ->
        </activity>
    </application>
</manifest>
```

Now you are ready to use ActiveAndroid. If you have an custom application class, check out more details for how to approach that in the [installation guide](https://github.com/pardom/ActiveAndroid/wiki/Getting-started).

### Usage

#### Defining Models

First, we define our models:

```java
import com.activeandroid.Model;

@Table(name = "Items")
public class Item extends Model {
    // This is how you avoid duplicates
    @Column(name = "remote_id", unique = true, onUniqueConflict = Column.ConflictAction.REPLACE)
    public int remoteId;
    @Column(name = "Name")
    public String name;
    @Column(name = "Category")
    public Category category;
    
    // Make sure to have a default constructor for every ActiveAndroid model
    public Item(){
       super();
    }
    
    public Item(String remoteId, String name, Category category){
        super();
        this.remoteId = remoteId;
        this.name = name;
        this.category = category;
    }
}

@Table(name = "Categories")
public class Category extends Model {
    // This is how you avoid duplicates
    @Column(name = "remote_id", unique = true, onUniqueConflict = Column.ConflictAction.REPLACE)
    public int remoteId;
    @Column(name = "Name")
    public String name;

    // Make sure to have a default constructor for every ActiveAndroid model
    public Category(){
       super();
    }

    // Used to return items from another table based on the foreign key
    public List<Item> items() {
        return getMany(Item.class, "Category");
    }
}
```

Note that **ActiveAndroid creates a local id (mId)** in addition to our manually managed remoteId (unique) which is the id on the server (for networked applications). 

#### CRUD Operations

Now we can create, modify and delete records for these models backed by SQLite:

```java
// Create a category
Category restaurants = new Category();
restaurants.remoteId = 1;
restaurants.name = "Restaurants";
restaurants.save();

// Create an item 
Item item = new Item();
item.remoteId = 1;
item.category = restaurants;
item.name = "Outback Steakhouse";
item.save();

// Deleting items
Item item = Item.load(Item.class, 1);
item.delete();
// or
new Delete().from(Item.class).where("remote_id = ?", 1).execute();
```

#### Querying Records

We can query records with:

```java
@Table(name = "Items")
public class Item extends Model {
    // ...
    public static List<Item> getAll(Category category) {
        // This is how you execute a query
        return new Select()
          .from(Item.class)
          .where("Category = ?", category.getId())
          .orderBy("Name ASC")
          .execute();
    }
}
```

That's ActiveAndroid in a nutshell. 

#### Official Reference Guides

Check these official reference guides for more detailed information:

 * [Install ActiveAndroid and perform initial setup](https://github.com/pardom/ActiveAndroid/wiki/Getting-started)
 * [Define your application models](https://github.com/pardom/ActiveAndroid/wiki/Creating-your-database-model)
 * [Persist data using save](https://github.com/pardom/ActiveAndroid/wiki/Saving-to-the-database)
 * [Query the local database](https://github.com/pardom/ActiveAndroid/wiki/Querying-the-database)
 * [Perform data schema migrations](https://github.com/pardom/ActiveAndroid/wiki/Schema-migrations)

Be sure to review the common questions below.

## Common Questions

> Problem: I am getting an "SQLiteException: no such table" error when I try to run the app

This is because ActiveAndroid only generates the schema if there is no existing database file. In order to "regenerate" the schema after creating a new model, the easiest way is to uninstall the app from the emulator and allow it to be fully re-installed. This is because this clears the database file and triggers ActiveAndroid to recreate the tables based on the annotated models in the project.

> Problem: I am getting a NullPointerException when trying to save a model

This is because ActiveAndroid needs you to save all objects separately. Before saving a tweet for example, be sure to save the associated user object first. So when you have a tweet that references a user be sure to `user.save()` before you call `tweet.save()` since storing the user requires the local id to be set and assigned as the foreign key for the tweet.

> Question: How does ActiveAndroid handle duplicate IDs?  For example, I want to make sure no duplicate twitter IDs are inserted.  Is there a way to specify a column is primary key in the model?

The solution is to mark the column recording the unique id of an object as a unique column acting as your pseudo primary key. As [explained here](https://github.com/pardom/ActiveAndroid/issues/22), the annotation is:

```java
@Table(name = "items")
public class SampleModel extends Model {
    // Ensure the remoteId must be unique. If a duplicate tries to save, replace the 
    @Column(name = "remote_id", unique = true, onUniqueConflict = Column.ConflictAction.REPLACE)
    private int remoteId;

    // ... set the remote id based on the json response
}
```

Make sure to **uninstall the app** afterward on the emulator to ensure the schema changes take effect.

> Question: I read somewhere that ActiveAndroid automatically creates another auto-increment ID column, is this true?  What field names should I avoid using?

This is true, see [creating your database model](https://github.com/pardom/ActiveAndroid/wiki/Creating-your-database-model) for more details. The column is called ["mId"](https://github.com/pardom/ActiveAndroid/blob/master/src/com/activeandroid/Model.java#L40)

> Question: How do you specify the data type (int, text)?  Does AA automatically know what the column type should be?

The type is inferred automatically from the type of the field.

> Question: How do I store dates into ActiveAndroid?

AA supports serializing Date fields automatically. Simply:

```java
@Column(name = "timestamp", index = true)
private Date timestamp; 
```

and the date will be serialized to SQLite. You can parse strings into a Date object using [SimpleDateFormat](http://docs.oracle.com/javase/7/docs/api/java/text/SimpleDateFormat.html):

```java
public void setDateFromString(String date) {
    SimpleDateFormat sf = new SimpleDateFormat("EEE MMM dd HH:mm:ss ZZZZZ yyyy");
    sf.setLenient(true);
    this.timestamp = sf.parse(date);
}
```

> Question: How do you represent a 1-1 relationship?  

Check out the [relationships section](https://github.com/pardom/ActiveAndroid/wiki/Creating-your-database-model#relationships) if you haven't yet. There are many ways to manage this. You can declare a type "User" and then that field will be associated with a [foreign key representing the user](https://github.com/pardom/ActiveAndroid/blob/master/src/com/activeandroid/Model.java#L135). You can manage this process by simply constructing and assigning the user object to a field of the parent object and then calling save on the parent. You should make sure to call save separately on the associated object in most cases.

> Question: How do I delete all the records from a table?

You can delete individual records using the `.delete` method and you can delete all records matching a particular condition with:

```java
new Delete().from(Item.class).execute(); // all records
// or based on a conditional clause
new Delete().from(Item.class).where("Id = ?", 1).execute();
```

This allows for bulk deletes.

> Question: Is it possible to do joins with ActiveAndroid? 

Joins are done using the query language AA provides: https://github.com/pardom/ActiveAndroid/blob/master/src/com/activeandroid/query/From.java read more here: https://github.com/pardom/ActiveAndroid/wiki/Querying-the-database

> Question: What are the best practices when interacting with the sqlite in Android, is ORM/DAO the way to go?

I've seen people use both SQLiteOpenHelper and several different ORMs. It's common to use the SQLiteOpenHelper in cases where an ORM breaks down or isn't necessary. Since Models are typically formed anyways though and persistence on Android in many cases can map very closely to objects, ORMs like AA can be helpful especially for simple mappings.

## References

* [ActiveAndroid Website](http://www.activeandroid.com/)
* [AA Getting Started](https://github.com/pardom/ActiveAndroid/wiki/Getting-started)
* [AA Models](https://github.com/pardom/ActiveAndroid/wiki/Creating-your-database-model)
* [AA Saving](https://github.com/pardom/ActiveAndroid/wiki/Saving-to-the-database)
* [AA Querying](https://github.com/pardom/ActiveAndroid/wiki/Querying-the-database)
* [AA Migrations](https://github.com/pardom/ActiveAndroid/wiki/Schema-migrations)
* [AA Pre-populated](https://github.com/pardom/ActiveAndroid/wiki/Pre-populated-databases)