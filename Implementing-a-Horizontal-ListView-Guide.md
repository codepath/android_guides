## Overview

One common challenge on Android that often arises is the need to create a horizontal ListView that works like a regular ListView but scrolls horizontally instead. This is common for news, images, etc. 

![Horizontal ListView](http://i.imgur.com/z3xLN7M.png)

Unfortunately, there is no "canonical" officially supported solution for this. Instead all we have is a series of half-baked unfinished libraries. This can make developing Horizontal ListView more challenging then you would initially suspect.

This guide is a step-by-step walkthrough of implementing a Horizontal ListView using the available third-party libraries. 

### Introducing TwoWayView

Of all the third-party libraries currently out there for Horizontal ListView, the most complete and useable is called [TwoWayView](https://github.com/lucasr/twoway-view). Even though there is a warning about this not being production ready, this is still probably the best view short of implementing one yourself. 

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
     android:layout_width="match_parent"
     android:layout_height="match_parent"
     android:drawSelectorOnTop="false"
     tools:context=".MainActivity" />
```

### Populating Data into TwoWayView

Now we can populate data into the `TwoWayView` much like any ListView as described in [[the Adapter guide|Using-an-ArrayAdapter-with-ListView]]. For example, we might create a very simple array of items:

```java
ArrayList<String> items = new ArrayList<String>();
items.add("Item 1");
items.add("Item 2");
items.add("Item 3");
items.add("Item 4");
// ...
```

and then construct a super simple `ArrayAdapter` and populate the TwoWayView with:

Take into account that the `layout_width` param is defined as `match_parent` in `simple_list_item_1`, so if you use it, the list will show just the first element. So, please create a new layout like this:
```xml
<?xml version="1.0" encoding="utf-8"?>
<TextView xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@android:id/text1"
    android:layout_width="wrap_content"
    android:layout_height="match_parent"
    android:textAppearance="?android:attr/textAppearanceListItemSmall"
    android:gravity="center_vertical" />
```
Name it as simple_list_item_1 and use the following code to test it:
```java
ArrayAdapter<String> aItems = new ArrayAdapter<String>(this, R.layout.simple_list_item_1, items);
TwoWayView lvTest = (TwoWayView) findViewById(R.id.lvItems);
lvTest.setAdapter(aItems);
```
The view will now present the items horizontally like:

![View Screen](http://i.imgur.com/76YnbHp.png)

### Conclusion

Here we have implemented a simple Horizontal ListView using a popular third-party library. Obviously this example is very simple but you can follow the steps in the [[the Adapter guide|Using-an-ArrayAdapter-with-ListView]] to create arbitrarily complex list items.

## References

 * <http://profi.co/android-horizontal-list-view-library-with-item-scroll/>
 * <http://lucasr.org/2013/02/21/introducing-twowayview/>
 * <https://github.com/lucasr/twoway-view>
 * <https://github.com/MeetMe/Android-HorizontalListView>
 * <https://github.com/dinocore1/DevsmartLib-Android>
 * <https://github.com/twotoasters/HorizontalImageScroller-Android>
 * <https://developer.android.com/preview/material/ui-widgets.html>