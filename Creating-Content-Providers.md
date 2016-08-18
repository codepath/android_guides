## Overview

Content providers are Android’s central mechanism that enables you to access data of other applications – mostly information stored in databases or flat files. As such content providers are one of Android’s central component types to support the modular approach common to Android. Without content providers accessing data of other apps would be a mess.

Content providers support the four basic operations, normally called CRUD-operations. CRUD is the acronym for create, read, update and delete. With content providers those objects simply represent data – most often a record (tuple) of a database – but they could also be a photo on your SD-card or a video on the web. Let's take a look at how to create a content provider.

## Contract Classes
Before creating your ContentProvider, it is a good idea to create contract classes that define your database schema. Let's consider creating a movie database. Inside of our `MovieContract` class, we need to define a few properties: a content authority, which is a unique identifier for our database, a base URI, and path names for each table:

```java
    /**
     * The Content Authority is a name for the entire content provider, similar to the relationship
     * between a domain name and its website. A convenient string to use for content authority is
     * the package name for the app, since it is guaranteed to be unique on the device.
     */
    public static final String CONTENT_AUTHORITY = "com.androidessence.moviedatabase";

    /**
     * The content authority is used to create the base of all URIs which apps will use to
     * contact this content provider.
     */
    private static final Uri BASE_CONTENT_URI = Uri.parse("content://" + CONTENT_AUTHORITY);

    /**
     * A list of possible paths that will be appended to the base URI for each of the different
     * tables.
     */
    public static final String PATH_MOVIE = "movie";
    public static final String PATH_GENRE = "genre";
```

For each of the tables, you should create a class that extends from BaseColumns, which includes an `_ID` string that is used for the auto increment id of each table. In addition, you'll define a URI for that table, the MIME types of return queries (which are either a directory of multiple rows, or a single item), and a method to build a URI for an individual row in that table. Below are examples for our MovieTable and GenreTable:

```java
    /**
     * Create one class for each table that handles all information regarding the table schema and
     * the URIs related to it.
     */
    public static final class MovieEntry implements BaseColumns {
        // Content URI represents the base location for the table
        public static final Uri CONTENT_URI =
                BASE_CONTENT_URI.buildUpon().appendPath(PATH_MOVIE).build();

        // These are special type prefixes that specify if a URI returns a list or a specific item
        public static final String CONTENT_TYPE =
                "vnd.android.cursor.dir/" + CONTENT_URI  + "/" + PATH_MOVIE;
        public static final String CONTENT_ITEM_TYPE =
                "vnd.android.cursor.item/" + CONTENT_URI + "/" + PATH_MOVIE;

        // Define the table schema
        public static final String TABLE_NAME = "movieTable";
        public static final String COLUMN_NAME = "movieName";
        public static final String COLUMN_RELEASE_DATE = "movieReleaseDate";
        public static final String COLUMN_GENRE = "movieGenre";

        // Define a function to build a URI to find a specific movie by it's identifier
        public static Uri buildMovieUri(long id){
            return ContentUris.withAppendedId(CONTENT_URI, id);
        }
    }

    public static final class GenreEntry implements BaseColumns{
        public static final Uri CONTENT_URI =
                BASE_CONTENT_URI.buildUpon().appendPath(PATH_GENRE).build();

        public static final String CONTENT_TYPE =
                "vnd.android.cursor.dir/" + CONTENT_URI + "/" + PATH_GENRE;
        public static final String CONTENT_ITEM_TYPE =
                "vnd.android.cursor.item/" + CONTENT_URI + "/" + PATH_GENRE;

        public static final String TABLE_NAME = "genreTable";
        public static final String COLUMN_NAME = "genreName";

        public static Uri buildGenreUri(long id){
            return ContentUris.withAppendedId(CONTENT_URI, id);
        }
    }
```

