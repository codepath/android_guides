## Overview

For maximum control over local data, developers can use SQLite directly by leveraging [SQLiteOpenHelper](http://developer.android.com/reference/android/database/sqlite/SQLiteOpenHelper.html) for executing SQL requests and managing a local database. 

In this guide, we'll use the example of building a database to persist user created "Posts" to demonstrate SQLite and SQLiteOpenHelper.

If you want to use SQLite directly but reduce the verbosity of working with the database, check out our [[Easier SQL with Cupboard]] guide for a middle ground between SQLite and a [[full-fledged ORM|ActiveAndroid-Guide]].

## Defining the Database Handler

We need to write our own class to handle database operations such as creation, upgrading, reading and writing. Database operations are defined using the [SQLiteOpenHelper](http://developer.android.com/reference/android/database/sqlite/SQLiteOpenHelper.html):

```java
public class PostsDatabaseHelper extends SQLiteOpenHelper {
    // Database Info
    private static final String DATABASE_NAME = "postsDatabase";
    private static final int DATABASE_VERSION = 1;

    // Table Names
    private static final String TABLE_POSTS = "posts";
    private static final String TABLE_USERS = "users";

    // Post Table Columns
    private static final String KEY_POST_ID = "id";
    private static final String KEY_POST_USER_ID_FK = "userId";
    private static final String KEY_POST_TEXT = "text";

    // User Table Columns
    private static final String KEY_USER_ID = "id";
    private static final String KEY_USER_NAME = "userName";
    private static final String KEY_USER_PROFILE_PICTURE_URL = "profilePictureUrl";

    public PostsDatabaseHelper(Context context) {
        super(context, DATABASE_NAME, null, DATABASE_VERSION);
    }
    
    // Called when the database connection is being configured.
    // Configure database settings for things like foreign key support, write-ahead logging, etc.
    @Override
    public void onConfigure(SQLiteDatabase db) {
        super.onConfigure(db);
        db.setForeignKeyConstraintsEnabled(true);
    }

    // Called when the database is created for the FIRST time.
    // If a database already exists on disk with the same DATABASE_NAME, this method will NOT be called.
    @Override
    public void onCreate(SQLiteDatabase db) {
        String CREATE_POSTS_TABLE = "CREATE TABLE " + TABLE_POSTS +
                "(" +
                    KEY_POST_ID + " INTEGER PRIMARY KEY," + // Define a primary key
                    KEY_POST_USER_ID_FK + " INTEGER REFERENCES " + TABLE_USERS + "," + // Define a foreign key
                    KEY_POST_TEXT + " TEXT" +
                ")";

        String CREATE_USERS_TABLE = "CREATE TABLE " + TABLE_USERS +
                "(" +
                    KEY_USER_ID + " INTEGER PRIMARY KEY," +
                    KEY_USER_NAME + " TEXT," +
                    KEY_USER_PROFILE_PICTURE_URL + " TEXT" +
                ")";

        db.execSQL(CREATE_POSTS_TABLE);
        db.execSQL(CREATE_USERS_TABLE);
    }
    
    // Called when the database needs to be upgraded.
    // This method will only be called if a database already exists on disk with the same DATABASE_NAME,
    // but the DATABASE_VERSION is different than the version of the database that exists on disk.
    @Override
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
        if (oldVersion != newVersion) {
            // Simplest implementation is to drop all old tables and recreate them
            db.execSQL("DROP TABLE IF EXISTS " + TABLE_POSTS);
            db.execSQL("DROP TABLE IF EXISTS " + TABLE_USERS);
            onCreate(db);
        }
    }
}
```

**Important Note:** The SQLite database is lazily initialized. This means that it isn't actually created until it's first accessed through a call to `getReadableDatabase()` or `getWriteableDatabase()`. This also means that any methods that call `getReadableDatabase()` or `getWriteableDatabase()` should be done on a background thread as there is a possibility that they might be kicking off the initial creation of the database.

## Singleton Pattern

Often a SQLite database will be used across your entire application; within services, applications, fragments, and more. For this reason, best practices often advise you to apply the singleton pattern to your `SQLiteOpenHelper` instances to avoid memory leaks and unnecessary reallocations. The best solution is to **make your database instance a singleton instance** across the entire application's lifecycle.

```java
public class PostsDatabaseHelper extends SQLiteOpenHelper { 
  private static PostsDatabaseHelper sInstance;

  // ...

  public static synchronized PostsDatabaseHelper getInstance(Context context) {
    // Use the application context, which will ensure that you 
    // don't accidentally leak an Activity's context.
    // See this article for more information: http://bit.ly/6LRzfx
    if (sInstance == null) {
      sInstance = new PostsDatabaseHelper(context.getApplicationContext());
    }
    return sInstance;
  }

  /**
   * Constructor should be private to prevent direct instantiation.
   * Make a call to the static method "getInstance()" instead.
   */
  private PostsDatabaseHelper(Context context) {
    super(context, DATABASE_NAME, null, DATABASE_VERSION);
  }
}
```

The static `getInstance()` method ensures that only one `PostsDatabaseHelper` will ever exist at any given time. If the `sInstance` object has not been initialized, one will be created. If one has already been created then it will simply be returned. Then we can access our database connection with:

```java
// In any activity just pass the context and use the singleton method
PostsDatabaseHelper helper = PostsDatabaseHelper.getInstance(this);
```

See [this android design patterns article](http://www.androiddesignpatterns.com/2012/05/correctly-managing-your-sqlite-database.html) for more information.

## Defining our Models

In order to access our records from the database more easily, we should create a model class for each of our resources. In this case, let's define a `Post` and a `User` model:

```java
public class Post {
    public User user;
    public String text;
}
```

```java
public class User {
    public String userName;
    public String profilePictureUrl;
}
```

Now we can interact with our data using the models.

## CRUD Operations (Create, Read, Update, Delete)

We'll walk through examples of creating, reading, updating, and deleting posts / users in our database.

### Inserting New Records

```java
public class PostsDatabaseHelper extends SQLiteOpenHelper {
    // ...existing methods...

    // Insert a post into the database
    public void addPost(Post post) {
        // Create and/or open the database for writing
        SQLiteDatabase db = getWritableDatabase();

        // It's a good idea to wrap our insert in a transaction. This helps with performance and ensures
        // consistency of the database.
        db.beginTransaction();
        try {
            // The user might already exist in the database (i.e. the same user created multiple posts).
            long userId = addOrUpdateUser(post.user);

            ContentValues values = new ContentValues();
            values.put(KEY_POST_USER_ID_FK, userId);
            values.put(KEY_POST_TEXT, post.text);

            // Notice how we haven't specified the primary key. SQLite auto increments the primary key column.
            db.insertOrThrow(TABLE_POSTS, null, values);
            db.setTransactionSuccessful();
        } catch (Exception e) {
            Log.d(TAG, "Error while trying to add post to database");
        } finally {
            db.endTransaction();
        }
    }

    // Insert or update a user in the database
    // Since SQLite doesn't support "upsert" we need to fall back on an attempt to UPDATE (in case the
    // user already exists) optionally followed by an INSERT (in case the user does not already exist).
    // Unfortunately, there is a bug with the insertOnConflict method
    // (https://code.google.com/p/android/issues/detail?id=13045) so we need to fall back to the more
    // verbose option of querying for the user's primary key if we did an update.
    public long addOrUpdateUser(User user) {
        // The database connection is cached so it's not expensive to call getWriteableDatabase() multiple times.
        SQLiteDatabase db = getWritableDatabase();
        long userId = -1;

        db.beginTransaction();
        try {
            ContentValues values = new ContentValues();
            values.put(KEY_USER_NAME, user.userName);
            values.put(KEY_USER_PROFILE_PICTURE_URL, user.profilePictureUrl);

            // First try to update the user in case the user already exists in the database
            // This assumes userNames are unique
            int rows = db.update(TABLE_USERS, values, KEY_USER_NAME + "= ?", new String[]{user.userName});

            // Check if update succeeded
            if (rows == 1) {
                // Get the primary key of the user we just updated
                String usersSelectQuery = String.format("SELECT %s FROM %s WHERE %s = ?",
                        KEY_USER_ID, TABLE_USERS, KEY_USER_NAME);
                Cursor cursor = db.rawQuery(usersSelectQuery, new String[]{String.valueOf(user.userName)});
                try {
                    if (cursor.moveToFirst()) {
                        userId = cursor.getInt(0);
                        db.setTransactionSuccessful();
                    }
                } finally {
                    if (cursor != null && !cursor.isClosed()) {
                        cursor.close();
                    }
                }
            } else {
                // user with this userName did not already exist, so insert new user
                userId = db.insertOrThrow(TABLE_USERS, null, values);
                db.setTransactionSuccessful();
            }
        } catch (Exception e) {
            Log.d(TAG, "Error while trying to add or update user");
        } finally {
            db.endTransaction();
        }
        return userId;
    }
}
```
**Note:** If you are inserting a large number of records, you might want to use a compiled [SQLiteStatement](http://developer.android.com/reference/android/database/sqlite/SQLiteStatement.html). You can read more about the performance benefits on [this blog](http://www.techrepublic.com/blog/software-engineer/turbocharge-your-sqlite-inserts-on-android/).

### Querying Records

```java
public class PostsDatabaseHelper extends SQLiteOpenHelper {
    // ...existing methods...

    public List<Post> getAllPosts() {
        List<Post> posts = new ArrayList<>();

        // SELECT * FROM POSTS
        // LEFT OUTER JOIN USERS
        // ON POSTS.KEY_POST_USER_ID_FK = USERS.KEY_USER_ID
        String POSTS_SELECT_QUERY =
                String.format("SELECT * FROM %s LEFT OUTER JOIN %s ON %s.%s = %s.%s",
                        TABLE_POSTS,
                        TABLE_USERS,
                        TABLE_POSTS, KEY_POST_USER_ID_FK,
                        TABLE_USERS, KEY_USER_ID);

        // "getReadableDatabase()" and "getWriteableDatabase()" return the same object (except under low
        // disk space scenarios)
        SQLiteDatabase db = getReadableDatabase();
        Cursor cursor = db.rawQuery(POSTS_SELECT_QUERY, null);
        try {
            if (cursor.moveToFirst()) {
                do {
                    User newUser = new User();
                    newUser.userName = cursor.getString(cursor.getColumnIndex(KEY_USER_NAME));
                    newUser.profilePictureUrl = cursor.getString(cursor.getColumnIndex(KEY_USER_PROFILE_PICTURE_URL));

                    Post newPost = new Post();
                    newPost.text = cursor.getString(cursor.getColumnIndex(KEY_POST_TEXT));
                    newPost.user = newUser;
                    posts.add(newPost);
                } while(cursor.moveToNext());
            }
        } catch (Exception e) {
            Log.d(TAG, "Error while trying to get posts from database");
        } finally {
            if (cursor != null && !cursor.isClosed()) {
                cursor.close();
            }
        }
        return posts;
    }
}
```

### Updating Records

```java
public class PostsDatabaseHelper extends SQLiteOpenHelper {
    // ...existing methods...

    // Update the user's profile picture url
    public int updateUserProfilePicture(User user) {
        SQLiteDatabase db = this.getWritableDatabase();

        ContentValues values = new ContentValues();
        values.put(KEY_USER_PROFILE_PICTURE_URL, user.profilePictureUrl);

        // Updating profile picture url for user with that userName
        return db.update(TABLE_USERS, values, KEY_USER_NAME + " = ?",
                new String[] { String.valueOf(user.userName) });
    }
}
```

### Deleting Records

```java
public class PostsDatabaseHelper extends SQLiteOpenHelper {
    // ...existing methods...

    public void deleteAllPostsAndUsers() {
        SQLiteDatabase db = getWritableDatabase();
        db.beginTransaction();
        try {
            // Order of deletions is important when foreign key relationships exist.
            db.delete(TABLE_POSTS, null, null);
            db.delete(TABLE_USERS, null, null);
            db.setTransactionSuccessful();
        } catch (Exception e) {
            Log.d(TAG, "Error while trying to delete all posts and users");
        } finally {
            db.endTransaction();
        }
    }
}
```

## Using our Database Handler

We can now leverage our database handler and models to persist data to our SQLite store:

```java
public class SQLiteExampleActivity extends AppCompatActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_sqlite_example);

        // Create sample data
        User sampleUser = new User();
        sampleUser.userName = "Steph";
        sampleUser.profilePictureUrl = "https://i.imgur.com/tGbaZCY.jpg";

        Post samplePost = new Post();
        samplePost.user = sampleUser;
        samplePost.text = "Won won!";

        // Get singleton instance of database
        PostsDatabaseHelper databaseHelper = PostsDatabaseHelper.getInstance(this);
        
        // Add sample post to the database
        databaseHelper.addPost(samplePost);

        // Get all posts from database
        List<Post> posts = databaseHelper.getAllPosts();
        for (Post post : posts) {
            // do something
        }
    }
}
```

**Note:** In many cases, rather than interacting with SQL directly, Android apps can leverage one of the many available [[higher-level ORMs|Persisting-Data-to-the-Device#object-relational-mappers]] (object relational mappers) to persist Java models to a database table instead.

## Full Database Handler Source

The full source code for the database handler above can be found here for reference:

```java
public class PostsDatabaseHelper extends SQLiteOpenHelper {
    // Database Info
    private static final String DATABASE_NAME = "postsDatabase";
    private static final int DATABASE_VERSION = 1;

    // Table Names
    private static final String TABLE_POSTS = "posts";
    private static final String TABLE_USERS = "users";

    // Post Table Columns
    private static final String KEY_POST_ID = "id";
    private static final String KEY_POST_USER_ID_FK = "userId";
    private static final String KEY_POST_TEXT = "text";

    // User Table Columns
    private static final String KEY_USER_ID = "id";
    private static final String KEY_USER_NAME = "userName";
    private static final String KEY_USER_PROFILE_PICTURE_URL = "profilePictureUrl";

    private static PostsDatabaseHelper sInstance;

    public static synchronized PostsDatabaseHelper getInstance(Context context) {
        // Use the application context, which will ensure that you 
        // don't accidentally leak an Activity's context.
        // See this article for more information: http://bit.ly/6LRzfx
        if (sInstance == null) {
            sInstance = new PostsDatabaseHelper(context.getApplicationContext());
        }
        return sInstance;
    }

    /**
     * Constructor should be private to prevent direct instantiation.
     * Make a call to the static method "getInstance()" instead.
     */
    private PostsDatabaseHelper(Context context) {
        super(context, DATABASE_NAME, null, DATABASE_VERSION);
    }
    
    // Called when the database connection is being configured.
    // Configure database settings for things like foreign key support, write-ahead logging, etc.
    @Override
    public void onConfigure(SQLiteDatabase db) {
        super.onConfigure(db);
        db.setForeignKeyConstraintsEnabled(true);
    }

    // Called when the database is created for the FIRST time.
    // If a database already exists on disk with the same DATABASE_NAME, this method will NOT be called.
    @Override
    public void onCreate(SQLiteDatabase db) {
        String CREATE_POSTS_TABLE = "CREATE TABLE " + TABLE_POSTS +
                "(" +
                    KEY_POST_ID + " INTEGER PRIMARY KEY," + // Define a primary key
                    KEY_POST_USER_ID_FK + " INTEGER REFERENCES " + TABLE_USERS + "," + // Define a foreign key
                    KEY_POST_TEXT + " TEXT" +
                ")";

        String CREATE_USERS_TABLE = "CREATE TABLE " + TABLE_USERS +
                "(" +
                    KEY_USER_ID + " INTEGER PRIMARY KEY," +
                    KEY_USER_NAME + " TEXT," +
                    KEY_USER_PROFILE_PICTURE_URL + " TEXT" +
                ")";

        db.execSQL(CREATE_POSTS_TABLE);
        db.execSQL(CREATE_USERS_TABLE);
    }
    
    // Called when the database needs to be upgraded.
    // This method will only be called if a database already exists on disk with the same DATABASE_NAME,
    // but the DATABASE_VERSION is different than the version of the database that exists on disk.
    @Override
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
        if (oldVersion != newVersion) {
            // Simplest implementation is to drop all old tables and recreate them
            db.execSQL("DROP TABLE IF EXISTS " + TABLE_POSTS);
            db.execSQL("DROP TABLE IF EXISTS " + TABLE_USERS);
            onCreate(db);
        }
    }

    // Insert a post into the database
    public void addPost(Post post) {
        // Create and/or open the database for writing
        SQLiteDatabase db = getWritableDatabase();

        // It's a good idea to wrap our insert in a transaction. This helps with performance and ensures
        // consistency of the database.
        db.beginTransaction();
        try {
            // The user might already exist in the database (i.e. the same user created multiple posts).
            long userId = addOrUpdateUser(post.user);

            ContentValues values = new ContentValues();
            values.put(KEY_POST_USER_ID_FK, userId);
            values.put(KEY_POST_TEXT, post.text);

            // Notice how we haven't specified the primary key. SQLite auto increments the primary key column.
            db.insertOrThrow(TABLE_POSTS, null, values);
            db.setTransactionSuccessful();
        } catch (Exception e) {
            Log.d(TAG, "Error while trying to add post to database");
        } finally {
            db.endTransaction();
        }
    }

    // Insert or update a user in the database
    // Since SQLite doesn't support "upsert" we need to fall back on an attempt to UPDATE (in case the
    // user already exists) optionally followed by an INSERT (in case the user does not already exist).
    // Unfortunately, there is a bug with the insertOnConflict method
    // (https://code.google.com/p/android/issues/detail?id=13045) so we need to fall back to the more
    // verbose option of querying for the user's primary key if we did an update.
    public long addOrUpdateUser(User user) {
        // The database connection is cached so it's not expensive to call getWriteableDatabase() multiple times.
        SQLiteDatabase db = getWritableDatabase();
        long userId = -1;

        db.beginTransaction();
        try {
            ContentValues values = new ContentValues();
            values.put(KEY_USER_NAME, user.userName);
            values.put(KEY_USER_PROFILE_PICTURE_URL, user.profilePictureUrl);

            // First try to update the user in case the user already exists in the database
            // This assumes userNames are unique
            int rows = db.update(TABLE_USERS, values, KEY_USER_NAME + "= ?", new String[]{user.userName});

            // Check if update succeeded
            if (rows == 1) {
                // Get the primary key of the user we just updated
                String usersSelectQuery = String.format("SELECT %s FROM %s WHERE %s = ?",
                        KEY_USER_ID, TABLE_USERS, KEY_USER_NAME);
                Cursor cursor = db.rawQuery(usersSelectQuery, new String[]{String.valueOf(user.userName)});
                try {
                    if (cursor.moveToFirst()) {
                        userId = cursor.getInt(0);
                        db.setTransactionSuccessful();
                    }
                } finally {
                    if (cursor != null && !cursor.isClosed()) {
                        cursor.close();
                    }
                }
            } else {
                // user with this userName did not already exist, so insert new user
                userId = db.insertOrThrow(TABLE_USERS, null, values);
                db.setTransactionSuccessful();
            }
        } catch (Exception e) {
            Log.d(TAG, "Error while trying to add or update user");
        } finally {
            db.endTransaction();
        }
        return userId;
    }

    // Get all posts in the database
    public List<Post> getAllPosts() {
        List<Post> posts = new ArrayList<>();

        // SELECT * FROM POSTS
        // LEFT OUTER JOIN USERS
        // ON POSTS.KEY_POST_USER_ID_FK = USERS.KEY_USER_ID
        String POSTS_SELECT_QUERY =
                String.format("SELECT * FROM %s LEFT OUTER JOIN %s ON %s.%s = %s.%s",
                        TABLE_POSTS,
                        TABLE_USERS,
                        TABLE_POSTS, KEY_POST_USER_ID_FK,
                        TABLE_USERS, KEY_USER_ID);

        // "getReadableDatabase()" and "getWriteableDatabase()" return the same object (except under low
        // disk space scenarios)
        SQLiteDatabase db = getReadableDatabase();
        Cursor cursor = db.rawQuery(POSTS_SELECT_QUERY, null);
        try {
            if (cursor.moveToFirst()) {
                do {
                    User newUser = new User();
                    newUser.userName = cursor.getString(cursor.getColumnIndex(KEY_USER_NAME));
                    newUser.profilePictureUrl = cursor.getString(cursor.getColumnIndex(KEY_USER_PROFILE_PICTURE_URL));

                    Post newPost = new Post();
                    newPost.text = cursor.getString(cursor.getColumnIndex(KEY_POST_TEXT));
                    newPost.user = newUser;
                    posts.add(newPost);
                } while(cursor.moveToNext());
            }
        } catch (Exception e) {
            Log.d(TAG, "Error while trying to get posts from database");
        } finally {
            if (cursor != null && !cursor.isClosed()) {
                cursor.close();
            }
        }
        return posts;
    }

    // Update the user's profile picture url
    public int updateUserProfilePicture(User user) {
        SQLiteDatabase db = this.getWritableDatabase();

        ContentValues values = new ContentValues();
        values.put(KEY_USER_PROFILE_PICTURE_URL, user.profilePictureUrl);

        // Updating profile picture url for user with that userName
        return db.update(TABLE_USERS, values, KEY_USER_NAME + " = ?",
                new String[] { String.valueOf(user.userName) });
    }

    // Delete all posts and users in the database
    public void deleteAllPostsAndUsers() {
        SQLiteDatabase db = getWritableDatabase();
        db.beginTransaction();
        try {
            // Order of deletions is important when foreign key relationships exist.
            db.delete(TABLE_POSTS, null, null);
            db.delete(TABLE_USERS, null, null);
            db.setTransactionSuccessful();
        } catch (Exception e) {
            Log.d(TAG, "Error while trying to delete all posts and users");
        } finally {
            db.endTransaction();
        }
    }    
}
```

## SQLite Database Debugging 

When working with SQLite, opening and inspecting the SQLite database can be helpful while debugging issues. 
You can leverage the [[Stetho library|Debugging-with-Stetho]] to view your data directly, or you can use the following command-line tools to retrieve the data.

The commands below will show how to get at the data (whether running on an emulator or an actual device). The commands should be performed **within the terminal or command-line**. Once you have the data, there are desktop SQLite viewers such as [DB Browser for SQLite](http://sqlitebrowser.org/) or [SQLite Professional](https://itunes.apple.com/us/app/sqlite-professional-read-only/id635299994?mt=12) to help inspect the SQLite data graphically.

### On an Emulator

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

### On a Device

There isn't a `SQLite3` executable on the device so our only option is to download the database file with:

```bash
./adb shell run-as <app package name> chmod 666 /data/data/<app package name>/databases/<database file name>
./adb shell cp /data/data/<app package name>/databases/<database file name> /sdcard/
./adb pull /sdcard/<database file name>
```

### Using Android Device Monitor

You can go to `Tools`->`Android Device Monitor`->`File Explorer` and look inside the `/data/<app package name>/databases` and download the file locally.  You can then use one of the previously mentioned SQLite desktop viewers.

## References

* [Vogella SQLite Tutorial](http://www.vogella.com/articles/AndroidSQLite/article.html)
* [AndroidHive Tutorial](http://www.androidhive.info/2011/11/android-sqlite-database-tutorial/)
* [Techotopia Tutorial](http://www.techotopia.com/index.php/An_Android_SQLite_Database_Tutorial)
* [Programming Android SQLite Tutorial](https://github.com/bmeike/ProgrammingAndroidExamples/tree/master/MicroJobs/src/com/oreilly/demo/android/pa/microjobs)
* [Optimizing Insert Performance](http://www.techrepublic.com/blog/software-engineer/turbocharge-your-sqlite-inserts-on-android/)