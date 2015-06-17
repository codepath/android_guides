## Overview

An ActionBar is located at the top of an activity and it can display any number of status or navigation related elements such as title, icon, buttons, or arbitrary action-related views.

Older Android devices have a hardware Option button to open the menu. The ActionBar replaces that concept in modern Android apps.

This is typically used for displaying the title of the application and providing a primary navigation for the app. The ActionBar can contain primary action buttons as well as tabs to control navigation within any app.

**Note** that we will be using the support library and `AppCompatActivity` for these examples since the library provides maximum compatibility with pre-3.0 Android versions. The APIs and usage are the same with the standard ActionBar just with small changes to the imported classes and class names.  If you are not currently using the support library, check out this [[migration guide|Migrating to the AppCompat Library]].

## Usage

In the [[Defining ActionBar|Defining-The-ActionBar#actionbar-basics]] cliffnotes we looked at the basics of adding items to the ActionBar and handling clicks. In this section, we take a look at how to use `AppCompatActivity` to support all Android versions and also at several powerful and extensible ActionBar features:

 * Using the split action bar to have a top and bottom menu
 * Adding an Action View (action_layout) and SearchView
 * Using ActionProvider and ShareActionProvider to enable richer functions
 * Configuring Home Icon to navigate "Up"

### Setting up AppCompatActivity

In order to ensure that the ActionBar works on all Android versions, we are going to use [AppCompatActivity](https://developer.android.com/reference/android/support/v7/app/AppCompatActivity.html) to setup our support ActionBar.

First, add the following line to your `app/build.gradle` file:

```gradle
apply plugin: 'com.android.application'

//...

dependencies {
    // ...
    compile 'com.android.support:appcompat-v7:22.2.0' 
    // or if not compiling to SDK 21 see note below and use:
    // compile 'com.android.support:appcompat-v7:20.0.0'  
}
```

Make sure to bump your `compileSdkVersion` to API version 21.  Otherwise the AppCompat library will not assume the Material Design themes should be included.

```gradle
android {

   compileSdkVersion 21
   buildToolsVersion "22.0.1" 
 
}
```

Once it's added, be sure to sync your project with the gradle file (`Tools => Android => Sync Project with Gradle Files`) and make sure any applicable activities are now extending from `AppCompatActivity` in order to enable
the compatibility fragments and action bar:

```java
public class MainActivity extends AppCompatActivity {
   // ...
}
```

and change the parent theme to a support compatible theme in `values/styles.xml` such as `Theme.AppCompat.Light.DarkActionBar`:

```xml
<resources>
    <!-- Base application theme. -->
    <style name="AppTheme" parent="Theme.AppCompat.Light.DarkActionBar">
        <!-- Customize your theme here. -->
    </style>
</resources>
```

Now the support ActionBar is configured and ready to be used. However, one should note that the `res/menu` xml files now need to use an `app:` prefix for `showAsAction` as shown below:

```xml
<menu xmlns:android="http://schemas.android.com/apk/res/android"
      xmlns:app="http://schemas.android.com/apk/res-auto">
    <item
        android:id="@+id/action_refresh" 
        android:title="@string/action_refresh"
        android:icon="@drawable/ic_action_refresh"
        app:showAsAction="ifRoom" />
</menu>
```

Note the use of `app:showAsAction` instead of `android:showAsAction` in the case of adding support action bar items.

### Populating Action Buttons

The action bar provides users access to the most important action items relating to the app's current context. Those that appear directly in the action bar with an icon and/or text are known as action buttons. 

For the support ActionBar, just change `onCreateOptionsMenu` to use the compatibility inflater:

```java
@Override
public boolean onCreateOptionsMenu(Menu menu) {
    // Inflate the menu; this adds items to the action bar if it is present.
    getMenuInflater().inflate(R.menu.menu_main, menu);
    return true;
}
```

See the [[Defining the ActionBar|Defining-The-ActionBar#actionbar-basics]] cliffnotes
for the basic details of adding items and handling clicks.


### Configuring a Split-Action Bar

Split action bar provides a separate bar at the bottom of the screen to display all action items when the activity is running on a narrow screen (such as a portrait-oriented handset). This is helpful when you want to display a top and bottom row of actions for a context.

To add support for the split-action bar, just add the option to the manifest for that activity:

```xml
<manifest ...>
    <activity android:uiOptions="splitActionBarWhenNarrow" ... >
        <meta-data android:name="android.support.UI_OPTIONS"
                   android:value="splitActionBarWhenNarrow" />
    </activity>
</manifest>
```

You can control the order of items within the ActionBar using `orderinCategory` where each menu item has an integer specified and lower integers are displayed with a higher priority:

```xml
<item
   android:id="@+id/menu_ordinary"
   android:orderInCategory="200"
   app:showAsAction="ifRoom"
   android:title="Ordinary" />

<item
   android:id="@+id/menu_important"
   android:orderInCategory="20"
   app:showAsAction="ifRoom"
   android:title="Important" />
```

Using split action bar also allows navigation tabs to collapse into the main action bar if you remove the icon and title leaving only tabs at the top and all the actions on the bottom with:

```java
public class MainActivity extends AppCompatActivity {
	@Override
	protected void onCreate(Bundle savedInstanceState) {
	    super.onCreate(savedInstanceState);
	    setContentView(R.layout.activity_main);
	    setupTabs();
	}

	private void setupTabs() {
	    ActionBar actionBar = getSupportActionBar();
	    actionBar.setNavigationMode(ActionBar.NAVIGATION_MODE_TABS);
	    actionBar.setDisplayShowHomeEnabled(false);
	    actionBar.setDisplayShowTitleEnabled(false);
	}
}
```

### Custom ActionBar Styles

We can tweak the ActionBar styles and properties by customizing our own ActionBar theme styles. For example, we could add the following to `res/values/styles.xml`:

```xml
<resources>

    <!-- Define your colors in `res/values/colors.xml` -->
    <color name="simple_yellow">#ECD078</color>
    <color name="primary_blue">#53777A</color>

    <style name="AppTheme" parent="Theme.AppCompat.Light.DarkActionBar">
        <item name="android:actionBarStyle">@style/MyActionBar</item>
        <item name="android:actionBarTabTextStyle">@style/MyActionBarTabText</item>

        <!-- Support library compatibility -->
        <item name="actionBarStyle">@style/MyActionBar</item>
        <item name="actionBarTabTextStyle">@style/MyActionBarTabText</item>
    </style>

    <style name="MyActionBar" parent="@style/Widget.AppCompat.Light.ActionBar.Solid.Inverse">
        <item name="android:background">@color/simple_yellow</item>
        <item name="android:titleTextStyle">@style/MyActionBar.TitleTextStyle</item>

        <!-- Support library compatibility -->
        <item name="background">@color/simple_yellow</item>
        <item name="titleTextStyle">@style/MyActionBar.TitleTextStyle</item>
    </style>

    <style name="MyActionBar.TitleTextStyle"
        parent="@style/TextAppearance.AppCompat.Widget.ActionBar.Title">
        <item name="android:textColor">@color/primary_blue</item>
    </style>

    <style name="MyActionBarTabText" parent="@style/Widget.AppCompat.ActionBar.TabText">
        <item name="android:textColor">@color/primary_blue</item>
    </style>

</resources>
```

Now verify the theme for the application or activity within the `AndroidManifest.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.codepath.example.servicesdemo"
    ...>
    <application
        ...
        android:theme="@style/AppTheme" >
</manifest>
```

Now your properties and styles will take affect within the ActionBar. See the [actionbar styling demo code](https://github.com/codepath/android-actionbar-style-demo) for a working example. If you want to style the tabs for the ActionBar, see our [[Tabs Styling Cliffnotes|ActionBar-Tabs-with-Fragments#styling-tabs]]. 

Check out this [styling the ActionBar](http://developer.android.com/guide/topics/ui/actionbar.html#Style) section for more details. For an easier way to skin the ActionBar, check out the [ActionBar Style Generator](http://jgilfelt.github.com/android-actionbarstylegenerator) tool for easy styling.

### Custom ActionBar Layout

In certain cases, you might want to change the styling of the ActionBar title more significantly. For example, you may want to tweak the icon, change the size of the title, tweak the color, or center the text. In order to achieve this, you can replace the default title with your own custom XML view. First, define your custom ActionBar XML in `res/layout/actionbar_title.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical" >
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center"
        android:text="@string/app_title"
        android:textColor="#ffffff"
        android:id="@+id/mytext"
        android:textSize="18dp" />
</LinearLayout>
```

Now we've defined the XML layout desired and we need to load this custom XML file and replace the ActionBar title with our customized XML inside the Activity by calling [setCustomView](http://developer.android.com/reference/android/app/ActionBar.html#setCustomView\(int\)):

```java
// in onCreate
getSupportActionBar().setDisplayOptions(ActionBar.DISPLAY_SHOW_CUSTOM); 
getSupportActionBar().setCustomView(R.layout.actionbar_title);
```

At this point, we now have replaced the default ActionBar with our preferred layout and have complete control over it's appearance. If you want to **include the app icon with the custom layout** you need to append `DISPLAY_SHOW_HOME` as well:

```java
getSupportActionBar().setDisplayOptions(ActionBar.DISPLAY_SHOW_CUSTOM | ActionBar.DISPLAY_SHOW_HOME); 
```

Note you can **still define an `onCreateOptionsMenu` method** in your Activity to define the action buttons.  This custom view will then share space with the action buttons, which normally are placed to the right side of the Action Bar.

### Adding ActionView Items

Unlike regular menu items, an action view is a widget that appears in the action bar as a substitute for an action button. ActionView is a full fledged 
view that is constructed via a layout XML file which is embedded into the ActionBar. First, you need to create the XML layout that will be embedded in the ActionBar in `res/layout/action_view_button.xml`:

```xml
<!-- res/layout/action_view_button.xml -->
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="horizontal" >
    <Button
        android:id="@+id/btnCustomAction"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Button" />
</LinearLayout>
```

Next, we can attach that layout to any item by specifying the `app:action_layout` property:

```xml
<menu xmlns:android="http://schemas.android.com/apk/res/android" >
    <item
        android:id="@+id/miActionButton"
        app:showAsAction="ifRoom"
        app:actionLayout="@layout/action_view_button" 
        android:title="Loading..." />
</menu>
```

and now the views specified in the layout are embedded in the ActionBar. We can access a reference to the embedded ActionView in the Activity by overriding the `onPrepareOptionsMenu` method:

```java
@Override
public boolean onPrepareOptionsMenu(Menu menu) {
	MenuItem actionViewItem = menu.findItem(R.id.miActionButton);
        // Retrieve the action-view from menu
	View v = MenuItemCompat.getActionView(actionViewItem);
        // Find the button within action-view
	Button b = (Button) v.findViewById(R.id.btnCustomAction);
	// Handle button click here
	return super.onPrepareOptionsMenu(menu);
}
```

Using `ActionView` can help you add any custom views you'd like to your ActionBar.

### Adding SearchView to ActionBar

One common example of an ActionView is the built-in `SearchView` which provides a simple search control within your application located in the ActionBar. First, we need to add the `SearchView` action item in the menu xml:

```xml
<!-- res/menu/menu.xml -->
<menu xmlns:android="http://schemas.android.com/apk/res/android" xmlns:app="http://schemas.android.com/apk/res-auto" >
   <item android:id="@+id/action_search"
          android:orderInCategory="5"
          android:title="Search"
          android:icon="@android:drawable/ic_menu_search"
          app:showAsAction="ifRoom|collapseActionView"
          app:actionViewClass="android.support.v7.widget.SearchView" />
</menu>
```

Notice that the showAsAction attribute also includes the "collapseActionView" value which collapses the search into an icon until clicked. Now we need to hook up a listener for when a search is performed:

```java
@Override
public boolean onCreateOptionsMenu(Menu menu) {
    MenuInflater inflater = getMenuInflater();
    inflater.inflate(R.menu.main, menu);
    MenuItem searchItem = menu.findItem(R.id.action_search);
    SearchView searchView = (SearchView) MenuItemCompat.getActionView(searchItem);
    searchView.setOnQueryTextListener(new OnQueryTextListener() {
       @Override
       public boolean onQueryTextSubmit(String query) {
            // perform query here
            return true;
       }

       @Override
       public boolean onQueryTextChange(String newText) {
           return false;
       }
   });
   return super.onCreateOptionsMenu(menu);
}
```

See [this writeup](http://antonioleiva.com/actionbarcompat-action-views/) for details on the compatibility action views outlined above. For more advanced searching functionality, check out the [Creating a Search Interface](http://developer.android.com/guide/topics/search/search-dialog.html) guide.

See [this tutorial](http://ramannanda.blogspot.com/2014/10/android-searchview-integration-with.html) for a guide on how to add autocomplete to your searchview in the actionbar.

### Using ActionProvider and ShareActionProvider

Similar to an action view, an action provider replaces an action button with a customized layout. However, unlike an action view, an action provider takes control of all the action's behaviors and an action provider can display a submenu when pressed.

You can build your own action provider by extending the ActionProvider class, but Android provides some pre-built action providers such as [ShareActionProvider](http://developer.android.com/reference/android/support/v7/widget/ShareActionProvider.html) which facilitates a "share" action by showing a list of possible apps for sharing. 

You can learn about this provider in the [[Sharing Content with Intents]] guide. You can also see the [ActionProvider section](http://developer.android.com/guide/topics/ui/actionbar.html#ActionProvider) of the ActionBar guide for more details.

### Navigating Up with the App Icon

To enable the app icon as an Up button, call [setDisplayHomeAsUpEnabled](http://developer.android.com/reference/android/support/v7/app/ActionBar.html#setDisplayHomeAsUpEnabled(boolean)). "Up" in contrast to  Back" takes the user to the logical parent screen of the current screen. This is not based on the navigation history but rather on the relationship between screens. For example, in a mail client "Back" might take the user to a previous email but "Up" would always take the user to the list of mail in the inbox.

First, specify that the home icon should be used as "Up":

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_details);

    ActionBar actionBar = getSupportActionBar();
    actionBar.setDisplayHomeAsUpEnabled(true);
    ...
}
```

#### Compile-time Configuration

To specify the "up" activity at compile-time we can set the logical parent of an activity in the `AndroidManifest.xml`:

```xml
<activity 
        android:name="com.example.myfirstapp.ChildActivity"
        android:label="@string/title_activity_display_message"
        android:parentActivityName="com.example.myfirstapp.ParentActivity" >
        <!-- Parent activity meta-data to support API level 7+ -->
        <meta-data
            android:name="android.support.PARENT_ACTIVITY"
            android:value="com.example.myfirstapp.ParentActivity" />
</activity>
```

And now when the home icon is pressed on the child, the parent activity will always be shown. 

#### Run-time Configuration

If we want to set the "up" button dynamically rather than in the manifest, we can override the `getSupportParentActivityIntent()` method on the activity returning the desired intent based on the argument passed in:

```java
public static final String PACKAGE_NAME = "com.myapplication.";
public static final String PARENT_NAME_EXTRA = "ParentClassName";

@Override
public Intent getSupportParentActivityIntent() {
    // Extract the class name of our parent
    Intent parentIntent = getIntent();
    String className = parentIntent.getStringExtra(PARENT_NAME_EXTRA);
    // Create intent based on the parent class name
    Intent newIntent = null;
    try {
         //you need to define the class with package name
         newIntent = new Intent(this, Class.forName(PACKAGE_NAME + className));
    } catch (ClassNotFoundException e) {
        e.printStackTrace();
    }
    // Return the created intent as the "up" activity
    return newIntent;
}
```

Then when launching a new activity, we need to supply the `ParentClassName` as an extra to control the desired parent:

```java
Intent intent = new Intent(this, ChildActivity.class);
intent.putExtra(ChildActivity.PARENT_NAME_EXTRA, "ParentActivity");
startActivity(intent);
```

Read more about ["Up" navigation](http://developer.android.com/training/implementing-navigation/ancestral.html) within the official guide.

## Libraries

* [AppCompatActivity](https://developer.android.com/reference/android/support/v7/app/AppCompatActivity.html) - The support library for ActionBar which provides backwards compatibility to nearly all Android versions.
* [StyleGenerator](http://jgilfelt.github.com/android-actionbarstylegenerator) - Use this nifty web tool to generate an ActionBar theme easily.

## References

 * <http://antonioleiva.com/actionbarcompat-how-to-use/>
 * <http://developer.android.com/guide/topics/ui/actionbar.html>
 * <http://developer.android.com/design/patterns/actionbar.html>
 * <http://developer.android.com/reference/android/app/ActionBar.html>
 * <http://www.vogella.com/articles/AndroidActionBar/article.html>
 * <http://jgilfelt.github.io/android-actionbarstylegenerator/>
 * <http://blog.stylingandroid.com/archives/1144>
 * <http://nikolakirev.com/android-tutorial-adding-buttons-to-the-action-bar-guest-post/>
 * <http://writecodeeasy.blogspot.com/2012/07/androidtutorial-actionbar.html>
 * <http://www.grokkingandroid.com/adding-actionviews-to-your-actionbar/>
 * <http://www.intertech.com/Blog/android-action-bar-from-the-options-menu/>
 * <http://android-developers.blogspot.com/2011/04/customizing-action-bar.html>