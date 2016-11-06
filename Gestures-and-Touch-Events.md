## Overview

Gesture recognition and handling touch events is an important part of developing user interactions. Handling standard events such as clicks, long clicks, key presses, etc are very basic and handled in other guides. This guide is focused on handling other more specialized gestures such as:

 * Swiping in a direction
 * Double tapping for zooming
 * Pinch to zoom in or out
 * Dragging and dropping
 * Effects while scrolling a list

You can see a visual guide of common gestures on the [gestures design patterns](http://developer.android.com/design/patterns/gestures.html) guide. See the new Material Design information about the [touch mechanics](http://www.google.com/design/spec/patterns/gestures.html#gestures-touch-mechanics) behind gestures too.

## Usage

### Handling Touches

At the heart of all gestures is the [onTouchListener](http://developer.android.com/reference/android/view/View.OnTouchListener.html) and the `onTouch` method which has access to [MotionEvent](http://developer.android.com/reference/android/view/MotionEvent.html) data. Every view has an `onTouchListener` which can be specified:

```java
myView.setOnTouchListener(new OnTouchListener() {
    @Override
    public boolean onTouch(View v, MotionEvent event) {
        // Interpret MotionEvent data
        // Handle touch here
        return true;
    }
});
```

Each `onTouch` event has access to the [MotionEvent](http://developer.android.com/reference/android/view/MotionEvent.html) which describe movements in terms of an **action code** and a **set of axis values**. The action code specifies the state change that occurred such as a pointer going down or up. The axis values describe the position and other movement properties:

 * `getAction()` - Returns an integer constant such as `MotionEvent.ACTION_DOWN`, `MotionEvent.ACTION_MOVE`, and `MotionEvent.ACTION_UP`
 * `getX()` - Returns the x coordinate of the touch event
 * `getY()` - Returns the y coordinate of the touch event

Note that every touch event can be **propagated through the entire affected view hierarchy**. Not only can the touched view respond to the event but every layout that contains the view has an opportunity as well. Refer to the [[understanding touch events|Gestures-and-Touch-Events#understanding-touch-events]] section for a detailed overview.

### Handling Multi Touch Events

Note that `getAction()` normally includes information about both the action as well as the pointer index.  In single-touch events, there is only one pointer (set to 0), so no [bitmap mask](http://developer.android.com/reference/android/view/MotionEvent.html#ACTION_MASK) is needed.  In multiple touch events (i.e pinch open or pinch close), however, there are multiple fingers involved and a non-zero pointer index may be included when calling `getAction()`. As a result, there are other methods that should be used to determine the touch event:

 * `getActionMasked()` - extract the action event without the pointer index
 * `getActionIndex()` - extract the pointer index used

The events associated with other pointers usually start with `MotionEvent.ACTION_POINTER` such as  `MotionEvent.ACTION_POINTER_DOWN` and `MotionEvent.ACTION_POINTER_UP`.  The `getPointerCount()` on the MotionEvent can be used to determine how many pointers are active in this touch sequence.

## Gesture Detectors

Within an `onTouch` event, we can then use a [GestureDetector](http://developer.android.com/reference/android/view/GestureDetector.html) to understand gestures based on a series of motion events. Gestures are often used for user interactions within an app. Let's take a look at how to implement common gestures.

For easy gesture detection using a third-party library, check out the popular [Sensey](https://github.com/nisrulz/Sensey) library which greatly simplifies the process of attaching multiple gestures to your views.

### Double Tapping

You can enable double tap events for any view within your activity using the [OnDoubleTapListener](https://gist.github.com/nesquena/b2f023bb04190b2653c7). First, copy the code for `OnDoubleTapListener` into your application and then you can apply the listener with:

```java
myView.setOnTouchListener(new OnDoubleTapListener(this) {
  @Override
  public void onDoubleTap(MotionEvent e) {
    Toast.makeText(MainActivity.this, "Double Tap", Toast.LENGTH_SHORT).show();
  }
});
```

Now that view will be able to respond to a double tap event and you can handle the event accordingly.

### Swipe Gesture Detection

Detecting finger swipes in a particular direction is best done using the built-in [onFling](http://developer.android.com/reference/android/view/GestureDetector.OnGestureListener.html#onFling\(android.view.MotionEvent, android.view.MotionEvent, float, float\)) event in the `GestureDetector.OnGestureListener`. 

A helper class that makes handling swipes as easy as possible can be found in the [OnSwipeTouchListener](https://gist.github.com/nesquena/ed58f34791da00da9751) class. Copy the `OnSwipeTouchListener` class to your own application and then you can use the listener to manage the swipe events with:

```java
myView.setOnTouchListener(new OnSwipeTouchListener(this) {
  @Override
  public void onSwipeDown() {
    Toast.makeText(MainActivity.this, "Down", Toast.LENGTH_SHORT).show();
  }
  
  @Override
  public void onSwipeLeft() {
    Toast.makeText(MainActivity.this, "Left", Toast.LENGTH_SHORT).show();
  }
  
  @Override
  public void onSwipeUp() {
    Toast.makeText(MainActivity.this, "Up", Toast.LENGTH_SHORT).show();
  }
  
  @Override
  public void onSwipeRight() {
    Toast.makeText(MainActivity.this, "Right", Toast.LENGTH_SHORT).show();
  }
});
```

With that code in place, swipe gestures should be easily manageable. 

#### RecyclerView Swipe Detection

If you intend to implement pull-to-refresh capabilities in your RecyclerView, you can leverage the built-in `SwipeRefreshLayout` as described [[here|Implementing-Pull-to-Refresh-Guide#recyclerview-with-swiperefreshlayout]].  If you wish to handle your own swipe detection, you can use the new `OnFlingListener` as described in [[this section|Using-the-RecyclerView#swipe-detection]].

#### ListView Swipe Detection

If you are interested in having a ListView that recognizes swipe gestures for each item, consider using the popular third-party library [android-swipelistview](https://github.com/47deg/android-swipelistview) which is a ListView replacement that supports swipe-eable items. Once setup, you can configure a layout that will appear when the item is swiped. 

Check out the [swipelistview](https://github.com/47deg/android-swipelistview#demo) project for more details but the general usage looks like:

```xml
 <com.fortysevendeg.swipelistview.SwipeListView
 xmlns:swipe="http://schemas.android.com/apk/res-auto"
 android:id="@+id/example_lv_list"
 android:listSelector="#00000000"
 android:layout_width="match_parent"
 android:layout_height="wrap_content"
 swipe:swipeFrontView="@+id/front"
 swipe:swipeBackView="@+id/back"
 swipe:swipeActionLeft="reveal"
 swipe:swipeActionRight="dismiss"
 swipe:swipeMode="both"
 swipe:swipeCloseAllItemsWhenMoveList="true"
 />
```

and then define the individual list item layout with:

```xml
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
             android:layout_width="match_parent"
             android:layout_height="match_parent">
    <LinearLayout
            android:id="@+id/back"
            android:tag="back"
            style="@style/ListBackContent">
        <Button
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:id="@+id/example_row_b_action_1"
                style="@style/ListButtonAction"
                android:text="@string/open"/>
    </LinearLayout>
    <RelativeLayout
            android:orientation="vertical"
            android:id="@+id/front"
            android:tag="front"
            style="@style/ListFrontContent">
        <ImageView
                style="@style/ListImage"
                android:id="@+id/example_row_iv_image"/>
    </RelativeLayout>
</FrameLayout>
```

Now `front` will be displayed by default and if I swipe left on an item, then the `back` will be displayed for that item. This simplifies swipes for the common case of menus for a ListView.

Another more recent alternative is the [AndroidSwipeLayout](https://github.com/daimajia/AndroidSwipeLayout) library which can be more flexible and is worth checking out as an alternative.

### Pinch to Zoom

Supporting Pinch to Zoom in and out is fairly straightforward thanks to the [ScaleGestureDetector](http://developer.android.com/reference/android/view/ScaleGestureDetector.html) class. Easiest way to manage pinch events is to subclass a view and manage the pinch event from within:

```java
public class ScaleableTextView extends TextView 
        implements OnTouchListener, OnScaleGestureListener {

  ScaleGestureDetector mScaleDetector = 
      new ScaleGestureDetector(getContext(), this);

  public ScaleableTextView(Context context, AttributeSet attrs) {
    super(context, attrs);
  }

  @Override
  public boolean onScale(ScaleGestureDetector detector) {
    // Code for scale here
    return true;
  }

  @Override
  public boolean onScaleBegin(ScaleGestureDetector detector) {
    // Code for scale begin here
    return true;
  }

  @Override
  public void onScaleEnd(ScaleGestureDetector detector) {
    // Code for scale end here
  }

  @Override
  public boolean onTouch(View v, MotionEvent event) {
    if (mScaleDetector.onTouchEvent(event))
      return true;
    return super.onTouchEvent(event);
  }
}
```

Using the `ScaleGestureDetector` makes managing this fairly straightforward. 

#### Zooming Image View

One of the most common use cases for a pinch or pannable view is for an ImageView that displays a Photo which can be zoomed or panned around on screen similar to the Facebook client. To achieve the zooming image view, rather than developing this yourself, be sure to check out the [PhotoView](https://github.com/chrisbanes/PhotoView) third-party library. Using the PhotoView just requires the XML:

```xml
<uk.co.senab.photoview.PhotoView
    android:id="@+id/iv_photo"
    android:layout_width="match_parent"
    android:layout_height="match_parent" />
```

and then in the Java:

```java
// Any implementation of ImageView can be used!
mImageView = (ImageView) findViewById(R.id.iv_photo);
// Set the image bitmap
mImageView.setImageDrawable(someBitmap);
// Setup view attacher
PhotoViewAttacher mAttacher = new PhotoViewAttacher(mImageView);
```

Check out the [PhotoView](https://github.com/chrisbanes/PhotoView) readme and sample for more details. You can also check the [TouchImageView](https://github.com/MikeOrtiz/TouchImageView) library which is a nice alternative.

### Scrolling Lists

Scrolling is a common gesture associated with lists of items within a `ListView` or `RecyclerView`. Often the scrolling is associated with the hiding of certain elements (toolbar) or the shrinking or morphing of elements such as a parallax header. If you are using a `RecyclerView`, check out the [addOnScrollListener](https://developer.android.com/reference/android/support/v7/widget/RecyclerView.html#addOnScrollListener\(android.support.v7.widget.RecyclerView.OnScrollListener\)). With a `ListView`, we can use the [setOnScrollListener](http://developer.android.com/reference/android/widget/AbsListView.html#setOnScrollListener\(android.widget.AbsListView.OnScrollListener\)) instead. 

With Android "M" and the release of the [[Design Support Library]], the [CoordinatorLayout](https://developer.android.com/reference/android/support/design/widget/CoordinatorLayout.html) was introduced which enables handling changes associated with the scrolling of a `RecyclerView`. Review the [[Handling Scrolls with CoordinatorLayout]] guide for a detailed breakdown of how to manage scrolls using this new layout to collapse the toolbar or hide and reveal header content.  

### Dragging and Dropping

Dragging and dropping views is not particularly difficult to do thanks to the [OnDragListener](http://developer.android.com/reference/android/view/View.OnDragListener.html) built in since API 11. Unfortunately, to support gingerbread managing drag and drop becomes much more manual as you have to implement it using the `onTouch` handlers. With API 11 and above, you can leverage the built in drag handling.

First, we want to attach an `onTouch` handler on the **views that are draggable** which will start the drag by creating a `DragShadow` with the [DragShadowBuilder](http://developer.android.com/reference/android/view/View.DragShadowBuilder.html) which is then dragged around the Activity once [startDrag](http://developer.android.com/reference/android/view/View.html#startDrag\(android.content.ClipData, android.view.View.DragShadowBuilder, java.lang.Object, int\)) is invoked on the view:

```java
// This listener is attached to the view that should be draggable
draggableView.setOnTouchListener(new OnTouchListener() {
    public boolean onTouch(View view, MotionEvent motionEvent) {
        if (motionEvent.getAction() == MotionEvent.ACTION_DOWN) {
            // Construct draggable shadow for view
	    DragShadowBuilder shadowBuilder = new View.DragShadowBuilder(view);
            // Start the drag of the shadow
	    view.startDrag(null, shadowBuilder, view, 0);
            // Hide the actual view as shadow is being dragged
	    view.setVisibility(View.INVISIBLE);
	    return true;
	} else {
	    return false;
	}
   }
});
```

If we want to add "drag" or "drop" events, we should create a `DragListener` that is **attached to a drop zone** for the draggable object. We hook up the listener and manage the different dragging and dropping events for the zone:

```java
// This listener is attached to the view that should be a drop target
viewDropZone.setOnDragListener(new OnDragListener() {
  // Drawable for when the draggable enters the drop target
  Drawable enteredZoneBackground = getResources().getDrawable(R.drawable.shape_border_green);
  // Drawable for the default background of the drop target
  Drawable defaultBackground = getResources().getDrawable(R.drawable.shape_border_red);

  @Override
  public boolean onDrag(View v, DragEvent event) {
      // Get the dragged view being dropped over a target view
      final View draggedView = (View) event.getLocalState();
      switch (event.getAction()) {
      case DragEvent.ACTION_DRAG_STARTED:
          // Signals the start of a drag and drop operation.
          // Code for that event here
          break;
      case DragEvent.ACTION_DRAG_ENTERED:
          // Signals to a View that the drag point has 
          // entered the bounding box of the View. 
          v.setBackground(enteredZoneBackground);
          break;
      case DragEvent.ACTION_DRAG_EXITED:
          // Signals that the user has moved the drag shadow 
          // outside the bounding box of the View. 
          v.setBackground(defaultBackground);
          break;
      case DragEvent.ACTION_DROP:
          // Signals to a View that the user has released the drag shadow, 
          // and the drag point is within the bounding box of the View. 
          // Get View dragged item is being dropped on
          View dropTarget = v;
          // Make desired changes to the drop target below
          dropTarget.setTag("dropped");
          // Get owner of the dragged view and remove the view (if needed)
          ViewGroup owner = (ViewGroup) draggedView.getParent();
          owner.removeView(draggedView);
          break;
      case DragEvent.ACTION_DRAG_ENDED:
          // Signals to a View that the drag and drop operation has concluded.
          // If event result is set, this means the dragged view was dropped in target
          if (event.getResult()) { // drop succeeded
              v.setBackground(enteredZoneBackground);
          } else { // drop did not occur
              // restore the view as visible
              draggedView.post(new Runnable() {
                  @Override
                  public void run() {
                      draggedView.setVisibility(View.VISIBLE);
                  }
              });
              // restore drop zone default background
              v.setBackground(defaultBackground);
          }
      default:
          break;
      }
      return true;
  }
});
```

Check out the [vogella dragging tutorial](http://www.vogella.com/articles/AndroidDragAndDrop/article.html) or [the javapapers dragging tutorial](http://javapapers.com/android/android-drag-and-drop/) for a detailed look at handling dragging and dropping. Read the official [drag and drop](http://developer.android.com/guide/topics/ui/drag-drop.html) guide for a more detail overview. 

### Shake Detection

Detecting when the device is shaked requires using the sensor data to determine movement. We can whip up a special listener which manages this shake recognition for us. First, copy the [ShakeListener](https://gist.github.com/nesquena/5d1922a5a50fcfb4436a) into your project. Now, we can implement `ShakeListener.Callback` in any activity:

```java
public class MainActivity extends Activity 
    implements ShakeListener.Callback {

  @Override
  public void shakingStarted() {
    // Code on started here
  }
  
  @Override
  public void shakingStopped() {
    // Code on stopped here
  }
}
```

Now we just have to implement the expected behavior for the shaking event in the two methods from the callback.

### MultiTouch Events

For additional multi-touch events such as "rotation" of fingers, finger movement events, etc., be sure to check out libraries such as [Sensey](https://github.com/nisrulz/Sensey) and [multitouch-gesture-detectors](https://github.com/Almeros/android-gesture-detectors) third-party library. Read the [documentation](http://code.almeros.com/android-multitouch-gesture-detectors#.UmTf0JQ6VZ8) for more details about how to handle multi-touch gestures. Also, for a more generic approach, read the official [multitouch guide](http://developer.android.com/training/gestures/multi.html).  See this [blog post](http://android-developers.blogspot.com/2010/06/making-sense-of-multitouch.html) for more details about how multi-touch events work.

## Understanding Touch Events

This section briefly summarizes touch propagation within the view hierarchy. There are three distinct touch related methods which will be outlined below:

| Order | Method                 | Invoked On  |    Description                                    |
| ----- | -------------------    | ----------- | ----------------------                            |
| 1st   | dispatchTouchEvent     | A, VG, V    | Dispatch touch events to affected child views     | 
| 2nd   | onInterceptTouchEvent  | VG          | Intercept touch events before passing to children | 
| 3rd   | onTouchEvent           | VG, V       | Handle the touch event and stop propogation       | 
 
"Order" above defines which of these methods gets invoked first when a touch is initiated. Note that above in "invoked on" **A** represents `Activity`, **VG** is `ViewGroup`, **V** is `View` describing where the method is invoked during a touch event.

Keep in mind that a **gesture is simply a series of touch events** as follows:

1. **DOWN**. Begins with a single DOWN event when the user touches the screen
2. **MOVE**. Zero or more MOVE events when the user moves the finger around
3. **UP**. Ends with a single UP (or CANCEL) event when the user releases the screen

These touch events trigger a very particular set of method invocations on affected views. To further illustrate, assume a "View C" is contained within a "ViewGroup B" which is then contained within "ViewGroup A" such as:

<a href="http://balpha.de/2013/07/android-development-what-i-wish-i-had-known-earlier/"><img src="https://i.imgur.com/mLB5ACK.png" /></a>

Review this example carefully as all sections below will be referring to the example presented here.

### Touch Event Propagation

When a touch DOWN event occurs on "View C" and **the view has registered a touch listener**, the following series of actions happens as the [onTouchEvent](http://developer.android.com/reference/android/view/View.html#onTouchEvent%28android.view.MotionEvent%29) is triggered on the view:

* The DOWN touch event is passed to "View C" `onTouchEvent` and the **boolean result** of **TRUE** or **FALSE** determines if the action is captured.
* **Returning TRUE:** If the "View C" `onTouchEvent` returns true then this view captures the gesture
  * Because "View C" returns true and is handling the gesture, the event is not passed to "ViewGroup B"'s nor "ViewGroup A"'s `onTouchEvent` methods.
  * Because View C says it's handling the gesture, any additional events in this gesture will also be passed to "View C"'s `onTouchEvent` method until the gesture ends with an UP touch event. 
* **Returning FALSE:** If the "View C" `onTouchEvent` returns false then the gesture is propagated upwards
  - The DOWN event is passed upward to "ViewGroup B" `onTouchEvent` method, and the boolean result determines if the event continues to propagate.
  - If "ViewGroup B" doesn't return true then the event is passed upward to "ViewGroup A" `onTouchEvent`

### Intercepting Touches

In addition to the `onTouchEvent`, there is also a separate [onInterceptTouchEvent](http://developer.android.com/reference/android/view/ViewGroup.html#onInterceptTouchEvent%28android.view.MotionEvent%29) that exists **only on ViewGroups** such as layouts. Before the `onTouchEvent` is called on any `View`, all its ancestors are first given the chance to intercept this event. In other words, **a containing layout can choose to steal the event** from a touched view **before the view even receives the event**. With this added in, the series of events from above become:

* The DOWN event on "View C" is first passed to "ViewGroup A" `onInterceptTouchEvent`, which can return false or true depending on if it wants to intercept the touch. 
* If false, the event is then passed to "ViewGroup B" `onInterceptTouchEvent` can also return false or true depending on if it wants to intercept the touch. 
* Next the DOWN event is passed to "View C" `onTouchEvent` which can return true to handle the event.
* Additional touch events within the same gesture are still passed to A and B's `onInterceptTouchEvent` before being called on "View C" `onTouchEvent` even if the ancestors chose not to previously intercept.

The takeaway here is that **any viewgroup can choose to implement `onInterceptTouchEvent`** and effectively decide to steal touch events from any of the child views. If the children choose not respond to the touch once received, then the touch event is propagated back upwards through the `onTouchEvent` of each of the containing `ViewGroup` as described in the previous section.

### Touch Dispatch

As the chart much earlier shows, all of this above behavior of touch and interception is being managed by "dispatchers" via the [dispatchTouchEvent](http://developer.android.com/reference/android/view/ViewGroup.html#dispatchTouchEvent%28android.view.MotionEvent%28) method invoked on each view. Revealing this behavior, as soon as a touch event occurs on top of "View C", the following dispatching occurs:

1. The `Activity.dispatchTouchEvent()` is called for the current activity containing the view. 
2. If the activity chooses not to "consume" the event (and stop propagation), the event is passed to the "ViewGroup A" `dispatchTouchEvent` since A is the outermost viewgroup affected by the touch.
3. "ViewGroup A" `dispatchTouchEvent` will trigger the "ViewGroup A" `onInterceptTouchEvent` first and if that method chooses not to intercept, the touch event is then sent to the "ViewGroup B" `dispatchTouchEvent`.
4. In turn, the "ViewGroup B" `dispatchTouchEvent` will trigger the "ViewGroup B" `onInterceptTouchEvent` and if that method chooses not to intercept, the touch event is then sent to the "ViewGroup C" `dispatchTouchEvent`.
5. "ViewGroup C" `dispatchTouchEvent` then invokes the "ViewGroup C" `onTouchEvent`.

To recap, the `dispatchTouchEvent` is called at every level of the way starting with the `Activity`. The dispatcher is responsible for identifying which methods to invoke next. On a `ViewGroup`, the dispatcher triggers the `onInterceptTouchEvent`, before triggering the `dispatchTouchEvent` on the next child in the view hierarchy. 

### Further Reading

The explanation above has been simplified and abridged for clarity. For additional reading on the touch propagation system, please review [this detailed article](http://codetheory.in/understanding-android-input-touch-events/) as well as this doc on [ViewGroup touch handling](http://developer.android.com/training/gestures/viewgroup.html) and this [useful blog post](http://balpha.de/2013/07/android-development-what-i-wish-i-had-known-earlier/).

## Libraries

* [Sensey](https://github.com/nisrulz/Sensey) - Android library which makes detecting gestures a breeze.
* [AndroidSlidingUpPanel](https://github.com/umano/AndroidSlidingUpPanel) - Sliding panel up from the bottom
* [PhotoView](https://github.com/chrisbanes/PhotoView) - ImageView for Android that supports zooming, by various touch gestures.
* [DraggablePanel](https://github.com/pedrovgs/DraggablePanel) - Cool draggable panels like YouTube
* [AndroidSwipeLayout](https://github.com/daimajia/AndroidSwipeLayout) 
* [android-swipelistview](https://github.com/47deg/android-swipelistview) - A List View with swipeable cells
* [gesticulate](https://github.com/aglover/gesticulate) - Android swipe detection made simple
* [android-gesture-detectors](https://github.com/Almeros/android-gesture-detectors) - small framework for gesture detection

## References

 * <http://developer.android.com/training/gestures/index.html>
 * <http://developer.android.com/design/patterns/gestures.html>
 * <http://developer.android.com/training/gestures/detector.html>
 * <http://developer.android.com/training/gestures/multi.html>
 * <http://mobile.tutsplus.com/tutorials/android/android-gesture/>
 * <http://www.codeproject.com/Articles/319401/Simple-Gestures-on-Android>
 * <http://www.vogella.com/articles/AndroidTouch/article.html>
 * <http://androidrises.blogspot.com/2012/10/draw-line-on-finger-touch.html>
 * <http://mrbool.com/how-to-work-with-swipe-gestures-in-android/28088>
 * <http://www.codeproject.com/Articles/319401/Simple-Gestures-on-Android>
 * <http://developer.android.com/guide/topics/ui/drag-drop.html>
 * <http://javapapers.com/android/android-drag-and-drop/>
 * <http://balpha.de/2013/07/android-development-what-i-wish-i-had-known-earlier/>
 * <http://developer.android.com/training/gestures/viewgroup.html>