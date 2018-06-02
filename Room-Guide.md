## Overview

Room is Google's new persistence library designed to make it easier to build offline apps.  It tries to expose APIs that can leverage the full power of SQL while still providing an abstraction layer for managing the data as Java objects.  It also works well seamlessly with Google's [Architecture Components library](https://developer.android.com/topic/libraries/architecture/) for building robust high-quality production apps and can also be used along with the [[Paging Library|Paging Library Guide]] for handling large data sets.

## Setup

The section below describes how to setup using Room.

### Gradle configuration

First, make sure you have the Google Maven repository added:

```gradle
allprojects {
    repositories {
        jcenter()
        google() // allow since Gradle v4.1
    }
}
```

Next, within your `app/build.gradle`, add Room to your dependency list.  We create a separate variable to store the version number to make it easier to change later:

```gradle
ext {
   roomVersion = "1.1.0"
}

dependencies {
  // Room for simple persistence with an ORM
  implementation "android.arch.persistence.room:runtime:$roomVersion"
  annotationProcessor "android.arch.persistence.room:compiler:$roomVersion"  
}
```

### Define schema locations

When compiling the code, the schemas for the table will be stored in a `schemas/` directory assuming this statement has been included your `app/build.gradle` file.  These schemas should be checked into your code based.


```gradle
android {
    defaultConfig {

        javaCompileOptions {
            annotationProcessorOptions {
                arguments = ["room.schemaLocation": "$projectDir/schemas".toString()]
            }
        }
    }

}
```

### Defining your Tables


Annotate your models with `@Entity`:

```java
import android.arch.persistence.room.Entity;

@Entity
public class Organization {
   // ... field definitions that map to columns go here ...
}
```

Next, we need to add annotations for each of the fields within our model class that will map to columns in the database table:

```java
import android.arch.persistence.room.ColumnInfo;
import android.arch.persistence.room.Entity;
import android.arch.persistence.room.PrimaryKey;

@Entity
public class Organization {

  @ColumnInfo
  @PrimaryKey(autoGenerate=true)
  Long id;

  @ColumnInfo
  String name;
}
```

**NOTE**: You must define at least one column to be the primary key.   You can also define composite primary keys by reviewing [this section](https://developer.android.com/training/data-storage/room/defining-data#primary-key).  Use the `autoGenerate=true` annotation to make the column auto-increment.  (Note that if you add this later, you need to update the schema of the database.)

### Defining Foreign Keys

Room allows foreign keys to be defined between objects, but explicitly disallows traversing these relations automatically.  For instance, trying to lookup an `Organization` object in `User` object in memory cannot be done.  Defining the constraints between these two tables however can still be done:


```java
@Entity(foreignKeys = @ForeignKey(entity=Organization.class, parentColumns="id", childColumns="organization_id"))
public class User  {
    @ColumnInfo
    @PrimaryKey(autoGenerate=true)
    Long id;

    @ColumnInfo
    String name;

    @ColumnInfo
    Long organization_id;
}
```

### Defining your data access Objects

We can declare the types of SQL queries we want to perform in our data access object (DAO) layer.  The following annotations can be used to get, insert, and delete a User object:

```java
@Dao
public interface UserDao {
  @Query("SELECT * FROM User where userId := :id")
  public User getById(int id);

  @Insert(onConflict = OnConflictStrategy.REPLACE)
  public Long insertUser(User user);

  @Delete
  public void deleteUser(User user);
}
```

### Creating the database

Create a `MyDatabase.java` file and annotate your class with the `@Database` decorator to declare your database.  It should contain both the name to be used for creating the table, as well as the version number. 

```java
// bump version number if your schema changes
@Database(entities={User.class, Organization.class}, version=1)
public abstract class MyDatabase extends RoomDatabase {
  // Declare your data access objects as abstract
  public abstract UserDao userDao();

  // Database name to be used
  public static final String NAME = "MyDataBase";
```

**Note**: if you decide to change the schema for any tables you create later, you will need to bump the version number.  The version number should always be incremented (and never downgraded) to avoid conflicts with older database versions.  Making schema changes will update the definitions in the `app/schemas` directory as setup in the [[previous step| Room-Guide#define-schema-locations]].  You will find the actual SQL definitions to create these tables there. 

### Instantiating Room

Next, we need to instantiate `Room` in a [[custom application class|Understanding-the-Android-Application-Class]].  If you do not have an `Application` object, create one in `MyApplication.java` as shown below:

```java
public class MyApplication extends Application {
    MyDatabase myDatabase;

    @Override
    public void onCreate() {
        super.onCreate();
        public class RestClientApp extends Application {
            // when upgrading versions, kill the original tables by using fallbackToDestructiveMigration()
            myDatabase = Room.databaseBuilder(this, MyDatabase.class, MyDatabase.NAME).fallbackToDestructiveMigration().build();
          }

        public MyDatabase getMyDatabase() {
            return myDatabase;
        }
    }
}
```

Modify your `AndroidManifest.xml` file to reference this Application object for the `android:name` property:

```xml
<application
        android:name=".MyApplication"
        ...>
  <!-- ... -->
</application>
```

## Basic CRUD operations

There are a few key aspects of Room to note that differ slightly from traditional object relational mapping (ORM) frameworks:

* Usually, ORM frameworks expose a limited subset of queries.  Tables are declared as Java objects, and the relations between these tables dictate the types of queries that can be performed.  In Room, SQL inserts, updates, deletes, and complex joins are declared as **Data Access Objects (DAO)**.  The data returned from these queries simply are mapped to a Java object that will hold this information in memory.  In this way, no assumptions are made about how the data can be accessed.  The table definitions are separate from the actual queries performed.

* ORMs typically handle one-to-one and one-to-many relationships by determining the relationship between tables and expose an interface or a set of APIs that perform the SQL queries behind the scenes.  Because of the [performance implications](https://youtu.be/MfHsPGQ6bgE?t=27m37s), Room requires handling these relationships explicitly.   The query needs to be defined as DAO objects, and the data returned will need to be mapped to Java objects.

* One additional change is that Room doesn't require defining your SQL tables as a single Java object.  It allows you to maintain encapsulation between objects by allowing other Java objects to be included as part of a single table.  
Basic creation, read, update, and delete (CRUD) statements need to be defined in your data access object (DAO).  One major difference is that queries cannot be done on the main thread.  In addition, the objects can be inserted with an [[AsyncTask|Creating-and-Executing-Async-Tasks]] or using the `runInTransaction() method`:

```java
final Organization organization = new Organization();
organization.setName("CodePath");

final User user = new User();
user.setName("John Doe");

// Get the DAO
final UserDao userDao = ((RestApplication) getApplicationContext()).getMyDatabase().userDao();

// Define the task
((RestApplication) getApplicationContext()).getMyDatabase().runInTransaction(new Runnable() {
  @Override
  public void run() {
    userDao.insertOrganization(organization);
    userDao.insertModel(user);
  }
});
```
### Querying Rows

All queries must be defined in the Data Access Object (DAO):

If we want to perform a User query that also retrieves the organization info into memory, we can do the following:
```java
// declare inner join here
  @Query("SELECT User.*, Organization.name AS organization_name FROM User INNER JOIN Organization " +
        "ON User.organization_id = Organization.id WHERE User.id = :id")
  public UserWithOrganization getWithOrgById(int id);
```

Notice how the organization name has been renamed using the SQL `AS` query.  This helps to sidestep naming conflicts since both the `User` and `Organization` both share the same field name.  We can then need to declare a regular Java object that includes both a `User` and the organization name:

```java
class UserWithOrganization {
   // @Embedded notation flattens the properties of the User object into the object, preserving encapsulation.
   @Embedded User user;

   // organization_name renamed during SELECT query w/ Organization.name AS organizaiton_name
   @ColumnInfo(name = "organization_name")
   String organizationName;

}
```

You can then access the properties of User within this object as we would as a normal Java object:

```java
UserWithOrganization userWithOrg = new UserWithOrganization();
User user = userWithOrg.user;
```

Suppose you wanted to query all the columns in the Organization table and have this information stored in this object.  The problem of course is that the `User` and `Organization` both have an `id` column and conflict.  To get around this issue, the `prefix` parameter can be used with the `@Embedded` notation:

```java
class UserWithOrganization {
   // @Embedded notation flattens the properties of the User object into the object, preserving encapsulation.
   @Embedded User user;

   // All fields returned from the Organization table must be prefixed with org_ (i.e. org_id, org_name, etc.)
   @Embedded(prefix="org_") Organization organization;

}
```

Our data access object would then be the following:

```java
// declare inner join here
  @Query("SELECT User.*, Organization.name AS org_name, Organization.id as org_id FROM User INNER JOIN Organization " +
        "ON User.organization_id = Organization.id WHERE User.id = :id")
  public UserWithOrganization getWithOrgById(int id);
```

We can then perform this query through an AsyncTask:

```java
final UserDao userDao = ((RestApplication) getApplicationContext()).getMyDatabase().userDao();

AsyncTask<Void, Void, Void> task = new AsyncTask<Void, Void, Void>() {
  @Override
  protected Void doInBackground(Void... ) {
    UserWithOrganization userWithOrganization = userDao.getWithOrgById(1);

    return null;
  };
};

// Make sure to run execute() to run this task!
task.execute();

```

### Updating Rows

Updating rows require defining an `@Insert` annotation.  The primary key will be returned if the return type is declared as a `Long` type:

```java
@Dao
public interface UserDao {
  @Insert(onConflict = OnConflictStrategy.REPLACE)
  // return primary key as Long
  public Long insertUser(User user);
```

Multiple primary keys can also be returned if the DAO includes multiple parameters:

```java
@Dao
public interface UserDao {
  @Insert(onConflict = OnConflictStrategy.REPLACE)
  public Long[] insertUser(User... users);
```

Remember again that you need to dispatch the task in a separate thread:

```java
final UserDao userDao = ((RestApplication) getApplicationContext()).getMyDatabase().userDao();

AsyncTask<User, Void, Void> task = new AsyncTask<User, Void, Void>() {

  @Override
  protected Void doInBackground(User... users) {
      userDao.insertUsers(users);
  }

task.execute(user);  
```
### Deleting Rows

Deleting requires either a class annotated as an Entity or a collection:

```java
@Delete
public void deleteUser(User user);

@Delete
public void deleteAll(User user1, User user2);
```

## Database Transactions

You can leverage the `runInTransaction()` method that comes with Room:

```java
((RestApplication) getApplicationContext()).getMyDatabase().runInTransaction(new Runnable() {
  @Override
  public void run() {
    UserDao userDao = ((RestApplication) getApplicationContext()).getMyDatabase().userDao();
    userDao.insertOrganization(organization);
    userDao.insertUser(user);
  }
});
```
## Common Questions

> Question: How do I inspect the SQLite data stored on the device?

In order to inspect the persisted data, we need to [[use adb to query or download the data|Local-Databases-with-SQLiteOpenHelper#sqlite-database-debugging]].  You can also take a look at using the [[Stetho|Troubleshooting-Common-Issues#database-inspection]] library, which provides a way to use Chrome to inspect the local data.

> Question: How does Room handle duplicate IDs?  For example, I want to make sure no duplicate twitter IDs are inserted.  Is there a way to specify a column is the primary key in the model?

Simply annotate the post ID column as the `@PrimaryKey` annotation:

```java
@Entity
public class SampleModel {

    @PrimaryKey
    @ColumnInfo
    int remoteId;

    // ... set the remote id based on the json response
}
```

Make sure to **uninstall the app** afterward on the emulator to ensure the schema changes take effect. Note that you may need to manually ensure that you don't attempt to re-create existing objects by verifying they are not already in the database **as shown below**.

> Question: How do you specify the data type (int, text)?  Does Room automatically know what the column type should be?

The type is inferred automatically from the type of the field.

> Question: How do I store dates into Room?

Room supports type converters:

```java
@ColumnInfo
Date timestamp;
```

Then you would define a `DataConverter` class similar to the following:


```java
public class DateConverter {
    @TypeConverter
    public static Date toDate(Long timestamp) {
        return timestamp == null ? null : new Date(timestamp);
    }

    @TypeConverter
    public static Long toTimestamp(Date date) {
        return date == null ? null : date.getTime();
    }
}
```
> Question: Is it possible to do joins with Room?

You must define the queries directly as Data Access Objects (DAO).  The return type defined in these interfaces must match the data returned.  See the example above in the [[querying rows|Room-Guide#querying-rows]] section.

There are a few key aspects of Room to note that differ slightly from traditional object relational mapping (ORM) frameworks:

* Usually, ORM frameworks expose a limited subset of queries.  Tables are declared as Java objects, and the relations between these tables dictate the types of queries that can be performed.  In Room, SQL inserts, updates, deletes, and complex joins are declared as **Data Access Objects (DAO)**.  The data returned from these queries simply are mapped to a Java object that will hold this information in memory.  In this way, no assumptions are made about how the data can be accessed.  The table definitions are separate from the actual queries performed.

* ORMs typically handle one-to-one and one-to-many relationships by determining the relationship between tables and expose an interface or a set of APIs that perform the SQL queries behind the scenes.  Because of the [performance implications](https://youtu.be/MfHsPGQ6bgE?t=27m37s), Room requires handling these relationships explicitly.   The query needs to be defined as DAO objects, and the data returned will need to be mapped to Java objects.

* One additional change is that Room doesn't require defining your SQL tables as a single Java object.  It allows you to maintain encapsulation between objects by allowing other Java objects to be included as part of a single table.  

## References

* <https://www.youtube.com/watch?v=MfHsPGQ6bgE>
