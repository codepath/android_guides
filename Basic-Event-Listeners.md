## Overview

Event Listening in Android development is largely centered around the `View` object. 

### View Event Listeners

Any View (Button, TextView, etc) has many event listeners that can be attached using the `setOnEvent` pattern which involves passing a class that implements a particular event interface. The listeners available to any `View` include:

 * `setOnClickListener` - Callback when the view is clicked
 * `setOnDragListener` - Callback when the view is dragged
 * `setOnFocusChangeListener` - Callback when the view changes focus
 * `setOnGenericMotionListener` - Callback for arbitrary gestures
 * `setOnHoverListener` - Callback for hovering over the view
 * `setOnKeyListener` - Callback for pressing a hardware key when view has focus
 * `setOnLongClickListener` - Callback for pressing and holding a view
 * `setOnTouchListener` - Callback for touching down or up on a view


**Using Java**

In Java Code, attaching to any event works roughly the same way. Let's take the `OnClickListener` as an example. First, you need a reference to the view and then you need to use the `set` method associated with that listener and pass in a class implementing a [particular interface](http://developer.android.com/reference/android/view/View.OnClickListener.html). For example:

```java
Button btnExample = (Button) findViewById(R.id.btnExample);
btnExample.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        // Do something here	
    }
});
```

This pattern works for any of the view-based event listeners.

**Using XML**

In addition `onClick` has a unique shortcut that allows the method to specified within the layout XML. So rather than attaching the event manually in the Java, the method can be attached in the view. For example:

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

 * `setOnItemClickListener` - Callback when an item contained is clicked
 * `setOnItemLongClickListener` - Callback when an item contained is clicked and held
 * `setOnItemSelectedListener` - Callback when an item is selected

This works similarly to a standard listener, simply implementing [the correct interface](http://developer.android.com/reference/android/widget/AdapterView.OnItemLongClickListener.html):

```java
lvItems.setOnItemLongClickListener(new OnItemLongClickListener() {
    @Override
    public boolean onItemLongClick(AdapterView<?> adapter, View item, int pos, long rowId) {
        // Do something here
    }
});
```

**Troubleshooting: Item Click Not Firing** If the item is more complex and does not seem to be properly responding to clicks after setting up the handler, the views inside the item might be drawing the focus. Check out [this stackoverflow post](http://stackoverflow.com/questions/6703390/listview-setonitemclicklistener-not-working-by-adding-button) and add the property `android:descendantFocusability="blocksDescendants"` to the root layout within the template for the item. 

### EditText Common Listeners

In addition to the listeners described above, there are a few other common listeners for input fields in particular. 

 * `addTextChangedListener` - Fires each time the text in the field is being changed
 * `setOnEditorActionListener` - Fires when an "action" button on the soft keyboard is pressed

#### TextChangedListener

If you want to handle an event as the text in the view is being changed, you only need to look as far as the [addTextChangedListener](http://developer.android.com/reference/android/widget/TextView.html#addTextChangedListener\(android.text.TextWatcher\)) method on an EditText (or even TextView):

```java
EditText etValue = (EditText) findViewById(R.id.etValue);
etValue.addTextChangedListener(new TextWatcher() {
	@Override
	public void onTextChanged(CharSequence s, int start, int before, int count) {
		// Fires right as the text is being changed (even supplies the range of text)
	}
	
	@Override
	public void beforeTextChanged(CharSequence s, int start, int count,
			int after) {
		// Fires right before text is changing
	}
	
	@Override
	public void afterTextChanged(Editable s) {
		// Fires right after the text has changed
		tvDisplay.setText(s.toString());
	}
});
```

This is great for any time you want to have the UI update as the user enters text.

#### OnEditorActionListener

Another case is when you want an action to occur once the user has finished typing text with the [[Soft Keyboard|Working-with-the-Soft-Keyboard]]. Keep in mind that this is especially useful when you can see the virtual keyboard which is disabled by default in the emulator but can be enabled as explained in [this graphic](http://imgur.com/a/kf1s9).

First, we need to setup an "action" button for our text field. To setup an "action button" such as a [Done  button on the soft Keyboard](http://imgur.com/WAwMn9k.png), simply configure your EditText with the following properties:

```xml
<EditText
   android:inputType="text"
   android:singleLine="true"
   android:imeOptions="actionDone"
   android:id="@+id/etValue"
   android:layout_width="wrap_content"
   android:layout_height="wrap_content">
   <requestFocus />
</EditText>
```

In particular, `singleLine` and `imeOptions` are required for the [Done button](http://imgur.com/WAwMn9k.png) to display. Now, we can hook into a [editor listener](http://developer.android.com/reference/android/widget/TextView.html#setOnEditorActionListener\(android.widget.TextView.OnEditorActionListener\)) for when the done button is pressed with:

```java
etValue.setOnEditorActionListener(new OnEditorActionListener() {
	@Override
	public boolean onEditorAction(TextView v, int actionId, KeyEvent event) {
		if (actionId == EditorInfo.IME_ACTION_DONE) {
                        String text = v.getText().toString();
			Toast.makeText(MainActivity.this, text, Toast.LENGTH_SHORT).show();
			return true;
		}
		return false;
	}
});
```

This is often great whenever a user needs to type text and then explicitly have an action performed when they are finished. There are many [imeOptions](http://developer.android.com/reference/android/widget/TextView.html#attr_android:imeOptions) for different situations.

### Input View Listeners

Similarly to EditText, many common input views have listeners of their own including NumberPicker has [setOnValueChangedListener](http://developer.android.com/reference/android/widget/NumberPicker.html#setOnValueChangedListener\(android.widget.NumberPicker.OnValueChangeListener\)) and SeekBar has [setOnSeekBarChangeListener](http://developer.android.com/reference/android/widget/SeekBar.html#setOnSeekBarChangeListener\(android.widget.SeekBar.OnSeekBarChangeListener\)) which allow us to listen for changes:

```java
NumberPicker npValue = (NumberPicker) findViewById(R.id.npValue);
npValue.setOnValueChangedListener(new OnValueChangeListener() {
	@Override
	public void onValueChange(NumberPicker picker, int oldVal, int newVal) {
		// Changes based on number here
	}
});
```

Almost all input views have similar methods available.

## References

 * <http://developer.android.com/guide/topics/ui/controls/button.html>
 * <http://developer.android.com/reference/android/widget/Button.html>
 * <http://developer.android.com/guide/topics/ui/ui-events.html>
 * <http://developer.android.com/reference/android/view/View.html#setOnClickListener>
 * [setOnEditorActionListener](http://developer.android.com/reference/android/widget/TextView.html#setOnEditorActionListener\(android.widget.TextView.OnEditorActionListener\)) Docs
 * [addTextChangedListener](http://developer.android.com/reference/android/widget/TextView.html#addTextChangedListener\(android.text.TextWatcher\)) Docs