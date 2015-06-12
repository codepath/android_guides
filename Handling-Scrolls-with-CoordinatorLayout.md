## Overview

[CoordinatorLayout] 
(https://developer.android.com/reference/android/support/design/widget/CoordinatorLayout.html) enables the ability to accomplish many of the [scrolling effects](http://www.google.com/design/spec/patterns/scrolling-techniques.html) as described in Google's Material Design spec.  For instance, it enables the ability to expand or compress the headline space as a user scrolls down to view the main content.   See the following <a href="http://material-design.storage.googleapis.com/publish/material_v_4/material_ext_publish/0B6Okdz75tqQsV09qdnY1WkRLTmM/patterns-scrolling-techniques-flexible_space_xhdpi_003.mp4">video clip</a> for one example of what it can enable.

CoordinatorLayout is basically a `FrameLayout` that also helps manage the scrolling effects for views defined within it.  Currently, there are several ways provided in this framework that allow it to work without needing to write your own custom animation code.  These effects include:

* Sliding the Floating Action Button up and down to make space for the Snackbar.
* Adjusting ToolBar or header space to expand or contract room to  for the main content.
* Controlling which views should expand or collapse and at what rate, including [parallax scrolling effects](https://ihatetomatoes.net/demos/parallax-scroll-effect/) animations.

These standard ways are described in this [blog post](http://android-developers.blogspot.com/2015/05/android-design-support-library.html).

You can also setup your own custom behaviors, such as the one discussed in using [CoordinatorLayout with Floating Action Buttons](http://guides.codepath.com/android/Floating-Action-Buttons#using-coordinatorlayout).  Because the source code does not appear to be publicly available, the best way to understand how to implement custom behaviors is by using Android Studio and navigating up to study the `FloatingActionButton` and `AppBarLayout` default behaviors.  Assuming you are using Android Studio v1.2 or higher, the decompiler code should enable you to understand better what's happening.


## Setup

Make sure to follow the [[Design Support Library]] instructions first.

## Floating Action Buttons and Snackbars

The CoordinatorLayout can be used to create floating effects using the `layout_anchor` and `layout_gravity` attributes.  See the [[Floating Action Buttons]] guide for more information.

When a [[Snackbar|Displaying the Snackbar]] is rendered, it normally appears at the bottom of the visible screen.  To make room, the floating action button must be moved up to provide space.  

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

## Collapsing and Hiding Toolbar and Headers

 
