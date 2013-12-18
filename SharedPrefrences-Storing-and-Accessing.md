## Overview

Often you'll find it is necessary to store certain options persistantly throughout the lifetime of the application. Using the SharedPrefrences class is the perfect way to do this! This tutorial will cover storing and accessing data using the SharedPrefrences class. 

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
