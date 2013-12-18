## Overview

In Android apps, there are often settings pages that contain different options the user can tweak. The [PreferenceFragment](http://developer.android.com/reference/android/preference/PreferenceFragment.html) contains a hierarchy of preference objects displayed on screen in a list. These preferences will automatically save to SharedPreferences as the user interacts with them.

To retrieve an instance of `SharedPreferences` that the preference hierarchy in this fragment will use, call `PreferenceManager.getDefaultSharedPreferences(android.content.Context)` with a context in the same package as this fragment.

### Storing Data with SharedPrefrences

In order to store data to the SharedPrefrences you need to first instantiate an instance of the SharedPrefrences like so.

`SharedPreferences mSettings = getActivity().getSharedPreferences("Settings", 0);`

The string Settings is the name of the settings file you wish to access. If it does not exist it will be created. The mode value of 0 designates the default behavior.

The next step is to create and Editor instance of SharedPrefrences like so.

`SharedPreferences.Editor editor = mSettings.edit();`

Then you can begin to add data to the Settings file declared when you instantiated the SharedPrefrences like so.

    Integer id = 1;
    String username = "john";
    editor.putString("username", username);
    editor.putInt("id", id);

Once you are finished adding data you need to 'commit()' the edits by calling.

    editor.commit();

That's the last step. Your data is stored and you can then access using the below method.


### Accessing Stored Data from SharedPrefrences

Once you have stored some data to your SharedPrefrences you may retrieve this value and other by using the following method.

First you need to instantiate and instance of your shared preferences. 

`SharedPreferences mSettings = getActivity().getSharedPreferences("Settings", 0);`

The string Settings is the name of the settings file you wish to access. If it does not exist it will be created. The mode value of 0 designates the default behavior.

The final step is to access the data like so.

`String cookieName = mSettings.getString("cookieName", "missing");`

This will either grab the value that was previously set with the key of "cookieName" or will return the string "missing" if it is not found. That's all there is to it.



## References

 * <http://developer.android.com/reference/android/preference/PreferenceFragment.html>
 * <http://developer.android.com/guide/topics/ui/settings.html>
 * <http://www.mysamplecode.com/2011/11/android-shared-preferences-example_12.html>
 * <http://mobile.tutsplus.com/tutorials/android/android-user-interface-design-building-application-preference-screens/>
 * <http://gmariotti.blogspot.com/2013/01/preferenceactivity-preferencefragment.html>