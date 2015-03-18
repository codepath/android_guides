## Overview

Floating action buttons (or **FAB**) are: “A special case of promoted actions. They are distinguished by a circled icon floating above the UI and have special motion behaviors, related to morphing, launching, and its transferring anchor point.” For example, if we are using an email app and we are listing the inbox folder, one promoted action can be a new mail.

<img src="https://github.com/makovkastar/FloatingActionButton/raw/master/art/demo.gif" width="185" alt="FAB1" />&nbsp;
<img src="http://i.imgur.com/SBbLXo2.png" width="200" alt="FAB2" />

The floating action button represents **the primary action** within an application. More info and use cases of the FAB button can be found in Google’s official [design specs found here](http://www.google.com/design/spec/patterns/promoted-actions.html#promoted-actions-floating-action-button).

## Usage

There are many popular libraries for making the implementation of floating actions quick and easy. The two most popular ones are [makovkastar/FloatingActionButton](https://github.com/makovkastar/FloatingActionButton) and [futuresimple/android-floating-action-button](https://github.com/futuresimple/android-floating-action-button).

### With FloatingActionButton

Using [makovkastar/FloatingActionButton](https://github.com/makovkastar/FloatingActionButton) library makes floating buttons quite simple to setup. See the [library readme](https://github.com/makovkastar/FloatingActionButton/blob/master/README.md) and the [sample code](https://github.com/makovkastar/FloatingActionButton/tree/master/sample/src/main/java/com/melnykov/fab/sample) for reference.

First, add as a dependency to your `app/build.gradle`:

```gradle
dependencies {
    compile 'com.melnykov:floatingactionbutton:1.2.0'
}
```

Next, let's add the `com.melnykov.fab.FloatingActionButton` to our layout XML:

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

Note the addition of the `xmlns:fab` to the attributes of the root layout. The button should be placed in the bottom right corner of the screen. Next, we can optionally associate the FAB with a `ListView` or `RecyclerView` so the button will hide as the list is scrolled down and revealed as the list is scrolled up:

```java
ListView listView = (ListView) findViewById(android.R.id.list);
FloatingActionButton fab = (FloatingActionButton) findViewById(R.id.fab);
fab.attachToListView(listView);
```

#### Adjust Button Type

We can adjust the FAB button type to "normal" or "mini":

```xml
<com.melnykov.fab.FloatingActionButton
    ...
    fab:fab_type="mini" />
```

#### Show or Hiding FAB

Show/hide the button explicitly:

```java
fab.show();
fab.hide();

fab.show(false); // Show without an animation
fab.hide(false); // Hide without an animation
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
    public void onScroll(AbsListView view, int firstVisibleItem, int visibleItemCount, int totalItemCount) {
        Log.d("ListViewFragment", "onScroll()");
    }
});
```

### Manual Implementations

Instead of using a library we can also develop the floating action buttons manually. For manual implementations of the floating action button, see this [big nerd ranch guide](http://www.bignerdranch.com/blog/floating-action-buttons-in-android-l/) and the [survivingwithandroid walkthrough](http://www.survivingwithandroid.com/2014/09/android-floating-action-button.html)

## References

* <http://www.google.com/design/spec/components/buttons.html#buttons-floating-action-button>
* <https://github.com/makovkastar/FloatingActionButton>
* <https://github.com/futuresimple/android-floating-action-button>
* <http://www.bignerdranch.com/blog/floating-action-buttons-in-android-l/>
* <http://prolificinteractive.com/blog/2014/07/24/android-floating-action-button-aka-fab/>
* <http://www.survivingwithandroid.com/2014/09/android-floating-action-button.html>