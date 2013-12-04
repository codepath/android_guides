## Overview

Layout that allows the user to swipe left and right through "pages" of content which are usually different fragments. This is a common navigation mode to use instead of [[ActionBar Tabs with Fragments]].

## Usage

A `ViewPager` is a layout which can be added to any layout XML file inside a root layout:

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">
 
    <android.support.v4.view.ViewPager
        android:id="@+id/vpPager"
        android:layout_width="match_parent"
        android:layout_height="wrap_content">
    </android.support.v4.view.ViewPager>
</LinearLayout>
```

## References

* <http://architects.dzone.com/articles/android-tutorial-using>
* <http://developer.android.com/training/animation/screen-slide.html>
* <http://developer.android.com/reference/android/support/v4/view/ViewPager.html>
* <http://android-developers.blogspot.com/2011/08/horizontal-view-swiping-with-viewpager.html>
* <http://viewpagerindicator.com/>
* <http://mobile.tutsplus.com/tutorials/android/android-user-interface-design-horizontal-view-paging/>