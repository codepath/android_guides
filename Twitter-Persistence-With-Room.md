## Overview

**Note**: this guide provides supporting material and code snippets for Twitter Persistence [video walkthrough](https://www.youtube.com/watch?v=qzg9tT03ukE&list=PLrT2tZ9JRrf6keV69n7hsy2VCaqQ4LWsf&index=5&t=0s)

In this guide we will look at using [Room]() for persisting Tweets and Users retrieved from Twitter API. Room provides nice abstractions on top of SQLite database. Those abstractions might look complicated for an app like Twitter. However, in production applications this approach shines and helps developers save lots of time.

When user opens the Twitter app it makes a network request, receives response and parses Tweet objects from JSON. Those objects are then stored in the application memory. This data will be lost whenever a user re-requests list of tweets or closes the application. Instead, we want to do the following:

0. Retrieve list of Tweets from database and display them to the user
1. Make a request for latest tweets from API
2. Parse response into objects and save objects into database
3. Update view with fresh data if needed

When using infinite scrolling it might come especially handy, since using database allows to free memory and save user's cell data by avoiding repeating network requests.

## Room Components 

There are 3 main components in Room database:

1. Database class. You will usually have only one instance of this class in your app and you'll use it to retrieve Data Access Object.
2. Entities. Those are your data classes (Tweet, User). Each Entity is usually matched by corresponding database table, where fields of the class become columns of the table.
3. Data Access Objects (DAOs). This is probably the most important abstraction layer where interaction with the db happens. Instead of directly executing SQL queries, DAOs provide an interface for inserting, updating and retrieving data from the database. 

![Room architecture from Google Developers](https://developer.android.com/images/training/data-storage/room_architecture.png)

## Setup

Within your `app/build.gradle`, add Room to your dependency list.  We create a separate variable to store the version number to make it easier to change later:

```gradle
ext {
   roomVersion = "2.1.0"
}

dependencies {
  // Room for simple persistence with an ORM
  implementation "androidx.room:room-runtime:$roomVersion"
  annotationProcessor "androidx.room:room-compiler:$roomVersion"  
}
```

**Note:** you only need to set path to schema location if you planning on using [migrations](https://developer.android.com/training/data-storage/room/migrating-db-versions#export-schema). In this guide you can skip this step.


## Entities - defining tables for User and Tweets

### Adding User entity

As we mentioned earlier, Entities are your data classes which Room can store in the form of database tables. Let's annotate User class as a Room `@Entity`:

```java
@Entity
public class User {
    public String name;
    public String screenName;
    public String profileImageUrl;

    // constructors and json parsing omitted 
}
```


The fields of a class will become columns of the database table. In order to do that we should add `@ColumnInfo` annotation:

```java
@Entity
public class User {
    
    @ColumnInfo
    public String name;
    
    @ColumnInfo
    public String screenName;
    
    @ColumnInfo(name = "profileImageUrl")
    public String profileImageUrl;

    // constructors and json parsing omitted
}
```

**Note:** `@ColumnInfo` annotation has [a few attributes](https://developer.android.com/reference/android/arch/persistence/room/ColumnInfo#public-methods), most notably `name`. By default column name is derived from the field name. However, you can add prefix or change column name using this attribute when needed.


Last, but not least we need to declare a [Primary Key](https://developer.android.com/reference/android/arch/persistence/room/PrimaryKey). It's a requirement for every Entity to have a Primary Key unless it's declared by a super class. In our case, User doesn't have a super class, but we can use Twitter user id as a Primary Key:

```java
@Parcel
@Entity
public class User {
    @ColumnInfo
    @PrimaryKey
    public long id;

    @ColumnInfo
    public String name;

    @ColumnInfo
    public String screenName;

    @ColumnInfo
    public String profileImageUrl;

    public static User fromJson(JSONObject jsonObject) throws JSONException {
        User user = new User();
        user.name = jsonObject.getString("name");
        
        // we should read the value of id from JSON now
        user.id = jsonObject.getLong("id");
        user.screenName = jsonObject.getString("screen_name");
        user.profileImageUrl = jsonObject.getString("profile_image_url_https");
        return user;
    }
}
```

**Note:** `@PrimaryKey` has `autoGenerate` attribute that allows to automatically create a unique identifier. It can be handy when you're creating and inserting new rows into your db table. However, in the case with Twitter User the server has already assigned the unique identifier for every `User` and we will use it.

### Defining Tweet table

First, let's add `@Entity` and `@ColumnInfo` annotations to `Tweet` class, just like we did with `User`:

```java
@Parcel
@Entity()
public class Tweet {

    @ColumnInfo
    @PrimaryKey
    public long id;

    @ColumnInfo
    public String body;

    @ColumnInfo
    public String createdAt;

    // how do we save a complex data type like User?!?
    public User user;
}
```

**Note:** Room does not support nested objects out of the box. There are a couple ways to go about it: [define relationships between objects](https://developer.android.com/training/data-storage/room/relationships) or [use TypeConverter for complex data](https://developer.android.com/training/data-storage/room/referencing-data). In this guide we will describe how to create one-to-many relationship between `User` and `Tweet`.

### Using Foreign key for User

First, we going to add a new field that will point to a user id in the Users table. Since we are going to use `userId` provided by Twitter API we need to add this code to JSON parsing as well.

```java
@Parcel
@Entity
public class Tweet {

    @ColumnInfo
    @PrimaryKey
    public long id;

    @ColumnInfo
    public String body;

    @ColumnInfo
    public String createdAt;

    @ColumnInfo
    public long userId;

    // this field will be ignored by Room, but still can be used in other places in the Twitter app
    @Ignore 
    public User user;

    public static Tweet fromJson(JSONObject jsonObject) throws JSONException {
        Tweet tweet = new Tweet();
        tweet.body = jsonObject.getString("text");
        tweet.id = jsonObject.getLong("id");
        tweet.createdAt = jsonObject.getString("created_at");
        User user = User.fromJson(jsonObject.getJSONObject("user"));
        tweet.user = user;

        // Capture user id assigned by the server
        tweet.userId = user.id;
        return tweet;
    }
}
```

Now it's time to connect `userId` to the entity. For this we use [ForeignKey annotation](https://developer.android.com/reference/android/arch/persistence/room/ForeignKey). `@ForeignKey` allows to specify the entity it points to (`User.class`), the column in the parent table (this is `id` from Users table) and the column in the child or current table (`userId`). 

```java
@Parcel
@Entity(foreignKeys = @ForeignKey(entity = User.class, parentColumns = "id", childColumns = "userId"))
public class Tweet {
	// see columns above
}
```

**Note:** Defining foreign key will help ensure compile time checks. However, it won't automatically populate `User` objects when requesting `Tweet`s from Tweet table. We will learn how to do it later in this guide.

## Creating Data Access Objects aka DAOs

DAOs are central part for implementing persistence with Room. Unlike other ORMs, in Room you don't use query builders or explicit queries to fetch data from the database. Instead, you define DAO interfaces and specify methods for data manipulation (Create, Retrieve, Update and Delete).

This approach has a couple benefits:

- Encapsulation and Separation of concern. In Room you can create different DAOs to access different parts of data.
- Testability. Room provides methods and mocks that allow you to test your DAO interfaces and ensure correctness of your db code.
- Compile-time safety. Room generates Java code and will fail if you make error in your SQL query (as opposed to crashing app at runtime).

A sample DAO could look like this:

```java
@Dao
public interface TweetDao {
	@Querry("SELECT * FROM Tweet ORDER BY createdAt DESC")
	List<Tweet> getTweets();

	@Insert
	void insertModel(Tweet... tweet);
}
```

**Note:** you do have to write some SQL code when using DAOs. However, Room makes it more safe and concise.

See [Accessing data using Room DAOs](https://developer.android.com/training/data-storage/room/accessing-data) for more details and examples.

### Retrieving Tweets with Users

When we defined `Tweet` entity we created a Foreign Key to User table. Now we need to use this key to populate data about Tweets and Users. One approach is to define a new class that will have both `Tweet` and `User` as follows:

```java
public class TweetWithUser {

    // @Embedded notation flattens the properties of the User object into the object, preserving encapsulation.
    @Embedded
    User user;

    // Prefix is needed to resolve ambiguity between fields: user.id and tweet.id, user.createdAt and tweet.createdAt
    @Embedded(prefix = "tweet_")
    Tweet tweet;
}
```

Let's add this class to our DAO class:

```java
@Dao
public interface TweetDao {

    @Query("SELECT * FROM Tweet ORDER BY createdAt DESC")
    List<TweetWithUser> recentItems();
}
```

Now we will need to use some SQL knowledge and define a query that will contain all the fields from both Tweet and User tables. For this we use `JOIN` keyword:

```java
@Dao
public interface TweetDao {

    @Query("SELECT Tweet.body AS tweet_body, Tweet.createdAt as tweet_createdAt, " +
            "User.* FROM Tweet INNER JOIN User ON Tweet.userId = User.id " +
            "ORDER BY Tweet.id DESC LIMIT 5")
    List<TweetWithUser> recentItems();
}
```

Notice how we used `tweet_body` and `tweet_createdAt` prefixes to resolve ambiguity between `user.createdAt` and `tweet.createdAt` fields. 

Check out [Querying multiple tables](https://developer.android.com/training/data-storage/room/accessing-data#query-multiple-tables) for more examples and info. 

### Inserting data

Inserting data is rather easy and doesn't require writing any SQL code. Just create a method in your DAO class that accepts entity object and add `@Insert` annotation:

```java
@Dao
public interface TweetDao {

 	// retrieving tweets is omitted

 	@Insert(onConflict = OnConflictStrategy.REPLACE)
 	void insertModel(Tweet... tweet);

 	@Insert(onConflict = OnConflictStrategy.REPLACE)
 	void insertModel(User... user);
}
```

Conflict resolution strategy allows you to decide how to handle cases when you're inserting an object that already exists (based on primary key). See more options in the [reference documentation](https://developer.android.com/reference/android/arch/persistence/room/OnConflictStrategy).

## Accessing Room database

Access to Room database is achieved via a special Database class. It has a couple requirements:

1. It has to be abstract
2. Should be inherited from `RoomDatabase`
3. All methods should be abstract

Here is what our TwitterDatabase might look like:

```java
@Database(entities={Tweet.class, User.class}, version=1)
public abstract class TwitterDatabase extends RoomDatabase {
    public abstract TweetDao tweetDao();

    // Database name to be used
    public static final String NAME = "TwitterDataBase";
}
```

The database class provides an interface to retrieve instances of DAO classes. In this case we only use one - `TweetDao`.

In order to instantiate database and get a DAO object we use special Room builder. Since we are going to use single database everywhere in Twitter app let's add it to `TwitterApp` class: 

```java
public class TwitterApp extends Application {

    TwitterDatabase twitterDatabase;

    @Override
    public void onCreate() {
        super.onCreate();
        // when upgrading versions, kill the original tables by using
		// fallbackToDestructiveMigration()
        twitterDatabase = Room.databaseBuilder(this, TwitterDatabase.class,
                TwitterDatabase.NAME).fallbackToDestructiveMigration().build();
    }

    public TwitterDatabase getTwitterDatabase() {
        return twitterDatabase;
    }
```

Now we can access database from anywhere in the app by calling:

```java
((TwitterApp) getApplicationContext()).getTwitterDatabase()
```

## Saving and retrieving tweets

First, let's get reference to DAO in the `TimelineActivity`:

```java
public class TimelineActivity extends AppCompatActivity {

	TweetDao tweetDao;

	@Override
    protected void onCreate(Bundle savedInstanceState) {
    	super.onCreate(savedInstanceState);
		// ... 

		tweetDao = ((TwitterApp) getApplicationContext()).getTwitterDatabase().tweetDao();
    }

}
```

**Note:** An important limitation of the Room library is that all interactions with the database should be handled off of the main thread. In this guide we will use simple `AsyncTask` wrapper to perform a call on a non-UI thread.

Before we will be able to retrieve any data we should store it in the database. A good rule of thumb is to store data as soon as it was received from the server. Modify `populateHomeTimeline()` method to save received data into the database:

```java
private void populateHomeTimeline() {
    twitterClient.getHomeTimeline(new JsonHttpResponseHandler() {
        @Override
        public void onSuccess(int statusCode, Headers headers, JSON json) {
            Log.d(TAG, "onSuccess");
            try {
                adapter.clear();
                final List<Tweet> freshTweets = Tweet.fromJsonArray(json.jsonArray);
                final List<User> freshUsers = User.fromJsonTweetArray(json.jsonArray);
                adapter.addAll(freshTweets);
                
                // Interaction with Database can't happen on the Main thread
                AsyncTask.execute(new Runnable() {
                    @Override
                    public void run() {
                    	// runInTransaction() allows to perform multiple db actions in a batch thus preserving consistency
                        ((TwitterApp) getApplicationContext()).getTwitterDatabase().runInTransaction(new Runnable() {
                            @Override
                            public void run() {
                            	// Inserting both Tweets and Users to their respective tables
                                tweetDao.insertModel(freshUsers.toArray(new User[0]));
                                tweetDao.insertModel(freshTweets.toArray(new Tweet[0]));
                            }
                        });
                    }
                });

                // Now we call setRefreshing(false) to signal refresh has finished
                swipeContainer.setRefreshing(false);
            } catch (JSONException e) {
                e.printStackTrace();
            }
        }

    });
}
```

Very similar to storing data we will retrieve existing tweets using DAO:

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
	// ...
	tweetDao = ((TwitterApp) getApplicationContext()).getTwitterDatabase().tweetDao();
    
	// Remember to always move DB queries off of the Main thread
    AsyncTask.execute(new Runnable() {
        @Override
        public void run() {
        	// Request list of Tweets with Users using DAO
            List<TweetWithUser> tweetsFromDatabase = tweetDao.recentItems();
            adapter.clear();
            Log.i(TAG, "Showing data from database");

            // TweetWithUser has to be converted Tweet objects with nested User objects (see next snippet) 
            List<Tweet> tweetList = TweetWithUser.getTweetList(tweetsFromDatabase);
            adapter.addAll(tweetList);
        }
    });
}
```

For convenience let's add a method to `TweetWithUser` that will convert pairs of `Tweet` and `User` into `Tweet` objects that have proper `User` object:

```java
public static List<Tweet> getTweetList(List<TweetWithUser> tweetWithUserList) {
    List<Tweet> tweets = new ArrayList<>();
    for (int i = 0; i < tweetWithUserList.size(); i++) {
        TweetWithUser tweetWithUser = tweetWithUserList.get(i);
        Tweet tweet = tweetWithUser.tweet;
        tweet.user = tweetWithUser.user;
        tweets.add(tweet);
    }
    return tweets;
}

```

## Conclusion

In this guide we created a persistence layer that saves Tweets and Users into local database using Room library. We learned about 3 main components:

- Entities
- DAOs
- Room Database

In Twitter Persistence task we encountered a challenge to store nested object and learned about one-to-many relationship to overcome this challenge. In the end we made it possible to show a list of previously loaded tweets even when a user is offline.

## References

* <https://developer.android.com/training/data-storage/room>
* <https://medium.com/@magdamiu/android-room-persistence-library-relations-75bbe02e8522>
* <https://www.youtube.com/watch?v=MfHsPGQ6bgE>
* <https://guides.codepath.com/android/Room-Guide>