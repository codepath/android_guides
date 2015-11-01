This guide is about troubleshooting common Parse issues that people encounter.

## Common Issues 

> Can't save object, I get "com.parse.ParseException: object not found for update"

This is usually an ACL Error which can be fixed by reviewing [this post](https://parse.com/questions/comparseparseexception-object-not-found-for-update-error-when-the-object-exists), [this one](https://www.parse.com/questions/android-object-not-found-for-update) and also [this post](https://parse.com/questions/how-to-update-objects-in-android).

> Can't construct query based on an associated object

If your data model has association pointer of other object, parse requires us to pass the associated object and not the object's id. For example,  Post(ParseObject) has a Author(ParseObject) relationship and author attribute is ParsePointer. If you have author object id "yiadffao89" and want to find all posts by an author:

```java
new ParseQuery("Post")
    .whereEqualTo("author",  ParseObject.createWithoutData(Author.class, "yiadffao89 "))
    .find();
```

> Compound equality constraints are seemingly ignored!

Rather than checking for multiple equality values, we need to instead use the `whereNotContainedIn` condition to include all equality conditions within a single compound condition.

```java
// FAILS
ParseQuery = ParseQuery.getQuery(Sample.class);
query.whereNotEqualTo("key", "valueA");
query.whereNotEqualTo("key", "valueB");
// WORKS
query.whereNotContainedIn("key", ImmutableList.of("valueA", "valueB"));
```

## Exploring Parse

Parse has quirks. Several are outlined below.

### Extending ParseUser

A common use case that comes up is trying to **extend the ParseUser class** to create your own custom `ParseUser` subclass. This causes issues with Parse and [is not recommended](https://github.com/ParsePlatform/ParseUI-Android/issues/2). Instead you can explore using a delegate pattern like this instead:

```java
public class CustomUser {
  private ParseUser user;

  void CustomUser(ParseUser obj) {
    user = obj;
  }

  void setSomeField(int someField) {
    user.put("some_field", 56);
    user.saveInBackground();
  }

  Date getCreatedAt() {
    return user.getCreatedAt();
  }
}
```  

While this isn't as convenient, it works more reliably.

### Subclassing a ParseObject Subclass

In certain cases, you may find yourself subclassing a Parse subclass to try to follow a single type inheritance pattern with parse. For example:

```java
@ParseClassName("Stove")
public class Stove extends ParseObject {
   // ...
}

@ParseClassName("ElectricStove")
public class ElectricStove extends Stove {
  public ElectricStove() {
  }

  public ElectricStove(String url, String brandName) {
     super(url, brandName);
  }

    // ...
}
```

This doesn't appear to be possible at this point as Parse Android SDK does not support this. Rather, we can use an identifier to specify what type of "Stove" a particular stove object is. Refer to this [stackoverflow example](http://stackoverflow.com/a/31682925) for specifics. 

### Passing ParseObject through Intent

Often with Android development, you need to pass an object from one `Activity` to another. This is done using the Intent system and passing objects as extras within a bundle. Unfortunately, `ParseObject` does not currently implement `Parcelable` or `Serializable`. See [[the available workarounds here|Building-Data-driven-Apps-with-Parse#passing-objects-between-activities]].

### Minimizing Number of Queries

When using Parse, you can often have many different relations between models causing your app to send multiple queries out to get back all the required information. Whenever possible try to reduce the queries sent using "include" statements. Refer to the [relational queries](https://parse.com/docs/android/guide#queries-relational-queries) guide for more details. One caveat is that include only works for pointers to objects and as such does not work with [many-to-many relational data with ParseRelation](https://parse.com/docs/android/guide#objects-relational-data). See [this parse support post](https://www.parse.com/questions/can-i-use-include-in-a-query-to-include-all-members-of-a-parserelation-error-102) for further clarification.

In addition, in certain situations making multiple queries to get all the required data for a screen is unavoidable. In these cases, encapsulating all the separate requests into a single object fetching java object can make data retrieval much more manageable in your activity. See [this android guides issue for a rough example](https://github.com/codepath/android_guides/issues/50). The idea here is to wrap all the queries to Parse in a container object which aggregates all the data and fires a listener once all callbacks have returned.

### Batch Save with Parse

In certain cases, there are many objects that need to be created and saved at once. In this case, we can use batch inserts to significantly speed up posting objects to parse with the [`saveAllInBackground`](https://parse.com/docs/android/api/com/parse/ParseObject.html#saveAllInBackground\(java.util.List\)) static method on `ParseObject`:

```java
List<ParseObject> objList = new ArrayList<ParseObject>();
// Add new ParseObject instances to the list
objList.add(...);
// Save all created ParseObject instances at once
ParseObject.saveAllInBackground(objList, new SaveCallback() {
   @Override
   public void done(ParseException e) {
       if (e == null) {
           Toast.makeText(getApplicationContext(), "saved", Toast.LENGTH_LONG).show();
       } else {
           Toast.makeText(getApplicationContext(), e.getMessage().toString(), Toast.LENGTH_LONG).show();
       }
   }
});
```

Avoid ever calling save on multiple `ParseObject` instances in quick succession.

## Local Datastore and Public Read Access

It looks like public read access to data is necessary for local datastore to work. Local datastore returned no results when role-specific read access was setup.

### Caching vs. Pinning

Even though it may not be apparent from Parse documentation, [caching](https://parse.com/docs/android_guide#queries-caching) and [pinning](https://parse.com/docs/android_guide#localdatastore-pin) are different concepts. Mixing the two results in an exception like so:

```
Caused by: java.lang.IllegalStateException: Method not allowed when Pinning is enabled.
            at com.parse.ParseQuery.checkPinningEnabled(ParseQuery.java:595)
            at com.parse.ParseQuery.setCachePolicy(ParseQuery.java:610)
```

With caching, hasCachedResult() determines whether a specific result is in the cache.

But there doesn't seem to be a way to determine if a set of objects has been pinned. 

## Providing Complete Offline Support

Let's say you want your app to work completely offline. That is, when it launches for the first time, it checks if there is data in the [local datastore](https://parse.com/docs/android_guide#localdatastore). The app works with the local data if it is available, otherwise it fetches data from the remote store and [pins](https://parse.com/docs/android_guide#localdatastore-pin) it locally. When creating new data, the app pins it to the local datastore as well as persists it remotely (based on network availability). To refresh the local cache, the app provides a pull-to-refresh or some similar mechanism.

**Note**: The default Parse functionality removes the locally pinned data after [syncing local changes](https://parse.com/docs/android_guide#localdatastore-saving). 

### Fetching Data 

```java

// Fetch from local. If not found, fetch from remote
progressBarVisible();
query.fromPin(pinName);
query.findInBackground(...) {
   // inside success callback
   if (no results found) {
      fetchRemoteData();
   } else {
      // add data to adapter
      progressBarInvisible();
   }
});


// Fetch from remote and replace local pin
fetchRemoteData() {

   newQuery.findInBackground(...) {
      // inside success callback
      // add data to adapter
      progressBarInvisible();
      unpinAndRepin(data);
   });
}

// Unpin old data and pin new data to local datastore
unpinAndRepin(data) {

   ParseObject.unpinAllInBackground(pinName, data, new DeleteCallback() { ... });
   // Add the latest results for this query to the cache.
   ParseObject.pinAllInBackground(pinName, data);
}
```

### Persisting Data

```java

// First pin locally
onSubmit(data) {
   progressBarVisible();
   pinDataLocally(data);
}

// After local pinning is successful, try to persist remotely.
pinDataLocally(data) {

   setProperACL();
   data.pinInBackground(data, new SaveCallback() {
      // inside success callback
      persistDataRemotely();
      progressBarInvisible();
   }
}

persistDataRemotely(data) {
   data.saveEventually(...) { ... }
}
```

## Configuring Proper ACLs

Parse takes its [access control lists](https://parse.com/docs/android_guide#users-acls) very seriously--every object has public read and _owner-only_ write access by default. If you have multiple users adding data to your Parse application and they are not all part of the same [role](https://parse.com/docs/android_guide#roles), it's possible that data created by one user cannot be modified by another. The error message is rather cryptic:

```
exception: com.parse.ParseException: object not found for update
```

Review related threads [here](https://parse.com/questions/comparseparseexception-object-not-found-for-update-error-when-the-object-exists), [here](https://www.parse.com/questions/android-object-not-found-for-update) and also [here](https://parse.com/questions/how-to-update-objects-in-android).

**Note**: It may **not** be possible to fix the ACLs manually for all rows in Parse DB and they keep reverting to their original values.

### Solution

As a one-time operation, fix your ACLs.

```java

// Define a new role with desired read and write access.
final ParseACL roleACL = new ParseACL();
roleACL.setPublicReadAccess(true);
final ParseRole role = new ParseRole("Engineer", roleACL);
 
// Add users to this role.
final ParseQuery<ParseUser> userQuery = ParseUser.getQuery();
userQuery.findInBackground(new FindCallback<ParseUser>() {
   @Override
   public void done(List<ParseUser> parseUsers, ParseException e) {
      if (e == null) {
         for (final ParseUser user : parseUsers) {
            role.getUsers().add(user);
         }
 
         role.saveInBackground(new SaveCallback() {
            @Override
            public void done(ParseException e) {
               if (e == null) {
                  Log.d("debug", "Saved role " + role.getName());
               } else {
                  Log.d("debug", "Couldn't save role " + e.toString());
               }
            }
         });
      } else {
         Log.d("debug", "Couldn't get users " + e.toString());
      }
   }
});
```

```java
// Set ACLs on the data based on the role.
final ParseQuery<ParseObject> query = ...
final ParseACL roleACL = new ParseACL();
roleACL.setPublicReadAccess(true);
roleACL.setRoleReadAccess("Engineer", true);
roleACL.setRoleWriteAccess("Engineer", true);
query.findInBackground(new FindCallback<ParseObject>() {

   @Override
   public void done(List<ParseObject> parseObjects, ParseException e) {
      for (ParseObject obj : parseObjects) {
         final MyObj myObj = (MyObj) obj;
         myObj.setACL(roleACL);
         myObj.saveInBackground(...) { }
});
```

For every new object accessible by this role, be sure to set the ACL too.

```java
final ParseACL roleACL = new ParseACL();
roleACL.setPublicReadAccess(true);
roleACL.setRoleReadAccess("Engineer", true);
roleACL.setRoleWriteAccess("Engineer", true);
myObj.setACL(reportACL);
```

## Troubleshooting network issues

There are several different ways for inspecting the network traffic between Parse and your emulator or device.  You can send all network calls through LogCat, or you can leverage Facebook's [Stetho](https://code.facebook.com/posts/393927910787513/stetho-a-new-debugging-platform-for-android/) project to monitor the traffic using Chrome's network inspector.  See also the [blog posting](http://blog.parse.com/announcements/parse-debugging-tools-identify-client-side-issues-faster/) for more details.

<img src="http://blog.parse.com/wp-content/uploads/2015/09/image-1024x585.png"/>

Add this dependency to your `app/build.gradle` file:

```gradle
dependencies {
  compile 'com.parse:parseinterceptors:0.0.1'
}
```

### Sending all network calls to LogCat

Before you initialize Parse, add the network interceptor:

```java
Parse.addParseNetworkInterceptor(new ParseLogInterceptor());
Parse.initialize(context, parseAppId, parseClientKey);
```

### Monitoring all network calls through Chrome

You can add the `ParseStethoInterceptor` before initializing Parse: 

```java
Parse.addParseNetworkInterceptor(new ParseStethoInterceptor());
Parse.initialize(context, parseAppId, parseClientKey);
```

Open Chrome and type `chrome://inspect`.  You should find a list of devices that you can track.  Click on the `inspect` and then click on the `Network` tab.