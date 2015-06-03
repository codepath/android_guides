## Overview

[SurfaceView](http://developer.android.com/reference/android/view/SurfaceView.html) provides a dedicated drawing surface embedded inside of a view hierarchy. You can control the format of this surface and, if you like, its size; the SurfaceView takes care of placing the surface at the correct location on the screen. 

One of the purposes of this class is to provide a surface in which a secondary thread can render into the screen. If you need to perform complex drawing operations many times a second, you may want to offload the drawing off the UIThread and that is the purpose of this view.

## Usage

**Incomplete: Fill this in with a relevant example**

## References

* <http://infinut.com/2013/10/10/to-opengl-or-to-surfaceview/>
* <http://developer.android.com/reference/android/view/SurfaceView.html>
* <http://javandroidevelop.blogspot.com/2012/09/all-about-surfaceview-android-lessons.html>
* <http://blog.wisecells.com/2012/06/04/surface-view-android/> 
* <http://blog.lemberg.co.uk/surface-view-playing-video>
* <https://github.com/google/grafika/blob/master/src/com/android/grafika/PlayMovieSurfaceActivity.java>
* <https://gist.github.com/ErikHellman/5840101>
* <http://stackoverflow.com/questions/12273976/camera-tutorial-for-android-using-surfaceview>