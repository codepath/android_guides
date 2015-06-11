## Overview

[CoordinatorLayout] 
(https://developer.android.com/reference/android/support/design/widget/CoordinatorLayout.html) enables the ability to accomplish many of the scrolling effects as described in Google's Material Design [spec](http://www.google.com/design/spec/patterns/scrolling-techniques.html).  For instance, it enables the ability to expand or contract the headline space as a user scrolls down or up the page.   See the following <a href="http://material-design.storage.googleapis.com/publish/material_v_4/material_ext_publish/0B6Okdz75tqQsV09qdnY1WkRLTmM/patterns-scrolling-techniques-flexible_space_xhdpi_003.mp4">video clip</a> for one example of what it can do:

CoordinatorLayout is basically a `FrameLayout` that also helps manage the scrolling effects for views contained within it.  Currently, there are several predefined ways provided in this framework that allow it to work without needing to write your own custom animation code.  These effects include:

* Sliding the Floating Action Button up and down to make space for the Snackbar.
* Expanding or contracting ToolBar or Header space for the main content.
* Controlling which elements should expand or collapse and what rate.  
* Enabling [parallax scrolling effects](https://ihatetomatoes.net/demos/parallax-scroll-effect/) animations.

These standard ways are described in this [blog post](http://android-developers.blogspot.com/2015/05/android-design-support-library.html).

You can also setup your own custom behaviors, such as the one discussed in using [CoordinatorLayout with Floating Action Buttons](http://guides.codepath.com/android/Floating-Action-Buttons#using-coordinatorlayout).

## Setup

Make sure to follow the [[Design Support Library]] instructions first.

