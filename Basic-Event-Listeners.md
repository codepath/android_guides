## Overview

Event Listening in Android development is largely centered around the `View` object. 

### View Event Listeners

Any View (Button, TextView, etc) has many event listeners that can be attached using the `setOnEvent` pattern which involves passing a class that implements a particular event interface. The listeners available to any `View` include:

 * setOnClickListener - Callback when the view is clicked
 * setOnDragListener - Callback when the view is dragged
 * setOnFocusChangeListener - Callback when the view changes focus
 * setOnGenericMotionListener - Callback for arbitrary gestures
 * setOnHoverListener - Callback for hovering over the view
 * setOnKeyListener - Callback for pressing a hardware key when view has focus
 * setOnLongClickListener - Callback for pressing and holding a view
 * setOnTouchListener - Callback for touching down or up on a view

In Java Code, attaching to any event works roughly the same way. Let's take the `OnClickListener` as an example. First, you need a reference to the view and then you need to the `set` method associated with that listener and pass in a class implementing a [particular interface](http://developer.android.com/reference/android/view/View.OnClickListener.html). For example:

```java
Button btnExample = (Button) findViewById(R.id.btnExample);
btnExample.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        // Do something here	
    }
});
```

This pattern works for any of the view-based event listeners. In addition `onClick` has a unique shortcut that allows the method to specified within the layout XML. So rather than attaching the event manually in the Java, the method can be attached in the view. For example:

```xml
<?xml version="1.0" encoding="utf-8"?>
<Button xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/button_send"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="@string/button_send"
    android:onClick="sendMessage" />
```

Within the Activity that hosts this layout, the following method handles the click event:

```java
/** Called when the user touches the button */
public void sendMessage(View view) {
    // Do something in response to button click
}
```

### AdapterView Event Listeners

In addition to the standard View listeners, `AdapterView` descendants have a few more key event listeners having to do with their items:

 * setOnItemClickListener - Callback when an item contained is clicked
 * setOnItemLongClickListener - Callback when an item contained is clicked and held
 * setOnItemSelectedListener - Callback when an item is selected

This works similarly to a standard listener, simply implementing [the correct interface](http://developer.android.com/reference/android/widget/AdapterView.OnItemLongClickListener.html):

```java
lvItems.setOnItemLongClickListener(new OnItemLongClickListener() {
    @Override
    public boolean onItemLongClick(AdapterView<?> adapter, View item, int pos, long rowId) {
        
    }
});
```

## References

 * <http://developer.android.com/guide/topics/ui/controls/button.html>
 * <http://developer.android.com/reference/android/widget/Button.html>
 * <http://developer.android.com/guide/topics/ui/ui-events.html>
 * <http://developer.android.com/reference/android/view/View.html#setOnClickListener>