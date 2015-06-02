In [[Common Navigation Paradigms]] cliffnotes, we discuss the various navigational structures available within Android apps. One of the most flexible is the [Navigation Drawer](http://developer.android.com/training/implementing-navigation/nav-drawer.html). The custom navigation drawer is totally managed by the user and can be read about on the [creating the navigation drawer](http://developer.android.com/training/implementing-navigation/nav-drawer.html#top) docs.

With the release of Android 5.0 Lollipop, the new material design style navigation drawer spans the full height of the screen and is displayed over the `ActionBar` and overlaps the translucent `StatusBar`. Read the [material design style navigation drawer](http://www.google.com/design/spec/patterns/navigation-drawer.html) document for specs on styling your navigation drawer.

<img src="http://i.imgur.com/Ag3t4KC.gif" width="350" />

Screenshots:

![Navigation Drawer](http://i.imgur.com/8nCjuWrl.jpg)

## Usage

This guide explains how to setup a basic material design style drawer filled with navigation items that switch different fragments into the content area. In this way, you can define multiple fragments, and then define the list of options which will display in the drawers items list. Each item when clicked will switch the relevant fragment into the activity's container view.

### Setup

Make sure to setup the Google [[Support Design Library]] before using Google's new [NavigationView](https://developer.android.com/reference/android/support/design/widget/NavigationView.html), announced as part of the Android M release.  The NavigationView should be backwards compatible with all versions down to Android 2.1.

### Download Nav Drawer Item icons

Download the following icons and add them to your drawable folders by copying and pasting them into the drawable folder or using the `New Image Asset` dialog to create versions for each density.

* [ic_one](http://i.imgur.com/PfXVA78.png)
* [ic_two](http://i.imgur.com/xzIdYlo.png)
* [ic_three](http://i.imgur.com/k5o1mCJ.png)

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

In your `res/values/strings.xml` add the following:

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string name="drawer_open">Open navigation drawer</string>
    <string name="drawer_close">Close navigation drawer</string>
</resources>
```

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
    android:minHeight="?attr/actionBarSize"
    android:background="?attr/colorPrimaryDark">
</android.support.v7.widget.Toolbar>
```

To use the `Toolbar` as an `ActionBar`, you need to disable the default `ActionBar`. This can be done by setting the app theme in `styles.xml` file.

```xml
<resources>
    <!-- Base application theme. -->
    <style name="AppTheme" parent="Theme.AppCompat.Light.NoActionBar">
    </style>
</resources>
```

### Setup Drawer in Activity

Next, let's setup a basic navigation drawer based on the following layout file which has the entire drawer setup in `res/layout/activity_main.xml`. Note that the `Toolbar` is added as the first child of the main content view by adding the include tag.

When `android:fitsSystemWindows` attribute is set to true for a view, the view would be laid out as if the `StatusBar` and the `ActionBar` were present i.e. the UI on top gets padding enough to not be obscured by the navigation bar. We want our main content view to have the navigation bar and hence `android:fitsSystemWindows` is set to true for the `LinearLayout`.

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
        android:fitsSystemWindows="true"
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
        android:layout_width="240dp"
        android:layout_height="match_parent"
        android:layout_gravity="start"
        android:background="@android:color/background_light"
        app:menu="@menu/drawer_view" />
</android.support.v4.widget.DrawerLayout>
```

Now, let's setup the drawer in our activity:

```java
public class MainActivity extends ActionBarActivity {
	private DrawerLayout dlDrawer;
	private ActionBarDrawerToggle drawerToggle;
	private Toolbar toolbar;

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);

		// Set a Toolbar to replace the ActionBar.
		toolbar = (Toolbar) findViewById(R.id.toolbar);
		setSupportActionBar(toolbar);
		drawerToggle = setupDrawerToggle();  // setup rotating hamburger
		// Find our drawer view
		dlDrawer = (DrawerLayout) findViewById(R.id.drawer_layout);
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
		// ActionBarDrawerToggle will take care of this.
		if (drawerToggle.onOptionsItemSelected(item)) {
			return true;
		}

		return super.onOptionsItemSelected(item);
	}

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

	private ActionBarDrawerToggle setupDrawerToggle() {
		return new ActionBarDrawerToggle(this, mDrawerLayout, toolbar, R.string.drawer_open,  R.string.drawer_close);
	}
}
```

## Responding to navigation element clicks

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

## Making Status Bar Translucent

To have the status bar translucent and have our drawer slide below it, we need to set `android:windowTranslucentStatus` to true. Because this style is not available for pre Kitkat devices, we'll add  `res/values-v19/styles.xml` file for API version 19 and onwards.  **Note**: If you modify your `res/values/styles.xml` directly with this `android:windowTranslucentStatus` line, you are likely to need to build only for SDK versions 19 or higher, which will obviously limit you from supporting many older devices.

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




## Custom Background for Selected Item

To change the color of the selected item in your navigation drawer, you need to define layout drawable to  state the list item state when normal and pressed. It needs overall three xml files. One is for normal state, second is for pressed state and third one to combine both the layouts.

![NavIconDrawable|250](http://i.imgur.com/lMDh4x2l.png)

Create a xml file under res > drawable folder named `list_item_bg_normal.xml` and paste the following code. (If you don’t see drawable folder, create a new folder and name it as drawable)

```xml
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="rectangle">
  <gradient
      android:startColor="#eeeeee"
      android:endColor="#eeeeee"
      android:angle="90" />
</shape>
```

Create another xml layout under res > drawable named `list_item_bg_pressed.xml` with following content.

```xml
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="rectangle">
  <gradient
      android:startColor="@android:color/holo_orange_dark"
      android:endColor="@android:color/holo_orange_dark"
      android:angle="90" />
</shape>
```

Create another xml file to combine both the drawable states under res > drawable named `list_selector.xml`.

```xml
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">

    <item android:drawable="@drawable/list_item_bg_pressed" android:state_pressed="true"/>
    <item android:drawable="@drawable/list_item_bg_pressed" android:state_activated="true"/>
    <item android:drawable="@drawable/list_item_bg_normal" android:state_activated="false"/>

</selector>
```

Update `android:background` attribute of your `RelativeLayout` under `res/layout/drawer_nav_item.xml`.

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:background="@drawable/list_selector"
    android:layout_height="48dp">

    <ImageView
        android:id="@+id/ivIcon"
        android:layout_width="25dp"
        android:layout_height="wrap_content"
        android:layout_alignParentLeft="true"
        android:layout_marginLeft="12dp"
        android:layout_marginRight="12dp"
        android:layout_centerVertical="true" />

    <TextView
        android:id="@+id/tvTitle"
        android:layout_width="wrap_content"
        android:layout_height="match_parent"
        android:layout_toRightOf="@id/ivIcon"
        android:minHeight="?android:attr/listPreferredItemHeightSmall"
        android:textAppearance="?android:attr/textAppearanceListItemSmall"
        android:gravity="center_vertical"
        android:paddingRight="40dp"/>

</RelativeLayout>
```

For a more in-depth look at a navigation drawer with icons and sections, check out this [detailed navigation drawer tutorial](http://www.michenux.net/android-navigation-drawer-748.html).

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

## Manual Implementation 

Instead of the above steps, an alternative is using a `DrawerLayout` directly in the layout as outlined here:

```xml
<android.support.v4.widget.DrawerLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/drawer_layout"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
 
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:fitsSystemWindows="true"
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
    <ListView
      android:id="@+id/lvDrawer"
      android:layout_width="240dp"
      android:layout_height="match_parent"
      android:layout_gravity="start"
      android:choiceMode="singleChoice"
      android:paddingTop="24dp"
      android:divider="@android:color/darker_gray"
      android:dividerHeight="0dp"
      android:background="@android:color/background_light" />
</android.support.v4.widget.DrawerLayout>
```

This requires manual setup of the drawer [as outlined here](https://developer.android.com/training/implementing-navigation/nav-drawer.html).

## References

* <http://www.michenux.net/android-navigation-drawer-748.html>
* <http://www.androidhive.info/2013/11/android-sliding-menu-using-navigation-drawer/>
* <http://android-developers.blogspot.com/2014/10/appcompat-v21-material-design-for-pre.html>
* <http://stackoverflow.com/questions/26440879/how-do-i-use-drawerlayout-to-display-over-the-actionbar-toolbar-and-under-the-st>
* <http://antonioleiva.com/navigation-view/>