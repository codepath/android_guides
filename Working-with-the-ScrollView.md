## Overview

When an app has layout content that might be longer than the height of the device and that content should be vertically scrollable, then we need to use a [ScrollView](http://developer.android.com/reference/android/widget/ScrollView.html).

<img src="http://i.imgur.com/1TaIN6g.gif" width="350" />

## Usage

### Vertically Scrolling

To make any content vertically scrollable, simply wrap that content in a `ScrollView`:

```xml
<ScrollView
  android:layout_width="match_parent"
  android:layout_height="match_parent">
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical" >   
        
          <TextView
              android:id="@+id/tv_long"
              android:layout_width="wrap_content"
              android:layout_height="match_parent"
              android:text="@string/really_long_string" >
          </TextView>
  
          <Button
              android:id="@+id/btn_act"
              android:layout_width="wrap_content"
              android:layout_height="wrap_content"
              android:text="View" >
          </Button>
    </LinearLayout>
</ScrollView>
```

**Note** that a ScrollView can only contain a **single child element** so if you need multiple things to be scrollable, you need to wrap that content into a layout as shown above. 

In certain situations, you want to position content beneath the end of the scrollable content area. For example for a "terms of service" where you can only accept once you've scrolled through all the content. In this case, you might need to apply the [android:fillViewport](http://developer.android.com/reference/android/widget/ScrollView.html#attr_android:fillViewport) property to "true". Read [this post by Romain Guy](http://www.curious-creature.org/2010/08/15/scrollviews-handy-trick/) for a detailed look at this use case.

### Scrollable TextView

Note that a `TextView` **doesn't require** a ScrollView and if you just need a scrolling `TextView` simply set the `scrollbars` property and apply the correct `MovementMethod`:

```xml
<TextView
    android:id="@+id/tv_long"
    android:layout_width="wrap_content"
    android:layout_height="match_parent"
    android:scrollbars = "vertical"
    android:text="@string/really_long_string" >
</TextView>
```

and then in the activity:

```java
TextView tv = (TextView) findViewById(R.id.tv_long);
tv.setMovementMethod(new ScrollingMovementMethod());
```

Now the TextView will automatically scroll vertically.

### Horizontally Scrolling

In other cases, we want content to horizontally scroll in which case we need to use the [HorizontalScrollView](http://developer.android.com/reference/android/widget/HorizontalScrollView.html) instead like this:

```xml
<HorizontalScrollView
  android:layout_width="match_parent"
  android:layout_height="match_parent">
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal" >   
        <!-- child views in here -->
    </LinearLayout>
</HorizontalScrollView>
```

and now you have a horizontally scrolling view.

## Resources

* <http://examples.javacodegeeks.com/android/core/ui/scrollview/android-scrollview-example/>
* <http://www.curious-creature.org/2010/08/15/scrollviews-handy-trick/>
* <http://www.androidhub4you.com/2013/05/simple-scroll-view-example-in-android.html>
* <http://developer.android.com/reference/android/widget/ScrollView.html>
* <http://developer.android.com/reference/android/widget/HorizontalScrollView.html>
* <http://www.whitesof.com/?q=tutorials/android/android-scroll-view-tutorial>