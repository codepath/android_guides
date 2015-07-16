## Overview

For maximum control over local data, developers can use SQLite directly by leveraging [SQLiteOpenHelper](http://developer.android.com/reference/android/database/sqlite/SQLiteOpenHelper.html) for executing SQL requests and managing a local database. 

If you want to use SQLite directly but reduce the verbosity of working with the database, check out our [[Easier SQL with Cupboard]] guide for a middle ground between SQLite and a [[full-fledged ORM|ActiveAndroid-Guide]].

### Defining the Database Handler

We need to write our own class to handle database operations such as creation, upgrading, reading and writing. Database operations are defined using the [SQLiteOpenHelper](http://developer.android.com/reference/android/database/sqlite/SQLiteOpenHelper.html):

```java
public class TodoItemDatabase extends SQLiteOpenHelper {
    // All Static variables
    // Database Version
    private static final int DATABASE_VERSION = 1;
 
    // Database Name
    private static final String DATABASE_NAME = "todoListDatabase";
 
    // Todo table name
    private static final String TABLE_TODO = "todo_items";
 
    // Todo Table Columns names
    private static final String KEY_ID = "id";
    private static final String KEY_BODY = "body";
    private static final String KEY_PRIORITY = "priority";
 
    public TodoItemDatabase(Context context) {
        super(context, DATABASE_NAME, null, DATABASE_VERSION);
    }
 
    // Creating our initial tables
    // These is where we need to write create table statements. 
    // This is called when database is created.
    @Override
    public void onCreate(SQLiteDatabase db) {
    	// Construct a table for todo items
        String CREATE_TODO_TABLE = "CREATE TABLE " + TABLE_TODO + "("
                + KEY_ID + " INTEGER PRIMARY KEY," + KEY_BODY + " TEXT,"
                + KEY_PRIORITY + " INTEGER" + ")";
        db.execSQL(CREATE_TODO_TABLE);
    }
 
    // Upgrading the database between versions
    // This method is called when database is upgraded like modifying the table structure, 
    // adding constraints to database, etc
    @Override
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
    	if (oldVersion != newVersion) {
           // Wipe older tables if existed
           db.execSQL("DROP TABLE IF EXISTS " + TABLE_TODO);
           // Create tables again
           onCreate(db);
    	}
    }
}
```

Here we've defined the table name, the names of the fields and how to create the initial database schema and upgrade the database when the version changes.

### Defining our Models

In order to access our records from the database more easily, we should create a model class for each of our resources. In this case, let's define a `TodoItem` model as follows:

```java
public class TodoItem {
	private int id;
	private String body;
	private int priority;
	
	public TodoItem(String body, int priority) {
		super();
		this.body = body;
		this.priority = priority;
	}

	public String getBody() {
		return body;
	}

	public void setBody(String body) {
		this.body = body;
	}

	public int getPriority() {
		return priority;
	}

	public void setPriority(int priority) {
		this.priority = priority;
	}

	public int getId() {
		return id;
	}

	public void setId(int id) {
		this.id = id;
	}
}
```

Now we can interact with our data using the models.

### Inserting New Records

We can create a method for inserting a model into our database:

```java
public class TodoItemDatabase extends SQLiteOpenHelper {
    // ...existing methods...

    // Insert record into the database
    public void addTodoItem(TodoItem item) {
    	// Open database connection
        SQLiteDatabase db = this.getWritableDatabase();
        // Define values for each field
        ContentValues values = new ContentValues();
        values.put(KEY_BODY, item.getBody()); 
        values.put(KEY_PRIORITY, item.getPriority()); 
        // Insert Row and get generated id
        long rowId = db.insertOrThrow(TABLE_TODO, null, values);
        // Closing database connection
        db.close();
    }
}
```

### Querying Records

We can create the following method to query a single todo item based on id:

```java
public class TodoItemDatabase extends SQLiteOpenHelper {
    // ...existing methods...

    // Returns a single todo item by id
    public TodoItem getTodoItem(int id) {
    	// Open database for reading
        SQLiteDatabase db = this.getReadableDatabase();
        // Construct and execute query
        Cursor cursor = db.query(TABLE_TODO,  // TABLE
        		new String[] { KEY_ID, KEY_BODY, KEY_PRIORITY }, // SELECT 
        		KEY_ID + "= ?", new String[] { String.valueOf(id) },  // WHERE, ARGS
        		null, null, "id ASC", "100"); // GROUP BY, HAVING, ORDER BY, LIMIT
        if (cursor != null)
            cursor.moveToFirst();
        // Load result into model object
        TodoItem item = new TodoItem(cursor.getString(1), cursor.getInt(2));
        item.setId(cursor.getInt(cursor.getColumnIndexOrThrow(KEY_ID)));
        // Close the cursor
        if (cursor != null)
            cursor.close();
        // return todo item
        return item;
    }
}
```

and then the following method for querying all todo items:

```java
public class TodoItemDatabase extends SQLiteOpenHelper {
    // ...existing methods...

    public List<TodoItem> getAllTodoItems() {
        List<TodoItem> todoItems = new ArrayList<TodoItem>();
        // Select All Query
        String selectQuery = "SELECT  * FROM " + TABLE_TODO;
     
        SQLiteDatabase db = this.getReadableDatabase();
        Cursor cursor = db.rawQuery(selectQuery, null);
     
        // looping through all rows and adding to list
        if (cursor.moveToFirst()) {
            do {
            	TodoItem item = new TodoItem(cursor.getString(1), cursor.getInt(2));
                item.setId(cursor.getInt(0));
                // Adding todo item to list
                todoItems.add(item);
            } while (cursor.moveToNext());
        }
        // Close the cursor
        if (cursor != null)
            cursor.close();
     
        // return todo list
        return todoItems;
    }
}
```

and the following method to get the total number of todo items:

```java
public class TodoItemDatabase extends SQLiteOpenHelper {
    // ...existing methods...

    public int getTodoItemCount() {
        String countQuery = "SELECT  * FROM " + TABLE_TODO;
        SQLiteDatabase db = this.getReadableDatabase();
        Cursor cursor = db.rawQuery(countQuery, null);
        // Close the cursor
        cursor.close();
        // return count
        return cursor.getCount();
    }
}
```

### Updating Records

We can create this method to update a record in the database:

```java
public class TodoItemDatabase extends SQLiteOpenHelper {
    // ...existing methods...

    public int updateTodoItem(TodoItem item) {
    	// Open database for writing
        SQLiteDatabase db = this.getWritableDatabase();
        // Setup fields to update
        ContentValues values = new ContentValues();
        values.put(KEY_BODY, item.getBody());
        values.put(KEY_PRIORITY, item.getPriority());
        // Updating row
        int result = db.update(TABLE_TODO, values, KEY_ID + " = ?",
                new String[] { String.valueOf(item.getId()) });
        // Close the database
        db.close();
        return result;
    }
}
```

### Deleting Records

We can create this method to delete a record from the database:

```java
public class TodoItemDatabase extends SQLiteOpenHelper {
    // ...existing methods...

    public void deleteTodoItem(TodoItem item) {
    	// Open database for writing
        SQLiteDatabase db = this.getWritableDatabase();
        // Delete the record with the specified id
        db.delete(TABLE_TODO, KEY_ID + " = ?",
                new String[] { String.valueOf(item.getId()) });
        // Close the database
        db.close();
    }
}
```

### Using our Database Handler

We can now leverage our database handler and models to persist data to our SQLite store:

```java
public class SQLiteExampleActivity extends Activity {
	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_sqlite_example);
		// Create our sqlite database object
		TodoItemDatabase db = new TodoItemDatabase(this);
		// Inserting todo items
		db.addTodoItem(new TodoItem("Get milk", 3));
		db.addTodoItem(new TodoItem("Pickup kids", 1));
		// Querying all todo items
		List<TodoItem> items = db.getAllTodoItems();
		// Print out properties
		for (TodoItem ti : items) {
			String log = "Id: " + ti.getId() + " , Body: " + ti.getBody() + 
					" , Priority: " + ti.getPriority();
			// Writing Todo Items to log 
			Log.d("Name: ", log);
		}
	}
}
```

**Note:** In many cases, rather than interacting with SQL directly, Android apps can leverage one of the many available [[higher-level ORMs|Persisting-Data-to-the-Device#object-relational-mappers]] (object relational mappers) to persist Java models to a database table instead.

### Full Database Handler Source

The full source code for the database handler above can be found here for reference:

```java
public class TodoItemDatabase extends SQLiteOpenHelper {
 
    // All Static variables
    // Database Version
    private static final int DATABASE_VERSION = 1;
 
    // Database Name
    private static final String DATABASE_NAME = "todoListDatabase";
 
    // Todo table name
    private static final String TABLE_TODO = "todo_items";
 
    // Todo Table Columns names
    private static final String KEY_ID = "id";
    private static final String KEY_BODY = "body";
    private static final String KEY_PRIORITY = "priority";
 
    public TodoItemDatabase(Context context) {
        super(context, DATABASE_NAME, null, DATABASE_VERSION);
    }
 
    // Creating our initial tables
    // These is where we need to write create table statements. 
    // This is called when database is created.
    @Override
    public void onCreate(SQLiteDatabase db) {
    	// Construct a table for todo items
        String CREATE_TODO_TABLE = "CREATE TABLE " + TABLE_TODO + "("
                + KEY_ID + " INTEGER PRIMARY KEY," + KEY_BODY + " TEXT,"
                + KEY_PRIORITY + " INTEGER" + ")";
        db.execSQL(CREATE_TODO_TABLE);
    }
 
    // Upgrading the database between versions
    // This method is called when database is upgraded like modifying the table structure, 
    // adding constraints to database, etc
    @Override
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
    	if (oldVersion != newVersion) {
           // Wipe older tables if existed
           db.execSQL("DROP TABLE IF EXISTS " + TABLE_TODO);
           // Create tables again
           onCreate(db);
    	}
    }
    
    // Insert record into the database
    public void addTodoItem(TodoItem item) {
    	// Open database connection
        SQLiteDatabase db = this.getWritableDatabase();
        // Define values for each field
        ContentValues values = new ContentValues();
        values.put(KEY_BODY, item.getBody()); 
        values.put(KEY_PRIORITY, item.getPriority()); 
        // Insert Row
        db.insertOrThrow(TABLE_TODO, null, values);
        db.close(); // Closing database connection
    }
    
    // Returns a single todo item by id
    public TodoItem getTodoItem(int id) {
    	// Open database for reading
        SQLiteDatabase db = this.getReadableDatabase();
        // Construct and execute query
        Cursor cursor = db.query(TABLE_TODO,  // TABLE
        		new String[] { KEY_ID, KEY_BODY, KEY_PRIORITY }, // SELECT 
        		KEY_ID + "= ?", new String[] { String.valueOf(id) },  // WHERE, ARGS
        		null, null, "id ASC", "100"); // GROUP BY, HAVING, ORDER BY, LIMIT
        if (cursor != null)
            cursor.moveToFirst();
        // Load result into model object
        TodoItem item = new TodoItem(cursor.getString(1), cursor.getInt(2));
        item.setId(cursor.getInt(cursor.getColumnIndexOrThrow(KEY_ID)));
        // Close the cursor
        if (cursor != null)
            cursor.close();
        // return todo item
        return item;
    }
    
    public List<TodoItem> getAllTodoItems() {
        List<TodoItem> todoItems = new ArrayList<TodoItem>();
        // Select All Query
        String selectQuery = "SELECT  * FROM " + TABLE_TODO;
     
        SQLiteDatabase db = this.getReadableDatabase();
        Cursor cursor = db.rawQuery(selectQuery, null);
     
        // looping through all rows and adding to list
        if (cursor.moveToFirst()) {
            do {
            	TodoItem item = new TodoItem(cursor.getString(1), cursor.getInt(2));
                item.setId(cursor.getInt(0));
                // Adding todo item to list
                todoItems.add(item);
            } while (cursor.moveToNext());
        }

        // Close the cursor
        if (cursor != null)
            cursor.close();
     
        // return todo list
        return todoItems;
    }
    
    public int getTodoItemCount() {
        String countQuery = "SELECT  * FROM " + TABLE_TODO;
        SQLiteDatabase db = this.getReadableDatabase();
        Cursor cursor = db.rawQuery(countQuery, null);
        // Close the cursor
        cursor.close();
        // return count
        return cursor.getCount();
    }
    
    public int updateTodoItem(TodoItem item) {
    	// Open database for writing
        SQLiteDatabase db = this.getWritableDatabase();
        // Setup fields to update
        ContentValues values = new ContentValues();
        values.put(KEY_BODY, item.getBody());
        values.put(KEY_PRIORITY, item.getPriority());
        // Updating row
        int result = db.update(TABLE_TODO, values, KEY_ID + " = ?",
                new String[] { String.valueOf(item.getId()) });
        // Close the database
        db.close();
        return result;
    }
    
    public void deleteTodoItem(TodoItem item) {
    	// Open database for writing
        SQLiteDatabase db = this.getWritableDatabase();
        // Delete the record with the specified id
        db.delete(TABLE_TODO, KEY_ID + " = ?",
                new String[] { String.valueOf(item.getId()) });
        // Close the database
        db.close();
    }
    
}
```

## Working with Multiple Tables

Once you understand the basics of SQLite above, be sure to review [this more advanced tutorial](http://www.androidhive.info/2013/09/android-sqlite-database-with-multiple-tables/) which explores working with SQLite when you have multiple associated tables.

## Singleton Pattern

Often SQLite will be used across your entire application; within services, applications, fragments, and more. For this reason, best practices often advise you to apply the singleton pattern to your `SQLiteOpenHelper` instances to avoid memory leaks and unnecessary reallocations. The best solution is to **make your database instance a singleton instance** across the entire application's lifecycle.

```java
public class DatabaseHelper extends SQLiteOpenHelper { 
  private static DatabaseHelper sInstance;

  // ...

  public static synchronized DatabaseHelper getInstance(Context context) {
    // Use the application context, which will ensure that you 
    // don't accidentally leak an Activity's context.
    // See this article for more information: http://bit.ly/6LRzfx
    if (sInstance == null) {
      sInstance = new DatabaseHelper(context.getApplicationContext());
    }
    return sInstance;
  }

  /**
   * Constructor should be private to prevent direct instantiation.
   * make call to static method "getInstance()" instead.
   */
  private DatabaseHelper(Context context) {
    super(context, DATABASE_NAME, null, DATABASE_VERSION);
  }
}
```

The static `getInstance()` method ensures that only one `DatabaseHelper` will ever exist at any given time. If the `sInstance` object has not been initialized, one will be created. If one has already been created then it will simply be returned. Then we can access our database connection with:

```java
// In any activity just pass the context and use singleton method
DatabaseHelper helper = DatabaseHelper.getInstance(this);
```

See [this android design patterns article](http://www.androiddesignpatterns.com/2012/05/correctly-managing-your-sqlite-database.html) for more information.

## SQLite Database Debugging 

When working with SQLite, opening and inspecting the SQLite database can be helpful while debugging issues. The commands below will show how to get at the data (whether running on an emulator or an actual device). The commands should be performed **within the terminal or command-line**. Once you have the data, there are [desktop SQLite viewers](http://sqlitebrowser.org/) to help inspect the SQLite data graphically.

### On an Emulator:

Use `SQLite3` to query the data on the emulator:

```bash
cd /path/to/my/sdk/platform-tools
./adb shell
run-as <app package name>
cd /data/data/<app package name>/databases
ls
chmod 666 <database file name>
sqlite3 <database file name>
> (semi-colon terminated commands can be run here to query the data)
> .exit
(copy full database path)
exit
```

For further inspection, we can download the database file with:

```
./adb wait-for-device pull /data/data/<app package name>/databases/<database file name>
```

### On a Device:

There isn't a `SQLite3` executable on the device so our only option is to download the database file with:

```bash
./adb shell run-as <app package name> chmod 666 /data/data/<app package name>/databases/<database file name>
./adb shell cp /data/data/<app package name>/databases/<database file name> /sdcard/
./adb pull /sdcard/<database file name>
```

## References

* [Vogella SQLite Tutorial](http://www.vogella.com/articles/AndroidSQLite/article.html)
* [AndroidHive Tutorial](http://www.androidhive.info/2011/11/android-sqlite-database-tutorial/)
* [Techotopia Tutorial](http://www.techotopia.com/index.php/An_Android_SQLite_Database_Tutorial)
* [Programming Android SQLite Tutorial](https://github.com/bmeike/ProgrammingAndroidExamples/tree/master/MicroJobs/src/com/oreilly/demo/android/pa/microjobs)
* [Multiple Tables](http://www.androidhive.info/2013/09/android-sqlite-database-with-multiple-tables/)