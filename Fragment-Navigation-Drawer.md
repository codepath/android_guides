In [[Common Navigation Paradigms]] cliffnotes, we discuss the various navigational structures available within Android applications. One of the most flexible is the [Navigation Drawer](http://developer.android.com/training/implementing-navigation/nav-drawer.html). The fully custom navigation drawer is totally managed by the user and can be read about on the [creating the navigation drawer](http://developer.android.com/training/implementing-navigation/nav-drawer.html#top) docs.

![Navigation Drawer](http://i.imgur.com/GG6JYZC.png)

This guide instead explains how to setup a basic drawer filled with navigation items that switch different fragments into the content area. In this way, you can define multiple fragments, and then define the list of options which will display in the drawers items list. Each item when clicked will switch the relevant fragment into the activity's container view.

### Download Assets

Next, be sure to [download the drawer image assets](http://developer.android.com/downloads/design/Android_Navigation_Drawer_Icon_20130516.zip) necessary and add the images into each of your drawable folders.

### Add Android Support v13 SDK

You need to bump your Minimum version in AndroidManifest.xml to API v13.

You'll also need to include android-support-v13.jar from the Android SDK directory (sdk/extras/android/support/v13/).

### Setup Drawer Layout Files

You also need to setup a view that will represent the individual drawer item in a layout file such as `res/layout/drawer_nav_item.xml`:

```xml
<TextView xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@android:id/text1"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:textAppearance="?android:attr/textAppearanceListItemSmall"
    android:gravity="center_vertical"
    android:paddingLeft="16dp"
    android:paddingRight="16dp"
    android:textColor="#111"
    android:background="?android:attr/activatedBackgroundIndicator"
    android:minHeight="?android:attr/listPreferredItemHeightSmall"/>
```

Then in your `res/values/strings.xml` add the following:

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string name="drawer_open">Open navigation drawer</string>
    <string name="drawer_close">Close navigation drawer</string>
</resources>
```

### Copy In FragmentNavigationDrawer

First, let's define our [FragmentNavigationDrawer.java](https://gist.github.com/nesquena/4e9f618b71c30842e89c) file by copying the text from the [linked gist](https://gist.github.com/nesquena/4e9f618b71c30842e89c). 

### Define Fragments

Next, you need to define your fragments that will be displayed within the drawer. These can be any support fragments you define within your application.

### Setup Drawer in Activity

Next, let's setup a basic navigation drawer based on the following layout file which has the entire drawer setup in `res/layout/activity_main.xml`:

```xml
<com.codepath.examples.navdrawerdemo.FragmentNavigationDrawer
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/drawer_layout"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    
    <!-- The main content view -->
    <FrameLayout
        android:id="@+id/flContent"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />
    
    <!-- The navigation drawer -->
    <ListView android:id="@+id/lvDrawer"
        android:layout_width="240dp"
        android:layout_height="match_parent"
        android:layout_gravity="start"
        android:choiceMode="singleChoice"
        android:divider="@android:color/darker_gray"
        android:dividerHeight="0dp"
        android:background="@android:color/background_light"
     />
</com.codepath.examples.navdrawerdemo.FragmentNavigationDrawer>
```

Now, let's setup the drawer in our activity:

```java
public class MainActivity extends FragmentActivity {
	private FragmentNavigationDrawer dlDrawer;

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		// Find our drawer view
		dlDrawer = (FragmentNavigationDrawer) findViewById(R.id.drawer_layout);
		// Setup drawer view
		dlDrawer.setupDrawerConfiguration((ListView) findViewById(R.id.lvDrawer), 
                     R.layout.drawer_nav_item, R.id.flContent);
		// Add nav items
		dlDrawer.addNavItem("First", "First Fragment", FirstFragment.class);
		dlDrawer.addNavItem("Second", "Second Fragment", SecondFragment.class);
		dlDrawer.addNavItem("Third", "Third Fragment", ThirdFragment.class);
		// Select default
		if (savedInstanceState == null) {
			dlDrawer.selectDrawerItem(0);	
		}
	}


	@Override
	public boolean onPrepareOptionsMenu(Menu menu) {
		// If the nav drawer is open, hide action items related to the content
		if (dlDrawer.isDrawerOpen()) {
			menu.findItem(R.id.mi_test).setVisible(false);
		}
		return super.onPrepareOptionsMenu(menu);
	}

	@Override
	public boolean onCreateOptionsMenu(Menu menu) {
		MenuInflater inflater = getMenuInflater();
		inflater.inflate(R.menu.main, menu);
		return super.onCreateOptionsMenu(menu);
	}

	@Override
	public boolean onOptionsItemSelected(MenuItem item) {
		// The action bar home/up action should open or close the drawer.
		// ActionBarDrawerToggle will take care of this.
		if (dlDrawer.getDrawerToggle().onOptionsItemSelected(item)) {
			return true;
		}

		return super.onOptionsItemSelected(item);
	}

	@Override
	protected void onPostCreate(Bundle savedInstanceState) {
		super.onPostCreate(savedInstanceState);
		// Sync the toggle state after onRestoreInstanceState has occurred.
		dlDrawer.getDrawerToggle().syncState();
	}

	@Override
	public void onConfigurationChanged(Configuration newConfig) {
		super.onConfigurationChanged(newConfig);
		// Pass any configuration change to the drawer toggles
		dlDrawer.getDrawerToggle().onConfigurationChanged(newConfig);
	}

}
```

Now if you run your application, you should see the navigation drawer and be able to select between your fragments.