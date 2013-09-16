## Overview

Navigation between views is an important part of any application. There are several ways to setup navigation in an Android application:

 * [ActionBar](http://developer.android.com/design/patterns/actionbar.html) - Using the ActionBar and/or ActionBar tabs to switch between different views
 * [Navigation Drawer](http://developer.android.com/training/implementing-navigation/nav-drawer.html) - Displays a vertical menu that slides in to allow navigation between views 
 * [Screen Map](http://developer.android.com/training/design-navigation/descendant-lateral.html#buttons) - Providing a series of buttons on screen that can be pressed to visit different views
 * [Swipe Views](http://developer.android.com/training/implementing-navigation/lateral.html) - Allow paging between views using a swipe gesture

These four represent the most common navigation paradigms in Android applications. The specifics for how to implement these can be found in the various links above.

### ActionBar Tabs

For a description of how to add ActionBar tabs to your Activity, check out the [fragments actionbar tabs cliffnotes](https://github.com/thecodepath/android_guides/wiki/Creating-and-Using-Fragments#fragments-and-tabs) for more details.

### Navigation Drawer

First, setting up the navigation drawer requires using the `android.support.v4.widget.DrawerLayout` as the root of the layout for the activity. This view has two nested views: first the `FrameLayout` that will display the fragments and second child is the `ListView` which contains the menu items. 

To create a basic navigation drawer that toggles between displaying different fragments, check out the
[[Fragment Navigation Drawer]] cliffnotes. For more details about creating a custom drawer check out the [Creating a Navigation Drawer](http://developer.android.com/training/implementing-navigation/nav-drawer.html#top) docs.

## References

 * <http://developer.android.com/design/patterns/navigation.html>
 * <http://developer.android.com/training/design-navigation/index.html>
 * <http://developer.android.com/training/implementing-navigation/nav-drawer.html>