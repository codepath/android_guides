## Overview

[SurfaceView](http://developer.android.com/reference/android/view/SurfaceView.html) provides a dedicated drawing surface embedded inside of a view hierarchy. You can control the format of this surface and, if you like, its size; the SurfaceView takes care of placing the surface at the correct location on the screen. 

One of the purposes of this class is to provide a surface in which a secondary thread can render into the screen. If you need to perform complex drawing operations many times a second, you may want to offload the drawing off the UIThread and that is the purpose of this view.

## Usage

**Needs Attention**

## References

* <http://developer.android.com/reference/android/view/SurfaceView.html>
* <http://javandroidevelop.blogspot.com/2012/09/all-about-surfaceview-android-lessons.html>
* <http://blog.wisecells.com/2012/06/04/surface-view-android/> 