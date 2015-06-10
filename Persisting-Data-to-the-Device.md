## Overview

The Android framework offers several options and strategies for persistence:

 * **Shared Preferences** - Easily save basic data as key-value pairs in a private persisted dictionary.
 * **Local Files** - Save arbitrary files to internal or external device storage.
 * **SQLite Database** - Persist data in tables within an application specific database.
 * **ORM** - Describe and persist model objects using a higher level query/update syntax.

### Shared Preferences

Settings can be persisted for your application by using [SharedPreferences](http://developer.android.com/reference/android/content/SharedPreferences.html) to persist key-value pairs.  

To retrieve an existing `username` key from your Shared Preferences, you can type:

```java
SharedPreferences pref =   
    PreferenceManager.getDefaultSharedPreferences(this);
String username = pref.getString("username", "n/a"); 
```

SharedPreferences can be edited by getting access to the Editor instance:

```java
SharedPreferences pref =   
    PreferenceManager.getDefaultSharedPreferences(this);
Editor edit = pref.edit();
edit.putString("username", "billy");
edit.putString("user_id", "65");
edit.commit(); 
```

### Local Files

Android can read/write files to internal as well as external storage. Applications have access to an application-specific directory where preferences and sqlite databases are also stored. Every Activity has helpers to get the writeable directory. File I/O API is a subset of the normal Java File API.

Writing files is as simple as getting the stream using `openFileOutput` method] and writing to it using a BufferedWriter:

```java
// Use Activity method to create a file in the writeable directory
FileOutputStream fos = openFileOutput("filename", MODE_WORLD_WRITEABLE);
// Create buffered writer
BufferedWriter writer = new BufferedWriter(new OutputStreamWriter(fos));
writer.write("Hi, I'm writing stuff");
writer.close();
```

Reading the file back is then just using a `BufferedReader` and then building the text into a StringBuffer:

```java
BufferedReader input = null;
input = new BufferedReader(
new InputStreamReader(openFileInput("myfile")));
String line;
StringBuffer buffer = new StringBuffer();
while ((line = input.readLine()) != null) {
  buffer.append(line + "\n");
}
String text = buffer.toString();
```

You can also inspect and transfer files to emulators or devices using the DDMS File Explorer perspective which allows you to access to filesystem on the device.

### SQLite

For maximum control, developers can use SQLite directly by leveraging the [SQLiteOpenHelper](http://developer.android.com/reference/android/database/sqlite/SQLiteOpenHelper.html) for executing SQL requests and managing a local database:

```java
public class TodoItemDatabase extends SQLiteOpenHelper { 
    public TodoItemDatabase(Context context) {
        super(context, DATABASE_NAME, null, DATABASE_VERSION);
    }
 
    // These is where we need to write create table statements. 
    // This is called when database is created.
    @Override
    public void onCreate(SQLiteDatabase db) {
    	// SQL for creating the tables
    }
 
    // This method is called when database is upgraded like 
    // modifying the table structure, 
    // adding constraints to database, etc
    @Override
    public void onUpgrade(SQLiteDatabase db, int oldVersion, 
        int newVersion) {
    	// SQL for upgrading the tables
    }
}
```

Check out our [[managing databases with SQLiteOpenHelper|Local-Databases-with-SQLiteOpenHelper]] guide for a more detailed look at working with SQLite. In many cases, rather than interacting with SQL directly, Android apps can leverage one of the many available higher-level ORMs (object relational mappers) to persist Java models to a database table as shown below. If needed, we can also [[access the SQLite database for debugging|Local-Databases-with-SQLiteOpenHelper#sqlite-database-debugging]].

### Object Relational Mappers

Instead of accessing the SQLite database directly, there is no shortage of higher-level wrappers for managing SQL persistence. There are many popular ORMs for Android, but one of the easiest to use is [ActiveAndroid](https://github.com/pardom/ActiveAndroid/wiki/Getting-started) ([[cliffnotes|ActiveAndroid-Guide]]). Here's a few alternatives as well:

 * [SugarORM](http://satyan.github.io/sugar/index.html) - Very easy syntax, uses reflection to infer data ([[cliffnotes|Clean-Persistence-with-Sugar-ORM]])
 * [Siminov](http://siminov.github.io/android-orm/) - Another viable alternative syntax
 * [greenDAO](http://greendao-orm.com/) - Slightly different take (DAO vs ORM)
 * [ORMLite](http://ormlite.com/sqlite_java_android_orm.shtml) - Lightweight and speed is prioritized
 * [androrm](http://www.androrm.net/) - Newer but promising take on a simple ORM

For this class, we selected ActiveAndroid. With ActiveAndroid, building models that are SQLite backed is easy and explicit using annotations. Instead of manually creating and updating tables and managing SQL queries, simply annotate your model classes to associate fields with database columns:

```java
@Table(name = "Users")
public class User extends Model {
  @Column(name = "Name")
  public String name;

  @Column(name = "Age")
  public int age;
  
  // Make sure to define this constructor (with no arguments)
  // If you don't querying will fail to return results!
  public User() {
    super();
  }
  
  // Be sure to call super() on additional constructors as well
  public User(String name, int age){
    super();
    this.name = name;
    this.age = age;
  }
}
```

Inserting or updating objects no longer requires manually constructing SQL statements, just use the model class as an ORM:

```java
User user = new User();
user.name = "Jack";
user.age = 25;
user.save();
// or delete easily too
user.delete();
```

ActiveAndroid queries map to SQL queries and are built by chaining methods.

```java
List<User> users = new Select()
    .from(User.class).where("age > ?", 25)
    .limit(25).offset(0)
    .orderBy("age ASC").execute();
```

This will automatically query the database and return the results as a List for use. For more information, check out our [[ActiveAndroid Guide|ActiveAndroid-Guide]] for links to more resources and answers to common questions. As needed, we can also [[access the SQLite database for debugging|Local-Databases-with-SQLiteOpenHelper#sqlite-database-debugging]].

## References

 * <http://developer.android.com/guide/topics/data/data-storage.html>
 * <http://developer.android.com/training/basics/data-storage/index.html>
 * <http://mobile.tutsplus.com/tutorials/android/data-management-options-for-android-applications/>
 * <http://developer.android.com/reference/android/content/SharedPreferences.html>