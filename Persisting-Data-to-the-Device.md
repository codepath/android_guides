## Overview

The Android framework offers several options and strategies for persistence:

 * Shared Preferences - Easily save basic data as key-value pairs in a private persisted dictionary.
 * Local Files - Save arbitrary files to internal or external device storage.
 * SQLite Database - Persist data in tables within an application specific database.
 * ORM - Describe and persist model objects using a higher level query/update syntax.

### Shared Preferences

Settings can be persisted for your application by using [SharedPreferences](http://developer.android.com/reference/android/content/SharedPreferences.html) to persist key-value pairs:

```java
SharedPreferences pref =   
    PreferenceManager.getDefaultSharedPreferences(this);
String username = pref.getString("username", "n/a"); 
```

SharedPreferences can be edited by getting access to the Editor instance:

```java
SharedPreferences pref =   
    PreferenceManager.getDefaultSharedPreferences(this);
Editor edit = pref.edit();
edit.putString("username", "billy");
edit.putString("user_id", "65");
edit.commit(); 
```

### Local Files

### ActiveAndroid ORM

## References

 * <http://developer.android.com/guide/topics/data/data-storage.html>
 * <http://developer.android.com/training/basics/data-storage/index.html>
 * <http://mobile.tutsplus.com/tutorials/android/data-management-options-for-android-applications/>
 * <http://developer.android.com/reference/android/content/SharedPreferences.html>