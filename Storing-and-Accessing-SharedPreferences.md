## Overview

Often you'll find it is necessary to store certain options persistently throughout the lifetime of the application. Using the SharedPreferences interface is the perfect way to do this! This tutorial will cover storing and accessing data using the SharedPreferences interface. 

### Storing Data with SharedPreferences

In order to store data to the SharedPreferences you need to first instantiate an instance of the SharedPreferences like so.

#### Specifying a Preference File

```java
SharedPreferences sharedPreferences = getSharedPreferences("Settings", Context.MODE_PRIVATE);
```

The string `Settings` is the name of the preference file you wish to access. If it does not exist, it will be created. The mode value of 0 designates the default behavior, which is to allow read access to only to the application.  There are other read/write permissions that can be specified, but are no longer encouraged for [security reasons](http://developer.android.com/reference/android/content/Context.html#MODE_WORLD_READABLE).

#### Using a Default Preferences File

If you wish to have a common preference file and don't wish to specify a file, you can also use default shared preferences too:


```java
SharedPreferences sharedPreferences = PreferenceManager.getDefaultSharedPreferences(getApplicationContext());
```

Using this way will default the preference file to be stored as `/data/data/com.package.name/shared_prefs/com.package.name_preferences.xml`. 

#### Editing Preferences

When the user interacts with a preference UI, e.g., an EditTextPreference and changes its default value, then the new value is stored in the default preference file.
If there is a need to edit a preferences file without user interaction, it can be done by creating an Editor instance of SharedPreferences like so.

```java
SharedPreferences.Editor editor = sharedPreferences.edit();
```

Then you can begin to add data to the Settings file declared when you instantiated the SharedPreferences like so.

```java
int id = 1;
editor.putString("keyName", "newValue");
editor.putInt("id", id);
```

Once you are finished adding data you need to 'apply()' the edits by calling.

```java
editor.apply();
```

That's the last step. Your data is stored and you can then access your data using the method below.

### Accessing Stored Data from SharedPrefrences

Once you have stored some data to your SharedPrefrences you may retrieve this value and others by using the following method.

First you need to instantiate an instance of your shared preferences. 

```java
SharedPreferences sharedPreferences = getSharedPreferences("Settings", Context.MODE_PRIVATE);
```

The string Settings is the name of the settings file you wish to access. If it does not exist it will be created. The mode value of 0 designates the default behavior.

The final step is to access the data like so.

```java
String setting = sharedPreferences.getString("keyName", "defaultValue");
```

This will either grab the value that was previously set with the key of "keyName" or will return the string "defaultValue" if it is not found. That's all there is to it.

## References

* <http://developer.android.com/reference/android/content/SharedPreferences.html>