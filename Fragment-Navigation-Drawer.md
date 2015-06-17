In [[Common Navigation Paradigms]] cliffnotes, we discuss the various navigational structures available within Android apps. One of the most flexible is the [Navigation Drawer](http://developer.android.com/training/implementing-navigation/nav-drawer.html).  During the I/O Conference 2015, Google
released [NavigationView] (https://developer.android.com/reference/android/support/design/widget/NavigationView.html), which makes it far easier to create it than the [previously documented instructions](http://developer.android.com/training/implementing-navigation/nav-drawer.html#top).

With the release of Android 5.0 Lollipop, the new material design style navigation drawer spans the full height of the screen and is displayed over the `ActionBar` and overlaps the translucent `StatusBar`. Read the [material design style navigation drawer](http://www.google.com/design/spec/patterns/navigation-drawer.html) document for specs on styling your navigation drawer.

<img src="http://i.imgur.com/hPOFJUf.gif" width="350" />

## Usage

This guide explains how to setup a basic material design style drawer filled with navigation items that switch different fragments into the content area. In this way, you can define multiple fragments, and then define the list of options which will display in the drawers items list. Each item when clicked will switch the relevant fragment into the activity's container view.

### Setup

Make sure to setup the Google [[Design Support Library]] before using Google's new [NavigationView](https://developer.android.com/reference/android/support/design/widget/NavigationView.html), announced as part of the Android M release.  The NavigationView should be backwards compatible with all versions down to Android 2.1.

### Download Nav Drawer Item icons

Download the following icons and add them to your drawable folders by copying and pasting them into the drawable folder or using the `New Image Asset` dialog to create versions for each density.  

* [ic_one](http://i.imgur.com/PfXVA78.png)
* [ic_two](http://i.imgur.com/xzIdYlo.png)
* [ic_three](http://i.imgur.com/k5o1mCJ.png)

If you use the `New Image Asset` dialog, choose a black foreground color and change the resource name.

<img src="http://i.imgur.com/DE49F1R.png" width="600"/>

Download the menu icon from Google's official [Material Design icon set](https://github.com/google/material-design-icons) and use `New Image Asset` to create versions of each density too.  Save this file as `ic_menu.png`.

* [ic_menu_white_36dp.png](https://github.com/google/material-design-icons/blob/master/navigation/drawable-xxxhdpi/ic_menu_white_36dp.png)

### Setup Drawer Resources

Create a `menu/drawer_view.xml` file:

```xml
<menu xmlns:android="http://schemas.android.com/apk/res/android">

    <group android:checkableBehavior="single">
        <item
            android:id="@+id/nav_first_fragment"
            android:icon="@drawable/ic_one"
            android:title="First" />
        <item
            android:id="@+id/nav_second_fragment"
            android:icon="@drawable/ic_two"
            android:title="Second" />
        <item
            android:id="@+id/nav_third_fragment"
            android:icon="@drawable/ic_three"
            android:title="Third" />
    </group>
</menu>
```

Note that you can set one of these elements to be default selected by using `android:checked="true"`.  

You can also create subheaders too and group elements together:

```xml
 <item android:title="Sub items">
        <menu>
            <item
                android:icon="@drawable/ic_dashboard"
                android:title="Sub item 1" />
            <item
                android:icon="@drawable/ic_forum"
                android:title="Sub item 2" />
        </menu>
    </item>
```

<img src="http://imgur.com/zoDqDKM.png"/>

### Define Fragments

Next, you need to define your fragments that will be displayed within the drawer. These can be any support fragments you define within your application. Make sure that all the fragments extend from **android.support.v4.app.Fragment**.

### Setup Toolbar

In order to slide our navigation drawer over the ActionBar, we need to use the new [Toolbar](http://guides.codepath.com/android/Defining-The-ActionBar#toolbar-basics) widget as defined in the AppCompat v21 library. The `Toolbar` can be embedded into your view hierarchy which makes sure that the drawer slides over the `ActionBar`.

Create a new layout file `res/layout/toolbar.xml` with the following code:

```xml
<android.support.v7.widget.Toolbar
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/toolbar"
    android:layout_height="wrap_content"
    android:layout_width="match_parent"
    android:fitsSystemWindows="true"
    android:minHeight="?attr/actionBarSize"
    app:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar"
    android:background="?attr/colorPrimaryDark">
</android.support.v7.widget.Toolbar>
```

Note that when the `android:fitsSystemWindows` attribute is set to true for a view, the view would be laid out as if the `StatusBar` and the `ActionBar` were present i.e. the UI on top gets padding enough to not be obscured by the navigation bar.  Without this attribute, there is not enough padding factored into consideration for the `ToolBar`:

<img src="http://imgur.com/HaOAmoh.png"/>

We want our main content view to have the navigation bar and hence `android:fitsSystemWindows` is set to true for the `Toolbar`.

To use the `Toolbar` as an `ActionBar`, you need to disable the default `ActionBar`. This can be done by setting the app theme in `styles.xml` file.   

```xml
<resources>
    <!-- Base application theme. -->
    <style name="AppTheme" parent="Theme.AppCompat.Light.NoActionBar">
        <item name="colorPrimary">#673AB7</item>
        <item name="colorPrimaryDark">#512DA8</item>
        <item name="colorAccent">#FF4081</item>
    </style>
</resources>
```

Also note that normally you should decide on your color scheme by going to [Material Palette](http://www.materialpalette.com/) and choosing a primary and dark primary color. For this example, we will pick purple-based colors as shown in the screenshot.

*Note*: If you forget to disable the ActionBar in `styles.xml`, you are likely to see a `java.lang.IllegalStateException` with an error message that reads `This Activity already has an action bar supplied by the window decor. Do not request Window.FEATURE_ACTION_BAR and set windowActionBar to false in your theme to use a Toolbar instead.`  If you see this message, you need to make sure to follow the previous steps.

### Setup Drawer in Activity

Next, let's setup a basic navigation drawer based on the following layout file which has the entire drawer setup in `res/layout/activity_main.xml`. Note that the `Toolbar` is added as the first child of the main content view by adding the include tag.

```xml
<android.support.v4.widget.DrawerLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/drawer_layout"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical">

        <!-- The ActionBar -->
        <include
            layout="@layout/toolbar"
            android:layout_width="match_parent"
            android:layout_height="wrap_content" />

        <!-- The main content view -->
        <FrameLayout
            android:id="@+id/flContent"
            android:layout_width="match_parent"
            android:layout_height="match_parent" />
    </LinearLayout>

    <!-- The navigation drawer -->
    <android.support.design.widget.NavigationView
        android:id="@+id/nvView"
        android:layout_width="wrap_content"
        android:layout_height="match_parent"
        android:layout_gravity="start"
        android:background="@android:color/white"
        app:menu="@menu/drawer_view" />
</android.support.v4.widget.DrawerLayout>
```

Now, let's setup the drawer in our activity.  We can also setup the menu icon too.

```java
public class MainActivity extends AppCompatActivity {
	private DrawerLayout mDrawer;
	private Toolbar toolbar;

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);

		// Set a Toolbar to replace the ActionBar.
		toolbar = (Toolbar) findViewById(R.id.toolbar);
		setSupportActionBar(toolbar);

		// Find our drawer view
		mDrawer = (DrawerLayout) findViewById(R.id.drawer_layout);

		// Set the menu icon instead of the launcher icon.
		final ActionBar ab = getSupportActionBar();
		ab.setHomeAsUpIndicator(R.drawable.ic_menu);
	 	ab.setDisplayHomeAsUpEnabled(true);

	}

	@Override
	public boolean onCreateOptionsMenu(Menu menu) {
		MenuInflater inflater = getMenuInflater();
		// Uncomment to inflate menu items to Action Bar
		// inflater.inflate(R.menu.main, menu);
		return super.onCreateOptionsMenu(menu);
	}

	@Override
	public boolean onOptionsItemSelected(MenuItem item) {
		// The action bar home/up action should open or close the drawer.
		switch (item.getItemId()) {
			case android.R.id.home:
				mDrawer.openDrawer(GravityCompat.START);
				return true;
		}

		return super.onOptionsItemSelected(item);
	}

	@Override
	protected void onPostCreate(Bundle savedInstanceState) {
		super.onPostCreate(savedInstanceState);
	}
}
```

### Navigating between Menu Items

Setup a handler to respond to click events on the navigation elements and swap out the fragment.

```java

@Override
    protected void onCreate(Bundle savedInstanceState) {

         // Find our drawer view
        nvDrawer = (NavigationView) findViewById(R.id.nvView);
        // Setup drawer view
        setupDrawerContent(nvDrawer);
    }

    private void setupDrawerContent(NavigationView navigationView) {
        navigationView.setNavigationItemSelectedListener(
                new NavigationView.OnNavigationItemSelectedListener() {
                    @Override
                    public boolean onNavigationItemSelected(MenuItem menuItem) {
                        selectDrawerItem(menuItem);
                        return true;
                    }
                });
    }

    public void selectDrawerItem(MenuItem menuItem) {
        // Create a new fragment and specify the planet to show based on
        // position
        Fragment fragment = null;

        Class fragmentClass;
        switch(menuItem.getItemId()) {
            case R.id.nav_first_fragment:
                fragmentClass = FirstFragment.class;
                break;
            case R.id.nav_second_fragment:
                fragmentClass = SecondFragment.class;
                break;
            case R.id.nav_third_fragment:
                fragmentClass = ThirdFragment.class;
                break;
            default:
                fragmentClass = FirstFragment.class;
        }

        try {
            fragment = (Fragment) fragmentClass.newInstance();
        } catch (Exception e) {
            e.printStackTrace();
        }

        // Insert the fragment by replacing any existing fragment
        FragmentManager fragmentManager = getActivity().getSupportFragmentManager();
        fragmentManager.beginTransaction().replace(R.id.flContent, fragment).commit();

        // Highlight the selected item, update the title, and close the drawer
        menuItem.setChecked(true);
        setTitle(menuItem.getTitle());
        dlDrawer.closeDrawers();
    }
```

### Add Navigation Header

<img src="http://i.imgur.com/Ri3c6Xz.png"/>

The NavigationView also accepts a custom attribute that can reference a layout that provides a header of our layout.  For instance, you can create a `layout/nav_header.xml` similar to the following:

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="192dp"
    android:background="?attr/colorPrimaryDark"
    android:padding="16dp"
    android:theme="@style/ThemeOverlay.AppCompat.Dark"
    android:orientation="vertical"
    android:gravity="bottom">

    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Header"
        android:textColor="@android:color/white"
        android:textAppearance="@style/TextAppearance.AppCompat.Body1"/>

</LinearLayout>
```

You would then reference this layout in your `NavigationView` with the `app:headerLayout` custom attribute:

```xml

 <!-- The navigation drawer -->
    <android.support.design.widget.NavigationView

        app:headerLayout="@layout/nav_header">

    </android.support.design.widget.NavigationView>
```

## Animate the Hamburger Icon

<img src="http://imgur.com/ekmWl7q.gif"/>

In order for the hamburger icon to animate to indicate the drawer is being opened and closed, we need to use the [ActionBarDrawerToggle](https://developer.android.com/reference/android/support/v7/app/ActionBarDrawerToggle.html) class.

In your `res/values/strings.xml` add the following:

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string name="drawer_open">Open navigation drawer</string>
    <string name="drawer_close">Close navigation drawer</string>
</resources>
```

We need to tie the DrawerLayout and Toolbar together:

```java
  protected void onCreate(Bundle savedInstanceState) { 
	// Set a Toolbar to replace the ActionBar.
		toolbar = (Toolbar) findViewById(R.id.toolbar);
		setSupportActionBar(toolbar);

		// Find our drawer view
		dlDrawer = (DrawerLayout) findViewById(R.id.drawer_layout);
		drawerToggle = setupDrawerToggle();

		// Tie DrawerLayout events to the ActionBarToggle
		dlDrawer.setDrawerListener(drawerToggle);
   }

   private ActionBarDrawerToggle setupDrawerToggle() {
        return new ActionBarDrawerToggle(this, dlDrawer, toolbar, R.string.drawer_open,  R.string.drawer_close);
   }
```

Next, we need to make sure we synchronize the state whenever the screen is restored or there is a configuration change (i.e screen rotation):

```java
    @Override
    protected void onPostCreate(Bundle savedInstanceState) {
        super.onPostCreate(savedInstanceState);
        // Sync the toggle state after onRestoreInstanceState has occurred.
        drawerToggle.syncState();
    }

    @Override
    public void onConfigurationChanged(Configuration newConfig) {
        super.onConfigurationChanged(newConfig);
        // Pass any configuration change to the drawer toggles
        drawerToggle.onConfigurationChanged(newConfig);
    }
```

We also need to change the `onOptionsItemSelected()` method and allow the ActionBarToggle to handle the events.  
 
```java
    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        if (drawerToggle.onOptionsItemSelected(item)) {
            return true;
        }
        return super.onOptionsItemSelected(item);
    }
```

The ActionBarToggle will perform the same function done previously but adds a bit more checks and allows mouse clicks on the icon to open and close the drawer.  See the [source code](https://github.com/android/platform_frameworks_support/blob/master/v7/appcompat/src/android/support/v7/app/ActionBarDrawerToggle.java#L285-291) for more context.

One thing to note is that you no longer need to the `ic_menu.png` and the setHomeAsUpIndicator() call to set the menu.  The ActionBarDrawerToggle renders a custom [DrawerArrowDrawable](
https://github.com/android/platform_frameworks_support/blob/master/v7/appcompat/src/android/support/v7/app/DrawerArrowDrawable.java) for you.

## Making Status Bar Translucent

<img src="http://imgur.com/o4WvT3k.gif"/>

To have the status bar translucent and have our drawer slide over it, we need to set `android:windowTranslucentStatus` to true. Because this style is not available for pre Kitkat devices, we'll add  `res/values-v19/styles.xml` file for API version 19 and onwards.  **Note**: If you modify your `res/values/styles.xml` directly with this `android:windowTranslucentStatus` line, you are likely to need to build only for SDK versions 19 or higher, which will obviously limit you from supporting many older devices.

In `res/values-v19/styles.xml` we can add the following:

```xml
<resources>
  <!-- Base application theme. -->
  <style name="AppTheme" parent="Theme.AppCompat.Light.NoActionBar">
    <!-- Customize your theme here. -->
    <item name="android:windowTranslucentStatus">true</item>
  </style>
</resources>
```

Now if you run your app, you should see the navigation drawer and be able to select between your fragments.

## Limitations

The current version of the design support library does come with its limitations. The main issue is with the system that highlights the current item in the navigation menu. The itemBackground attribute for the NavigationView does not handle the checked state of the item correctly: somehow either all items are highlighted or none of them are. This makes this attribute basically unusable for most apps. While working with submenu's in the navigation items, once again the highlighting refused to work as expected: updating the selected item in a submenu makes the highlight overlay disappear altogether. In the end it seems that managing the selected item is still a chore that has to be solved manually in the app itself.

## Alternative to Fragments

Although many navigation drawer examples show how fragments can be used with the navigation drawer, you can also use a `RelativeLayout`/`LinearLayout` if you wish to use the drawer as an overlay to your currently displayed Activity.  

Instead of `<FrameLayout>` you can substitute that for a `<LinearLayout>`

```xml
<android.support.v4.widget.DrawerLayout
        xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:id="@+id/drawer_layout">

        <LinearLayout
                android:id="@+id/content_frame"
                android:orientation="horizontal"
                android:layout_width="match_parent"
                android:layout_height="match_parent"/>

        <!-- The navigation drawer -->
        <ListView android:id="@+id/left_drawer"
                  android:layout_width="240dp"
                  android:layout_height="wrap_content"
                  android:layout_gravity="start"
                  android:choiceMode="singleChoice"
                  android:divider="@android:color/transparent"
                  android:dividerHeight="0dp"
                  android:background="#111"/>

</android.support.v4.widget.DrawerLayout>
```

Instead of this:

```java
// Insert the fragment by replacing any existing fragment
getFragmentManager().beginTransaction()
       .replace(R.id.content_frame, fragment)
       .commit();
```

You can instead use the `LinearLayout` container to inflate the Activity directly:

```java
LayoutInflater inflater = getLayoutInflater();
LinearLayout container = (LinearLayout) findViewById(R.id.content_frame);
inflater.inflate(R.layout.activity_main, container);
```

## References

* <http://android-developers.blogspot.com/2014/10/appcompat-v21-material-design-for-pre.html>
* <http://stackoverflow.com/questions/26440879/how-do-i-use-drawerlayout-to-display-over-the-actionbar-toolbar-and-under-the-st>
* <http://antonioleiva.com/navigation-view/>