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

> Parse LoginUI library’s sample projects are using Gradle and will not work with Eclipse.

Steps to import [ParseUI](https://github.com/ParsePlatform/ParseUI-Android) into Eclipse:

1. Create a new Eclipse project without any activity. (In wizard don’t select main activity).
2. Copy following from sample project to newly created eclipse project.
   * `AndroidManifest.xml`
   * `src` directory
   * `res` directory
   * Update `strings.xml` with your parse/facebook/twitter keys.
3. Clean and rebuild newly created eclipse project.
4. Run project

## Local Datastore and Public Read Access

It looks like public read access to data is necessary for local datastore to work. Local datastore returned no results when role-specific read access was setup.

## Caching vs. Pinning

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