[Parse](https://www.parse.com) is a service that provides one of the easiest ways to get a database and RESTful API up and running. If you want to build a mobile app and don't want to code the back-end, give Parse a try.

## Registration

First, we need to [sign up for a Parse account](https://www.parse.com/#signup) unless we are already registered. 

## Setup

Make sure you have an app prepared that you would like to integrate with Parse. Follow the steps on the [existing app page](https://www.parse.com/apps/quickstart#parse_data/mobile/android/native/existing) starting with [downloading the Parse SDK](https://parse.com/downloads/android/Parse/latest).  Make sure to edit the `app/build.gradle` file to make any changes (not the top-level `build.gradle` file).  You must also unzip the Parse SDK .ZIP file too.

<a target="_blank" href="https://www.parse.com/apps/quickstart#parse_data/mobile/android/native/existing"><img src="http://i.imgur.com/ldLJfil.png" alt="screen_1" width="500" /></a>

Inside the Parse SDK .zip file should be a  `Parse-X.X.X.jar` file.  Drag this JAR file (not the .ZIP file) into the "libs" folder of your Android app. See these [instructions for how to reveal the libs folder](http://stackoverflow.com/a/28020621/313399) in Android Studio.

<img src="http://i.imgur.com/EO6vp5S.png" alt="screen_2" width="400" />

Next, we need to create an `Application` class and initialize Parse. Be sure to replace the initialization line below with **your correct Parse keys**:

```java
public class ParseApplication extends Application {
    public static final String YOUR_APPLICATION_ID = "AERqqIXGvzH7Nmg45xa5T8zWRRjqT8UmbFQeeI";
    public static final String YOUR_CLIENT_KEY = "8bXPznF5eSLWq0sY9gTUrEF5BJlia7ltmLQFRh";

    @Override
    public void onCreate() {
        super.onCreate();

        // Add your initialization code here
        Parse.enableLocalDatastore(this);
        Parse.initialize(this, YOUR_APPLICATION_ID, YOUR_CLIENT_KEY);

        // ...
  }		
}
```

We also need to make sure to set the application instance above as the `android:name` for the application within the `AndroidManifest.xml`. This change in the manifest determines which application class is instantiated when the app is launched:

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.parsetododemo"
    android:versionCode="1"
    android:versionName="1.0" >
    <application
        android:allowBackup="true"
        android:name="com.codepath.example.parsetododemo.ParseApplication"
        android:icon="@drawable/ic_launcher"
        android:label="@string/app_name"
        android:theme="@style/AppTheme" >
    </application>
</manifest>
```

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

Now, let's test the SDK. We should be able to create a new object to verify that Parse is working with this application. Let's add the test code to `ParseApplication` as follows:

```java
public class ParseApplication extends Application {
    public static final String YOUR_APPLICATION_ID = "AERqqIXGvzH7Nmg45xa5T8zWRRjqT8UmbFQeeI";
    public static final String YOUR_CLIENT_KEY = "8bXPznF5eSLWq0sY9gTUrEF5BJlia7ltmLQFRh";

    @Override
    public void onCreate() {
        super.onCreate();

      	// Add your initialization code here
      	Parse.initialize(this, YOUR_APPLICATION_ID, YOUR_CLIENT_KEY);
    
        // Test creation of object
      	ParseObject testObject = new ParseObject("TestObject");
      	testObject.put("foo", "bar");
      	testObject.saveInBackground();
    }		
}
```

Run your app and a new object of class TestObject will be sent to the Parse Cloud and saved. Click on the "Test" button back on the [Parse quickstart guide](https://www.parse.com/apps/quickstart#parse_data/mobile/android/native/existing) to confirm data was successfully transmitted.

<img src="http://i.imgur.com/YloGilR.png" alt="screen_3" width="500" />

If you see "Congrats! You saved your first object", then Parse is setup successfully. If not, review the steps above to get Parse setup. 

If needed in your application, you might also want to [setup push notifications](https://www.parse.com/apps/quickstart#parse_push/android/existing) through Parse as well at this time.

## Working with Users

At the core of many apps, there is a notion of user accounts that lets users access their information in a secure manner. Parse has a specialized `ParseUser` as a part of their SDK which handles this functionality. Be sure to check out the [Users](https://www.parse.com/docs/android_guide#users) docs for a complete overview. See the API docs for [ParseUser](http://www.parse.com/docs/android/api/com/parse/ParseUser.html) for more details.

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
This call will asynchronously create a new user in your Parse App. Before it does this, it checks to make sure that both the username and email are unique. See the [signup up docs](https://www.parse.com/docs/android_guide#users-signup) for more details.

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

That's the basics of what you need to work with users. See more details by checking out the [User](https://www.parse.com/docs/android_guide#users) official docs.

You can also have a [Facebook Login](https://www.parse.com/docs/android_guide#fbusers-setup) or [Twitter Login](https://www.parse.com/docs/android_guide#twitterusers-setup) for your users easily following the guides linked.

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

See a list of [query constraints](https://www.parse.com/docs/android_guide#queries-constraints) here.

## Working With Data Objects

Storing data on Parse is built around the `ParseObject`. Each `ParseObject` contains key-value pairs of JSON-compatible data. This data is schemaless, which means that you don't need to specify ahead of time what keys exist on each `ParseObject`. Each `ParseObject` has a class name that you can use to distinguish different sorts of data. See the API docs for [ParseObject](http://www.parse.com/docs/android/api/com/parse/ParseObject.html) for more details.

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

See the [saving objects](https://www.parse.com/docs/android_guide#objects-saving) and [updating docs](https://www.parse.com/docs/android_guide#objects-updating) docs for more details. Also, check out the [relational data](https://www.parse.com/docs/android_guide#objects-pointers) section.

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

See [retrieving objects](https://www.parse.com/docs/android_guide#objects-retrieving) official docs for information on refreshing stale objects and more.

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

See a list of [query constraints](https://www.parse.com/docs/android_guide#queries-constraints) here and check the [queries overview](https://www.parse.com/docs/android_guide#queries) for explanation of compound queries and relational queries.

#### Objects by Querying GeoLocation

Often we might want to query objects within a certain radius of a coordinate (for example to display them on a map). With Parse, querying by `GeoPoint` to retrieve objects within a certain distance of a location is built in. Check the [AnyWall Tutorial](https://parse.com/tutorials/anywall-android) and the [whereWithinMiles](https://parse.com/docs/android/api/com/parse/ParseQuery.html#whereWithinMiles\(java.lang.String,%20com.parse.ParseGeoPoint,%20double\)) and related `where` conditions for more details. 

If you want to query this based on a map, first you can [add a listener for the map camera](https://developers.google.com/android/reference/com/google/android/gms/maps/GoogleMap#setOnCameraChangeListener\(com.google.android.gms.maps.GoogleMap.OnCameraChangeListener\)). Next, you can determine the [visible bounds of the map](http://stackoverflow.com/q/16056366) as shown there. Check the [[Maps Usage Guide|Google-Maps-API-v2-Usage]] for additional information on using the map.

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

You can also use `query.getFirst()` instead to retrieve the item in a synchronous style. Review the different [caching policies](https://parse.com/docs/android_guide#queries-caching) to understand how to make this fast.

While we could [implement parceling ourselves](http://www.androidbook.com/akc/display?url=DisplayNoteIMPURL&reportId=4539&ownerUserId=android) this is not ideal as it's pretty complex to manage the state of Parse objects. We could also [use a Proxy object](https://www.parse.com/questions/passing-around-parseobjects-in-android) to pass the data as well but this can be brittle.

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

For more details, check out the official [Relational Data](https://www.parse.com/docs/android_guide#objects-pointers) guide. For more complex many-to-many relationships, check out this official [join tables](https://www.parse.com/docs/relations_guide#manytomany-jointables) guide when the many-to-many requires additional metadata.

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

Parse now supports a more powerful form of [local data storage](https://parse.com/docs/android_guide#localdatastore) out of the box which can be used to store and retrieve ParseObjects, even when the network is unavailable. To enable this functionality, simply call Parse.enableLocalDatastore() before your call to initialize:

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

## Creating Data Classes

Suppose we had a simple todo application with user accounts and items persisted to Parse. The next step is to create our models using the [dashboard](https://www.parse.com/apps) to manage your new app. Visit the "Data Browser" for the correct application and let's create our `User` and `TodoItem` objects for our app.

First, **remove the test code that we added previously** and drop the "TestObject" listed in the browser to clear testing data.

<img src="http://i.imgur.com/ZFds0Fn.png" alt="screen_5" width="400" />

Next, select "New Class" and pick "User" to create the user object used to manage session authetication:

<img src="http://i.imgur.com/7hQZz3y.png" alt="screen_6" width="300" />

Let's also add our "Custom" class which can represent any custom data. In this case, we will create a `TodoItem` class:

<img src="http://i.imgur.com/MIZkV8L.png" alt="screen_7" width="300" />

Now, we need to add our custom columns to our class. In this case, let's add a "body" field to our `TodoItem` by selecting "+Col" and then selecting the type as a String and column name as "body":

<img src="http://i.imgur.com/qYel2NP.png" alt="screen_8" width="300" />

Once you've finished adding your columns to the class, you can create as many additional classes as necessary and configure their respective columns. Let's add a row of data to the class directly through the data browser:

<img src="http://i.imgur.com/QFMEnb7.png" alt="screen_9" width="600" />

We are now ready to access these classes within our application.

### Further Details

For the full summary of how to utilize the offline mode for Parse, be sure to review the [official local store guide](https://parse.com/docs/android_guide#localdatastore) in the Parse docs.

## Troubleshooting

Check out our [[Troubleshooting Common Issues with Parse]] guide for a detailed look at common issues encountered and related solutions.

## Additional Features

Parse has many powerful features in addition to the core functionality listed above:

* [Parse Push](https://parse.com/tutorials/android-push-notifications) - Super easy push notifications
* [Integrating with Facebook SDK](https://parse.com/tutorials/integrating-facebook-in-android) - Step by step for Facebook account integration
* [Using Parse Files](https://parse.com/docs/android_guide#files-classes) - Reference for how to store files (i.e images) to Parse
* [Mealspotting Tutorial](https://parse.com/tutorials/mealspotting) - Tutorial including how to store images associated to a record.
* [AnyWall Tutorial](https://parse.com/tutorials/anywall-android) - Tutorial including how to read geolocation and use this data within your app.
* [CloudCode Guide](https://parse.com/docs/cloud_code_guide#cloud_code) - Guide on how to write cloud-based code that adds error checking, validation or triggers "server-side" 

## References

 * <https://www.parse.com/docs/android_guide>
 * <https://www.parse.com/docs/android/api/?com/parse/ParseObject.html>