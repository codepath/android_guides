## Overview

Using the ActiveAndroid ORM makes managing client-side models extremely easy in simple cases. For more advanced or custom cases, you can use [[SQLiteOpenHelper|Local-Databases-with-SQLiteOpenHelper]] to manage the database communication directly. But for simple model mapping from JSON, ActiveAndroid keeps things simple.  

ActiveAndroid works like any **Object Relational Mapper** by mapping java classes to database tables and mapping java class member variables to the table columns. Through this process, **each table** maps to a **Java model** and **the columns** in the table represent the respective **data fields**. Similarly, each row in the database represents a particular object. This allows us to create, modify, delete and query our SQLite database using model objects instead of raw SQL.

For example, a "Tweet" model would be mapped to a "tweets" table in the database. The Tweet model might have a "body" field that maps to a body column in the table and a "timestamp" field that maps to a timestamp column. Through this process, each row would map to a particular tweet.

### Installation

You can download a [recent ActiveAndroid JAR](https://www.dropbox.com/s/cee2ldq6chuzplp/ActiveAndroid-3.1-08c6335cd.jar?dl=1) snapshot and drag this into the "libs" folder OR generate the latest JAR yourself based on the source as described below:

1. Download the latest build of [ActiveAndroid](https://github.com/pardom/ActiveAndroid/archive/master.zip).
2. Unzip the `master.zip` file.
3. Open up the **Terminal** and navigate to the directory that was just unzipped.
4. Type `ant` to build the project and generate a project jar file.
5. Move the generated jar file from `dist/ActiveAndroid.jar` to the `libs` directory in your **project home**.

```bash
mv dist/ActiveAndroid.jar <your_project_home>/libs/
```

### Configuration

Next, we need to configure the `application` node with the `name` property of `com.activeandroid.app.Application` to use the correct application class name within the `AndroidManifest.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    ...>
    <!- adjust the name property of your application node ->
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

First, we define our models by annotating the class with the table mapping and the member variables with the column mapping:

```java
import com.activeandroid.Model;
import com.activeandroid.annotation.Table;

@Table(name = "Items")
public class Item extends Model {
    // This is the unique id given by the server
    @Column(name = "remote_id", unique = true, onUniqueConflict = Column.ConflictAction.REPLACE)
    public long remoteId;
    // This is a regular field
    @Column(name = "Name")
    public String name;
    // This is an association to another activeandroid model
    @Column(name = "Category", onUpdate = ForeignKeyAction.CASCADE, onDelete = ForeignKeyAction.CASCADE)
    public Category category;
    
    // Make sure to have a default constructor for every ActiveAndroid model
    public Item(){
       super();
    }
    
    public Item(int remoteId, String name, Category category){
        super();
        this.remoteId = remoteId;
        this.name = name;
        this.category = category;
    }
}

@Table(name = "Categories")
public class Category extends Model {
    // This is the unique id given by the server
    @Column(name = "remote_id", unique = true)
    public long remoteId;
    // This is a regular field
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

Note that **ActiveAndroid creates a local id (mId)** in addition to our manually managed `remoteId` (unique) which is the id on the server (for networked applications). 

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
// or with
new Delete().from(Item.class).where("remote_id = ?", 1).execute();
```

#### Querying Records

We can query records with a simple query syntax using the `Select` object:

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

#### Executing Custom SQL

To run custom SQL with no need for a result, use the `SQLiteUtils.execSql` method:

```java
// Note nothing is returned from this
SQLiteUtils.execSql("DELETE FROM table_name");
```

If you need to execute a custom query and want to get a `List` of items back use `SQLiteUtils.rawQuery`:

```java
List<TodoItem> importantItems = 
  SQLiteUtils.rawQuery(TodoItem.class, 
     "SELECT * from todo_items where priority = ?", 
     new String[] { "high" });
```

See [SQLiteUtils implementation](https://github.com/pardom/ActiveAndroid/blob/master/src/com/activeandroid/util/SQLiteUtils.java#L105) for more details.

#### Populating ListView with CursorAdapter

Review this [[Custom CursorAdapter and ListViews|Populating a ListView with a CursorAdapter]] guide in order to load content from a `Cursor` into a `ListView`. In summary, in order to populate a `ListView` directly from the content within the ActiveAndroid SQLite database, we we can define this method on the model to retrieve a `Cursor` for the result set:

```java
public class TodoItem extends Model {
    // ...

    // Return cursor for result set for all todo items
    public static Cursor fetchResultCursor() {
        String tableName = Cache.getTableInfo(TodoItem.class).getTableName();
        // Query all items without any conditions
        String resultRecords = new Select(tableName + ".*, " + tableName + ".Id as _id").
            from(TodoItem.class).toSql();
        // Execute query on the underlying ActiveAndroid SQLite database
        Cursor resultCursor = Cache.openDatabase().rawQuery(resultRecords, null);
        return resultCursor;
    }
}
```

We need to define a custom `TodoCursorAdapter` as [[outlined here|Populating-a-ListView-with-a-CursorAdapter#defining-the-adapter]] in order to define which XML template to use for the cursor rows and how to populate the template views with the relevant data. 

Next, we can fetch the data cursor containing all todo items with `TodoItem.fetchResultCursor()` and populate the `ListView` using our custom `CursorAdapter`:

```java
// Find ListView to populate
ListView lvItems = (ListView) findViewById(R.id.lvItems);
// Get data cursor
Cursor todoCursor = TodoItem.fetchResultCursor();
// Setup cursor adapter
TodoCursorAdapter todoAdapter = new TodoCursorAdapter(this, todoCursor);
// Attach cursor adapter to ListView 
lvItems.setAdapter(todoAdapter);
```

That's all we have to do to load data from ActiveAndroid directly through a `Cursor` into a list.

#### Loading with Content Providers

Instead of using the underlying SQLite database directly, we can instead expose the ActiveAndroid data as a content provider with a few simple additions. First, override the default identity column for all ActiveAndroid models:

```java
@Table(name = "Items", id = BaseColumns._ID)
public class Item extends Model { ... }
```

Then you can the [SimpleCursorAdapter](http://developer.android.com/reference/android/widget/SimpleCursorAdapter.html) to populate adapters using the underlying database directly:

```java
// Define a SimpleCursorAdapter loading the body into the TextView in simple_list_item_1
SimpleCursorAdapter adapterTodo = new SimpleCursorAdapter(getActivity(),
  android.R.layout.simple_list_item_1, null,
  new String[] { "body" },
  new int[] { android.R.id.text1 },
  0);
// Attach the simple adapter to the list
myListView.setAdapter(adapterTodo);
```

You could also use [[a custom CursorAdapter|Populating a ListView with a CursorAdapter#defining-the-adapter]] instead for more flexibility. Next, we can load the data into the list using the content provider system through a `CursorLoader`:

```java
MyActivity.this.getSupportLoaderManager().initLoader(0, null, new LoaderCallbacks<Cursor>() {
        @Override
        public Loader<Cursor> onCreateLoader(int id, Bundle cursor) {
            return new CursorLoader(MyActivity.this,
                ContentProvider.createUri(TodoItem.class, null),
                null, null, null, null
            );
        }
        // ...
});
```

You must also register the content provider in your AndroidManifest.xml:

```xml
<application ...>
    <provider android:authorities="com.example" android:exported="false"    
              android:name="com.activeandroid.content.ContentProvider" />
    ...
</application>
```

See the full source code on the official [ActiveAndroid ContentProviders guide](https://github.com/pardom/ActiveAndroid/wiki/Using-the-content-provider).

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

This is because ActiveAndroid needs you to **save all objects separately**. Before saving a tweet for example, be sure to save the associated user object first. So when you have a tweet that references a user be sure to `user.save()` before you call `tweet.save()` since storing the user requires the local id to be set and assigned as the foreign key for the tweet.

> Question: How does ActiveAndroid handle duplicate IDs?  For example, I want to make sure no duplicate twitter IDs are inserted.  Is there a way to specify a column is the primary key in the model?

The first step is to mark the column recording the unique id of an object as a unique column acting as your pseudo primary key. As [explained here](https://github.com/pardom/ActiveAndroid/issues/22), the annotation is:

```java
@Table(name = "items")
public class SampleModel extends Model {
    // Ensure the remoteId must be unique. If a duplicate tries to save, replace the 
    @Column(name = "remote_id", unique = true, onUniqueConflict = Column.ConflictAction.REPLACE)
    private int remoteId;

    // ... set the remote id based on the json response
}
```

Make sure to **uninstall the app** afterward on the emulator to ensure the schema changes take effect. **Note** that you may need to manually ensure that you don't attempt to re-create existing objects by verifying they are not already in the database.

> Problem: `SQLiteConstraintException: foreign key constraint failed` when saving object

This error means that you are **attempting to save a object** which would create a duplicate row in the database. This means that the object you are trying to save has the same `remoteId` (or other unique column) as an object already in the database. 

With ActiveAndroid, be sure to avoid attempting to save duplicate data. Instead, you can check to ensure the data has not already been saved before saving a new object:

```java
@Table(name="users")
public class User extends Model {
    @Column(name = "remote_id", unique = true)
    public long remoteId;

    // Finds existing user based on remoteId or creates new user and returns
    public static User findOrCreateFromJson(JSONObject json) {
    	User user = User.fromJSON(json);
    	User existingUser = 
          new Select().from(User.class).where("remote_id = ?", user.remoteId).executeSingle(); 
    	if (existingUser != null) { 
    	    // found and return existing
    	    return existingUser;
    	} else {
    	    // create and return new
    	    user.save();
    	    return user;
    	}
    }    
}
```

Then when you want to create this record and avoid duplicates, you can just call:

```
User user = User.findOrCreateFromJson(objectJson);
// Returns either the existing user or the created user
```

This will avoid any foreign key constraint exceptions.

> Question: I read somewhere that ActiveAndroid automatically creates another auto-increment ID column, is this true?  What field names should I avoid using?

This is true, see [creating your database model](https://github.com/pardom/ActiveAndroid/wiki/Creating-your-database-model) for more details. The column is called ["mId"](https://github.com/pardom/ActiveAndroid/blob/master/src/com/activeandroid/Model.java#L40)

> Question: How do you specify the data type (int, text)?  Does AA automatically know what the column type should be?

The type is inferred automatically from the type of the field.

> Question: How do I store dates into ActiveAndroid?

AA supports serializing Date fields automatically. It is stored internally as a timestamp (INTEGER) in milliseconds.

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

```java
public static List<Model> findRecent(Date newerThan) {
    return new Select().from(Model.class).where("timestamp > ?", newerThan.getTimeInMillis()).execute();
}
```

> Question: How do you represent a 1-1 relationship?  

Check out the [relationships section](https://github.com/pardom/ActiveAndroid/wiki/Creating-your-database-model#relationships) if you haven't yet. There are many ways to manage this. You can declare a type "User" and then that field will be associated with a [foreign key representing the user](https://github.com/pardom/ActiveAndroid/blob/master/src/com/activeandroid/Model.java#L135). 

```java
public class User extends Model {
  public String email;
  public Address address;
}
```

You can manage this process by simply constructing and assigning the user object to a field of the parent object and then calling save on the parent. 

```java
User u = new User("bob@foo.com");
u.address = new Address("135 Hesby St, Los Angeles, CA");
u.address.save();
u.save();
```

You should make sure to **call save separately on the associated object** in most cases.

> Question: How do I delete all the records from a table?

You can delete individual records using the `.delete` method and you can delete all records matching a particular condition with:

```java
new Delete().from(Item.class).execute(); // all records
// or based on a conditional clause
new Delete().from(Item.class).where("Id = ?", 1).execute();
```

This allows for bulk deletes.

> Question: Is it possible to do joins with ActiveAndroid? 

Joins are done using the query language AA provides in the [From class](https://github.com/pardom/ActiveAndroid/blob/master/src/com/activeandroid/query/From.java). You can read more here on the [Querying the Database docs](https://github.com/pardom/ActiveAndroid/wiki/Querying-the-database)

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