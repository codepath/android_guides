## Overview

Navigation between views is an important part of any application. There are several ways to setup navigation in an Android application:

 * [ActionBar](http://developer.android.com/design/patterns/actionbar.html) - Using the ActionBar and/or ActionBar tabs to switch between different views
 * [Navigation Drawer](http://developer.android.com/training/implementing-navigation/nav-drawer.html) - Displays a vertical menu that slides in to allow navigation between views 
 * [Screen Map](http://developer.android.com/training/design-navigation/descendant-lateral.html#buttons) - Providing a series of buttons on screen that can be pressed to visit different views
 * [Swipe Views](http://developer.android.com/training/implementing-navigation/lateral.html) - Allow paging between views using a swipe gesture

These four represent the most common navigation paradigms in Android applications. The specifics for how to implement these can be found in the various links above.

### ActionBar Tabs

For a description of how to add ActionBar tabs to your Activity, check out the [Fragments cliffnotes](https://github.com/thecodepath/android_guides/wiki/Creating-and-Using-Fragments#fragments-and-tabs) for more details.

### Navigation Drawer

First, setting up the navigation drawer requires using the `android.support.v4.widget.DrawerLayout` as the root of the layout for the activity. This view has two nested views: first the `FrameLayout` that will display the fragments and second child is the `ListView` which contains the menu items. See [Creating a Navigation Drawer](http://developer.android.com/training/implementing-navigation/nav-drawer.html#top) for more details about creating a custom NavigationDrawer.

The following explains how to setup a basic NavigationDrawer that switches different fragments into the content area. First, let's define our [FragmentNavigationDrawer.java](https://gist.github.com/nesquena/4e9f618b71c30842e89c) file by copying the text from the [linked gist](https://gist.github.com/nesquena/4e9f618b71c30842e89c). 

Next, be sure to [download the drawer image assets](http://developer.android.com/downloads/design/Android_Navigation_Drawer_Icon_20130516.zip) necessary and add them into each of your drawable folders.

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

Next, you need to define your fragments that will be displayed within the drawer. These can be any support fragments you define within your application.

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

Now if you run your application, you should see the navigation drawer.

## References

 * <http://developer.android.com/design/patterns/navigation.html>
 * <http://developer.android.com/training/design-navigation/index.html>
 * <http://developer.android.com/training/implementing-navigation/nav-drawer.html>