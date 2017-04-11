This guide is about troubleshooting common Parse issues that people encounter.

## Common Issues

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

### Leveraging Multi-level Relational Queries

In the case where we want to execute a query with Parse that resembles a complex SQL join query, the best way to achieve this is using inner queries as [outlined briefly here](http://docs.parseplatform.org/android/guide/#relational-queries). The key to inner queries is constructing a query on an associated object and then using [whereMatchesQuery](http://parseplatform.org/Parse-SDK-Android/api/com/parse/ParseQuery.html#whereMatchesQuery\(java.lang.String,%20com.parse.ParseQuery\)) to form the outer query.

For example, if we had a Post which had associated entries each of which had a user (`Post => Entry => User`), we could get all posts which have at least one entry belonging to a particular user with the following multi-level query:

```java
// Get current user object
User me = (User) User.getCurrentUser();
/* Equivalent SQL query (to help you to understand):
 SELECT * FROM Post WHERE Post.entry IN
     (SELECT * FROM Entry WHERE Entry.user = me) */
// Build inner query (finding entries that have the associated user)
ParseQuery<Entry> innerQuery = ParseQuery.getQuery(Entry.class);
innerQuery.whereEqualTo("user", me);
// Build outer query (finding posts with an entry that match the inner query)
ParseQuery<Post> query = ParseQuery.getQuery(Post.class);
query.whereMatchesQuery("entry", innerQuery);
// Include entries and associated users in the results
query.include("entry");
query.include("entry.user");
// Execute the query
query.findInBackground(new FindCallback<Post>() {
    @Override
    public void done(List<Post> postsList, ParseException e) {
        ...
    }
});
```

With this mechanism of inner queries we can recreate a lot of the relational logic used with SQL databases. Note that there is also a mechanism called [whereMatchesKeyInQuery](http://parseplatform.org/Parse-SDK-Android/api/com/parse/ParseQuery.html#whereMatchesKeyInQuery\(java.lang.String,%20java.lang.String,%20com.parse.ParseQuery\)) which adds a constraint to the query that requires a particular key's value to match a specified value.

### Minimizing Number of Queries

When using Parse, you can often have many different relations between models causing your app to send multiple queries out to get back all the required information. Whenever possible try to reduce the queries sent using "include" statements. Refer to the [relational queries](http://docs.parseplatform.org/android/guide/#relational-queries) guide for more details. One caveat is that include only works for pointers to objects and as such does not work with [many-to-many relational data with ParseRelation](http://docs.parseplatform.org/android/guide/#relational-data).

In addition, in certain situations making multiple queries to get all the required data for a screen is unavoidable. In these cases, encapsulating all the separate requests into a single object fetching java object can make data retrieval much more manageable in your activity. See [this android guides issue for a rough example](https://github.com/codepath/android_guides/issues/50). The idea here is to wrap all the queries to Parse in a container object which aggregates all the data and fires a listener once all callbacks have returned.

### Batch Save with Parse

In certain cases, there are many objects that need to be created and saved at once. In this case, we can use batch inserts to significantly speed up posting objects to parse with the [`saveAllInBackground`](http://parseplatform.org/Parse-SDK-Android/api/com/parse/ParseObject.html#saveAllInBackground\(java.util.List\)) static method on `ParseObject`:

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

Even though it may not be apparent from Parse documentation, [caching](http://docs.parseplatform.org/android/guide/#caching-queries) and [pinning](http://docs.parseplatform.org/android/guide/#pinning) are somewhat different concepts. Mixing the two results in an exception like so:

```
Caused by: java.lang.IllegalStateException: Method not allowed when Pinning is enabled.
            at com.parse.ParseQuery.checkPinningEnabled(ParseQuery.java:595)
            at com.parse.ParseQuery.setCachePolicy(ParseQuery.java:610)
```

With caching, hasCachedResult() determines whether a specific result is in the cache.

But there doesn't seem to be a way to determine if a set of objects has been pinned.

## Providing Complete Offline Support

Let's say you want your app to work completely offline. That is, when it launches for the first time, it checks if there is data in the [local datastore](http://docs.parseplatform.org/android/guide/#local-datastore). The app works with the local data if it is available, otherwise it fetches data from the remote store and [pins](hhttp://docs.parseplatform.org/android/guide/#pinning) it locally. When creating new data, the app pins it to the local datastore as well as persists it remotely (based on network availability). To refresh the local cache, the app provides a pull-to-refresh or some similar mechanism.

**Note**: The default Parse functionality removes the locally pinned data after [syncing local changes](http://docs.parseplatform.org/android/guide/#syncing-local-changes).

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

Parse takes its [access control lists](http://docs.parseplatform.org/android/guide/#security-for-user-objects) very seriously--every object has public read and _owner-only_ write access by default. If you have multiple users adding data to your Parse application and they are not all part of the same [role](hhttp://docs.parseplatform.org/android/guide/#security-for-other-objects), it's possible that data created by one user cannot be modified by another. The error message is rather cryptic:

```
exception: com.parse.ParseException: object not found for update
```

Note that it may **not** be possible to fix the ACLs manually for all rows in Parse DB and they keep reverting to their original values.

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
  compile 'com.parse:parseinterceptors:0.0.2'
  compile 'com.facebook.stetho:stetho:1.3.0' // Parse interceptor needs this library
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
