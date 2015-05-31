## Overview

Android has support for many different input controls for accepting input from the user. The common input controls include:

 * [Buttons](http://developer.android.com/guide/topics/ui/controls/button.html)
 * [Text Fields](http://developer.android.com/guide/topics/ui/controls/text.html)
 * [Checkboxes](http://developer.android.com/guide/topics/ui/controls/checkbox.html)
 * [Radio Buttons](http://developer.android.com/guide/topics/ui/controls/radiobutton.html)
 * [Spinners](http://developer.android.com/guide/topics/ui/controls/spinner.html)
 * [SeekBar](http://developer.android.com/reference/android/widget/SeekBar.html)
 * [RatingBar](http://developer.android.com/reference/android/widget/RatingBar.html)
 * [NumberPicker](http://developer.android.com/reference/android/widget/NumberPicker.html)
 * [Switch](http://developer.android.com/reference/android/widget/Switch.html)
 * [Date and Time Pickers](http://developer.android.com/guide/topics/ui/controls/pickers.html)

Adding an input control to your UI is as simple as adding the appropriate XML element with required attributes to your layout file:

![Imgur](http://i.imgur.com/OWGeaH9.png)

The important part of input is deciding which input to use and how to properly configure the input view.

### Input Reference

There are many input fields available, use the handy chart below to determine which inputs to use for different situations:

| Type            | Inputs                                |
| --------------- | ----------------------------------    |
| Single Action   | [Button](http://developer.android.com/guide/topics/ui/controls/button.html), [ImageButton](https://developer.android.com/reference/android/widget/ImageButton.html) |
| Free Text       | [[TextView|Working-with-the-TextView]], [AutoCompleteTextView](https://developer.android.com/reference/android/widget/AutoCompleteTextView.html) |
| Integer         | [[TextView|Working-with-the-TextView]], [[NumberPicker|Working-with-Input-Views#numberpicker]], [SeekBar](http://developer.android.com/reference/android/widget/SeekBar.html) |
| Boolean         | [[Checkboxes|Working-with-Input-Views#checkboxes]],  [Switch](http://developer.android.com/reference/android/widget/Switch.html) |
| Single Choice    | [[Spinner|Working-with-Input-Views#spinners]], [[Radio Buttons|Working-with-Input-Views#radio-buttons]], [AutoCompleteTextView](https://developer.android.com/reference/android/widget/AutoCompleteTextView.html) |
| Multiple Choice | [[Checkboxes|Working-with-Input-Views#checkboxes]] |
| Dates, Times    | [[DateTimePicker|Working-with-Input-Views#date-and-time-pickers]] |
| Rating          | [RatingBar](http://developer.android.com/reference/android/widget/RatingBar.html) |

There are many third-party libraries available to improve input selection within apps. See this [list of UI libraries](https://github.com/wasabeef/awesome-android-ui) and our [[Must Have Libraries]] guide for more information.

### Text Fields

If you are using an EditText, you should always specify the [hint](http://guides.codepath.com/android/Working-with-the-EditText#displaying-placeholder-hints) and [input type](http://guides.codepath.com/android/Working-with-the-EditText#customizing-the-input-type).  

### Date and Time Pickers

Often within an app you will need the user to select a time or date for an event or other object. There are [native date and time pickers](http://developer.android.com/guide/topics/ui/controls/pickers.html) but they do not make for a polished user experience. Better options for date and time picking are below:

 * [DateTimePicker](https://github.com/flavienlaurent/datetimepicker) - Contains the beautiful DatePicker and TimePicker that can be seen in the new Google Agenda app.
 * [BetterPickers](https://github.com/derekbrameyer/android-betterpickers) - DialogFragments modeled after the AOSP Clock and Calendar apps to improve UX for picking time, date, numbers, and other things.

Using one of these should make selecting dates and times much easier. See [this list of picker libraries](https://android-arsenal.com/tag/27) for alternatives. 

### Checkboxes

Checkboxes allow the user to select one or more options from a set. Typically, you should present each checkbox option in a vertical list. Add them to the view:

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <CheckBox android:id="@+id/checkbox_meat"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/meat"
        android:onClick="onCheckboxClicked"/>
    <CheckBox android:id="@+id/checkbox_cheese"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/cheese"
        android:onClick="onCheckboxClicked"/>
</LinearLayout>
```

and then in the activity:

```java
public void onCheckboxClicked(View view) {
    // Is the view now checked?
    boolean checked = ((CheckBox) view).isChecked();
    
    // Check which checkbox was clicked
    switch(view.getId()) {
        case R.id.checkbox_meat:
            if (checked)
                // Put some meat on the sandwich
            else
                // Remove the meat
            break;
        case R.id.checkbox_cheese:
            if (checked)
                // Cheese me
            else
                // I'm lactose intolerant
            break;
    }
}
```

Using this pattern you can manage the checkbox events and take the user input. Take a look at the [Official Checkbox Guide](http://developer.android.com/guide/topics/ui/controls/checkbox.html) for more details.

### Radio Buttons

Radio buttons allow the user to select one option from a set. If it's not necessary to show all options side-by-side, use a spinner instead.

To create each radio button option, create a RadioButton in your layout. However, because radio buttons are mutually exclusive, you must group them together inside a RadioGroup. By grouping them together, the system ensures that only one radio button can be selected at a time.

```xml
<?xml version="1.0" encoding="utf-8"?>
<RadioGroup xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="vertical">
    <RadioButton android:id="@+id/radio_pirates"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/pirates"
        android:onClick="onRadioButtonClicked"/>
    <RadioButton android:id="@+id/radio_ninjas"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/ninjas"
        android:onClick="onRadioButtonClicked"/>
</RadioGroup>
```

and within the activity:

```java
public void onRadioButtonClicked(View view) {
    // Is the button now checked?
    boolean checked = ((RadioButton) view).isChecked();
    
    // Check which radio button was clicked
    switch(view.getId()) {
        case R.id.radio_pirates:
            if (checked)
                // Pirates are the best
            break;
        case R.id.radio_ninjas:
            if (checked)
                // Ninjas rule
            break;
    }
}
```

Check out the [official radio buttons guide](http://developer.android.com/guide/topics/ui/controls/radiobutton.html) for more details.

### Spinners

Spinners provide a quick way to select one value from a set of options. Create a spinner like any view and specify the `android:entries` attribute to specify the set of options:

```xml
<Spinner
    android:id="@+id/mySpinner"
    android:layout_width="wrap_content"
    android:entries="@array/planets_arrays"
    android:prompt="@string/planets_prompt"
    android:layout_height="wrap_content" />
```

and then specify the string array of options in `res/values/planets_array.xml`:

```java
<?xml version="1.0" encoding="utf-8"?>
<resources>
	<string-array name="planets_array">
		<item>Mercury</item>
		<item>Venus</item>
		<item>Earth</item>
		<item>Mars</item>
	</string-array>
</resources>
```

Check out the [Spinners](http://developer.android.com/guide/topics/ui/controls/spinner.html) guide for more details. Note that customizing a spinner's text requires using a [custom array adapter and layout file](http://stackoverflow.com/questions/9476665/how-to-change-spinner-text-size-and-text-color).

#### Getting and Setting Values

Get the selected item out a spinner using:

```java
String value = spinner.getSelectedItem().toString();
```

Setting spinner item based on value (rather than item position):

```java
public void setSpinnerToValue(Spinner spinner, String value) {
	int index = 0;
	SpinnerAdapter adapter = spinner.getAdapter();
	for (int i = 0; i < adapter.getCount(); i++) {
		if (adapter.getItem(i).equals(value)) {
			index = i;
			break; // terminate loop
		}
	}
	spinner.setSelection(index);
}
```

#### Custom ArrayAdapter Source

You can also load a spinner using an adapter for a dynamic source of options using an Adapter:

```java
Spinner spinner = (Spinner) findViewById(R.id.spinner);
// Create an ArrayAdapter using the string array and a default spinner layout
ArrayAdapter<CharSequence> adapter = ArrayAdapter.createFromResource(this,
        R.array.planets_array, android.R.layout.simple_spinner_item);
// Specify the layout to use when the list of choices appears
adapter.setDropDownViewResource(android.R.layout.simple_spinner_dropdown_item);
// Apply the adapter to the spinner
spinner.setAdapter(adapter);
```

#### Multiple Select Spinner

By default, the spinner only allows the user to select one option from the list. Check out the following resources surrounding multiple selection spinners:

 * [MultiSelectSpinner](https://github.com/wongk/MultiSelectSpinner) - Simple multi-select library
 * [MultiSelect Tutorial 1](http://v4all123.blogspot.in/2013/09/spinner-with-multiple-selection-in.html)
 * [MultiSelect Tutorial 2](http://asnehal.wordpress.com/2012/04/03/multi-select-drop-down-list-in-android/)
 * [Simple Multi Dialog](http://labs.makemachine.net/2010/03/android-multi-selection-dialogs/)

Note that we can also use a `ListView` in this case instead to avoid a spinner altogether.

### NumberPicker

This is a widget that enables the user to select a number from a predefined range. First, we put the NumberPicker within the layout XML:

```xml
 <NumberPicker 
        android:id="@+id/np_total"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"  
/>
```

Then we **must define the desired range at runtime** in the Activity:

```java
public class DemoPickerActivity extends Activity {
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_demo_picker);
        NumberPicker numberPicker = 
            (NumberPicker) findViewById(R.id.np_total);
        numberPicker.setMinValue(0); 
        numberPicker.setMaxValue(100);    
        numberPicker.setWrapSelectorWheel(true);
        numberPicker.setDescendantFocusability(NumberPicker.FOCUS_BLOCK_DESCENDANTS);
    }
}
```

Note we set the range with `setMinValue` and `setMaxValue` and made the selector wrap with [setWrapSelectorWheel](http://developer.android.com/reference/android/widget/NumberPicker.html#setWrapSelectorWheel\(boolean\)).

If you don't want the soft keyboard to popup and take focus on number picker click, you can set DescendantFocusability to be [NumberPicker.FOCUS_BLOCK_DESCENDANTS] (http://developer.android.com/reference/android/view/ViewGroup.html#setDescendantFocusability\(int\)).

If you want to listen as the value changes, we can use the `OnValueChangeListener` listener:

```java
// within onCreate
numberPicker.setOnValueChangedListener(new NumberPicker.OnValueChangeListener() {
    @Override
    public void onValueChange(NumberPicker picker, int oldVal, int newVal) {
        Log.d("DEBUG", "Selected number in picker is " + newVal);
    }
});
```

You can also call `getValue` to retrieve the numeric value any time. See the [NumberPicker docs](http://developer.android.com/reference/android/widget/NumberPicker.html#setWrapSelectorWheel\(boolean\)) for more details.

## References

 * <http://developer.android.com/guide/topics/ui/controls.html>