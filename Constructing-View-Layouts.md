## Overview

View Layouts are a type of View class whose primary purpose is to organize and position other view controls. These layout classes (LinearLayout, RelativeLayout, etc. ) are used to display child controls, such as text controls or buttons on the screen. 

Android activities (screens) use layouts as a container for view controls, and layouts can actually contain other nested layouts as well. Nearly all Android activities have layout containers similar to the way that most HTML documents use "divs" to contain other content.

There are two very commonly used layouts and then many more specialized layouts that are used in only very particular cases. The bread and butter layouts are **LinearLayout** and **RelativeLayout**. 

## LinearLayout

In a linear layout, like the name suggests, all the elements are displayed in a linear fashion either *horizontally* or *vertically* and this behavior is specified in `android:orientation` which is an attribute of the node `LinearLayout`.

Example of LinearLayout snippet:

```xml
<?xml version="1.0" encoding="utf-8"?>
<!-- Parent linear layout with vertical orientation -->
<LinearLayout
  xmlns:android="http://schemas.android.com/apk/res/android"
  android:orientation="vertical"
  android:layout_width="match_parent"
  android:layout_height="match_parent">
   
  <TextView android:layout_width="fill_parent" android:layout_height="wrap_content"
            android:text="Email:" android:padding="5dip"/>
             
  <EditText android:layout_width="fill_parent" android:layout_height="wrap_content"
            android:layout_marginBottom="10dip"/>            
</LinearLayout>
```

## RelativeLayout



## References

 * <http://developer.android.com/guide/topics/ui/declaring-layout.html>
 * <http://developer.android.com/guide/topics/ui/layout/linear.html>
 * <http://developer.android.com/guide/topics/ui/layout/relative.html>
 * <http://www.learn-android.com/2010/01/05/android-layout-tutorial/>
 * <http://mobile.tutsplus.com/tutorials/android/android-layout/>
 * <http://www.androidhive.info/2011/07/android-layouts-linear-layout-relative-layout-and-table-layout/>
 * <http://logc.at/2011/10/18/when-to-use-linearlayout-vs-relativelayout/>