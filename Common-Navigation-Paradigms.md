## Overview

Navigation between views is an important part of any application. There are several ways to setup navigation in an Android application:

 * [Swipe Views](http://developer.android.com/training/implementing-navigation/lateral.html) - Allow paging between views using a swipe gesture
 * [Navigation Drawer](http://developer.android.com/training/implementing-navigation/nav-drawer.html) - Displays a vertical menu that slides in to allow navigation between views 
 * [ActionBar](http://developer.android.com/design/patterns/actionbar.html) - Using the ActionBar and/or ActionBar tabs to switch between different views
 * [Screen Map](http://developer.android.com/training/design-navigation/descendant-lateral.html#buttons) - Providing a series of buttons on screen that can be pressed to visit different views

These four represent the most common navigation paradigms in Android applications. The specifics for how to implement these can be found in the various links above.

### Swiping Views

Check out the [[ViewPager with FragmentPagerAdapter]] cliffnotes for more details of how to add swipe-able views as a form of navigation. We can use a [[tab indicator|Sliding-Tabs-with-PagerSlidingTabStrip]] to display tabs on top of a swiping view.

### Navigation Drawer

To create a basic navigation drawer that toggles between displaying different fragments, check out the
[[Fragment Navigation Drawer]] cliffnotes. For more details about creating a custom drawer check out the [Creating a Navigation Drawer](http://developer.android.com/training/implementing-navigation/nav-drawer.html#top) docs.  

### ActionBar Tabs

Check out the [[ActionBar Tabs|ActionBar Tabs with Fragments#fragments-and-tabs]] cliffnotes for more details of how to add ActionBar tabs to your Activity. Note that **ActionBar Tabs is deprecated** since Android API 21. 

## References

 * <http://developer.android.com/design/patterns/navigation.html>
 * <http://developer.android.com/training/design-navigation/index.html>
 * <http://developer.android.com/training/implementing-navigation/nav-drawer.html>