[Parse](http://parseplatform.org/) is an open-source platform that provides one of the easiest ways to get a database and RESTful API up and running. If you want to build a mobile app and don't want to code the back-end by hand, give Parse a try.

<img src="https://i.imgur.com/Xo47jSc.gif" alt="Parse" width="750" />

### Parse on Heroku

We can deploy our own Parse data store and push notifications systems to [Heroku](https://www.heroku.com/) leveraging the [server open-sourced by Parse](https://github.com/ParsePlatform/parse-server-example). Parse is built on top of the MongoDB database which can be added to Heroku using MongoLab.

To get started setting up our own Parse backend, check out our [[configuring a Parse Server]] guide. Once the Parse server is configured, we can [initialize Parse within our Android app](http://guides.codepath.com/android/Configuring-a-Parse-Server#enabling-client-sdk-integration) pointing the client to our self-hosted URL. After that, the functions demonstrated in this guide work the same as they did before.

### Alternatives to Parse

A comprehensive list of alternatives can be [reviewed here](https://github.com/relatedcode/ParseAlternatives). One of the primary alternatives is Google's [Firebase](http://firebase.google.com), which provides a hosted solution for analytics, crash reporting, and a realtime JSON database.  One major difference is that Parse still provides many powerful constructs for querying data, whereas Firebase requires you to perform this querying based on child/parent relations.  See [this guide](https://firebase.google.com/support/guides/parse-android) for more information to porting Parse applications to Firebase.

## What is Parse?

Parse is an open-source Android SDK and back-end solution that enables developers to build mobile apps with shared data quickly and without writing any back-end code or custom APIs.

<img src="https://i.imgur.com/LylIn7w.png" />

Parse is a Node.js application which is deployed onto a host such as Heroku (or AWS) and then creates an automatic API for user authentication and storing data to a MongoDB document store. Parse has the following features included by combining the mobile SDK and back-end service:

 * User registration and authentication
 * Connecting user with Facebook to create a user account.
 * Creating, querying, modifying and deleting arbitrary data models
 * Makes sending push notifications easier
 * Uploading files to a server for access across clients

In short, Parse makes building mobile app ideas much easier!

## Setup

### Installing the Parse SDK

Setting up Parse starts with [[deploying your own Parse instance|Configuring-a-Parse-Server#setting-a-new-parse-server]] to Heroku or another app hosting provider.

Open the `app/build.gradle` in your project and add the following dependencies:

```gradle
dependencies {
    compile 'com.parse.bolts:bolts-android:1.4.0'
    compile 'com.parse:parse-android:1.16.3'
    compile 'com.squareup.okhttp3:logging-interceptor:3.8.1' // for logging API calls to LogCat
}
```

| Package           | Version                                                                                                  |     
|-------------------|:--------------------------------------------------------------------------------------------------------:|
| Parse Bolts       | ![](https://maven-badges.herokuapp.com/maven-central/com.parse.bolts/bolts-android/badge.svg?style=flat) |
| Parse Android SDK | ![](https://maven-badges.herokuapp.com/maven-central/com.parse/parse-android/badge.svg?style=flat)       |

Select `Tools -> Android -> Sync Project with Gradle Files` to load the libraries through Gradle. When you sync, it will import everything automatically. You can see the imported files in the External Libraries folder.

### Initializing the SDK

Next, we need to create an `Application` class and initialize Parse. Be sure to replace the initialization line below with **your correct Parse keys**:

```java
public class ParseApplication extends Application {
    @Override
    public void onCreate() {
        super.onCreate();

        // Use for troubleshooting -- remove this line for production
        Parse.setLogLevel(Parse.LOG_LEVEL_DEBUG);

        // Use for monitoring Parse OkHttp traffic        
        // Can be Level.BASIC, Level.HEADERS, or Level.BODY
        // See http://square.github.io/okhttp/3.x/logging-interceptor/ to see the options.
        OkHttpClient.Builder builder = new OkHttpClient.Builder();
        HttpLoggingInterceptor httpLoggingInterceptor = new HttpLoggingInterceptor();
        httpLoggingInterceptor.setLevel(HttpLoggingInterceptor.Level.BODY);
        builder.networkInterceptors().add(httpLoggingInterceptor);

        // set applicationId, and server server based on the values in the Heroku settings.
        // clientKey is not needed unless explicitly configured
        // any network interceptors must be added with the Configuration Builder given this syntax
        Parse.initialize(new Parse.Configuration.Builder(this)
                .applicationId("myAppId") // should correspond to APP_ID env variable
                .clientKey(null)  // set explicitly unless clientKey is explicitly configured on Parse server
                .clientBuilder(builder)
                .server("https://my-parse-app-url.herokuapp.com/parse/").build());
    }
}
```

The `/parse/` path needs to match the `PARSE_MOUNT` environment variable, which is set to this value by default.

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

Run your app and a new object of class `TestObject` will be sent to the Parse Cloud and saved. See [[browsing Parse data|Configuring-a-Parse-Server#browsing-parse-data]] for more information about how to check this data.

If needed in your application, you might also want to [[setup push notifications|Push-Notifications-Setup-for-Parse]] through Parse as well at this time.

You should be setup now!  Follow the remaining documentation guide to understand how to leverage Parse for your entire backend.  

## Working with Users

At the core of many apps, there is a notion of user accounts that lets users access their information in a secure manner. Parse has a specialized `ParseUser` as a part of their SDK which handles this functionality. Be sure to check out the [Users](http://docs.parseplatform.org/android/guide/#users) docs for a complete overview. See the API docs for [ParseUser](http://parseplatform.org/Parse-SDK-Android/api/com/parse/ParseUser.html) for more details.

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
This call will asynchronously create a new user in your Parse App. Before it does this, it checks to make sure that both the username and email are unique. See the [signup up docs](http://docs.parseplatform.org/android/guide/#signing-up) for more details.

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

If the credentials are correct, the `ParseUser` will be passed back accordingly. You can now access the cached current user for your application at any time in order to determine the session status:

```java
ParseUser currentUser = ParseUser.getCurrentUser();
if (currentUser != null) {
  // do stuff with the user
} else {
  // show the signup or login screen
}
```

A user can be signed back out with:

```java
ParseUser.logOut();
ParseUser currentUser = ParseUser.getCurrentUser(); // this will now be null
```

That's the basics of what you need to work with users. See more details by checking out the [User](http://docs.parseplatform.org/android/guide/#users) official docs.

You can also have a [Facebook Login](http://docs.parseplatform.org/android/guide/#facebook-users) or [Twitter Login](http://docs.parseplatform.org/android/guide/#twitter-users) for your users easily following the guides linked.

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

See a list of [query constraints](http://docs.parseplatform.org/android/guide/#query-constraints) here.

## Working With Data Objects

Storing data on Parse is built around the `ParseObject`. Each `ParseObject` contains key-value pairs of JSON-compatible data. This data is schema-less, which means that you don't need to specify ahead of time what keys exist on each `ParseObject`. Each `ParseObject` has a class name that you can use to distinguish different sorts of data. See the API docs for [ParseObject](http://parseplatform.org/Parse-SDK-Android/api/com/parse/ParseObject.html) for more details.

### Creating Parse Models

When using Parse, the best practice is to create models that represent our data and that subclass `ParseObject` to allow for Parse persistence. Suppose we wanted to create a `TodoItem` model:

```java
import com.parse.ParseObject;
import com.parse.ParseClassName;

@ParseClassName("TodoItem")
public class TodoItem extends ParseObject {
  // Ensure that your subclass has a public default constructor

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

Note that there are two ways to save an object: `saveInBackground` which executes immediately and `saveEventually` which will store the update on the device and push to the server once internet access is available.

See the [saving objects](http://docs.parseplatform.org/android/guide/#saving-objects) and [updating docs](http://docs.parseplatform.org/android/guide/#updating-objects) docs for more details. Also, check out the [relational data](http://docs.parseplatform.org/android/guide/#relations) section.

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

See [retrieving objects](http://docs.parseplatform.org/android/guide/#retrieving-objects) official docs for information on refreshing stale objects and more.

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

See a list of [query constraints](http://docs.parseplatform.org/android/guide/#query-constraints) here and check the [queries overview](http://docs.parseplatform.org/android/guide/#queries) for explanation of compound queries and relational queries.

#### Objects by Querying GeoLocation

Often we might want to query objects within a certain radius of a coordinate (for example to display them on a map). With Parse, querying by `GeoPoint` to retrieve objects within a certain distance of a location is built in. Check the [AnyWall Tutorial](https://github.com/rosstapson/AnyWall) and the [whereWithinMiles](http://parseplatform.org/Parse-SDK-Android/api/com/parse/ParseQuery.html#whereWithinMiles\(java.lang.String,%20com.parse.ParseGeoPoint,%20double\)) and related `where` conditions for more details.

If you want to query this based on a map, first you can [add a listener for the map camera](https://developers.google.com/android/reference/com/google/android/gms/maps/GoogleMap#setOnCameraChangeListener\(com.google.android.gms.maps.GoogleMap.OnCameraChangeListener\)). Next, you can determine the [visible bounds of the map](http://stackoverflow.com/q/16056366) as shown there. Check the [[Maps Usage Guide|Google-Maps-API-v2-Usage]] for additional information on using the map.

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

See [this guide](http://parseplatform.org/docs/parse-server/guide/#live-queries) for more details.  Parse Live Queries rely on the websocket protocol, which creates a bidirectional channel between the client and server and periodically exchange ping/pong frames to validate the connection is still alive.  Websocket URLs are usually prefixed with ws:// or wss:// (secure) URLs.  Heroku instances already provide websocket support, but if you are deploying to a different server (Amazon), you may need to make sure that TCP port 80 or TCP port 443 are available.

You will need to also setup the client SDK by adding this dependency to your `app/build.gradle` config:

```gradle
dependencies {
  // add Parse dependencies too
  compile 'com.parse:parse-livequery-android:1.0.2'
}
```

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

Often with Android development, you need to pass an object from one Activity to another. This is done using the `Intent` system and passing objects as extras within a bundle. Unfortunately, `ParseObject` does not currently implement Parcelable or Serializable.

The simplest way to pass data between activities in Parse is simply to pass the object ID into the Intent:

```java
Intent i = new Intent(this, SomeNewActivity.class);
i.putExtra("todo_id", myTodoItem.getObjectId());
startActivity(i);
```

and then refetch the object using the object ID within the child Activity:

```java
String todoId = getIntent().getStringExtra("todo_id");
ParseQuery<TodoItem> query = ParseQuery.getQuery(TodoItem.class);
// First try to find from the cache and only then go to network
query.setCachePolicy(ParseQuery.CachePolicy.CACHE_ELSE_NETWORK); // or CACHE_ONLY
// Execute the query to find the object with ID
query.getInBackground(todoId, new GetCallback<TodoItem>() {
  public void done(TodoItem item, ParseException e) {
    if (e == null) {
       // item was found
    }
  }
}
```

You can also use `query.getFirst()` instead to retrieve the item in a synchronous style. Review the different [caching policies](http://parseplatform.org/docs/android/guide/#caching-queries) to understand how to make this fast.

While we could [implement parceling ourselves](http://www.androidbook.com/akc/display?url=DisplayNoteIMPURL&reportId=4539&ownerUserId=android) this is not ideal as it's pretty complex to manage the state of Parse objects.

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

For more details, check out the official [Relational Data](http://docs.parseplatform.org/android/guide/#using-pointers) guide. For more complex many-to-many relationships, check out this official [join tables](http://docs.parseplatform.org/android/guide/#using-join-tables) guide when the many-to-many requires additional metadata.

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

Parse now supports a more powerful form of [local data storage](http://parseplatform.org/docs/android/guide/#local-datastore) out of the box which can be used to store and retrieve ParseObjects, even when the network is unavailable. To enable this functionality, simply call Parse.enableLocalDatastore() before your call to initialize:

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

For the full summary of how to utilize the offline mode for Parse, be sure to review the [official local store guide](http://parseplatform.org/docs/android/guide/#local-datastore) in the Parse docs.

## Using the Data Browser

Suppose we had a simple todo application with user accounts and items persisted to Parse. The next step is to setup and create our models using the [Parse dashboard](http://guides.codepath.com/android/Configuring-a-Parse-Server#parse-dashboard) to manage your new app. Visit the "Data Browser" for the correct application and let's create our `User` and `TodoItem` objects for our app.

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

Parse has full support for storing images and files uploaded by an application. Photos are stored using the `ParseFile` construct [described in more detail here](http://parseplatform.org/docs/android/guide/#files). Refer to the following resources for more details:

 * [Parse Docs on File Uploads](http://parseplatform.org/docs/android/guide/#files)
 * [Parse Image Upload Tutorial](http://www.androidbegin.com/tutorial/android-parse-com-image-upload-tutorial) - Tutorial on using `ParseFile` to upload images.
 * [Mealspotting Sample App](https://github.com/rufflez/MealSpottingTutorial) - Detailed app sample showing how to store images associated to a record. Here's a [related video](https://www.youtube.com/watch?v=X9ttMA5ca9U).
 * [[CodePath Camera and Gallery Guide|Accessing-the-Camera-and-Stored-Media]] - Guide on capturing photos with the camera.
 * [[Handling Files after API 24|Sharing-Content-with-Intents#sharing-files-with-api-24-or-higher]] - Section on how to store files using `FileProvider`
 * [How to upload images onto Parse post](http://stackoverflow.com/questions/31227547/how-to-upload-image-to-parse-in-android) - Good StackOverflow post on uploading images to Parse


### Geo Location

Parse has support for geolocation services and querying:

* [AnyWall Sample App](https://github.com/rosstapson/AnyWall) - Sample app including how to read geolocation and use this data within your app.

### Push Notifications

Parse supports push notifications made easy:

* [[Parse Push Messaging|Push-Messaging]] - CodePath Guide
* [[Parse Push|Configuring-a-Parse-Server#enabling-push-notifications]] - Super easy push notifications

### Facebook SDK

For a quick way of incorporating Facebook login, check out Parse's [UI Library](https://github.com/ParsePlatform/ParseUI-Android/wiki/Login-Library).  It leverages Parse's [FacebookUtils](https://github.com/ParsePlatform/ParseFacebookUtils-Android/) library, which acts as a wrapper for associating `ParseUser` objects with Facebook users.  

The official step by step instructions for integrating Parse with Facebook SDK is located [here](http://parseplatform.org/docs/android/guide/#facebook-users).  The manual process of integrating with Facebook's SDK is discussed below.

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
  compile 'com.facebook.android:facebook-android-sdk:4.10.0'
  compile 'com.parse:parsefacebookutils-v4-android:1.10.4@aar'
}
```

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

### Server Code

Running server-side code on Parse:

* [CloudCode Guide](http://parseplatform.org/docs/android/guide/#use-cloud-code) - Guide on how to write cloud-based code that adds error checking, validation or triggers "server-side"

## References

 * <http://docs.parseplatform.org/android/guide/>
 * <http://parseplatform.github.io/Parse-SDK-Android/api/>
