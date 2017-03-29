## Overview

For most Android applications, maintaining persistent synchronization between local and cloud data source can be troubling, complex, and frustrating, yet it is essential. Luckily, Android provides the SyncAdapter framework to batch synchronization efficiently for us! 

The SyncAdapter can be invoked manually or can be set to sync periodically at regular intervals. There are also a plethora of other awesome features that using the SyncAdapter framework allows, such as account settings and a nice logo in the 'Accounts' tab in Android settings... which implicates its reliability on the AccountManager framework as well.

To use the SyncAdapter framework, you must create a few components: 
1. _ContentProvider_ (used to update local content), 
2. _Authenticator_ (can be stubbed or used to retrieve tokens such as OAuth tokens), 
3. _AuthenticatorService_ (Used only by Android to use authenticator if needed),
4. _SyncAdapter_ (Performs the actual synchronization),
5. _SyncService_ (Used only by Android to run your SyncAdapter)

**As a hypothetical use case, we'll pretend that we are creating a SyncAdapter to synchronize pretend RSS feed with our application herein.**

## Prelude
Before being able to jump into building our SyncAdapter, we must create a few components before we can start.

### Create Model Classes
We need to create a simple class to model what our hypothetical article's data should look like.
```java
/**
 * Represents a very simple RSS article.
 */
public class Article {
    private String id;
    private String title;
    private String content;
    private String link;


    public Article() {}

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    public String getContent() {
        return content;
    }

    public void setContent(String content) {
        this.content = content;
    }

    public String getLink() {
        return link;
    }

    public void setLink(String link) {
        this.link = link;
    }
}
```

### Create a Parsing Utility
We need to create a utility that will convert JSON data into our Article model.
```java
/**
 * This is an example parser to 'parse' pretend json news feed.
 */
public class ArticleParser {
    public static Article parse(JSONObject jsonArticle) {
        Article article = new Article();
        article.setId(jsonArticle.optString("id"));
        article.setTitle(jsonArticle.optString("title"));
        article.setContent(jsonArticle.optString("content"));
        article.setLink(jsonArticle.optString("link"));
        return article;
    }
}
```

### Create Contract Classes
Just like recommended in other Wiki pages on here, it's a good idea to define your database and provider structure concretely. (Refer to this great [article](http://guides.codepath.com/android/Creating-Content-Providers) for a more detailed explanation)
```java
/**
 * Define all local entities in this class.
 */
public final class ArticleContract {
    // ContentProvider information
    public static final String CONTENT_AUTHORITY = "com.example.sync";
    static final Uri BASE_CONTENT_URI = Uri.parse("content://" + CONTENT_AUTHORITY);
    static final String PATH_ARTICLES = "articles";

    // Database information
    static final String DB_NAME = "articles_db";
    static final int DB_VERSION = 1;


    /**
     * This represents our SQLite table for our articles.
     */
    public static abstract class Articles {
        public static final String NAME = "articles";
        public static final String COL_ID = "articleId";
        public static final String COL_TITLE = "articleTitle";
        public static final String COL_CONTENT = "articleContent";
        public static final String COL_LINK = "articleLink";

        // ContentProvider information for articles
        public static final Uri CONTENT_URI =
                BASE_CONTENT_URI.buildUpon().appendPath(PATH_ARTICLES).build();
        public static final String CONTENT_TYPE =
                "vnd.android.cursor.dir/" + CONTENT_URI + "/" + PATH_ARTICLES;
        public static final String CONTENT_ITEM_TYPE =
                "vnd.android.cursor.item/" + CONTENT_URI + "/" + PATH_ARTICLES;
    }
}
```

