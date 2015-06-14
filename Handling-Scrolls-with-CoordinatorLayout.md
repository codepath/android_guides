## Overview

[CoordinatorLayout] 
(https://developer.android.com/reference/android/support/design/widget/CoordinatorLayout.html) extends the ability to accomplish many of the Google's Material Design [scrolling effects](http://www.google.com/design/spec/patterns/scrolling-techniques.html).  Currently, there are several ways provided in this framework that allow it to work without needing to write your own custom animation code.  These effects include:

* Sliding the Floating Action Button up and down to make space for the Snackbar.

  <img src="http://imgur.com/zF9GGsK.gif" width="350"/>

* Expanding or contracting the Toolbar or header space to make room for the main content.

  <img src="http://imgur.com/X5AIH0P.gif" width="350"/>

* Controlling which views should expand or collapse and at what rate, including [parallax scrolling effects](https://ihatetomatoes.net/demos/parallax-scroll-effect/) animations.

  <img src="http://imgur.com/1JHP0cP.gif"/>

## Setup

Make sure to follow the [[Design Support Library]] instructions first.

## Floating Action Buttons and Snackbars

The CoordinatorLayout can be used to create floating effects using the `layout_anchor` and `layout_gravity` attributes.  See the [[Floating Action Buttons]] guide for more information.

When a [[Snackbar|Displaying the Snackbar]] is rendered, it normally appears at the bottom of the visible screen.  To make room, the floating action button must be moved up to provide space.  

<img src="http://imgur.com/zF9GGsK.gif" width="350"/>

So long as the CoordinatorLayout is used as the primary layout, this animation effect will occur for you automatically.   The floating action button has a [default behavior](https://developer.android.com/reference/android/support/design/widget/FloatingActionButton.Behavior.html) that detects Snackbar views being added and animates the button above the height of the Snackbar.

```xml
 <android.support.design.widget.CoordinatorLayout
        android:id="@+id/main_content"
        xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto"
        android:layout_width="match_parent"
        android:layout_height="match_parent">

   <android.support.v7.widget.RecyclerView
         android:id="@+id/rvToDoList"
         android:layout_width="match_parent"
         android:layout_height="match_parent"></android.support.v7.widget.RecyclerView>

   <android.support.design.widget.FloatingActionButton
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="bottom|right"
        android:layout_margin="16dp"
        android:src="@mipmap/ic_launcher"
        app:layout_anchor="@id/rvToDoList"
        app:layout_anchorGravity="bottom|right|end"/>
 </android.support.design.widget.CoordinatorLayout>
```

## Expanding and Collapsing Toolbars

<img src="http://imgur.com/X5AIH0P.gif" width="350"/>

The first step is to make sure you are not using the deprecated ActionBar.  Make sure to follow the [Using the ToolBar as ActionBar](http://guides.codepath.com/android/Defining-The-ActionBar#using-toolbar-as-actionbar) guide.  Also make sure that the CoordinatorLayout is the main layout container.

```xml
<android.support.design.widget.CoordinatorLayout xmlns:android="http://schemas.android.com/apk/res/android"
 xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/main_content"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:fitsSystemWindows="true">

      <android.support.v7.widget.Toolbar
                android:id="@+id/toolbar"
                android:layout_width="match_parent"
                android:layout_height="?attr/actionBarSize"
                app:popupTheme="@style/ThemeOverlay.AppCompat.Light" />

</android.support.design.widget.CoordinatorLayout>
```

### Responding to Scroll Events

Next, we must make the Toolbar responsive to scroll events using a container layout called
[AppBarLayout](http://developer.android.com/reference/android/support/design/widget/AppBarLayout.html):

```xml
<android.support.design.widget.AppBarLayout
        android:id="@+id/appbar"
        android:layout_width="match_parent"
        android:layout_height="@dimen/detail_backdrop_height"
        android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar"
        android:fitsSystemWindows="true">

  <android.support.v7.widget.Toolbar
                android:id="@+id/toolbar"
                android:layout_width="match_parent"
                android:layout_height="?attr/actionBarSize"
                app:popupTheme="@style/ThemeOverlay.AppCompat.Light" />

 </android.support.design.widget.AppBarLayout>
```

**Note**: AppBarLayout currently expects to be the first child nested within a CoordinatorLayout according to the official [Google docs] (http://developer.android.com/reference/android/support/design/widget/AppBarLayout.html).

Next, we need to define an association between the AppBarLayout and the View that will be scrolled.  Add an `app:layout_behavior` to a RecyclerView or any other View capable of nested scrolling such as [NestedScrollView](http://stackoverflow.com/questions/25136481/what-are-the-new-nested-scrolling-apis-for-android-l).  The support library contains a special string resource `@string/appbar_scrolling_view_behavior` that maps to [AppBarLayout.ScrollingViewBehavior](https://developer.android.com/reference/android/support/design/widget/AppBarLayout.ScrollingViewBehavior.html), which is used to notify the `AppBarLayout` when scroll events occur on this particular view.  The behavior must be established on the view that triggers the event.

```xml
 <android.support.v7.widget.RecyclerView
        android:id="@+id/rvToDoList"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:layout_behavior="@string/appbar_scrolling_view_behavior">
```

When a CoordinatorLayout notices this attribute declared in the RecyclerView, it will search across the other views contained within it for any related views associated by the behavior.  In this particular case, the `AppBarLayout.ScrollingViewBehavior` describes a dependency between the RecyclerView and AppBarLayout.   Any scroll events to the RecyclerView should trigger changes to the AppBarLayout layout or any views contained within it.

Scroll events in the RecyclerView trigger changes inside views declared within `AppBarLayout` by using the `app:layout_scrollFlags` attribute:

```xml
    <android.support.design.widget.AppBarLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:fitsSystemWindows="true"
        android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar">

            <android.support.v7.widget.Toolbar
                android:id="@+id/toolbar"
                android:layout_width="match_parent"
                android:layout_height="?attr/actionBarSize"
                app:layout_scrollFlags="scroll|enterAlways"/>

 </android.support.design.widget.AppBarLayout>
```

The `scroll` flag used within the attribute `app:layout_scrollFlags` must be enabled in order for the View to scroll off the screen.  Otherwise, it will remain pinned to the top.    Other flags that can be used include:
 
   * `enterAlways`: The view will become visible when scrolling up.
   * `enterAlwaysCollapsed`: Assuming you have declared a minHeight and `enterAlways` is declared, your view will only appear at its minimum height and expand to the full height when the scrolling view reaches to the top.
   * `exitUntilCollapsed`: Assuming you have declared a minHeight, the view will disappear once the minimum height is reached.

Keep in mind to order all your views with the scroll flag first.  This way, the views that collapse will exit first while leaving the pinned elements at the top.

At this point, you should notice that the Toolbar responds to scroll events.  

<img src="http://imgur.com/Hl2Asb1.gif" width="350"/>

### Creating Collapsing Effects

If we want to create the collapsing toolbar effect, we must wrap the Toolbar inside CollapsingToolbarLayout:

```xml
<android.support.design.widget.CollapsingToolbarLayout
            android:id="@+id/collapsing_toolbar"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:fitsSystemWindows="true"
            app:contentScrim="?attr/colorPrimary"
            app:expandedTitleMarginEnd="64dp"
            app:expandedTitleMarginStart="48dp"
            app:layout_scrollFlags="scroll|exitUntilCollapsed">
            
            <android.support.v7.widget.Toolbar
                android:id="@+id/toolbar"
                android:layout_width="match_parent"
                android:layout_height="?attr/actionBarSize"
                app:layout_scrollFlags="scroll|enterAlways"></android.support.v7.widget.Toolbar>
        </android.support.design.widget.CollapsingToolbarLayout>
```

Your result should now appears as:

<img src="http://imgur.com/X5AIH0P.gif" width="350"/>

Normally, we set the title of the Toolbar. Now, we need to set the title on the CollapsingToolBarLayout instead of the Toolbar.

```java
   CollapsingToolbarLayout collapsingToolbar =
                (CollapsingToolbarLayout) findViewById(R.id.collapsing_toolbar);
        collapsingToolbar.setTitle("Title");
```

### Creating Parallax Animations

The CollapsingToolbarLayout also enables us to do more advanced animations, such as using an ImageView that fades out as it collapses.  The title can also change in height as the user scrolls.

<img src="http://imgur.com/ah4l5oj.gif" width="350"/>

To create this effect, we add an ImageView and declare an `app:layout_collapseMode="parallax"` attribute to the tag.

```xml
    <android.support.design.widget.CollapsingToolbarLayout
            android:id="@+id/collapsing_toolbar"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:fitsSystemWindows="true"
            app:contentScrim="?attr/colorPrimary"
            app:expandedTitleMarginEnd="64dp"
            app:expandedTitleMarginStart="48dp"
            app:layout_scrollFlags="scroll|exitUntilCollapsed">

            <android.support.v7.widget.Toolbar
                android:id="@+id/toolbar"
                android:layout_width="match_parent"
                android:layout_height="?attr/actionBarSize"
                app:layout_scrollFlags="scroll|enterAlways"></android.support.v7.widget.Toolbar>
            <ImageView
                android:src="@drawable/cheese_1"
                app:layout_scrollFlags="scroll|enterAlways|enterAlwaysCollapsed"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:scaleType="centerCrop"
                app:layout_collapseMode="parallax"
                android:minHeight="100dp"/>

        </android.support.design.widget.CollapsingToolbarLayout>
```

## Custom Behaviors

One example of a custom behavior is discussed in using [CoordinatorLayout with Floating Action Buttons](http://guides.codepath.com/android/Floating-Action-Buttons#using-coordinatorlayout).  

CoordinatorLayout works by searching through any child view that has a [CoordinatorLayout Behavior](http://developer.android.com/reference/android/support/design/widget/CoordinatorLayout.Behavior.html) defined either statically as XML with a `app:layout_behavior` tag or programmatically with the View class annotated with the `@DefaultBehavior` decorator.  When a scroll event happens, CoordinatorLayout attempts to trigger other child views that are declared as dependencies.

To define your own a CoordinatorLayout Behavior, the layoutDependsOn() and onDependentViewChanged() should be implemented.  For instance, AppBarLayout.Behavior has these two key methods defined.  This behavior is used to trigger a change on the AppBarLayout when a scroll event happens. 
 
```java

  public boolean layoutDependsOn(CoordinatorLayout parent, View child, View dependency) {
            return dependency instanceof AppBarLayout;
        }

   public boolean onDependentViewChanged(CoordinatorLayout parent, View child, View dependency) {
            // check the behavior triggered
            android.support.design.widget.CoordinatorLayout.Behavior behavior = ((android.support.design.widget.CoordinatorLayout.LayoutParams)dependency.getLayoutParams()).getBehavior();
            if(behavior instanceof AppBarLayout.Behavior) {
            // do stuff here
            }
   }       
```

The best way to understand how to implement these custom behaviors is by studying the AppBarLayout.Behavior and FloatingActionButtion.Behavior examples.  Although the source code is not publicly available yet, you can use the decompiler integrated with Android Studio 1.2 to examine how they work by navigating up the source tree.

## References

* <http://android-developers.blogspot.com/2015/05/android-design-support-library.html>