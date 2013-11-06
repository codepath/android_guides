## Overview

An ActionBar is located at the top of an activity and it can display any number of status or navigation related elements such as title, icon, buttons, or arbitrary action-related views.

Older Android devices have a hardware Option button which use to open a menu but the ActionBar replaces that concept in modern Android apps.

This is typically used for displaying the title of the application and providing a primary navigation for the app. The ActionBar can contain primary action buttons as well as tabs to control navigation within any app.

**Note** that we will be using ActionBarSherlock for these examples since the library provides maximum compatibility with pre-3.0 Android versions. The APIs and usage are the same with the standard ActionBar just with "Sherlock" prepended to the front of each class name.

## Usage

In the [Defining ActionBar](https://github.com/thecodepath/android_guides/wiki/Defining-The-ActionBar#actionbar-basics) cliffnotes we looked at the basics of adding items to the ActionBar and handling clicks. In this section, we take a look at how to use `ActionBarSherlock` to support all Android versions and also at several powerful and extensible ActionBar features:

 * Using the split action bar to have a top and bottom menu
 * Adding an Action View (action_layout) and SearchView
 * Using ActionProvider and ShareActionProvider to enable richer functions
 * Configuring Home Icon to navigate "Up"

### Setting up ActionBarSherlock

In order to ensure that the ActionBar works on all Android versions, we are going to use [ActionBarSherlock](<http://actionbarsherlock.com/usage.html>) to setup our ActionBar.

First, [download](http://actionbarsherlock.com/download.html) the library and then set it up according to the [usage guide](http://actionbarsherlock.com/usage.html). 

Once it's setup, you should make sure any applicable activities are now extending from `SherlockFragmentActivity` in order to enable
the ActionBarSherlock compatibility fragments and action bar:

```java
public class MainActivity extends SherlockFragmentActivity {
   // ...
}
```

and change the theme to a sherlock compatible theme in the manifest:

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest …>
    <application
        android:allowBackup="true"
        android:icon="@drawable/ic_launcher"
        android:label="@string/app_name"
        android:theme="@style/Theme.Sherlock.Light.DarkActionBar" >
        <!-- … !>
    </application>
</manifest>
```

Now ActionBarSherlock is configured and ready to be used.

### Populating Action Buttons

The action bar provides users access to the most important action items relating to the app's current context. Those that appear directly in the action bar with an icon and/or text are known as action buttons. 

For ActionBarSherlock, just change `onCreateOptionsMenu` to use the support inflater:

```java
@Override
public boolean onCreateOptionsMenu(Menu menu) {
	MenuInflater inflater = getSupportMenuInflater();
	inflater.inflate(R.menu.main, menu);
	return super.onCreateOptionsMenu(menu);
}
```

See the [Defining ActionBar](https://github.com/thecodepath/android_guides/wiki/Defining-The-ActionBar#actionbar-basics) cliffnotes
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
   android:showAsAction="ifRoom"
   android:title="Ordinary"/>

<item
   android:id="@+id/menu_important"
   android:orderInCategory="20"
   android:showAsAction="ifRoom"
   android:title="Important"/>
```

Using split action bar also allows navigation tabs to collapse into the main action bar if you remove the icon and title leaving only tabs at the top and all the actions on the bottom with:

```java
public class MainActivity extends SherlockFragmentActivity {
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
```

### Custom ActionBar Layout

In certain cases, you might want to change the styling of the ActionBar title. For example, you may want to remove the icon, change the size of the title, tweak the color, or center the text. In order to achieve this, you need to replace the title with your own custom XML view. First, define your custom ActionBar XML in `res/layout/actionbar_title.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="#000000"
    android:orientation="vertical" >
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center"
        android:text="@string/app_title"
        android:textColor="#ffffff"
        android:id="@+id/mytext"
        android:textSize="25dp" />
</LinearLayout>
```

Now we've defined the XML layout desired and we need to load this custom XML file and replace the ActionBar title with our customized XML inside the Activity by calling [setCustomView](http://developer.android.com/reference/android/app/ActionBar.html#setCustomView(int\)):

```java
// in onCreate
getSupportActionBar().setDisplayOptions(ActionBar.DISPLAY_SHOW_CUSTOM); 
getSupportActionBar().setCustomView(R.layout.actionbar_title);
```

At this point, we now have replaced the default ActionBar with our preferred layout and have complete control over it's appearance. If you want to **include the app icon with the custom layout**:

```java
getActionBar().setDisplayOptions(ActionBar.DISPLAY_SHOW_CUSTOM | ActionBar.DISPLAY_SHOW_HOME); 
```

Note that this can also help ensure that the tabs appear below the title when using `ActionBarSherlock` caused by [a bug with the ActionBar](https://github.com/JakeWharton/ActionBarSherlock/issues/327).

### Custom ActionBar Styles

In addition, to the ability to change the ActionBar layout, we can also tweak the ActionBar styles and properties by customizing our own theme. For example, we could add the following to `res/values/styles.xml`:

```xml
<resources>
    <style name="MyCustomTheme" parent="@android:style/Theme.Holo.Light">
        <item name="android:actionBarStyle">@style/MyActionBar</item>
    </style>

    <style name="MyActionBar" parent="@android:style/Widget.Holo.Light.ActionBar">
        <item name="android:background">#ffffff</item>
        <item name="android:textColor">#000000</item>
    </style>
</resources>
```

Now tweak the theme for the application or activity within the `AndroidManifest.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.codepath.example.servicesdemo"
    ...>
    <application
        ...
        android:theme="@style/MyCustomTheme" >
</manifest>
```

Now your properties and styles will take affect within the ActionBar. For an easier way to skin the ActionBar, check out the [StyleGenerator](http://jgilfelt.github.com/android-actionbarstylegenerator) tool for easy style tweaking.

### Adding ActionView Items

Unlike regular menu items, an action view is a widget that appears in the action bar as a substitute for an action button. ActionView is a full fledged 
view that is constructed via a layout XML file which is embedded into the ActionBar. First, you need to create the XML layout that will be embedded
in the ActionBar:

```xml
<!-- res/layout/action_view_button -->
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="horizontal" >
    <Button
        android:id="@+id/button1"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Button" />
</LinearLayout>
```

Next, we can attach that layout to any item by specifying the `android:action_layout` property:

```java
<menu xmlns:android="http://schemas.android.com/apk/res/android" >
    <item
        android:id="@+id/s"
        android:orderInCategory="40"
        android:showAsAction="ifRoom"
        android:actionLayout="@layout/test"
</menu>
```

and now the views specified in the layout are embedded in the ActionBar. We can access the embedded ActionView in the Activity by overrding the `onPrepareOptionsMenu` method:

```java
@Override
public boolean onPrepareOptionsMenu(Menu menu) {
	MenuItem actionViewItem = menu.findItem(R.id.menu_second);
	View v = actionViewItem.getActionView();
	Button b = (Button) v.findViewById(R.id.button1);
	// Handle button click here
	return super.onPrepareOptionsMenu(menu);
}
```

Using ActionView can help you add anything to your ActionBar.

### Adding SearchView to ActionBar

One common example of an ActionView is the built-in `SearchView` which provides a simple search control within your application located in the ActionBar. First, we need to add the `SearchView` action item in the menu xml:

```xml
<!-- res/menu/menu.xml -->
<menu xmlns:android="http://schemas.android.com/apk/res/android" >
   <item android:id="@+id/action_search"
          android:orderInCategory="5"
          android:title="Search"
          android:icon="@drawable/ic_action_search"
          android:showAsAction="ifRoom|collapseActionView"
          android:actionViewClass="com.actionbarsherlock.widget.SearchView" />
</menu>
```

Notice that the showAsAction attribute also includes the "collapseActionView" value which collapses the search into an icon until clicked. Now we need to hook up a listener for when a search is performed:

```java
@Override
public boolean onCreateOptionsMenu(Menu menu) {
    MenuInflater inflater = getSupportMenuInflater();
    inflater.inflate(R.menu.main, menu);
    MenuItem searchItem = menu.findItem(R.id.action_search);
    searchView = (SearchView) searchItem.getActionView();
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

For more advanced searching functionality, check out the [Creating a Search Interface](http://developer.android.com/guide/topics/search/search-dialog.html) guide.

### Using ActionProvider and ShareActionProvider

Similar to an action view, an action provider replaces an action button with a customized layout. However, unlike an action view, an action provider takes control of all the action's behaviors and an action provider can display a submenu when pressed.

You can build your own action provider by extending the ActionProvider class, but Android provides some pre-built action providers such as [ShareActionProvider](http://developer.android.com/reference/android/support/v7/widget/ShareActionProvider.html) which facilitates a "share" action by showing a list of possible apps for sharing.

See the [ActionProvider section](http://developer.android.com/guide/topics/ui/actionbar.html#ActionProvider) of the ActionBar guide for more details.

### Navigating Up with the App Icon

To enable the app icon as an Up button, call [setDisplayHomeAsUpEnabled](http://developer.android.com/reference/android/support/v7/app/ActionBar.html#setDisplayHomeAsUpEnabled(boolean)). "Up" in contrast to  Back" takes the user to the logical parent screen of the current screen. This is not based on the navigation history but rather on the relationship between screens. For example, in a mail client "Back" might take the user to a previous email but 
"Up" would always take the user to the list of mail in the inbox.

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

Next, specify the logical parent of an activity in the manifest:

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

And now when the home icon is pressed on the child, the parent activity will always be shown. Read more about ["Up" navigation](http://developer.android.com/training/implementing-navigation/ancestral.html) on the official guide.

## Libraries

* [ActionBarSherlock](<http://actionbarsherlock.com/usage.html>) - An essential extension to ActionBar which supports almost all Android versions.
* [StyleGenerator](http://jgilfelt.github.com/android-actionbarstylegenerator) - Use this nifty web tool to generate an ActionBar theme easily.

## References

 * <http://developer.android.com/guide/topics/ui/actionbar.html>
 * <http://developer.android.com/design/patterns/actionbar.html>
 * <http://actionbarsherlock.com/usage.html>
 * <http://developer.android.com/reference/android/app/ActionBar.html>
 * <http://www.vogella.com/articles/AndroidActionBar/article.html>
 * <http://jgilfelt.github.io/android-actionbarstylegenerator/>
 * <http://blog.stylingandroid.com/archives/1144>
 * <http://nikolakirev.com/android-tutorial-adding-buttons-to-the-action-bar-guest-post/>
 * <http://writecodeeasy.blogspot.com/2012/07/androidtutorial-actionbar.html>
 * <http://www.grokkingandroid.com/adding-actionviews-to-your-actionbar/>
 * <http://www.intertech.com/Blog/android-action-bar-from-the-options-menu/>
 * <http://android-developers.blogspot.com/2011/04/customizing-action-bar.html>