### Create SQLite Database
After defining our database structure, we will need to create, manage, and access it accordingly using the SQLiteOpenHelper class. For detailed information on using this, refer to this [article](http://guides.codepath.com/android/Creating-Content-Providers#SQLiteOpenHelper). 

There are several ways to build this class, but I'm using the Singleton pattern to provide access to the SQLiteDatabase class.
```java
/**
 * Notice how we are inheriting {@link SQLiteOpenHelper}, this requires we do:
 * (1) Call parent constructor (specify database info)
 * (2) Implement {@link #onCreate(SQLiteDatabase)} (create our tables here)
 * (3) Implement {@link #onUpgrade(SQLiteDatabase, int, int)} (update our tables here)
 *
 * {@link #db} stores a reference to our database we want to use.
 */
public final class DatabaseClient extends SQLiteOpenHelper {
    private static volatile DatabaseClient instance;
    private final SQLiteDatabase db;


    private DatabaseClient(Context c) {
        super(c, DB_NAME, null, DB_VERSION);
        this.db = getWritableDatabase();
    }

    /**
     * We use a Singleton to prevent leaking the SQLiteDatabase or Context.
     * @return {@link DatabaseClient}
     */
    public static DatabaseClient getInstance(Context c) {
        if (instance == null) {
            synchronized (DatabaseClient.class) {
                if (instance == null) {
                    instance = new DatabaseClient(c);
                }
            }
        }
        return instance;
    }

    @Override
    public void onCreate(SQLiteDatabase db) {
        // Create any SQLite tables here
        createArticlesTable(db);
    }

    @Override
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
        // Update any SQLite tables here
        db.execSQL("DROP TABLE IF EXISTS [" + Articles.NAME + "];");
        onCreate(db);
    }

    /**
     * Provide access to our database.
     */
    public SQLiteDatabase getDb() {
        return db;
    }

    /**
     * Creates our 'articles' SQLite database table.
     * @param db {@link SQLiteDatabase}
     */
    private void createArticlesTable(SQLiteDatabase db) {
        db.execSQL("CREATE TABLE [" + Articles.NAME + "] ([" +
                Articles.COL_ID + "] TEXT UNIQUE PRIMARY KEY,[" +
                Articles.COL_TITLE + "] TEXT NOT NULL,[" +
                Articles.COL_CONTENT + "] TEXT,[" +
                Articles.COL_LINK + "] TEXT);");
    }
}
```

## Step 1: The ContentProvider
Now that we've created the necessary components to implement a ContentProvider, we can start. We need to create a ContentProvider for our hypothetical example to provide access to our local data source.

This ContentProvider is absolutely imperative as we use it to synchronize local data changes and utilize its content observer to potentially update the UI after syncing.

For an in-depth tutorial on creating a ContentProvider, refer to this fantastic [article](http://guides.codepath.com/android/Creating-Content-Providers).
```java
/**
 * This is the ContentProvider that will be used by our SyncAdapter to sync local data.
 */
public class ArticleProvider extends ContentProvider {
    // Use ints to represent different queries
    private static final int ARTICLE = 1;
    private static final int ARTICLE_ID = 2;

    private static final UriMatcher uriMatcher;
    static {
        // Add all our query types to our UriMatcher
        uriMatcher = new UriMatcher(UriMatcher.NO_MATCH);
        uriMatcher.addURI(ArticleContract.CONTENT_AUTHORITY, ArticleContract.PATH_ARTICLES, ARTICLE);
        uriMatcher.addURI(ArticleContract.CONTENT_AUTHORITY, ArticleContract.PATH_ARTICLES + "/#", ARTICLE_ID);
    }

    private SQLiteDatabase db;


    @Override
    public boolean onCreate() {
        this.db = DatabaseClient.getInstance(getContext()).getDb();
        return true;
    }

    @Nullable
    @Override
    public String getType(@NonNull Uri uri) {
        // Find the MIME type of the results... multiple results or a single result
        switch (uriMatcher.match(uri)) {
            case ARTICLE:
                return ArticleContract.Articles.CONTENT_TYPE;
            case ARTICLE_ID:
                return ArticleContract.Articles.CONTENT_ITEM_TYPE;
            default: throw new IllegalArgumentException("Invalid URI!");
        }
    }

    @Nullable
    @Override
    public Cursor query(@NonNull Uri uri, @Nullable String[] projection, @Nullable String selection, @Nullable String[] selectionArgs, @Nullable String sortOrder) {
        Cursor c;
        switch (uriMatcher.match(uri)) {
            // Query for multiple article results
            case ARTICLE:
                c = db.query(ArticleContract.Articles.NAME,
                        projection,
                        selection,
                        selectionArgs,
                        null,
                        null,
                        sortOrder);
                break;

            // Query for single article result
            case ARTICLE_ID:
                long _id = ContentUris.parseId(uri);
                c = db.query(ArticleContract.Articles.NAME,
                        projection,
                        ArticleContract.Articles.COL_ID + "=?",
                        new String[] { String.valueOf(_id) },
                        null,
                        null,
                        sortOrder);
                break;
            default: throw new IllegalArgumentException("Invalid URI!");
        }

        // Tell the cursor to register a content observer to observe changes to the
        // URI or its descendants.
        assert getContext() != null;
        c.setNotificationUri(getContext().getContentResolver(), uri);
        return c;
    }

    @Nullable
    @Override
    public Uri insert(@NonNull Uri uri, @Nullable ContentValues values) {
        Uri returnUri;
        long _id;

        switch (uriMatcher.match(uri)) {
            case ARTICLE:
                _id = db.insert(ArticleContract.Articles.NAME, null, values);
                returnUri = ContentUris.withAppendedId(ArticleContract.Articles.CONTENT_URI, _id);
                break;
            default: throw new IllegalArgumentException("Invalid URI!");
        }

        // Notify any observers to update the UI
        assert getContext() != null;
        getContext().getContentResolver().notifyChange(uri, null);
        return returnUri;
    }

    @Override
    public int update(@NonNull Uri uri, @Nullable ContentValues values, @Nullable String selection, @Nullable String[] selectionArgs) {
        int rows;
        switch (uriMatcher.match(uri)) {
            case ARTICLE:
                rows = db.update(ArticleContract.Articles.NAME, values, selection, selectionArgs);
                break;
            default: throw new IllegalArgumentException("Invalid URI!");
        }

        // Notify any observers to update the UI
        if (rows != 0) {
            assert getContext() != null;
            getContext().getContentResolver().notifyChange(uri, null);
        }
        return rows;
    }

    @Override
    public int delete(@NonNull Uri uri, @Nullable String selection, @Nullable String[] selectionArgs) {
        int rows;
        switch (uriMatcher.match(uri)) {
            case ARTICLE:
                rows = db.delete(ArticleContract.Articles.NAME, selection, selectionArgs);
                break;
            default: throw new IllegalArgumentException("Invalid URI!");
        }

        // Notify any observers to update the UI
        if (rows != 0) {
            assert getContext() != null;
            getContext().getContentResolver().notifyChange(uri, null);
        }
        return rows;
    }
}
```

### Declare the ContentProvider
In order to use our ContentProvider, we need to declare it in the `manifest.xml` file. It is important to note that:
1. `android:authorities` must match our ContentProvider's authority exactly, and
2. `android:syncable` must be true.
```
<provider
    android:name=".example.ArticleProvider"
    android:authorities="com.example.sync"
    android:exported="false"
    android:syncable="true"/>
```

## Steps 2 & 3: The Account and Authenticator
The next steps requires that we have an Account on the device. This seems like a lot of work or unnecessary, but it is worth it for a prodigious amount of reasons:
* Supports the SyncAdapter framework
* Supports various access tokens (access rights)
* Supports various account settings features
* Standardization for authentication
* Sharing your account across multiple apps, like Google does

There are many ways you can utilize account creation and authentication (check out the references), but for the scope of our example (syncing RSS feed), we just want a very simple account with stubbed authentication.

### Declare Proper Permissions
Before continuing, we must add these NEEDED permissions to the `manifest.xml` so we can use SyncAdapter, AccountManager, and Sync Settings.
```
<uses-permission android:name="android.permission.GET_ACCOUNTS"/>
<uses-permission android:name="android.permission.MANAGE_ACCOUNTS"/>
<uses-permission android:name="android.permission.AUTHENTICATE_ACCOUNTS"/>
<uses-permission android:name="android.permission.WRITE_SYNC_SETTINGS"/>
<uses-permission android:name="android.permission.READ_SYNC_SETTINGS" />
<uses-permission android:name="android.permission.READ_SYNC_STATS"/>
```

### Create Contract Class for Account
Again, it's a good idea to concretely define our account structure; especially since our account will be really simple.
```java
public final class AccountGeneral {
    /**
     * This is the type of account we are using. i.e. we can specify our app or apps
     * to have different types, such as 'read-only', 'sync-only', & 'admin'.
     */
    private static final String ACCOUNT_TYPE = "com.example.syncaccount";

    /**
     * This is the name that appears in the Android 'Accounts' settings.
     */
    private static final String ACCOUNT_NAME = "Example Sync";


    /**
     * Gets the standard sync account for our app.
     * @return {@link Account}
     */
    public static Account getAccount() {
        return new Account(ACCOUNT_NAME, ACCOUNT_TYPE);
    }

    /**
     * Creates the standard sync account for our app.
     * @param c {@link Context}
     */
    public static void createSyncAccount(Context c) {
        // Flag to determine if this is a new account or not
        boolean created = false;

        // Get an account and the account manager
        Account account = getAccount();
        AccountManager manager = (AccountManager)c.getSystemService(Context.ACCOUNT_SERVICE);

        // Attempt to explicitly create the account with no password or extra data
        if (manager.addAccountExplicitly(account, null, null)) {
            final String AUTHORITY = ArticleContract.CONTENT_AUTHORITY;
            final long SYNC_FREQUENCY = 60 * 60; // 1 hour (seconds)

            // Inform the system that this account supports sync
            ContentResolver.setIsSyncable(account, AUTHORITY, 1);

            // Inform the system that this account is eligible for auto sync when the network is up
            ContentResolver.setSyncAutomatically(account, AUTHORITY, true);

            // Recommend a schedule for automatic synchronization. The system may modify this based
            // on other scheduled syncs and network utilization.
            ContentResolver.addPeriodicSync(account, AUTHORITY, new Bundle(), SYNC_FREQUENCY);

            created = true;
        }
        
        // Force a sync if the account was just created
        if (created) {
            SyncAdapter.performSync();
        }
    }
}
```

### Create an Account Authenticator
The authenticator will perform all the actions on the account type. It will also know which activity to show for the user to enter their credential and where to find any stored auth-token that the server has previously returned. This can also be common to many different services under a single account type.

For example, Google's authenticator on Android authenticates the Google Mail service (Gmail), Google Calendar, Google Drive, and along with many other Google services.

We need to make an authenticator that extends the AbstractAccountAuthenticator class. This is really simple because we don't use any authentication, therefore, we stub it.
```java
/**
 * This is stubbed because we don't need any authentication to access the pretend RSS feed.
 */
public class AccountAuthenticator extends AbstractAccountAuthenticator {
    public AccountAuthenticator(Context c) {
        super(c);
    }
    
    @Override
    public Bundle addAccount(AccountAuthenticatorResponse response, String accountType, String authTokenType, String[] requiredFeatures, Bundle options) throws NetworkErrorException {
        return null;
    }

    @Override
    public Bundle confirmCredentials(AccountAuthenticatorResponse response, Account account, Bundle options) throws NetworkErrorException {
        return null;
    }

    @Override
    public Bundle getAuthToken(AccountAuthenticatorResponse response, Account account, String authTokenType, Bundle options) throws NetworkErrorException {
        return null;
    }

    @Override
    public String getAuthTokenLabel(String authTokenType) {
        return null;
    }

    @Override
    public Bundle updateCredentials(AccountAuthenticatorResponse response, Account account, String authTokenType, Bundle options) throws NetworkErrorException {
        return null;
    }

    @Override
    public Bundle editProperties(AccountAuthenticatorResponse response, String accountType) {
        return null;
    }

    @Override
    public Bundle hasFeatures(AccountAuthenticatorResponse response, Account account, String[] features) throws NetworkErrorException {
        return null;
    }
}
```

### Create the Authentication Service
For Android to use our AccountAuthenticator, it needs to run it in a bound Service that will allow it to do so. To see the in-depth of how AbstractAccountAuthenticator works, look at its Transport inner-class and read about AIDL for inter-process communication.

We need to create this bound Service so we can let Android use our `AccountAuthenticator` class.
```java
/**
 * This is used only by Android to run our {@link AccountAuthenticator}.
 */
public class AuthenticatorService extends Service {
    private AccountAuthenticator authenticator;


    @Override
    public void onCreate() {
        // Instantiate our authenticator when the service is created
        this.authenticator = new AccountAuthenticator(this);
    }

    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        // Return the authenticator's IBinder
        return authenticator.getIBinder();
    }
}
```

### (Optional) Declare Account Preferences
These preferences will be shown when accessing the account's preferences from the device Settings screen. This allows the user more control over their account. 

To create this, we need to create a resource package in 'res' called, 'xml', if not already created, and create 'syncsettings.xml'.
```
<?xml version="1.0" encoding="utf-8"?>
<PreferenceScreen
    xmlns:android="http://schemas.android.com/apk/res/android">
    <SwitchPreference
        android:key="use_sync"
        android:title="Toggle Options"
        android:summary="This is an example toggle setting"/>
</PreferenceScreen>
```

### Declare the Authenticator
We need to create a resource package in 'res' called, 'xml', if not already created, and create 'authenticator.xml'.
* `android:accountType=(string)` - identify our account type when an app wants to authenticate us (match EXACTLY in our AccountsGeneral)
* `android:label=(string)` - Name of the account on the device Settings
* `android:icon=(drawable)` - is the normal icon that will appear in the 'Accounts' Android settings
* `android:smallIcon=(drawable)` - (optional) is the small icon the will show for the account
* `android:accountPreferences=(xml)` - (optional) specifies extra settings for our account
```
<?xml version="1.0" encoding="utf-8"?>
<account-authenticator
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:accountType="com.someonew.syncaccount"
    android:icon="@mipmap/ic_launcher_round"
    android:smallIcon="@mipmap/ic_launcher_round"
    android:accountPreferences="@xml/syncsettings"
    android:label="@string/app_name"/>
```

### Declare the Authenticator Service
We need to declare our `AuthenticatorService` in the `manifest.xml` file.
```
<service android:name=".example.AuthenticatorService">
    <intent-filter>
        <action android:name="android.accounts.AccountAuthenticator"/>
    </intent-filter>
    <meta-data
        android:name="android.accounts.AccountAuthenticator"
        android:resource="@xml/authenticator"/>
</service>
```

## Steps 4 & 5: The SyncAdapter and SyncService
The next steps requires us to have our `SyncAdapter` and another bound Service to allow Android to run it. These are some of the benefits it will offer us:
* Status check and start synchronization when network is available
* Scheduler that synchronizes using criteria
* Auto synchronization if it had previously failed
* Saves battery power because the system will batch networking

### Create the SyncAdapter
To create our `SyncAdapter`, we must extend the AbstractThreadedSyncAdapter class. 

It's important to note that any logic that's performed in the `onPerformSync()` method, happens on a new Thread. Do NOT create a new Thread to run networking or long performing tasks because Android handles that for you!
```java
/**
 * This is used by the Android framework to perform synchronization. IMPORTANT: do NOT create
 * new Threads to perform logic, Android will do this for you; hence, the name.
 *
 * The goal here to perform synchronization, is to do it efficiently as possible. We use some
 * ContentProvider features to batch our writes to the local data source. Be sure to handle all
 * possible exceptions accordingly; random crashes is not a good user-experience.
 */
public class SyncAdapter extends AbstractThreadedSyncAdapter {
    private static final String TAG = "SYNC_ADAPTER";

    /**
     * This gives us access to our local data source.
     */
    private final ContentResolver resolver;


    public SyncAdapter(Context c, boolean autoInit) {
        this(c, autoInit, false);
    }

    public SyncAdapter(Context c, boolean autoInit, boolean parallelSync) {
        super(c, autoInit, parallelSync);
        this.resolver = c.getContentResolver();
    }

    /**
     * This method is run by the Android framework, on a new Thread, to perform a sync.
     * @param account Current account
     * @param extras Bundle extras
     * @param authority Content authority
     * @param provider {@link ContentProviderClient}
     * @param syncResult Object to write stats to
     */
    @Override
    public void onPerformSync(Account account, Bundle extras, String authority, ContentProviderClient provider, SyncResult syncResult) {
        Log.w(TAG, "Starting synchronization...");

        try {
            // Synchronize our news feed
            syncNewsFeed(syncResult);

            // Add any other things you may want to sync

        } catch (IOException ex) {
            Log.e(TAG, "Error synchronizing!", ex);
            syncResult.stats.numIoExceptions++;
        } catch (JSONException ex) {
            Log.e(TAG, "Error synchronizing!", ex);
            syncResult.stats.numParseExceptions++;
        } catch (RemoteException|OperationApplicationException ex) {
            Log.e(TAG, "Error synchronizing!", ex);
            syncResult.stats.numAuthExceptions++;
        }

        Log.w(TAG, "Finished synchronization!");
    }

    /**
     * Performs synchronization of our pretend news feed source.
     * @param syncResult Write our stats to this
     */
    private void syncNewsFeed(SyncResult syncResult) throws IOException, JSONException, RemoteException, OperationApplicationException {
        final String rssFeedEndpoint = "http://www.examplejsonnews.com";

        // We need to collect all the network items in a hash table
        Log.i(TAG, "Fetching server entries...");
        Map<String, Article> networkEntries = new HashMap<>();

        // Parse the pretend json news feed
        String jsonFeed = download(rssFeedEndpoint);
        JSONArray jsonArticles = new JSONArray(jsonFeed);
        for (int i = 0; i < jsonArticles.length(); i++) {
            Article article = ArticleParser.parse(jsonArticles.optJSONObject(i));
            networkEntries.put(article.getId(), article);
        }

        // Create list for batching ContentProvider transactions
        ArrayList<ContentProviderOperation> batch = new ArrayList<>();

        // Compare the hash table of network entries to all the local entries
        Log.i(TAG, "Fetching local entries...");
        Cursor c = resolver.query(ArticleContract.Articles.CONTENT_URI, null, null, null, null, null);
        assert c != null;
        c.moveToFirst();

        String id;
        String title;
        String content;
        String link;
        Article found;
        for (int i = 0; i < c.getCount(); i++) {
            syncResult.stats.numEntries++;

            // Create local article entry
            id = c.getString(c.getColumnIndex(ArticleContract.Articles.COL_ID));
            title = c.getString(c.getColumnIndex(ArticleContract.Articles.COL_TITLE));
            content = c.getString(c.getColumnIndex(ArticleContract.Articles.COL_CONTENT));
            link = c.getString(c.getColumnIndex(ArticleContract.Articles.COL_LINK));

            // Try to retrieve the local entry from network entries
            found = networkEntries.get(id);
            if (found != null) {
                // The entry exists, remove from hash table to prevent re-inserting it
                networkEntries.remove(id);

                // Check to see if it needs to be updated
                if (!title.equals(found.getTitle())
                    || !content.equals(found.getContent())
                    || !link.equals(found.getLink())) {
                    // Batch an update for the existing record
                    Log.i(TAG, "Scheduling update: " + title);
                    batch.add(ContentProviderOperation.newUpdate(ArticleContract.Articles.CONTENT_URI)
                            .withSelection(ArticleContract.Articles.COL_ID + "='" + id + "'", null)
                            .withValue(ArticleContract.Articles.COL_TITLE, found.getTitle())
                            .withValue(ArticleContract.Articles.COL_CONTENT, found.getContent())
                            .withValue(ArticleContract.Articles.COL_LINK, found.getLink())
                            .build());
                    syncResult.stats.numUpdates++;
                }
            } else {
                // Entry doesn't exist, remove it from the local database
                Log.i(TAG, "Scheduling delete: " + title);
                batch.add(ContentProviderOperation.newDelete(ArticleContract.Articles.CONTENT_URI)
                        .withSelection(ArticleContract.Articles.COL_ID + "='" + id + "'", null)
                        .build());
                syncResult.stats.numDeletes++;
            }
            c.moveToNext();
        }
        c.close();

        // Add all the new entries
        for (Article article : networkEntries.values()) {
            Log.i(TAG, "Scheduling insert: " + article.getTitle());
            batch.add(ContentProviderOperation.newInsert(ArticleContract.Articles.CONTENT_URI)
                    .withValue(ArticleContract.Articles.COL_ID, article.getId())
                    .withValue(ArticleContract.Articles.COL_TITLE, article.getTitle())
                    .withValue(ArticleContract.Articles.COL_CONTENT, article.getContent())
                    .withValue(ArticleContract.Articles.COL_LINK, article.getLink())
                    .build());
            syncResult.stats.numInserts++;
        }

        // Synchronize by performing batch update
        Log.i(TAG, "Merge solution ready, applying batch update...");
        resolver.applyBatch(ArticleContract.CONTENT_AUTHORITY, batch);
        resolver.notifyChange(ArticleContract.Articles.CONTENT_URI, // URI where data was modified
                null, // No local observer
                false); // IMPORTANT: Do not sync to network
    }

    /**
     * A blocking method to stream the server's content and build it into a string.
     * @param url API call
     * @return String response
     */
    private String download(String url) throws IOException {
        // Ensure we ALWAYS close these!
        HttpURLConnection client = null;
        InputStream is = null;

        try {
            // Connect to the server using GET protocol
            URL server = new URL(url);
            client = (HttpURLConnection)server.openConnection();
            client.connect();

            // Check for valid response code from the server
            int status = client.getResponseCode();
            is = (status == HttpURLConnection.HTTP_OK)
                    ? client.getInputStream() : client.getErrorStream();

            // Build the response or error as a string
            BufferedReader br = new BufferedReader(new InputStreamReader(is));
            StringBuilder sb = new StringBuilder();
            for (String temp; ((temp = br.readLine()) != null);) {
                sb.append(temp);
            }

            return sb.toString();
        } finally {
            if (is != null) { is.close(); }
            if (client != null) { client.disconnect(); }
        }
    }

    /**
     * Manual force Android to perform a sync with our SyncAdapter.
     */
    public static void performSync() {
        Bundle b = new Bundle();
        b.putBoolean(ContentResolver.SYNC_EXTRAS_MANUAL, true);
        b.putBoolean(ContentResolver.SYNC_EXTRAS_EXPEDITED, true);
        ContentResolver.requestSync(AccountGeneral.getAccount(),
                ArticleContract.CONTENT_AUTHORITY, b);
    }
}
```

### Declare the SyncAdapter
We need to create a resource package in 'res' called, 'xml', if not already created, and create 'syncadapter.xml'.
* `android:contentAuthority=(string)` - Specifies our ContentProvider's authority, must EXACTLY match
* `android:accountType=(string)` - must match EXACTLY the account type defined in our `AccountGeneral`
* `android:userVisible=(true|false)` - True if sync is visible to the user
* `android:allowParallelSyncs=(true|false)` - True if can sync in parallel
* `android:isAlwaysSyncable=(true|false)` - True if can be synced at anytime
* `android:supportsUploading=(true|false)` - True if uploads to the server
```
<?xml version="1.0" encoding="utf-8"?>
<sync-adapter
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:contentAuthority="com.example.sync"
    android:accountType="com.example.syncaccount"
    android:userVisible="true"
    android:allowParallelSyncs="false"
    android:isAlwaysSyncable="true"
    android:supportsUploading="true"/>
```

### Create a Sync Service
For Android to use our `SyncAdapter`, it needs to run it in a bound Service that will allow it to do so. To see the in-depth of how AbstractThreadedSyncAdapter works, read about AIDL for inter-process communication.

We need to create this bound Service so we can let Android use our `SyncAdapter` class.
```java
/**
 * This is used only by Android to run our {@link SyncAdapter}.
 */
public class SyncService extends Service {
    /**
     * Lock use to synchronize instantiation of SyncAdapter.
     */
    private static final Object LOCK = new Object();
    private static SyncAdapter syncAdapter;


    @Override
    public void onCreate() {
        // SyncAdapter is not Thread-safe
        synchronized (LOCK) {
            // Instantiate our SyncAdapter
            syncAdapter = new SyncAdapter(this, false);
        }
    }

    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        // Return our SyncAdapter's IBinder
        return syncAdapter.getSyncAdapterBinder();
    }
}
```

### Declare SyncService
We need to declare our `SyncService` in the `manifest.xml` file.
```
<service
    android:name=".example.SyncService"
    android:exported="true">
    <intent-filter>
        <action android:name="android.content.SyncAdapter"/>
    </intent-filter>
    <meta-data
        android:name="android.content.SyncAdapter"
        android:resource="@xml/syncadapter"/>
</service>
```

## How to Use Your New SyncAdapter
You now have created the wonderful ability of server synchronization! Here's an example of how to use `SynAdapter`.
```java
/**
 * Your SyncAdapter is good to go!
 *
 * Your SyncAdapter will run all on its own by Android if you specified it to sync
 * automatically and periodically. If not, you can force a sync using our performSync()
 * method we made.
 *
 * Use {@link android.database.ContentObserver} to get callbacks for data changes when
 * Android runs your SyncAdapter or when you manually run it.
 */
public class MainActivity extends AppCompatActivity {
    /**
     * This is our example content observer.
     */
    private ArticleObserver articleObserver;


    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        // Create your sync account
        AccountGeneral.createSyncAccount(this);

        // Perform a manual sync by calling this:
        SyncAdapter.performSync();


        // Setup example content observer
        articleObserver = new ArticleObserver();
    }

    @Override
    protected void onStart() {
        super.onStart();

        // Register the observer at the start of our activity
        getContentResolver().registerContentObserver(
                ArticleContract.Articles.CONTENT_URI, // Uri to observe (our articles)
                true, // Observe its descendants
                articleObserver); // The observer
    }

    @Override
    protected void onStop() {
        super.onStop();

        if (articleObserver != null) {
            // Unregister the observer at the stop of our activity
            getContentResolver().unregisterContentObserver(articleObserver);
        }
    }

    private void refreshArticles() {
        Log.i(getClass().getName(), "Articles data has changed!");
    }

    
    /**
     * Example content observer for observing article data changes.
     */
    private final class ArticleObserver extends ContentObserver {
        private ArticleObserver() {
            // Ensure callbacks happen on the UI thread
            super(new Handler(Looper.getMainLooper()));
        }

        @Override
        public void onChange(boolean selfChange, Uri uri) {
            // Handle your data changes here!!!
            refreshArticles();
        }
    }
}
```

## References
* <https://developer.android.com/training/sync-adapters/creating-sync-adapter.html>
* <http://www.c99.org/2010/01/23/writing-an-android-sync-provider-part-1>
* <https://habrahabr.ru/company/e-Legion/blog/216857>
* <https://github.com/Udinic/AccountAuthenticator>
* <http://guides.codepath.com/android/Creating-Content-Providers>