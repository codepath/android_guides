## Overview

An ActionBar is located at the top of an activity and it can display any number of status or navigation related elements such as title, icon, buttons, or arbitrary action-related views.

This is typically used for displaying the title of the application and providing a primary navigation for the app. The ActionBar can contain primary action buttons as well as a drawer toggle icon for displaying the navigation drawer.

<img src="https://i.imgur.com/EA42s2Q.png" width="500" />

**Note** that we will be using the support library and `AppCompatActivity` for these examples since the library provides maximum compatibility with pre-3.0 Android versions. The APIs and usage are the same with the standard ActionBar just with small changes to the imported classes and class names.  If you are not currently using the support library, check out this [[migration guide|Migrating to the AppCompat Library]].

## Usage

In the [[Defining ActionBar|Defining-The-ActionBar#actionbar-basics]] cliffnotes we looked at the basics of adding items to the ActionBar and handling clicks. In this section, we take a look at how to use `AppCompatActivity` to support all Android versions and also at several powerful and extensible ActionBar features:

 * Using the split action bar to have a top and bottom menu
 * Adding ActionView (`app:action_layout`) and `SearchView` widgets
 * Configuring icon order within ActionBar
 * Using `ActionProvider` and `ShareActionProvider` to enable richer functions
 * Configuring Home Icon to navigate "Up"
 * Inflating menu icons from fragments within the ActionBar

### Setting up AppCompatActivity

In order to ensure that the ActionBar works on all Android versions, we are going to use [AppCompatActivity](https://developer.android.com/reference/android/support/v7/app/AppCompatActivity.html) to setup our support ActionBar.  Follow the [[AppCompat setup guide|Migrating-to-the-AppCompat-Library]] to make sure you're including the library.  

Once the library has been added, be sure to sync your project with the gradle file (`Tools => Android => Sync Project with Gradle Files`) and make sure any applicable activities are now extending from `AppCompatActivity` in order to enable
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

### Configuring ActionBar Icon Order

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

This can be useful to be explicit with the desired order based on importance of actions.

### Custom ActionBar Styles

We can configure the ActionBar styles and properties by creating our own ActionBar theme styles. For example, we could add the following to `res/values/styles.xml`:

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

Now your properties and styles will take affect within the ActionBar. This results in the following:

<img src="https://i.imgur.com/B7OBUhO.png" width="500" />

See the [actionbar styling demo code](https://github.com/codepath/android-actionbar-style-demo) for a working example. If you want to style the tabs for the ActionBar, see our [[Tabs Styling Cliffnotes|ActionBar-Tabs-with-Fragments#styling-tabs]]. 

Check out this [styling the ActionBar](http://developer.android.com/guide/topics/ui/actionbar.html#Style) section for more details. For an easier way to skin the ActionBar, check out the [ActionBar Style Generator](http://jgilfelt.github.com/android-actionbarstylegenerator) tool for easy styling.

### Custom ActionBar Layout

In certain cases, you might want to change the styling of the ActionBar title more significantly. For example, you may want to tweak the icon, change the size of the title, tweak the color, or center the text. In order to achieve this, you can replace the default title with your own custom XML view. First, define your custom ActionBar XML in `res/layout/actionbar_title.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:gravity="center"
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
import android.support.v7.app.ActionBar;
// in Activity#onCreate
getSupportActionBar().setDisplayOptions(ActionBar.DISPLAY_SHOW_CUSTOM); 
getSupportActionBar().setCustomView(R.layout.actionbar_title);
```

At this point, we now have replaced the default ActionBar with our preferred layout and have complete control over its appearance. The above code results in:

<img src="https://i.imgur.com/N5Bvoej.png" width="500" />


If you want to **include the app icon with the custom layout** you need to append `DISPLAY_SHOW_HOME` as well:

```java
getSupportActionBar().setDisplayOptions(ActionBar.DISPLAY_SHOW_CUSTOM | ActionBar.DISPLAY_SHOW_HOME); 
```

Note you can **still define an `onCreateOptionsMenu` method** in your Activity to define the action buttons.  This custom view will then share space with the action buttons, which normally are placed to the right side of the Action Bar.

### Adding ActionView Items

If you want to provide a menu item beyond simply icon or text, such as providing a more interactive widget, an Action View enables you to do so.  The most common Action View is the `SearchView`, which collapses to show only the search icon and expands to show an `EditText` when the user has clicked an icon. You can also use an Action View to create a custom layout too.

An ActionView is a full fledged view that is constructed via a layout XML file which is embedded into the ActionBar. First, you need to create the XML layout that will be embedded in the ActionBar in `res/layout/action_view_button.xml`:

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
<menu xmlns:android="http://schemas.android.com/apk/res/android" xmlns:app="http://schemas.android.com/apk/res-auto">
    <item
        android:id="@+id/miActionButton"
        app:showAsAction="ifRoom"
        app:actionLayout="@layout/action_view_button" 
        android:title="Loading..." />
</menu>
```

and now the views specified in the layout are embedded in the ActionBar. The above code results in:

<img src="https://i.imgur.com/YhuRH56.png" width="500" />

We can access a reference to the embedded ActionView in the Activity by overriding the `onPrepareOptionsMenu` method:

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

One common example of an `ActionView` is the built-in `SearchView` which provides a simple search control within your application located in the ActionBar. First, we need to add the `SearchView` action item in the menu xml:

```xml
<!-- res/menu/menu.xml -->
<menu xmlns:android="http://schemas.android.com/apk/res/android" xmlns:app="http://schemas.android.com/apk/res-auto" >
   <item android:id="@+id/action_search"
          android:orderInCategory="5"
          android:title="Search"
          android:icon="@android:drawable/ic_menu_search"
          app:showAsAction="always|collapseActionView"
          app:actionViewClass="android.support.v7.widget.SearchView" />
</menu>
```

Notice that the `app:showAsAction` attribute also includes the "collapseActionView" value which collapses the search into an icon until clicked. The above code results in the search view in the action bar:
 
<img src="https://i.imgur.com/zW9vfCf.gif" width="500" />

Now we need to hook up a listener for when a search is performed:

```java
// Make sure to import the support version of the SearchView
import android.support.v7.widget.SearchView;

@Override
public boolean onCreateOptionsMenu(Menu menu) {
    MenuInflater inflater = getMenuInflater();
    inflater.inflate(R.menu.main, menu);
    MenuItem searchItem = menu.findItem(R.id.action_search);
    final SearchView searchView = (SearchView) MenuItemCompat.getActionView(searchItem);
    searchView.setOnQueryTextListener(new OnQueryTextListener() {
       @Override
       public boolean onQueryTextSubmit(String query) {
            // perform query here
      
            // workaround to avoid issues with some emulators and keyboard devices firing twice if a keyboard enter is used
            // see https://code.google.com/p/android/issues/detail?id=24599
            searchView.clearFocus();

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

We can expand the `SearchView` any time programmatically by calling `expandActionView` on the search view: 

```java
@Override
public boolean onCreateOptionsMenu(Menu menu) {
    // ... lookup the search view
    MenuItem searchItem = menu.findItem(R.id.action_search);
    final SearchView searchView = (SearchView) MenuItemCompat.getActionView(searchItem);
    // Expand the search view and request focus 
    searchItem.expandActionView();
    searchView.requestFocus();
}
```

#### Customizing or Extending SearchView

We can customize the search icon and text color with [this approach with styles](http://stackoverflow.com/a/26897024) or the Java code below:

```java
@Override
public boolean onCreateOptionsMenu(Menu menu) {
    // ... lookup the search view
    MenuItem searchItem = menu.findItem(R.id.action_search);
    final SearchView searchView = (SearchView) MenuItemCompat.getActionView(searchItem);
    // Use a custom search icon for the SearchView in AppBar
    int searchImgId = android.support.v7.appcompat.R.id.search_button; 
    ImageView v = (ImageView) searchView.findViewById(searchImgId);
    v.setImageResource(R.drawable.search_btn);
    // Customize searchview text and hint colors
    int searchEditId = android.support.v7.appcompat.R.id.search_src_text;
    EditText et = (EditText) searchView.findViewById(searchEditId);
    et.setTextColor(Color.BLACK);
    et.setHintTextColor(Color.BLACK);
}
```

We can customize the up button that appears when the search view is activated with [this approach](http://stackoverflow.com/a/35482445/1715285) or the code below:

Use the `app:collapseIcon` to modify the color/style of your drawable.

```xml
<android.support.v7.widget.Toolbar
         android:id="@+id/toolbar"
         android:layout_width="match_parent"
         android:layout_height="@dimen/toolbarHeight"
         app:collapseIcon="@drawable/collapseBackIcon" />
```

See [this writeup](http://antonioleiva.com/actionbarcompat-action-views/) for details on the compatibility action views outlined above. For more advanced searching functionality, check out the [Creating a Search Interface](http://developer.android.com/guide/topics/search/search-dialog.html) guide.

Refer to [this tutorial](http://ramannanda.blogspot.com/2014/10/android-searchview-integration-with.html) for a guide on how to **add autocomplete to your searchview** in the actionbar.

### Using ActionProvider and ShareActionProvider

Similar to an action view, an action provider replaces an action button with a customized layout. However, unlike an action view, an action provider takes control of all the action's behaviors and an action provider can display a submenu when pressed.

You can build your own action provider by extending the ActionProvider class, but Android provides some pre-built action providers such as [ShareActionProvider](http://developer.android.com/reference/android/support/v7/widget/ShareActionProvider.html) which facilitates a "share" action by showing a list of possible apps for sharing. 

<img src="https://i.imgur.com/MLuaXES.png" />

You can learn about this provider in the [[Sharing Content with Intents]] guide. You can also see the [ActionProvider section](http://developer.android.com/guide/topics/ui/actionbar.html#ActionProvider) of the ActionBar guide for more details.

### Navigating Up with the App Icon

To enable the app icon as an Up button, call [setDisplayHomeAsUpEnabled](http://developer.android.com/reference/android/support/v7/app/ActionBar.html#setDisplayHomeAsUpEnabled(boolean)). 

![Up Button](https://i.imgur.com/yEXdmG2.png)

"Up" in contrast to the Back button takes the user to the logical parent screen of the current screen. This is not based on the navigation history but rather on the relationship between screens. For example, in a mail client "Back" might take the user to a previous email but "Up" would always take the user to the list of mail in the inbox.

First, specify that the home icon should be used as "Up":

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_details);
    // Enable up icon
    getSupportActionBar().setDisplayHomeAsUpEnabled(true);
    ...
}
```

We can also explicitly override the "Up" button in the AppBar in the `onOptionsItemSelected` by checking for the  `android.R.id.home` id being selected:

```java
    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        switch (item.getItemId()) {
        // This is the up button
        case android.R.id.home:
            NavUtils.navigateUpFromSameTask(this);
            // overridePendingTransition(R.animator.anim_left, R.animator.anim_right);
            return true;
        default:
            return super.onOptionsItemSelected(item);
        }
    }
```

This allows us to configure the transition or otherwise handle behavior when the "up" button is pressed.

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

If you want to navigate up from current activity to its parent, but want that parent activity to preserve its state, then also specify the <singleTop> launch mode for the parent activity in `AndroidManifest.xml`. The existing parent activity instance will be re-used, receiving the intent through `onNewIntent()`.
```xml
android:launchMode="singleTop"
```

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

### Configuring the ActionBar from a Fragment

Configuring the action bar from a fragment is very similar to how it's done from an activity, with a couple small differences. By default, Android assumes that fragments don't want to contribute items to the action bar. When a fragment actually wants to add items to the action bar, it needs to tell Android about this by calling `setHasOptionsMenu(true)` in its `onCreate()` method:

```java
@Override
public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);

    // ...

    setHasOptionsMenu(true);
}
```

Now Android knows to call the fragment's `onCreateOptionsMenu(...)` and related methods. 

Keep in mind that any action bar items added by the fragment will be appended to any existing action bar items. This includes action bar items added by the containing activity. You can use the `orderInCategory` property of your action bar items to control the ordering yourself.

Adding items to the menu is very similar to how it's done in an activity. Inside a fragment, Android provides a `MenuInflater` for you in the callback:

```java
@Override
public void onCreateOptionsMenu(Menu menu, MenuInflater inflater) {
    // Inflate the menu; this adds items to the action bar.
    inflater.inflate(R.menu.my_menu, menu);
    
    // ...
}
```

Handling clicks is exactly the same as if you were in an activity. The only difference is that the fragment's `onOptionsItemSelected(...)` method only gets called if the activity's `onOptionsItemSelected(...)` doesn't handle the click.

```java
@Override
public boolean onOptionsItemSelected(MenuItem item) {
   // handle item selection
   switch (item.getItemId()) {
      case R.id.my_item:
         // Handle this selection
         return true;
      default:
         return super.onOptionsItemSelected(item);
   }
}
```

### Configuring a Split-Action Bar

Split action bar provides a separate bar at the bottom of the screen to display all action items when the activity is running on a narrow screen (such as a portrait-oriented handset). This is helpful when you want to display a top and bottom row of actions for a context.

For **Holo-based themes**, to add support for the split-action bar, just add the option to the manifest for that activity:

```xml
<manifest ...>
    <activity android:uiOptions="splitActionBarWhenNarrow" ... >
        <meta-data android:name="android.support.UI_OPTIONS"
                   android:value="splitActionBarWhenNarrow" />
    </activity>
</manifest>
```

**Note:** Split action bar **only works with Holo-based themes. `splitActionBarWhenNarrow` is **not supported** by `Theme.Material` or the `appcompat-v7` actionbar backport. If you wish to use either Theme.Material or appcompat-v7, you will **need to create your own** "split action bar", by having a [[Toolbar at the bottom|Using-the-App-ToolBar]] of the screen that you populate separately. See the [sample code for a split toolbar](https://github.com/commonsguy/cw-omnibus/tree/master/Toolbar/SplitActionBar2) here.

## Libraries

* [AppCompatActivity](https://developer.android.com/reference/android/support/v7/app/AppCompatActivity.html) - The support library for ActionBar which provides backwards compatibility to nearly all Android versions.
* [StyleGenerator](http://jgilfelt.github.com/android-actionbarstylegenerator) - Use this nifty web tool to generate an ActionBar theme easily.

## References

 * <http://antonioleiva.com/actionbarcompat-how-to-use/>
 * <http://developer.android.com/guide/topics/ui/actionbar.html>
 * <http://developer.android.com/design/patterns/actionbar.html>
 * <http://developer.android.com/reference/android/app/ActionBar.html>
 * <http://developer.android.com/guide/components/fragments.html>
 * <http://www.vogella.com/articles/AndroidActionBar/article.html>
 * <http://jgilfelt.github.io/android-actionbarstylegenerator/>
 * <http://blog.stylingandroid.com/archives/1144>
 * <http://nikolakirev.com/android-tutorial-adding-buttons-to-the-action-bar-guest-post/>
 * <http://writecodeeasy.blogspot.com/2012/07/androidtutorial-actionbar.html>
 * <http://www.grokkingandroid.com/adding-actionviews-to-your-actionbar/>
 * <http://www.intertech.com/Blog/android-action-bar-from-the-options-menu/>
 * <http://android-developers.blogspot.com/2011/04/customizing-action-bar.html>