## SQLiteOpenHelper
Now that we've defined what our database will look like, we need to actually create the database. To do this, we use the [SQLiteOpenHelper](http://developer.android.com/reference/android/database/sqlite/SQLiteOpenHelper.html) class. We can start by creating the constructor of the class, which calls the base class using the database version (which will be incremented every time the database changes), and the name of the database:

```java
    public class MovieDBHelper extends SQLiteOpenHelper{
        private static final int DATABASE_VERSION = 1;
        private static final String DATABASE_NAME = "movieList.db"; 
 
        public MovieDBHelper(Context context){
           super(context, DATABASE_NAME, null, DATABASE_VERSION);
        }
    }
```

There are two required implementations for this class: `onCreate()` and `onUpgrade()`. These are fairly self explanatory, the first is called when the database is created, and the second is called anytime `DATABASE_VERSION` is incremented. For this example, we will not handle updating the database:

```java
    /**
     * Called when the database is first created.
     * @param db The database being created, which all SQL statements will be executed on.
     */
    @Override
    public void onCreate(SQLiteDatabase db) {
        addGenreTable(db);
        addMovieTable(db);
    }

    /**
     * Called whenever DATABASE_VERSION is incremented. This is used whenever schema changes need
     * to be made or new tables are added.
     * @param db The database being updated.
     * @param oldVersion The previous version of the database. Used to determine whether or not
     *                   certain updates should be run.
     * @param newVersion The new version of the database.
     */
    @Override
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {

    }
```

Lastly, we will write the SQL code to create the database tables. Since this is not a SQL tutorial, I won't give them much explanation:

```java
    /**
     * Inserts the genre table into the database.
     * @param db The SQLiteDatabase the table is being inserted into.
     */
    private void addGenreTable(SQLiteDatabase db){
        db.execSQL(
                "CREATE TABLE " + MovieContract.GenreEntry.TABLE_NAME + " (" +
                        MovieContract.GenreEntry._ID + " INTEGER PRIMARY KEY, " +
                        MovieContract.GenreEntry.COLUMN_NAME + " TEXT UNIQUE NOT NULL);"
        );
    }

    /**
     * Inserts the movie table into the database.
     * @param db The SQLiteDatabase the table is being inserted into.
     */
    private void addMovieTable(SQLiteDatabase db){
        db.execSQL(
                "CREATE TABLE " + MovieContract.MovieEntry.TABLE_NAME + " (" +
                        MovieContract.MovieEntry._ID + " INTEGER PRIMARY KEY, " +
                        MovieContract.MovieEntry.COLUMN_NAME + " TEXT NOT NULL, " +
                        MovieContract.MovieEntry.COLUMN_RELEASE_DATE + " TEXT NOT NULL, " +
                        MovieContract.MovieEntry.COLUMN_GENRE + " INTEGER NOT NULL, " +
                        "FOREIGN KEY (" + MovieContract.MovieEntry.COLUMN_GENRE + ") " +
                        "REFERENCES " + MovieContract.GenreEntry.TABLE_NAME + " (" + MovieContract.GenreEntry._ID + "));"
        );
    }
```

## ContentProvider
Now that we have covered everything necessary to setup our provider, let's discuss how to actually write the [ContentProvider](http://developer.android.com/reference/android/content/ContentProvider.html) class. I'll break down step by step how to complete this class.

First, we need to define an integer identifier for each URI or query we plan to write. In this case we will have two for each query; A URI for all rows, and a URI for an individual row:

```java
public class MovieProvider extends ContentProvider {
    // Use an int for each URI we will run, this represents the different queries
    private static final int GENRE = 100;
    private static final int GENRE_ID = 101;
    private static final int MOVIE = 200;
    private static final int MOVIE_ID = 201;
}
```

Next, we can define our other class variables such as our SQLiteOpenHelper which is used to access the database itself, and a URIMatcher that will take in a URI and match it to the appropriate integer identifier we just defined:

