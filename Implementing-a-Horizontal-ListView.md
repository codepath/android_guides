## Overview

One common challenge on Android that often arises is the need to create a horizontal ListView that works like a regular ListView but scrolls horizontally instead. This is common for news, images, etc. 

![Horizontal ListView](http://i.imgur.com/z3xLN7M.png)

Unfortunately, there is no "canonical" officially supported solution for this. Instead all we have is a series of half-baked unfinished libraries. This can make developing Horizontal ListView more challenging then you would initially suspect.

This guide is a step-by-step walkthrough of implementing a Horizontal ListView using the available third-party libraries. 

### Introducing TwoWayView

Of all the third-party libraries currently out there for Horizontal ListView, the most complete and useable is called [TwoWayView](https://github.com/lucasr/twoway-view). Even though there is a warning about this not being production ready, this is still probably the best view short of [implementing one yourself](http://www.androiddevelopersolution.com/2012/11/horizontal-listview-in-android-example.html). 

### Installing TwoWayView

[Download TwoWayView](https://github.com/lucasr/twoway-view) as a ZIP and then install it as a [library project](http://imgur.com/a/N8baF). **Note:** If you have any trouble importing the version downloaded, try this modified version [compatible with Eclipse](https://www.dropbox.com/s/1zwlc17d2nyahf1/TwoWayView-eclipse.zip).

### Adding TwoWayView to Layout

First, let's add a style indicating the orientation of the ListView (horizontal or vertical) in `res/values/styles.xml`:

```xml
<style name="TwoWayView">
    <item name="android:orientation">horizontal</item>
</style>
```

In your Layout XML, use the following code to add the TwoWayView:

```xml
<org.lucasr.twowayview.TwoWayView 
     xmlns:android="http://schemas.android.com/apk/res/android"
     xmlns:tools="http://schemas.android.com/tools"
     xmlns:app="http://schemas.android.com/apk/res-auto"
     android:id="@+id/lvItems"
     style="@style/TwoWayView"
     android:layout_width="fill_parent"
     android:layout_height="fill_parent"
     android:drawSelectorOnTop="false"
     tools:context=".MainActivity" />
```

### Populating Data into TwoWayView

Now we can populate data into the `TwoWayView` much like any ListView as described in [the Adapter guide](https://github.com/thecodepath/android_guides/wiki/Using-an-ArrayAdapter-with-ListView). For example, we might create 

## References

 * <http://profi.co/android-horizontal-list-view-library-with-item-scroll/>
 * <http://lucasr.org/2013/02/21/introducing-twowayview/>
 * <https://github.com/lucasr/twoway-view>
 * <https://github.com/MeetMe/Android-HorizontalListView>
 * <https://github.com/dinocore1/DevsmartLib-Android>
 * <https://github.com/twotoasters/HorizontalImageScroller-Android>