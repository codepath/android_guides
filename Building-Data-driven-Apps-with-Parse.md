[Parse](https://parseplatform.org/) is an open-source platform that provides one of the easiest ways to get a database and RESTful API up and running. If you want to build a mobile app and don't want to code the back-end by hand, give Parse a try.

<img src="https://i.imgur.com/Xo47jSc.gif" alt="Parse" width="750" />

## Parse on back4app

We can deploy our own Parse data store and push notifications systems to [back4app](https://www.back4app.com/) leveraging the [server open-sourced by Parse](https://github.com/ParsePlatform/parse-server-example). 

To get started setting up our own Parse backend, check out our [[Configuring a Parse Server]] guide. Once the Parse server is configured, we can [initialize Parse within our Android app](https://guides.codepath.com/android/Configuring-a-Parse-Server#enabling-client-sdk-integration) pointing the client to our self-hosted URL. After that, the functions demonstrated in this guide work the same as they did before.

### Alternatives to Parse

A comprehensive list of alternatives can be [reviewed here](https://github.com/relatedcode/ParseAlternatives). One of the primary alternatives is Google's [Firebase](https://firebase.google.com), which provides a hosted solution for analytics, crash reporting, and a realtime JSON database.  One major difference is that Parse still provides many powerful constructs for querying data, whereas Firebase requires you to perform this querying based on child/parent relations.  See [this guide](https://firebase.google.com/support/guides/parse-android) for more information on porting Parse applications to Firebase.

## What is Parse?

Parse is an open-source Android SDK and back-end solution that enables developers to build mobile apps with shared data quickly and without writing any back-end code or custom APIs.

<img src="https://i.imgur.com/LylIn7w.png" />

Parse is a Node.js application that is deployed onto a host such as Heroku (or AWS) and then creates an automatic API for user authentication and storing data to a MongoDB document store. Parse has the following features included by combining the mobile SDK and back-end service:

 * User registration and authentication
 * Connecting user with Facebook to create a user account.
 * Creating, querying, modifying and deleting arbitrary data models
 * Makes sending push notifications easier
 * Uploading files to a server for access across clients

In short, Parse makes building mobile app ideas much easier!

## Setup

### Installing the Parse SDK

Setting up Parse starts with [[deploying your own Parse instance|Configuring-a-Parse-Server#setting-a-new-parse-server]] to back4app or another app hosting provider.

 Add the JitPack library to your `settings.gradle` file (**not** your module `build.gradle` file):

```gradle
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
        maven { url "https://jitpack.io" }
    }
}
```

Open the `app/build.gradle` in your project and add the following dependencies:

```gradle
dependencies {
    implementation 'com.github.parse-community.Parse-SDK-Android:parse:1.24.1'
    implementation 'com.squareup.okhttp3:logging-interceptor:4.7.2' // for logging API calls to LogCat
}
```

Notice that the version of Parse-SDK-Android was 1.25.0 when this documentation was released. [Check](https://jitpack.io/#parse-community/Parse-SDK-Android) the latest version.

Select `Tools -> Android -> Sync Project with Gradle Files` to load the libraries through Gradle. When you sync, it will import everything automatically. You can see the imported files in the External Libraries folder.

⚠️ **You may see multiple errors regarding "Duplicate class android.support...".**

![](https://i.imgur.com/WvyYnhA.png =200x)

If you see this error, you will need to add `android.enableJetifier=true` to your `gradle.properties` file, then click Sync Now. See sample in the screenshot below...

![](https://i.imgur.com/dCkR7Dl.png =200x)

After clicking Sync Now and re-running the app, this error should be resolved.

### Initializing the SDK

Next, we need to create an `Application` class and initialize Parse. 

The following section walks you through the process. You can also go to the API reference page and Getting Started section to follow the process.

<img src="https://i.imgur.com/HcJBjDH.gif" width="800" />

Be sure to replace the initialization line below with **your correct Parse keys**:

```java
public class ParseApplication extends Application {
    @Override
    public void onCreate() {
        super.onCreate();

        // Use for troubleshooting -- remove this line for production
        Parse.setLogLevel(Parse.LOG_LEVEL_DEBUG);

        // Use for monitoring Parse OkHttp traffic        
        // Can be Level.BASIC, Level.HEADERS, or Level.BODY
        // See https://square.github.io/okhttp/3.x/logging-interceptor/ to see the options.
        OkHttpClient.Builder builder = new OkHttpClient.Builder();
        HttpLoggingInterceptor httpLoggingInterceptor = new HttpLoggingInterceptor();
        httpLoggingInterceptor.setLevel(HttpLoggingInterceptor.Level.BODY);
        builder.networkInterceptors().add(httpLoggingInterceptor);

        // set applicationId, and server server based on the values in the back4app settings.
        // any network interceptors must be added with the Configuration Builder given this syntax
        Parse.initialize(new Parse.Configuration.Builder(this)
                .applicationId("myAppId") // should correspond to Application Id env variable
                .clientKey("yourClientKey")  // should correspond to Client key env variable
                .server("https://parseapi.back4app.com").build());
    }
}
```

```kotlin
class ParseApplication : Application() {
    override fun onCreate() {
        super.onCreate()

        // Use for troubleshooting -- remove this line for production
        Parse.setLogLevel(Parse.LOG_LEVEL_DEBUG)

        // Use for monitoring Parse OkHttp traffic        
        // Can be Level.BASIC, Level.HEADERS, or Level.BODY
        // See https://square.github.io/okhttp/3.x/logging-interceptor/ to see the options.
        val builder = OkHttpClient.Builder()
        val httpLoggingInterceptor = HttpLoggingInterceptor()
        httpLoggingInterceptor.setLevel(HttpLoggingInterceptor.Level.BODY)
        builder.networkInterceptors().add(httpLoggingInterceptor)

        // set applicationId, and server server based on the values in the back4app settings.
        // any network interceptors must be added with the Configuration Builder given this syntax
        Parse.initialize(
            Parse.Configuration.Builder(this)
                .applicationId("myAppId") // should correspond to Application ID env variable
                .clientKey("yourClientKey")  // should correspond to Client key env variable
                .server("https://parseapi.back4app.com").build())
    }
}
```

We also need to make sure to set the application instance above as the `android:name` for the application within the `AndroidManifest.xml`. This change in the manifest determines which application class is instantiated when the app is launched and also adding the application ID metadata tag:

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.parsetododemo"
    android:versionCode="1"
    android:versionName="1.0" >
    <application
        <!-- other attributes here -->
        android:name=".ParseApplication">
    </application>
</manifest>
```

### Setup network permissions

We also need to add a few important network permissions to the `AndroidManifest.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.parsetododemo"
    android:versionCode="1"
    android:versionName="1.0" >

    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />

   ...
</manifest>
```

### Testing Parse Client

Assuming you have access to the Parse instance, you can test the SDK to verify that Parse is working with this application.

Let's add the test code to `ParseApplication` as follows:

```java
public class ParseApplication extends Application {
    @Override
    public void onCreate() {
        super.onCreate();

      	// Your initialization code from above here
      	Parse.initialize(...);

        // New test creation of object below
      	ParseObject testObject = new ParseObject("TestObject");
      	testObject.put("foo", "bar");
      	testObject.saveInBackground();
    }		
}
```
```kotlin
class ParseApplication : Application() {
    override fun onCreate() {
        super.onCreate()

      	// Your initialization code from above here
      	Parse.initialize(...)

        // New test creation of object below
      	val testObject = ParseObject("TestObject")
        testObject.put("foo", "bar")
        testObject.saveInBackground()
    }		
}
```

Run your app and a new object of class `TestObject` will be sent to the Parse Cloud and saved. See [[browsing Parse data|Configuring-a-Parse-Server#browsing-parse-data]] for more information about how to check this data.

If needed in your application, you might also want to [[setup push notifications|Push-Notifications-Setup-for-Parse]] through Parse as well at this time.

You should be setup now!  Follow the remaining documentation guide to understand how to leverage Parse for your entire backend.  

## Working with Users

At the core of many apps, there is a notion of user accounts that lets users access their information in a secure manner. Parse has a specialized `ParseUser` as a part of their SDK which handles this functionality. Be sure to check out the [Users](https://docs.parseplatform.org/android/guide/#users) docs for a complete overview. See the API docs for [ParseUser](https://parseplatform.org/Parse-SDK-Android/api/com/parse/ParseUser.html) for more details.

### User Signup

Creating a new user account is the process of constructing a `ParseUser` object and calling `signUpInBackground`:

```java
// Create the ParseUser
ParseUser user = new ParseUser();
// Set core properties
user.setUsername("joestevens");
user.setPassword("secret123");
user.setEmail("email@example.com");
// Set custom properties
user.put("phone", "650-253-0000");
// Invoke signUpInBackground
user.signUpInBackground(new SignUpCallback() {
  public void done(ParseException e) {
    if (e == null) {
      // Hooray! Let them use the app now.
    } else {
      // Sign up didn't succeed. Look at the ParseException
      // to figure out what went wrong
    }
  }
});
```

```kotlin
// Create the ParseUser
val user = ParseUser()

// Set fields for the user to be created
user.setUsername(username)
user.setPassword(password)

user.signUpInBackground { e ->
    if (e == null) {
        // Hooray! Let them use the app now.
    } else {
        // Sign up didn't succeed. Look at the ParseException
        // to figure out what went wrong
    }
}
```

This call will asynchronously create a new user in your Parse App. Before it does this, it checks to make sure that both the username and email are unique. See the [signup up docs](https://docs.parseplatform.org/android/guide/#signing-up) for more details.

### User Session Login

We can allow a user to signin by calling `logInInBackground` and passing the user details:

```java
ParseUser.logInInBackground("joestevens", "secret123", new LogInCallback() {
  public void done(ParseUser user, ParseException e) {
    if (user != null) {
      // Hooray! The user is logged in.
    } else {
      // Signup failed. Look at the ParseException to see what happened.
    }
  }
});
```

```kotlin
ParseUser.logInInBackground("joestevens", "secret123", ({ user, e ->
  if (user != null) {
    // Hooray!  The user is logged in.
  } else {
    // Signup failed.  Look at the ParseException to see what happened.
  }})
)
```

If the credentials are correct, the `ParseUser` will be passed back accordingly. You can now access the cached current user for your application at any time in order to determine the session status:

```java
ParseUser currentUser = ParseUser.getCurrentUser();
if (currentUser != null) {
  // do stuff with the user
} else {
  // show the signup or login screen
}
```
```kotlin
val currentUser = ParseUser.getCurrentUser()
if (currentUser != null) {
  // do stuff with the user
} else {
  // show the signup or login screen
}
```

### User Logout

A user can be signed back out with:

```java
ParseUser.logOut();
ParseUser currentUser = ParseUser.getCurrentUser(); // this will now be null
```

```kotlin
ParseUser.logOut()
val currentUser = ParseUser.getCurrentUser() // this will now be null
```

That's the basics of what you need to work with users. See more details by checking out the [User](https://docs.parseplatform.org/android/guide/#users) official docs.

You can also have a [Facebook Login](https://docs.parseplatform.org/android/guide/#facebook-users) or [Twitter Login](https://docs.parseplatform.org/android/guide/#twitter-users) for your users easily following the guides linked.

### Querying Users

To query for users, you need to use the special user query:

```java
ParseQuery<ParseUser> query = ParseUser.getQuery();
query.whereGreaterThan("age", 20); // find adults
query.findInBackground(new FindCallback<ParseUser>() {
  public void done(List<ParseUser> objects, ParseException e) {
    if (e == null) {
        // The query was successful.
    } else {
        // Something went wrong.
    }
  }
});
```
```kotlin
val query = ParseUser.getQuery()
query.whereGreaterThan("age", 20) // find adults
query.findInBackground { objects, e ->
    if (e == null) {
        // The query was successful.
    } else {
        // Something went wrong.
    }
}
```

See a list of [query constraints](https://docs.parseplatform.org/android/guide/#query-constraints) here.

## Working With Data Objects

Storing data on Parse is built around the `ParseObject`. Each `ParseObject` contains key-value pairs of JSON-compatible data. This data is schema-less, which means that you don't need to specify ahead of time what keys exist on each `ParseObject`. Each `ParseObject` has a class name that you can use to distinguish different sorts of data. See the API docs for [ParseObject](https://parseplatform.org/Parse-SDK-Android/api/com/parse/ParseObject.html) for more details.

### Creating Parse Models

When using Parse, the best practice is to create models that represent our data and that subclass `ParseObject` to allow for Parse persistence. Suppose we wanted to create a `TodoItem` model:

```java
import com.parse.ParseObject;
import com.parse.ParseClassName;

@ParseClassName("TodoItem")
public class TodoItem extends ParseObject {
   // Ensure that your subclass has a public default constructor

    public static final String KEY_AUTHOR = "author";

    public ParseUser getAuthor() {
        return getParseUser(KEY_AUTHOR);
    }

    public void setAuthor(ParseUser author) {
        put(KEY_AUTHOR, author);
    }

}
```

```kotlin
import com.parse.ParseObject
import com.parse.ParseClassName

@ParseClassName("TodoItem")
class TodoItem : ParseObject() {
var author: ParseUser?
    get() = getParseUser("author")
    // putOrIgnore does a smart cast to Kotlin Any after an if check 
    set(author) = putOrIgnore("author", author)
}
```

We need to make sure to register this class with Parse **before** we call `Parse.initialize`:

```java
public class ParseApplication extends Application {
  @Override
  public void onCreate() {
    super.onCreate();
    // Register your parse models
    ParseObject.registerSubclass(TodoItem.class);
    // Add your initialization code here
    Parse.initialize(this, "7zBztvyG4hYQ9XghgfqYxfRcL3SMBYWAj0GUL", "iZWhgJRu6yKm3iNMbTaguLcNCV3qedijWL");
  }		
}
```
```kotlin
class ParseApplication : Application() {

    override fun onCreate() {
        super.onCreate()
        // Register your parse models
        ParseObject.registerSubclass(TodoItem::class.java)
        // Add your initialization code here
        Parse.initialize(...)
    }
```

Now we can add fields and constructors to our `TodoItem`:

```java
@ParseClassName("TodoItem")
public class TodoItem extends ParseObject {
  // Ensure that your subclass has a public default constructor
  public TodoItem() {
    super();
  }

  // Add a constructor that contains core properties
  public TodoItem(String body) {
    super();
    setBody(body);
  }

  // Use getString and others to access fields
  public String getBody() {
    return getString("body");
  }

  // Use put to modify field values
  public void setBody(String value) {
    put("body", value);
  }

  // Get the user for this item
  public ParseUser getUser()  {
    return getParseUser("owner");
  }

  // Associate each item with a user
  public void setOwner(ParseUser user) {
    put("owner", user);
  }
}
```

Notice that now our model has `getBody`, `setBody` as well as property methods for storing which user created the TodoItem.

**Note:** When creating Parse models, avoid **creating unnecessary member instance variables** and instead rely directly on `getString`-type methods to retrieve the values of database properties.

### Saving or Updating Objects

Let's suppose we wanted to save a `TodoItem` to the Parse database, create a new `TodoItem`, set the data attributes and then trigger a save with `saveInBackground`:

```java
TodoItem todoItem = new TodoItem("Do laundry");
// Set the current user, assuming a user is signed in
todoItem.setOwner(ParseUser.getCurrentUser());
// Immediately save the data asynchronously
todoItem.saveInBackground();
// or for a more robust offline save
// todoItem.saveEventually();
```
```kotlin
val todoItem = TodoItem("Do laundry")
// Set the current user, assuming a user is signed in
todoItem.setOwner(ParseUser.getCurrentUser())
// Immediately save the data asynchronously
todoItem.saveInBackground()
// or for a more robust offline save
// todoItem.saveEventually()
```

Note that there are two ways to save an object: `saveInBackground` which executes immediately and `saveEventually` which will store the update on the device and push to the server once internet access is available.

See the [saving objects](https://docs.parseplatform.org/android/guide/#saving-objects) and [updating docs](https://docs.parseplatform.org/android/guide/#updating-objects) docs for more details. Also, check out the [relational data](https://docs.parseplatform.org/android/guide/#relations) section.

### Querying Objects

#### Objects By Id

If you have the objectId, you can retrieve the whole `ParseObject` using a `ParseQuery`:

```java
// Specify which class to query
ParseQuery<TodoItem> query = ParseQuery.getQuery(TodoItem.class);
// Specify the object id
query.getInBackground("aFuEsvjoHt", new GetCallback<TodoItem>() {
  public void done(TodoItem item, ParseException e) {
    if (e == null) {
      // Access data using the `get` methods for the object
      String body = item.getBody();
      // Access special values that are built-in to each object
      String objectId = item.getObjectId();
      Date updatedAt = item.getUpdatedAt();
      Date createdAt = item.getCreatedAt();
      // Do whatever you want with the data...
      Toast.makeText(TodoItemsActivity.this, body, Toast.LENGTH_SHORT).show();
    } else {
      // something went wrong
    }
  }
});
```
```kotlin
// Specify which class to query
val query: ParseQuery<TodoItem> = ParseQuery.getQuery(TodoItem::class.java)
// Specify the object id
query.getInBackground("aFuEsvjoHt") { item, e ->
    if (e == null) {
        // Access data using the `get` methods for the object
        val body: String = item.getBody()
        // Access special values that are built-in to each object
        val objectId: String = item.getObjectId()
        val updatedAt: Date = item.getUpdatedAt()
        val createdAt: Date = item.getCreatedAt()
        // Do whatever you want with the data...
        Toast.makeText(this@TodoItemsActivity, body, Toast.LENGTH_SHORT).show()
    } else {
        // something went wrong
    }
}
```

See [retrieving objects](https://docs.parseplatform.org/android/guide/#retrieving-objects) official docs for information on refreshing stale objects and more.

#### Objects By Query Conditions

The general pattern is to create a `ParseQuery`, put conditions on it, and then retrieve a `List` of matching ParseObjects using the `findInBackground` method with a `FindCallback`. For example, to find all items created by a particular user:

```java
// Define the class we would like to query
ParseQuery<TodoItem> query = ParseQuery.getQuery(TodoItem.class);
// Define our query conditions
query.whereEqualTo("owner", ParseUser.getCurrentUser());
// Execute the find asynchronously
query.findInBackground(new FindCallback<TodoItem>() {
    public void done(List<TodoItem> itemList, ParseException e) {
        if (e == null) {
            // Access the array of results here
            String firstItemId = itemList.get(0).getObjectId();
            Toast.makeText(TodoItemsActivity.this, firstItemId, Toast.LENGTH_SHORT).show();
        } else {
            Log.d("item", "Error: " + e.getMessage());
        }
    }
});
```

See a list of [query constraints](https://docs.parseplatform.org/android/guide/#query-constraints) here and check the [queries overview](https://docs.parseplatform.org/android/guide/#queries) for explanation of compound queries and relational queries.

#### Objects by Querying GeoLocation

Often we might want to query objects within a certain radius of a coordinate (for example to display them on a map). With Parse, querying by `GeoPoint` to retrieve objects within a certain distance of a location is built in. Check the [AnyWall Tutorial](https://github.com/rosstapson/AnyWall) and the [whereWithinMiles](https://parseplatform.org/Parse-SDK-Android/api/com/parse/ParseQuery.html#whereWithinMiles-java.lang.String-com.parse.ParseGeoPoint-double-) and related `where` conditions for more details.

If you want to query this based on a map, first you can [add a listener for the map camera](https://developers.google.com/android/reference/com/google/android/gms/maps/GoogleMap#setOnCameraChangeListener\(com.google.android.gms.maps.GoogleMap.OnCameraChangeListener\)). Next, you can determine the [visible bounds of the map](https://stackoverflow.com/q/16056366) as shown there. Check the [[Maps Usage Guide|Google-Maps-API-v2-Usage]] for additional information on using the map.

#### Live Queries

One of the newer features of Parse is that you can monitor for live changes made to objects in your database  To get started, make sure you have defined the ParseObjects that you want in your NodeJS server.  Make sure to define a list of all the objects by declaring it in the `liveQuery` and `classNames listing`:

```javascript
let api = new ParseServer({
  ...,
  // Make sure to define liveQuery AND classNames
  liveQuery: {
    // define your ParseObject names here
    classNames: ['Post', 'Comment']
  }
});
```

See [this guide](https://docs.parseplatform.org/parse-server/guide/#live-queries) for more details.  Parse Live Queries rely on the websocket protocol, which creates a bidirectional channel between the client and server and periodically exchange ping/pong frames to validate the connection is still alive.  Websocket URLs are usually prefixed with ws:// or wss:// (secure) URLs.  If you are deploying to a different server (Amazon), you may need to make sure that TCP port 80 or TCP port 443 are available.

You will need to also setup the client SDK by adding this dependency to your `app/build.gradle` config:

```gradle
dependencies {
  // add Parse dependencies too
  implementation 'com.github.parse-community:ParseLiveQuery-Android:1.1.0'}
```

where `X.X.X` is the latest version <img src="https://jitpack.io/v/parse-community/ParseLiveQuery-Android.svg"/>

Next, instantiate the client:

```java
ParseLiveQueryClient parseLiveQueryClient = ParseLiveQueryClient.Factory.getClient();
```

Define the query pattern you wish to listen for events:
```java
ParseQuery<Comment> query = ParseQuery.getQuery(Comment.class);
```

Create a subscription to this `ParseQuery` instance:

```java
SubscriptionHandling<ParseObject> subscriptionHandling = parseLiveQueryClient.subscribe(parseQuery)
```

Finally, listen to the events. You can listen for `Event.UPDATE`, `Event.DELETE`, `Event.ENTER`, and `Event.LEAVE`.  An enter and leave event reflects changes to an existing ParseObject that either now fulfill the criteria or no longer do so.  See [this guide](https://github.com/ParsePlatform/parse-server/wiki/Parse-LiveQuery-Protocol-Specification) for more information about the live queries protocol specification.

```java
subscriptionHandling.handleEvent(SubscriptionHandling.Event.CREATE, new SubscriptionHandling.HandleEventCallback<Comment>() {
    @Override
    public void onEvent(ParseQuery<Comment> query, Comment object) {
        // HANDLING create event
    }
})
```

### Passing Objects Between Activities

Often with Android development, you need to pass an object from one Activity to another. This is done using the `Intent` system and passing objects as extras within a bundle. Fortunately, your subclasses of ParseObject are already Parcelable by default, since the `ParseObject library` implements Parcelable!

So, the simplest way to pass data between activities is simply to pass the entire Parse object through an Intent:

```java
Intent i = new Intent(this, SomeNewActivity.class);
i.putExtra("todo_object", myTodoItem);
startActivity(i);
```

you can then use your Parse object within the child Activity:

```java
TodoItem todoItem = getIntent().getParcelableExtra("todo_object");

Log.i("test", todoItem.getAuthor());
```

In your Java file for your model, be sure to add a `public static final String` to define the key for each property in your model. This helps the Parse library understand which column in your Parse Dashboard/database corresponds to each property in your model. An example would be the `KEY_AUTHOR` mentioned above in this article.

### Associations

Objects can have relationships with other objects. To model this behavior, any `ParseObject` can be used as a value in other `ParseObject`s. Internally, the Parse framework will store the referred-to object in just one place, to maintain consistency.

#### One-to-One Relationships

For example, each Comment in a blogging app might correspond to one Post. To create a new Post with a single Comment, you could write:

```java
@ParseClassName("Post")
public class Post extends ParseObject {
   // ...
}

@ParseClassName("Comment")
public class Comment extends ParseObject {
        // ...

	// Associate each comment with a user
	public void setOwner(ParseUser user) {
		put("owner", user);
	}

	// Get the user for this comment
	public ParseUser getOwner()  {
           return getParseUser("owner");
	}

	// Associate each comment with a post
	public void setPost(Post post) {
		put("post", post);
	}

	// Get the post for this item
	public Post getPost()  {
           return (Post) getParseObject("post");
	}
}

// Create the post
Post post = new Post("Welcome Spring!");
// Get the user
ParseUser currentUser = ParseUser.getCurrentUser();
// Create the comment
Comment comment = new Comment("Get milk and eggs");
comment.setPost(post);
comment.setOwner(currentUser);
comment.saveInBackground();
```

By default, when fetching an object, related `ParseObject`s are not fetched. We can preload (eagerly fetch these) by using the `include` method on any `ParseQuery`:

```java
ParseQuery<ParseObject> query = ParseQuery.getQuery(Comment.class);
// Include the post data with each comment
query.include("owner"); // the key which the associated object was stored
// Execute query with eager-loaded owner
query.findInBackground(new FindCallback<ParseObject>() {
 ....
}
```

We can eagerly load nested associations as well. If you have an objectA which has a column referencing objectB and then objectB has a column referencing objectC, you can get objectB and objectC by doing:

```java
ParseQuery<ParseObject> query = ParseQuery.getQuery(ObjectA.class);
query.include("ObjectB.ObjectC");
// ...execute query...
```

Otherwise, these associated objects can only be retrieved once they have been fetched separately:

```java
fetchedTodoItem.getCategory()
    .fetchIfNeededInBackground(new GetCallback<Category>() {
        public void done(Category object, ParseException e) {
          String title = category.getTitle();
        }
    });
```

#### Many-to-Many Relationships

You can also model a many-to-many relation using the `ParseRelation` object. This works similar to `ArrayList`, except that you don't need to download all the `ParseObject`s in a relation at once.

```java
@ParseClassName("Tag")
public class Tag extends ParseObject {
   // ...this is a tag to describe an item
}

@ParseClassName("TodoItem")
public class TodoItem extends ParseObject {
   public ParseRelation<Tag> getTagsRelation() {
      return getRelation("tags");
  }

  public void addTag(Tag tag) {
    getTagsRelation().add(tag);
    saveInBackground();
  }

  public void removeTag(Tag tag) {
     getTagsRelation().remove(tag);
     saveInBackground();
  }
}
```

By default, the list of objects in this relation are not downloaded. You can get the list of Posts by calling findInBackground on the ParseQuery returned by getQuery. The code would look like:

```java
fetchedTodoItem.getTagsRelation().getQuery().findInBackground(new FindCallback<Tag>() {
    void done(List<Tag> results, ParseException e) {
      if (e == null) {
        // results have all the Posts the current user liked.
      } else {
        // There was an error
      }
    }
});
```

For more details, check out the official [Relational Data](https://docs.parseplatform.org/android/guide/#using-pointers) guide. For more complex many-to-many relationships, check out this official [join tables](https://docs.parseplatform.org/android/guide/#using-join-tables) guide when the many-to-many requires additional metadata.

### Deleting Objects

To delete an object from the Parse Cloud:

```java
todoItem.deleteInBackground();
```

Naturally we can also delete in an offline manner with:

```java
todoItem.deleteEventually();
```

## Local Storage Mode

Parse now supports a more powerful form of [local data storage](https://docs.parseplatform.org/android/guide/#local-datastore) out of the box which can be used to store and retrieve ParseObjects, even when the network is unavailable. To enable this functionality, simply call Parse.enableLocalDatastore() before your call to initialize:

```java
// Within the Android Application where Parse is initialized
Parse.enableLocalDatastore(this);
Parse.initialize(this, PARSE_APPLICATION_ID, PARSE_CLIENT_KEY);
```

### Saving To Local Store

You can store a ParseObject in the local datastore by pinning it. Pinning a ParseObject is recursive, just like saving, so any objects that are pointed to by the one you are pinning will also be pinned:

```java
TodoItem todoItem = new TodoItem("Do laundry");
// Set the current user, assuming a user is signed in
todoItem.setOwner(ParseUser.getCurrentUser());
// Store object offline
todoItem.pinInBackground();
```

### Querying from Local Store

We can query from the local offline store with the `fromLocalDatastore` flag during any query operation:

```java
// Specify which class to query
ParseQuery<TodoItem> query = ParseQuery.getQuery(TodoItem.class);
// Flag indicates we will use offline store
query.fromLocalDatastore();
// Specify the object id
query.getInBackground("aFuEsvjoHt", new GetCallback<TodoItem>() {
  public void done(TodoItem item, ParseException e) {
     // ...
  }
});
```

### Further Details

For the full summary of how to utilize the offline mode for Parse, be sure to review the [official local store guide](https://docs.parseplatform.org/android/guide/#local-datastore) in the Parse docs.

## Using the Data Browser

Suppose we had a simple todo application with user accounts and items persisted to Parse. The next step is to setup and create our models using the [Parse dashboard](https://guides.codepath.com/android/Configuring-a-Parse-Server#parse-dashboard) to manage your new app. Visit the "Data Browser" for the correct application and let's create our `User` and `TodoItem` objects for our app.

First, **remove the test code that we added previously** and drop the "TestObject" listed in the browser to clear testing data.

<img src="https://i.imgur.com/ZFds0Fn.png" alt="screen_5" width="400" />

Next, select "New Class" and pick "User" to create the user object used to manage session authetication:

<img src="https://i.imgur.com/7hQZz3y.png" alt="screen_6" width="300" />

Let's also add our "Custom" class which can represent any custom data. In this case, we will create a `TodoItem` class:

<img src="https://i.imgur.com/MIZkV8L.png" alt="screen_7" width="300" />

Now, we need to add our custom columns to our class. In this case, let's add a "body" field to our `TodoItem` by selecting "+Col" and then selecting the type as a String and column name as "body":

<img src="https://i.imgur.com/qYel2NP.png" alt="screen_8" width="300" />

Once you've finished adding your columns to the class, you can create as many additional classes as necessary and configure their respective columns. Let's add a row of data to the class directly through the data browser:

<img src="https://i.imgur.com/QFMEnb7.png" alt="screen_9" width="600" />

We are now ready to access these classes within our application.

## Troubleshooting

Check out our [[Troubleshooting Common Issues with Parse]] guide for a detailed look at common issues encountered and related solutions.

## Additional Features

Parse has many powerful features in addition to the core functionality listed above.

### Uploading Photos

Parse has full support for storing images and files uploaded by an application. Photos are stored using the `ParseFile` construct [described in more detail here](https://docs.parseplatform.org/android/guide/#files).

First, make sure to create a [Parse model](https://guides.codepath.com/android/Building-Data-driven-Apps-with-Parse#creating-parse-models).  Next, you will need to add a `ParseFile` object:

```java
@ParseClassName("Post")
class Post extends ParseObject {

    public ParseFile getMedia() {
        return getParseFile("media");
    }

    public void setMedia(ParseFile parseFile) {
        put("media", parseFile);
    }
}
```

```kotlin
@ParseClassName("Post")
class Post : ParseObject() {

    var media: ParseFile?
        get() = getParseFile("media")
        set(file) {
            put("media", file)
        }
```

A `ParseFile` instance can be initialized with a `File` object or a byte array.  Follow the [[CodePath Camera and Gallery Guide|Accessing-the-Camera-and-Stored-Media]] guide for capturing photos with a camera.  

```java
// getPhotoFileUri() is defined in
// https://guides.codepath.com/android/Accessing-the-Camera-and-Stored-Media#using-capture-intent
File photoFile = getPhotoFileUri(photoFileName);

ParseFile parseFile = new ParseFile(photoFile);
```

```kotlin
// getPhotoFileUri() is defined in
// https://guides.codepath.com/android/Accessing-the-Camera-and-Stored-Media#using-capture-intent
val photoFile = getPhotoFileUri(photoFileName)

val parseFile = ParseFile(photoFile)
```

#### Rendering ParseFile objects

To render a ParseFile in an ImageView, you can use [Parse's UI Widget library](https://github.com/parse-community/ParseUI-Android) and use the `ParseImageView`, which provides helper methods to load the image asynchronously.  First, you need to add the Parse UI Widget to your `app/build.gradle` declaration:

```gradle
dependencies {
  implementation 'com.github.parse-community.ParseUI-Android:widget:0.0.6'
}
```

Instead of an `ImageView`, use the `com.parse.ParseImageView` class:

```xml
<com.parse.ParseImageView
        android:layout_width="match_parent"
        android:layout_height="300dp"/>
```

To load the `ParseFile`, use the `setParseFile()` method and call `loadInBackground()`:

```java
imageView.setParseFile(post.getMedia());
imageView.loadInBackground()
```

```kotlin
imageView.setParseFile(post.media)
imageView.loadInBackground()
```

Alternatively, if you have a third-party library such as [Glide](https://guides.codepath.com/android/Displaying-Images-with-the-Glide-Library) or Picasso, you can also reference the URL directly and don't need to use this special `ParseImageView` widget.

```java
Glide.with(view.context).load(post.getMedia().getUrl()).into(imageView);
```

```kotlin
Glide.with(view.context).load(post.media?.url).into(imageView)
```
### Geo Location

Parse has support for geolocation services and querying:

* [AnyWall Sample App](https://github.com/rosstapson/AnyWall) - Sample app including how to read geolocation and use this data within your app.

### Push Notifications

Parse supports push notifications made easy:

* [[Parse Push Messaging|Push-Messaging]] - CodePath Guide
* [[Parse Push|Configuring-a-Parse-Server#enabling-push-notifications]] - Super easy push notifications

### Facebook SDK

For a quick way of incorporating Facebook login, check out Parse's [UI Library](https://github.com/ParsePlatform/ParseUI-Android/wiki/Login-Library).  It leverages Parse's [FacebookUtils](https://github.com/ParsePlatform/ParseFacebookUtils-Android/) library, which acts as a wrapper for associating `ParseUser` objects with Facebook users.  

The official step by step instructions for integrating Parse with Facebook SDK is located [here](https://docs.parseplatform.org/android/guide/#facebook-users).  The manual process of integrating with Facebook's SDK is discussed below.

You will first need to create a [Facebook app](https://developers.facebook.com/apps) and get an Application ID.
If you are using open source Parse, make sure to set the `FACEBOOK_APP_ID` environment variable too.

You will also need to get access to your keystore hash and make sure to include it:

<img src="https://scontent.fsnc1-1.fna.fbcdn.net/hphotos-xaf1/t39.2178-6/851568_627654437290708_1803108402_n.png"/>

OS X:

```bash
keytool -exportcert -alias androiddebugkey -keystore ~/.android/debug.keystore | openssl sha1 -binary | openssl base64
```

Windows:
```bash
keytool -exportcert -alias androiddebugkey -keystore %HOMEPATH%\.android\debug.keystore | openssl sha1 -binary | openssl
base64
```

Next, you will need to include Parse's [FacebookUtils](https://github.com/ParsePlatform/ParseFacebookUtils-Android/) library, which provides an easy-to-use wrapper to work with the Facebook SDK, as well as Facebook's SDK:

```gradle
dependencies {
  // add other parse dependencies
  implementation 'com.facebook.android:facebook-android-sdk:4.33.0'
  implementation 'com.parse:parsefacebookutils-v4-android:X.X.X'
}
```

where `X.X.X` is the latest version: <img src="https://api.bintray.com/packages/parse/maven/ParseFacebookUtils-Android/images/download.svg">

You will then need to make sure to initialize these libraries by doing so in your `MainApplication.java` file:

```java
public class MainApplication extends Application {

  @Override
  public void onCreate() {
     super.onCreate();

     Parse.initialize(new Parse.Configuration.Builder(this)
           .applicationId("YOUR_APPLICATION_ID_HERE")
           .clientKey(null)
           .server("YOUR_HOSTED_PARSE_SERVER_URL/parse").build());

    // ParseFacebookUtils should initialize the Facebook SDK for you
    ParseFacebookUtils.initialize(this);
  }
```

**Note that you need to add /parse in the url in order for Parse to work, not just the hostname.**

Make sure to reference this `MainApplication` in your `AndroidManifest.xml` file:

```xml
<application
android:name=".MainApplication"
```

Make sure to add the `FacebookActivity` to your manifest file, as well as the application ID and permissions you wish to request.  

```xml
<activity android:name="com.facebook.FacebookActivity"
            android:configChanges=
                "keyboard|keyboardHidden|screenLayout|screenSize|orientation"
            android:theme="@android:style/Theme.Translucent.NoTitleBar"
            android:label="@string/app_name" />

        <meta-data
            android:name="com.facebook.sdk.ApplicationId"
            android:value="@string/facebook_app_id" />

        <meta-data
            android:name="com.facebook.sdk.PERMISSIONS"
            android:value="email" />
```

There are two ways to support Facebook login: one is to use your own custom button/icon, or to use Facebook's `LoginButton` class as described in the [walkthrough](https://developers.facebook.com/docs/facebook-login/android).  If you choose to use the `LoginButton` class, you should not use the `FacebookUtils` library to trigger a login.  The reason is that Facebook's `LoginButton` already has click handlers that will launch a login screen, so using Parse's code triggers two of these screens to appear.  In addition, you have to do more work to associate a `ParseUser` object with a Facebook user.  For this reason, it is simpler to use the custom button approach when integrating with Parse.

#### Custom button

For your `LoginActivity.java`, add this code to trigger a login.  If you wish to trigger this login, you may add your own custom button:

```xml
    <Button
        android:background="@color/com_facebook_blue"
        android:text="Log in with Facebook"
        android:id="@+id/login"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"/>
```

You can then attach a button click handler.

```java
button.setOnClickListener(new View.OnClickListener() {
                                      @Override
                                      public void onClick(View v) {

            ArrayList<String> permissions = new ArrayList();
            permissions.add("email");
            ParseFacebookUtils.logInWithReadPermissionsInBackground(MainActivity.this, permissions,
                    new LogInCallback() {
                        @Override
                        public void done(ParseUser user, ParseException err) {
                            if (err != null) {
                                Log.d("MyApp", "Uh oh. Error occurred" + err.toString());
                            } else if (user == null) {
                                Log.d("MyApp", "Uh oh. The user cancelled the Facebook login.");
                            } else if (user.isNew()) {
                                Log.d("MyApp", "User signed up and logged in through Facebook!");
                            } else {
                                Toast.makeText(MainActivity.this, "Logged in", Toast.LENGTH_SHORT)
                                        .show();
                                Log.d("MyApp", "User logged in through Facebook!");
                            }
                        }
                    });
        }
});
```

You must also override the `onActivityResult()` to capture the result after a user signs into Facebook:

```java
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    super.onActivityResult(requestCode, resultCode, data);
    ParseFacebookUtils.onActivityResult(requestCode, resultCode, data);
}
```

### Using with Android Paging Library

This is a new way of handling pagination and endless scrolling. For a simpler and more straightforward solution check out [[Endless Scrolling Guide|Endless-Scrolling-with-AdapterViews-and-RecyclerView]].

#### Prerequisites

First, follow the [[setup instructions|Paging-Library-Guide#setup]] for adding the Paging Library to your Gradle configuration.   In addition, make sure also to change your RecyclerView to use PagedListAdapter as described in this [[section|Paging-Library-Guide#modifying-your-recyclerview]].  You must take care of these first steps before proceeding to the section below.

#### Create Data Source

With the Parse library, we will define our data source using the `PositionalDataSource` class.  We can use this data source because the Parse SDK allows us to count the size for a query while also requesting a size from an arbitrary offset.  We need to provide this information in the `loadInitial()` method:

```java
public class ParsePositionalDataSource extends PositionalDataSource<Post> {

    // define basic query here
    public ParseQuery<Post> getQuery() {
        return ParseQuery.getQuery(Post.class).orderByDescending("createdAt");
    }

    @Override
    public void loadInitial(@NonNull LoadInitialParams params, @NonNull LoadInitialCallback<Post> callback) {
        // get basic query
        ParseQuery<Post> query = getQuery();

        // Use values passed when PagedList was created.
        query.setLimit(params.requestedLoadSize);
        query.setSkip(params.requestedStartPosition);

        try {
            // loadInitial() should run queries synchronously so the initial list will not be empty         
            // subsequent fetches can be async   
            int count = query.count();
            List<Post> posts = query.find();

            // return info back to PagedList
            callback.onResult(posts, params.requestedStartPosition, count);
        } catch (ParseException e) {
            // retry logic here
        }
    }
}
```

```kotlin
class ParsePositionalDataSource : PositionalDataSource<Post>() {

    fun getQuery() : ParseQuery<Post> {
        return ParseQuery.getQuery(Post::class.java).orderByDescending("createdAt")
    }

    override fun loadInitial(params: LoadInitialParams, callback: LoadInitialCallback<Post>) {
        // get basic query
        val query = getQuery()

        // Use values passed when PagedList was created.
        query.limit = params.requestedLoadSize
        query.skip = params.requestedStartPosition

        // run queries synchronously since function is called on a background thread
        val count = query.count()
        val posts = query.find()

        // return info back to PagedList
        callback.onResult(posts, params.requestedStartPosition, count)
    }
}
```

We also need to implement a `loadRange()` method, which looks similar to the `loadInitial()` method:

```java
    @Override
    public void loadRange(@NonNull LoadRangeParams params, @NonNull LoadRangeCallback<Post> callback) {
        // get basic query
        ParseQuery<Post> query = getQuery();

        query.setLimit(params.loadSize);

        // fetch the next set from a different offset
        query.setSkip(params.startPosition);

        try {
            // run queries synchronously since function is called on a background thread
            List<Post> posts = query.find();

            // return info back to PagedList
            callback.onResult(posts);
        } catch (ParseException e) {
            // retry logic here
        }
    }
```

```kotlin
    override fun loadRange(params: LoadRangeParams, callback: LoadRangeCallback<Post>) {
        val query = getQuery()

        query.limit = params.loadSize
        // fetch the next set from a different offset
        query.skip = params.startPosition

        // synchronous call
        val posts = query.find()

        // return info back to PagedList
        callback.onResult(posts)
    }
```

Next, we need to create a data source factory:

```java
public class ParseDataSourceFactory extends DataSource.Factory<Integer, Post> {

    @Override
    public DataSource<Integer, Post> create() {
        ParsePositionalDataSource source = new ParsePositionalDataSource();
        return source;
    }
}
```

```kotlin
class ParseDataSourceFactory : DataSource.Factory<Int, Post>() {

    override fun create(): DataSource<Int, Post> {
        val source = ParsePositionalDataSource()
        return source
    }
}
```

Finally, we need to create our PagedList:

```java
public abstract class MyActivity extends AppCompatActivity {

    // normally this data should be encapsulated in ViewModels, but shown here for simplicity
    LiveData<PagedList<Post>> posts;

    public void onCreate(Bundle savedInstanceState) {

        // initial page size to fetch can be configured here
        PagedList.Config pagedListConfig =
              new PagedList.Config.Builder().setEnablePlaceholders(true)
                      .setPrefetchDistance(10)
                      .setInitialLoadSizeHint(20)
                      .setPageSize(10).build();

        ParseDataSourceFactory sourceFactory = new ParseDataSourceFactory();

        posts = new LivePagedListBuilder<>(sourceFactory, pagedListConfig).build();

        posts.observe(this, new Observer<PagedList<Post>>() {
	    @Override
	    public void onChanged(@Nullable PagedList<Post> tweets) {
	        postAdapter.submitList(tweets);
            }
        });
    }
}
```

```kotlin
class MyActivity : AppCompatActivity() {

// normally this data should be encapsulated in ViewModels, but shown here for simplicity
var postList : LiveData<PagedList<Post>>

override fun onCreate(savedInstanceState: Bundle?) {
       postAdapter = PostAdapter()
       
       // initial page size to fetch can be configured here
       val pagedListConfig =
               PagedList.Config.Builder().setEnablePlaceholders(true)
                       .setPrefetchDistance(10)
                       .setInitialLoadSizeHint(20)
                       .setPageSize(10).build()

       val sourceFactory = ParseDataSourceFactory()
       postList = LivePagedListBuilder(sourceFactory, pagedListConfig).build()
   }
```

### Server Code

Running server-side code on Parse:

* [CloudCode Guide](https://docs.parseplatform.org/android/guide/#use-cloud-code) - Guide on how to write cloud-based code that adds error checking, validation or triggers "server-side"

## References

 * <https://docs.parseplatform.org/android/guide/>
 * <https://parseplatform.org/Parse-SDK-Android/api/>
 * <https://www.cronj.com/node-js-development-company.html>
