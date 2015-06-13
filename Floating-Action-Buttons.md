## Overview

[Floating action buttons](http://www.google.com/design/spec/components/buttons.html#buttons-floating-action-button) (or **FAB**) are: “A special case of promoted actions. They are distinguished by a circled icon floating above the UI and have special motion behaviors, related to morphing, launching, and its transferring anchor point.” 

For example, if we are using an email app and we are listing the inbox folder, the promoted action might be composing a new message.

<img src="https://github.com/makovkastar/FloatingActionButton/raw/master/art/demo.gif" width="185" alt="FAB1" />&nbsp;
<img src="http://i.imgur.com/SBbLXo2.png" width="200" alt="FAB2" />

The floating action button represents **the primary action** within a particular screen. More info and use cases of the FAB button from Google’s official [design specs can be found here](http://www.google.com/design/spec/components/buttons.html#buttons-floating-action-button).

## Usage

Google made available during Google I/O 2015 a support library to create floating action buttons library.  In the past, third-party libraries such as [makovkastar/FloatingActionButton](https://github.com/makovkastar/FloatingActionButton) and [futuresimple/android-floating-action-button](https://github.com/futuresimple/android-floating-action-button) had to be used.

### Design Support Library

Make sure to follow the [[Design Support Library]] setup instructions first.  

You should now be able to add the `android.support.design.widget.FloatingActionButton` view to the layout.  The `src` attribute references the icon that should be used for the floating action.  

```xml
       <android.support.design.widget.FloatingActionButton
          android:src="@drawable/ic_done"
          app:fabSize="normal"
          android:layout_width="wrap_content"
          android:layout_height="wrap_content" />
```

In addition, assuming `xmlns:app="http://schemas.android.com/apk/res-auto` is declared as namespace the top of your layout, you can also define a custom attribute [`fabSize`](http://developer.android.com/reference/android/support/design/widget/FloatingActionButton.html#attr_android.support.design:fabSize) that can reference whether the button should be `normal` or `mini`. 

To place the floating action button, you will use [CoordinatorLayout](http://developer.android.com/reference/android/support/design/widget/CoordinatorLayout.html).   A CoordinatorLayout helps facilitate interactions between views contained within it, which will be useful later to describe how to animate the button depending on scroll changes.  For now we can take advantage of a feature in CoordinatorLayout that allows us to hover one element over another.  We simply need to have the ListView and FloatingActionButton contained within the CoordinatorLayout and use the `layout_anchor` and `layout_anchorGravity` attributes. 

```xml
<android.support.design.widget.CoordinatorLayout
    android:id="@+id/main_content"
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

          <ListView
              android:id="@+id/lvToDoList"
              android:layout_width="match_parent"
              android:layout_height="match_parent"></ListView>

          <android.support.design.widget.FloatingActionButton
              android:layout_width="wrap_content"
              android:layout_height="wrap_content"
              android:layout_gravity="bottom|right"
              android:layout_margin="16dp"
              android:src="@drawable/ic_done"
              app:layout_anchor="@id/lvToDoList"
              app:layout_anchorGravity="bottom|right|end" />

</android.support.design.widget.CoordinatorLayout>
```

The button should be placed in the bottom right corner of the screen. The recommended margin for the bottom is **16dp for phones and 24dp for tablets**.    In the example above, 16dp was used.

The actual drawable size should be **24dp** according to the Google design specs.  

<a href="http://www.google.com/design/spec/components/buttons.html#buttons-floating-action-button"><img src="http://material-design.storage.googleapis.com/publish/v_2/material_ext_publish/0Bx4BSt6jniD7UGxlcmdDZFRZYU0/patterns_actions_fab3.png" width="300"></a>

### Animating the Floating Action Button

When a user scrolls down a page, the floating action button should disappear.  Once the page scrolls to the top, it should reappear.   

<img src="http://i.imgur.com/CEtWLA8.gif"/>

To animate this part, you will need to take advantage of [CoordinatorLayout](https://developer.android.com/reference/android/support/design/widget/CoordinatorLayout.html), which helps choreograph animations between views defined within this layout.  

#### Converting from ListView to RecyclerView

Currently, you need to convert your ListViews to use [RecyclerView](http://developer.android.com/reference/android/support/v7/widget/RecyclerView.html). RecyclerView is the successor to ListView as described in [this section](http://guides.codepath.com/android/Using-the-RecyclerView#compared-to-listview).  There is no support built-in for CoordinatorLayout to work with ListView according to [this Google post](https://plus.google.com/101784949561498190574/posts/KPbsTY4NANx). You can review 
this [guide](http://guides.codepath.com/android/Using-the-RecyclerView) to help make the transition.

```xml
<android.support.v7.widget.RecyclerView
         android:id="@+id/lvToDoList"
         android:layout_width="match_parent"
         android:layout_height="match_parent"
</android.support.v7.widget.RecyclerView>
```

You also must upgrade to the v22 version of RecyclerView.  Previous v21 versions will not work with CoordinatorLayout.  Make sure to bump your `build.gradle` file:

```gradle
    compile 'com.android.support:recyclerview-v7:22.2.0'
```

#### Using CoordinatorLayout

Next, you must implement a [CoordinatorLayout Behavior](https://developer.android.com/reference/android/support/design/widget/CoordinatorLayout.Behavior.html) for the Floating Action Button. This class will be used to define how the button should respond to other views contained within the same CoordinatorLayout.  

Create a file called `ScrollAwareFABBehavior.java` that extends from [FloatingActionButton.Behavior](https://developer.android.com/reference/android/support/design/widget/FloatingActionButton.Behavior.html).  Currently, the default behavior is used for the Floating Action Button to make room for the Snackbar as shown in [this video](http://material-design.storage.googleapis.com/publish/material_v_3/material_ext_publish/0B6Okdz75tqQsLWFucDNlYWEyeW8/components_snackbar_usage_fabdo_002.webm).    We want to extend this behavior to signal that we wish to handle scroll events in the vertical direction:

```java
public class ScrollAwareFABBehavior extends FloatingActionButton.Behavior {

    @Override
    public boolean onStartNestedScroll(CoordinatorLayout coordinatorLayout,
            FloatingActionButton child, View directTargetChild, View target, int nestedScrollAxes) {
        return nestedScrollAxes == ViewCompat.SCROLL_AXIS_VERTICAL || 
            super.onStartNestedScroll(coordinatorLayout, child, directTargetChild, target,
            nestedScrollAxes);
    }

}
```

Because scrolling will be handled by this class, a separate method [onNestedScroll()](https://developer.android.com/reference/android/support/design/widget/CoordinatorLayout.Behavior.html#onNestedScroll(android.support.design.widget.CoordinatorLayout, V, android.view.View, int, int, int, int)) will be called.   We can check the Y position and determine whether to animate in or out the button:

```java
public class ScrollAwareFABBehavior extends FloatingActionButton.Behavior {
    // ...

    @Override
    public void onNestedScroll(CoordinatorLayout coordinatorLayout, FloatingActionButton child,
            View target, int dxConsumed, int dyConsumed, int dxUnconsumed, int dyUnconsumed) {
        super.onNestedScroll(coordinatorLayout, child, target, dxConsumed, dyConsumed, dxUnconsumed,
                dyUnconsumed);

        if (dyConsumed > 0 && !this.mIsAnimatingOut && child.getVisibility() == View.VISIBLE) {
            animateOut(child);
        } else if (dyConsumed < 0 && child.getVisibility() != View.VISIBLE) {
            animateIn(child);
        }
    }

    // ...
}
```

The `FloatingActionButton.Behavior` base class already has an `animateIn()` and `animateOut()` method, as well as sets a private variable `mIsAnimatingOut`.  Because these methods and variables are declared as private however, we must reimplement the animation methods for now.

```java
public class ScrollAwareFABBehavior extends FloatingActionButton.Behavior {

    private static final android.view.animation.Interpolator INTERPOLATOR = 
        new FastOutSlowInInterpolator();
    private boolean mIsAnimatingOut = false;

    // Same animation that FloatingActionButton.Behavior uses to 
    // hide the FAB when the AppBarLayout exits
    private void animateOut(final FloatingActionButton button) {
        if (Build.VERSION.SDK_INT >= 14) {
           ViewCompat.animate(button).scaleX(0.0F).scaleY(0.0F).alpha(0.0F)
                    .setInterpolator(INTERPOLATOR).withLayer()
                    .setListener(new ViewPropertyAnimatorListener() {
                        public void onAnimationStart(View view) {
                            ScrollAwareFABBehavior.this.mIsAnimatingOut = true;
                        }

                        public void onAnimationCancel(View view) {
                            ScrollAwareFABBehavior.this.mIsAnimatingOut = false;
                        }

                        public void onAnimationEnd(View view) {
                            ScrollAwareFABBehavior.this.mIsAnimatingOut = false;
                            view.setVisibility(View.GONE);
                        }
                    }).start();
        } else {
            Animation anim = AnimationUtils.loadAnimation(button.getContext(), R.anim.fab_out);
            anim.setInterpolator(INTERPOLATOR);
            anim.setDuration(200L);
            anim.setAnimationListener(new Animation.AnimationListener() {
                public void onAnimationStart(Animation animation) {
                    ScrollAwareFABBehavior.this.mIsAnimatingOut = true;
                }

                public void onAnimationEnd(Animation animation) {
                    ScrollAwareFABBehavior.this.mIsAnimatingOut = false;
                    button.setVisibility(View.GONE);
                }

                @Override
                public void onAnimationRepeat(final Animation animation) {
                }
            });
            button.startAnimation(anim);
        }
    }

    // Same animation that FloatingActionButton.Behavior 
    // uses to show the FAB when the AppBarLayout enters
    private void animateIn(FloatingActionButton button) {
        button.setVisibility(View.VISIBLE);
        if (Build.VERSION.SDK_INT >= 14) {
            ViewCompat.animate(button).scaleX(1.0F).scaleY(1.0F).alpha(1.0F)
                    .setInterpolator(INTERPOLATOR).withLayer().setListener(null)
                    .start();
        } else {
            Animation anim = AnimationUtils.loadAnimation(button.getContext(), R.anim.fab_in);
            anim.setDuration(200L);
            anim.setInterpolator(INTERPOLATOR);
            button.startAnimation(anim);
        }
    }
}
```

The final step is to associate this CoordinatorLayout Behavior to the Floating Action Button.  We can define it within the XML declaration as a custom attribute `app:layout_behavior`:

```xml
<android.support.design.widget.FloatingActionButton    
    app:layout_behavior="com.codepath.floatingactionbuttontest.ScrollAwareFABBehavior" />
```

Because we are defining this behavior statically within the XML, we must also implement a constructor to enable layout inflation to work correctly. 

```java
public class ScrollAwareFABBehavior extends FloatingActionButton.Behavior {
    // ...

    public ScrollAwareFABBehavior(Context context, AttributeSet attrs) {
        super();
    }

   // ...
}
```

If you forget to implement this last step, you will see `Could not inflate Behavior subclass` error messages. See this [example code](https://github.com/ianhanniballake/cheesesquare/commit/aefa8b57e61266e4ad51bef36e669d69f7fd749c) for the full set of changes.

*Note*: Normally when implementing `CoordinatorLayout` behaviors, we need to implement [layoutDependsOn()](https://developer.android.com/reference/android/support/design/widget/CoordinatorLayout.Behavior.html#layoutDependsOn(android.support.design.widget.CoordinatorLayout, V, android.view.View)) and [onDependentViewChanged()](https://developer.android.com/reference/android/support/design/widget/CoordinatorLayout.Behavior.html#onDependentViewChanged(android.support.design.widget.CoordinatorLayout, V, android.view.View)), which are used to track changes in other views contained within the CoordinatorLayout.  Since we only need to monitor scroll changes, we use the existing behavior defined for the Floating Action Button, which is currently implemented to track changes for Snackbars and AppBarLayout as discussed in this [blog post](http://android-developers.blogspot.com/2015/05/android-design-support-library.html).

Note that there is a known bug that triggers a `NullPointerException` with RecyclerView when scrolling too fast documented [here] (https://code.google.com/p/android/issues/detail?id=174981).  It will be fixed in the next release for this library.

### With FloatingActionButton (Third-Party)

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

### Manual Implementations

Instead of using a library we can also develop the floating action buttons manually. For manual implementations of the floating action button, see this [big nerd ranch guide](http://www.bignerdranch.com/blog/floating-action-buttons-in-android-l/) and the [survivingwithandroid walkthrough](http://www.survivingwithandroid.com/2014/09/android-floating-action-button.html).

## References

* <http://www.google.com/design/spec/components/buttons.html#buttons-floating-action-button>
* <https://github.com/makovkastar/FloatingActionButton>
* <https://github.com/futuresimple/android-floating-action-button>
* <http://www.bignerdranch.com/blog/floating-action-buttons-in-android-l/>
* <http://prolificinteractive.com/blog/2014/07/24/android-floating-action-button-aka-fab/>
* <http://www.survivingwithandroid.com/2014/09/android-floating-action-button.html>
* <https://android.googlesource.com/platform/frameworks/support.git/+/master/design/src/android/support/design/widget/FloatingActionButton.java>
* <https://android.googlesource.com/platform/frameworks/support.git/+/master/design/res/values/styles.xml>