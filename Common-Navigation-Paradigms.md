## Overview

Navigation between views is an important part of any application. There are several ways to setup navigation in an Android application:

 * [ActionBar](http://developer.android.com/design/patterns/actionbar.html) - Using the ActionBar and/or ActionBar tabs to switch between different views
 * [Swipe Views](http://developer.android.com/training/implementing-navigation/lateral.html) - Allow paging between views using a swipe gesture
 * [Navigation Drawer](http://developer.android.com/training/implementing-navigation/nav-drawer.html) - Displays a vertical menu that slides in to allow navigation between views 
 * [Screen Map](http://developer.android.com/training/design-navigation/descendant-lateral.html#buttons) - Providing a series of buttons on screen that can be pressed to visit different views

These four represent the most common navigation paradigms in Android applications. The specifics for how to implement these can be found in the various links above.

### ActionBar Tabs

Check out the [[ActionBar Tabs|ActionBar Tabs with Fragments#fragments-and-tabs]] cliffnotes for more details of how to add ActionBar tabs to your Activity.

### Swiping Views

Check out the [[ViewPager with FragmentPagerAdapter]] cliffnotes for more details of how to add swipe-able views as a form of navigation.

### Navigation Drawer

To create a basic navigation drawer that toggles between displaying different fragments, check out the
[[Fragment Navigation Drawer]] cliffnotes. For more details about creating a custom drawer check out the [Creating a Navigation Drawer](http://developer.android.com/training/implementing-navigation/nav-drawer.html#top) docs.   

Although both of the navigation drawer examples show how fragments can be substituted with the navigation drawer, you can also use RelativeLayout/LinearLayout if you wish to use the drawer as an overlay to your currently displayed Activity.  

Instead of &lt;FrameLayout&gt; you can subsitute for &lt;LinearLayout&gt;

```java
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

Instead of:

```java
 // Insert the fragment by replacing any existing fragment
    FragmentManager fragmentManager = getFragmentManager();
    fragmentManager.beginTransaction()
                   .replace(R.id.content_frame, fragment)
                   .commit();
```

You can also use the LinearLayout container to inflate the Activity:

```java
LayoutInflater inflater = getLayoutInflater();
LinearLayout container = (LinearLayout) findViewById(R.id.content_frame);
inflater.inflate(R.layout.activity_main, container);
```

## References

 * <http://developer.android.com/design/patterns/navigation.html>
 * <http://developer.android.com/training/design-navigation/index.html>
 * <http://developer.android.com/training/implementing-navigation/nav-drawer.html>