```java
    private static final UriMatcher sUriMatcher = buildUriMatcher();
    private MovieDBHelper mOpenHelper;

    @Override
    public boolean onCreate() {
        mOpenHelper = new MovieDBHelper(getContext());
        return true;
    }

    /**
     * Builds a UriMatcher that is used to determine witch database request is being made.
     */
    public static UriMatcher buildUriMatcher(){
        String content = MovieContract.CONTENT_AUTHORITY;

        // All paths to the UriMatcher have a corresponding code to return
        // when a match is found (the ints above).
        UriMatcher matcher = new UriMatcher(UriMatcher.NO_MATCH);
        matcher.addURI(content, MovieContract.PATH_GENRE, GENRE);
        matcher.addURI(content, MovieContract.PATH_GENRE + "/#", GENRE_ID);
        matcher.addURI(content, MovieContract.PATH_MOVIE, MOVIE);
        matcher.addURI(content, MovieContract.PATH_MOVIE + "/#", MOVIE_ID);

        return matcher;
    }
```

The main thing to notice here is that if we see a URI that ends with a given path, it will match with the URI for that table, but if an id is appended to the path we are looking for a row with that id.

### getType

The `getType` method is used to find the MIME type of the results, either a directory of multiple results, or an individual item:

```java

    @Override
    public String getType(Uri uri) {
        switch(sUriMatcher.match(uri)){
            case GENRE:
                return MovieContract.GenreEntry.CONTENT_TYPE;
            case GENRE_ID:
                return MovieContract.GenreEntry.CONTENT_ITEM_TYPE;
            case MOVIE:
                return MovieContract.MovieEntry.CONTENT_TYPE;
            case MOVIE_ID:
                return MovieContract.MovieEntry.CONTENT_ITEM_TYPE;
            default:
                throw new UnsupportedOperationException("Unknown uri: " + uri);
        }
    }
```

### query

The query method takes in five parameters:
- uri: The URI (or table) that should be queried.
- projection: A string array of columns that will be returned in the result set.
- selection: A string defining the criteria for results to be returned.
- selectionArgs: Arguments to the above criteria that rows will be checked against.
- sortOrder: A string of the column(s) and order to sort the result set by.

In order to query the database, we will switch based on the matched URI integer and query the appropriate table as necessary.

```java
    @Override
    public Cursor query(Uri uri, String[] projection, String selection, String[] selectionArgs, String sortOrder) {
        final SQLiteDatabase db = mOpenHelper.getWritableDatabase();
        Cursor retCursor;
        switch(sUriMatcher.match(uri)){
            case GENRE:
                retCursor = db.query(
                        MovieContract.GenreEntry.TABLE_NAME,
                        projection,
                        selection,
                        selectionArgs,
                        null,
                        null,
                        sortOrder
                );
                break;
            case GENRE_ID:
                long _id = ContentUris.parseId(uri);
                retCursor = db.query(
                        MovieContract.GenreEntry.TABLE_NAME,
                        projection,
                        MovieContract.GenreEntry._ID + " = ?",
                        new String[]{String.valueOf(_id)},
                        null,
                        null,
                        sortOrder
                );
                break;
            case MOVIE:
                retCursor = db.query(
                        MovieContract.MovieEntry.TABLE_NAME,
                        projection,
                        selection,
                        selectionArgs,
                        null,
                        null,
                        sortOrder
                );
                break;
            case MOVIE_ID:
                _id = ContentUris.parseId(uri);
                retCursor = db.query(
                        MovieContract.MovieEntry.TABLE_NAME,
                        projection,
                        MovieContract.MovieEntry._ID + " = ?",
                        new String[]{String.valueOf(_id)},
                        null,
                        null,
                        sortOrder
                );
                break;
            default:
                throw new UnsupportedOperationException("Unknown uri: " + uri);
        }

        // Set the notification URI for the cursor to the one passed into the function. This
        // causes the cursor to register a content observer to watch for changes that happen to
        // this URI and any of it's descendants. By descendants, we mean any URI that begins
        // with this path.
        retCursor.setNotificationUri(getContext().getContentResolver(), uri);
        return retCursor;
    }
```

