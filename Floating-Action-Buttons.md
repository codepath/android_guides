## Overview

[Floating action buttons](http://www.google.com/design/spec/components/buttons.html#buttons-floating-action-button) (or **FAB**) are: “A special case of promoted actions. They are distinguished by a circled icon floating above the UI and have special motion behaviors, related to morphing, launching, and its transferring anchor point.” 

For example, if we are using an email app and we are listing the inbox folder, the promoted action might be composing a new message.

<img src="https://github.com/makovkastar/FloatingActionButton/raw/master/art/demo.gif" width="185" alt="FAB1" />&nbsp;
<img src="http://i.imgur.com/SBbLXo2.png" width="200" alt="FAB2" />

The floating action button represents **the primary action** within a particular screen. More info and use cases of the FAB button can be found in Google’s official [design specs found here](http://www.google.com/design/spec/components/buttons.html#buttons-floating-action-button).

## Usage

Google made available during Google I/O 2015 a support library to create floating action buttons library.  In the past, third-party libraries such as [makovkastar/FloatingActionButton](https://github.com/makovkastar/FloatingActionButton) and [futuresimple/android-floating-action-button](https://github.com/futuresimple/android-floating-action-button) had to be used.

### Google's official support library 

1. Make sure that you have at least Gradle v1.2.3 supported.  There are several issues with using older versions including some support library widgets fail to render correctly (see  https://code.google.com/p/android/issues/detail?id=170841).

   ```gradle
      dependencies {
        classpath 'com.android.tools.build:gradle:1.2.3'
      }
   ```

2. There is a new support design library that must be included.   This library also depends on the [AppCompat](http://android-developers.blogspot.com/2014/10/appcompat-v21-material-design-for-pre.html) library and support v4 library to be included.

   ```gradle
      dependencies {
        compile 'com.android.support:appcompat-v7:22.2.0'
        compile 'com.android.support:support-v4:22.2.0'
        compile 'com.android.support:design:22.2.0'
      }
   ```

   **Note** The [official Google docs](http://developer.android.com/tools/support-library/features.html#design) suggest that the Gradle line is `support-design`, but it   appears to be a typo.  A ticket is filed at https://code.google.com/p/android/issues/detail?id=175066.

3. You should now be able to add the `android.support.design.widget.FloatingActionButton`.  The `src` attribute references the icon that should be used for the floating action.  

   ```xml
        <android.support.design.widget.FloatingActionButton
        android:src="@mipmap/ic_launcher"
        app:fabSize="@dimen/mini"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentTop="true"
        android:layout_alignParentRight="true"
        android:layout_alignParentEnd="true"
        android:layout_marginRight="27dp"
        android:layout_marginEnd="27dp">

   ```

In addition, assuming you define `xmlns:app="http://schemas.android.com/apk/res-auto` at the top of your layout, you can also define a custom attribute [`fabSize`](http://developer.android.com/reference/android/support/design/widget/FloatingActionButton.html#attr_android.support.design:fabSize) that can reference whether the button should be small or large.  You will need to define inside `attrs.xml` a custom dimension:

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <item name="mini" type="dimen" format="integer">1</item>
</resources>
```

### Dimensions

The button should be placed in the bottom right corner of the screen. The recommended margin for the bottom is **16dp for phones and 24dp for tablets**.  

The drawable that you use should be **24dp** according to the Google design specs:

<a href="http://www.google.com/design/spec/components/buttons.html#buttons-floating-action-button"><img src="http://material-design.storage.googleapis.com/publish/v_2/material_ext_publish/0Bx4BSt6jniD7UGxlcmdDZFRZYU0/patterns_actions_fab3.png" width="300"></a>

#### Attaching to Lists

Next, we can optionally associate the FAB with a `ListView`, `ScrollView` or `RecyclerView` so the button will hide as the list is scrolled down and revealed as the list is scrolled up:

```java
ListView listView = (ListView) findViewById(android.R.id.list);
FloatingActionButton fab = (FloatingActionButton) findViewById(R.id.fab);
fab.attachToListView(listView); // or attachToRecyclerView
```

We can attach to a `RecyclerView` with `fab.attachToRecyclerView(recyclerView)` or a `ScrollView` with `fab.attachToScrollView(scrollView)`

#### Adjust Button Type

Floating action buttons come in two sizes: the default, which should be used in most cases, and the mini, which should only be used to create visual continuity with other elements on the screen. 

<img src="http://i.imgur.com/SP6RG9d.png" alt="Mini" width="200" />

We can adjust the FAB button type to "normal" or "mini":

```xml
<com.melnykov.fab.FloatingActionButton
    ...
    fab:fab_type="mini" />
```

#### Show or Hiding FAB

Show/hide the button explicitly:

```java
// SHOW OR HIDE with animation
fab.show();
fab.hide();
// OR without animations
fab.show(false);
fab.hide(false);
```

#### Listening to Scroll Events

We can listen to scroll events on the attached list in order to manage the state of our FAB with:

```java
FloatingActionButton fab = (FloatingActionButton) root.findViewById(R.id.fab);
fab.attachToListView(list, new ScrollDirectionListener() {
    @Override
    public void onScrollDown() {
        Log.d("ListViewFragment", "onScrollDown()");
    }

    @Override
    public void onScrollUp() {
        Log.d("ListViewFragment", "onScrollUp()");
    }
}, new AbsListView.OnScrollListener() {
    @Override
    public void onScrollStateChanged(AbsListView view, int scrollState) {
        Log.d("ListViewFragment", "onScrollStateChanged()");
    }

    @Override
    public void onScroll(AbsListView view, int firstVisibleItem, int visibleItemCount, 
        int totalItemCount) {
        Log.d("ListViewFragment", "onScroll()");
    }
});
```

### With FloatingActionButton (deprecated)

Using [makovkastar/FloatingActionButton](https://github.com/makovkastar/FloatingActionButton) library makes floating buttons quite simple to setup. See the [library readme](https://github.com/makovkastar/FloatingActionButton/blob/master/README.md) and the [sample code](https://github.com/makovkastar/FloatingActionButton/tree/master/sample/src/main/java/com/melnykov/fab/sample) for reference.

First, add as a dependency to your `app/build.gradle`:

```gradle
dependencies {
    compile 'com.melnykov:floatingactionbutton:1.2.0'
}
```

Next, let's add the `com.melnykov.fab.FloatingActionButton` to our layout XML.  Note the addition of the `xmlns:fab` to the attributes of the root layout:

```xml
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
             xmlns:fab="http://schemas.android.com/apk/res-auto"
             android:layout_width="match_parent"
             android:layout_height="match_parent">

    <ListView
            android:id="@android:id/list"
            android:layout_width="match_parent"
            android:layout_height="match_parent" />

    <com.melnykov.fab.FloatingActionButton
            android:id="@+id/fab"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="bottom|right"
            android:layout_margin="16dp"
            android:src="@drawable/ic_action_content_new"
            fab:fab_type="normal"
            fab:fab_shadow="true"
            fab:fab_colorNormal="@color/primary"
            fab:fab_colorPressed="@color/primary_pressed"
            fab:fab_colorRipple="@color/ripple" />
</FrameLayout>
```

### Manual Implementations

Instead of using a library we can also develop the floating action buttons manually. For manual implementations of the floating action button, see this [big nerd ranch guide](http://www.bignerdranch.com/blog/floating-action-buttons-in-android-l/) and the [survivingwithandroid walkthrough](http://www.survivingwithandroid.com/2014/09/android-floating-action-button.html).

## References

* <http://www.google.com/design/spec/components/buttons.html#buttons-floating-action-button>
* <https://github.com/makovkastar/FloatingActionButton>
* <https://github.com/futuresimple/android-floating-action-button>
* <http://www.bignerdranch.com/blog/floating-action-buttons-in-android-l/>
* <http://prolificinteractive.com/blog/2014/07/24/android-floating-action-button-aka-fab/>
* <http://www.survivingwithandroid.com/2014/09/android-floating-action-button.html>