### insert

The insert method takes in a `ContentValues` object, which is a key value pair of column names and values to be inserted. Similar to the query method, and the update and delete methods that follow, we will switch based on the URI and act on the appropriate table:

```java
    @Override
    public Uri insert(Uri uri, ContentValues values) {
        final SQLiteDatabase db = mOpenHelper.getWritableDatabase();
        long _id;
        Uri returnUri;

        switch(sUriMatcher.match(uri)){
            case GENRE:
                _id = db.insert(MovieContract.GenreEntry.TABLE_NAME, null, values);
                if(_id > 0){
                    returnUri =  MovieContract.GenreEntry.buildGenreUri(_id);
                } else{
                    throw new UnsupportedOperationException("Unable to insert rows into: " + uri);
                }
                break;
            case MOVIE:
                _id = db.insert(MovieContract.MovieEntry.TABLE_NAME, null, values);
                if(_id > 0){
                    returnUri = MovieContract.MovieEntry.buildMovieUri(_id);
                } else{
                    throw new UnsupportedOperationException("Unable to insert rows into: " + uri);
                }
                break;
            default:
                throw new UnsupportedOperationException("Unknown uri: " + uri);
        }

        // Use this on the URI passed into the function to notify any observers that the uri has
        // changed.
        getContext().getContentResolver().notifyChange(uri, null);
        return returnUri;
    }
```

### update and delete

The update and delete methods take in a selection string and arguments to define which rows should be updated or deleted. They differ in that the update method requires a ContentProvider object as well, for the columns in that row(s) that will be updated.

```java
    @Override
    public int delete(Uri uri, String selection, String[] selectionArgs) {
        final SQLiteDatabase db = mOpenHelper.getWritableDatabase();
        int rows; // Number of rows effected

        switch(sUriMatcher.match(uri)){
            case GENRE:
                rows = db.delete(MovieContract.GenreEntry.TABLE_NAME, selection, selectionArgs);
                break;
            case MOVIE:
                rows = db.delete(MovieContract.MovieEntry.TABLE_NAME, selection, selectionArgs);
                break;
            default:
                throw new UnsupportedOperationException("Unknown uri: " + uri);
        }

        // Because null could delete all rows:
        if(selection == null || rows != 0){
            getContext().getContentResolver().notifyChange(uri, null);
        }

        return rows;
    }

    @Override
    public int update(Uri uri, ContentValues values, String selection, String[] selectionArgs) {
        final SQLiteDatabase db = mOpenHelper.getWritableDatabase();
        int rows;

        switch(sUriMatcher.match(uri)){
            case GENRE:
                rows = db.update(MovieContract.GenreEntry.TABLE_NAME, values, selection, selectionArgs);
                break;
            case MOVIE:
                rows = db.update(MovieContract.MovieEntry.TABLE_NAME, values, selection, selectionArgs);
                break;
            default:
                throw new UnsupportedOperationException("Unknown uri: " + uri);
        }

        if(rows != 0){
            getContext().getContentResolver().notifyChange(uri, null);
        }

        return rows;
    }
```

##Using the Content Provider
In order to use the content provider, even from within your own app, you must update the AndroidManifest.xml file:  In the ``application`` node, add:


```

     <provider
            android:name=".MovieProvider"
            android:authorities="com.androidessence.moviedatabase"
            android:exported="false"
            android:protectionLevel="signature"
            android:syncable="true"/>
```

The code seen above, as well as a sample project can be found on Github: https://github.com/androidessence/MovieDatabase

## References

* <http://code.tutsplus.com/tutorials/android-sdk_content-providers--mobile-5549>
* <http://www.grokkingandroid.com/android-tutorial-writing-your-own-content-provider/>
* <http://developer.android.com/guide/topics/providers/content-providers.html>
* <https://thenewcircle.com/s/post/1375/android_content_provider_tutorial>
* <http://www.grokkingandroid.com/android-tutorial-content-provider-basics/>
* <http://androidessence.com/>