## Overview

Android has support for many different input controls for accepting input from the user. The common input controls include:

 * [Buttons](http://developer.android.com/guide/topics/ui/controls/button.html)
 * [[Text Fields|Working-With-Input-Views#text-fields]]
 * [[Checkboxes|Working-With-Input-Views#checkboxes]]
 * [[Radio Buttons|Working-With-Input-Views#radio-buttons]]
 * [[Spinners|Working-With-Input-Views#spinners]]
 * [SeekBar](http://developer.android.com/reference/android/widget/SeekBar.html)
 * [RatingBar](http://developer.android.com/reference/android/widget/RatingBar.html)
 * [[NumberPicker|Working-With-Input-Views#numberpicker]]
 * [Switch](http://developer.android.com/reference/android/widget/Switch.html)
 * [[Date and Time Pickers|Working-With-Input-Views#date-and-time-pickers]]

Adding an input control to your UI is as simple as adding an XML element with attributes to your layout file:

![Imgur](https://i.imgur.com/OWGeaH9.png)

The important part of input is deciding which input to use and how to properly configure the input view.

### Input Reference

There are many input fields available, use the handy chart below to determine which inputs to use for different situations:

| Type            | Inputs                                |
| --------------- | ----------------------------------    |
| Single Action   | [Button](http://developer.android.com/guide/topics/ui/controls/button.html), [ImageButton](https://developer.android.com/reference/android/widget/ImageButton.html) |
| Free Text       | [[EditText|Working-with-the-EditText]], [AutoCompleteTextView](https://developer.android.com/reference/android/widget/AutoCompleteTextView.html) |
| Integer         | [[EditText|Working-with-the-EditText]], [[NumberPicker|Working-with-Input-Views#numberpicker]], [SeekBar](http://developer.android.com/reference/android/widget/SeekBar.html) |
| Boolean         | [[Checkboxes|Working-with-Input-Views#checkboxes]],  [Switch](http://developer.android.com/reference/android/widget/Switch.html) |
| Single Choice    | [[Spinner|Working-with-Input-Views#spinners]], [[Radio Buttons|Working-with-Input-Views#radio-buttons]], [AutoCompleteTextView](https://developer.android.com/reference/android/widget/AutoCompleteTextView.html) |
| Multiple Choice | [[Checkboxes|Working-with-Input-Views#checkboxes]] |
| Dates, Times    | [[DateTimePicker|Working-with-Input-Views#date-and-time-pickers]] |
| Rating          | [RatingBar](http://developer.android.com/reference/android/widget/RatingBar.html) |

There are many third-party libraries available to improve input selection within apps. See this [list of UI libraries](https://github.com/wasabeef/awesome-android-ui) and our [[Must Have Libraries]] guide for more information.

### Text Fields

If you are using an EditText, you should always specify the [[hint|Working-with-the-EditText#displaying-placeholder-hints]] and [[input type|Working-with-the-EditText#customizing-the-input-type]].  

### Date and Time Pickers

Often within an app you will need the user to select a time or date for an event or other object. There are [native date and time pickers](http://developer.android.com/guide/topics/ui/controls/pickers.html) but they do not make for a polished user experience for pre Lollipop (API v21) devices.  Consider the difference:

| Description | Pre-Android 5.0 (API 20 and lower)        | Android 5.0 (API 21+)  |                                           
| ------------| -----------------------------|---------------------------------------------------------|
| Date Picker | <img src="https://imgur.com/PBSFVeb.png" width="300"/>  | <img src="https://imgur.com/8YqcxA7.png" width="300"/>   |  
| Time Picker | <img src="https://imgur.com/13yNQPA.png" width="300"/> | <img src="https://imgur.com/s358M0K.png" width="300"/>
                
If you intend to support older Android devices, the better options for date and time picking are below:

 * [BetterPickers](https://github.com/derekbrameyer/android-betterpickers) - DialogFragments modeled after the AOSP Clock and Calendar apps to improve UX for picking time, date, numbers, and other things (supports Android 2.3+ devices)
 * [DateTimePicker](https://github.com/flavienlaurent/datetimepicker) - Contains the beautiful DatePicker and TimePicker that can be seen in the new Google Agenda app (supports Android 2.1+ devices)
 * [MaterialDateTimePicker](https://github.com/wdullaer/MaterialDateTimePicker) - Material Design styled DatePicker and TimePicker (supports Android 4.0+ devices)

For large dedicated date pickers used for scheduling, check out these:

 * [TimesSquare](https://github.com/square/android-times-square) - If you want a full-screen date picker, check out this widely popular library. 
 * [Material-CalendarView](https://github.com/prolificinteractive/material-calendarview) - A Material design back port of Android's CalendarView. 

DateTimerPicker and MaterialDateTimePicker are both forks from the original Android open source datetime picker located [here](https://android.googlesource.com/platform/frameworks/opt/datetimepicker/).  MaterialDateTimePicker however does not use the Support Fragment Manager for reasons stated [here](https://github.com/wdullaer/MaterialDateTimePicker#why-not-use-supportdialogfragment), so if you need it in your project, you will need to download the original source, create a separate project, and modify the [import statements](https://android.googlesource.com/platform/frameworks/opt/datetimepicker/+/master/src/com/android/datetimepicker/date/DatePickerDialog.java#21).


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
        android:text="@string/meat" />
    <CheckBox android:id="@+id/checkbox_cheese"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/cheese" />
</LinearLayout>
```

We can access whether or not a checkbox is checked with:

```java
CheckBox checkCheese = (CheckBox) findViewById(R.id.checkbox_cheese);
// Check if the checkbox is checked
boolean isChecked = checkCheese.isChecked();
// Set the checkbox as checked
checkCheese.setChecked(true);
```

and in our activity, we can manage checkboxes using a checked listener with `OnCheckedChangeListener` as show below:

```java
// Defines a listener for every time a checkbox is checked or unchecked
CompoundButton.OnCheckedChangeListener checkListener = new CompoundButton.OnCheckedChangeListener() {
    @Override
    public void onCheckedChanged(CompoundButton view, boolean checked) {
        // compoundButton is the checkbox
        // boolean is whether or not checkbox is checked
        // Check which checkbox was clicked
        switch(view.getId()) {
            case R.id.checkbox_meat:
                if (checked) {
                    // Put some meat on the sandwich
                } else {
                    // Remove the meat
                }
                break;
            case R.id.checkbox_cheese:
                if (checked) {
                    // Cheese me
                } else {
                    // I'm lactose intolerant
                }
                break;
          }
    }
};

// This actually applies the listener to the checkboxes 
// by calling `setOnCheckedChangeListener` on each one
public void setupCheckboxes() {
    Checkbox checkCheese = (Checkbox) findViewById(R.id.checkbox_cheese);
    Checkbox checkMeat = (Checkbox) findViewById(R.id.checkbox_meat);
    checkCheese.setOnCheckedChangeListener(checkListener);
    checkMeat.setOnCheckedChangeListener(checkListener);
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

#### Customizing Spinner Items

Changing text size on the `<Spinner>` tag has no effect on the actual dropdown items.   To change their styles, you need to create a custom array adapter and layout file.  First, you should create a `spinner_item1.xml`

```xml
<TextView xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@android:id/text1"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    style:"@style/spinner_dropdown_style1"
    android:textColor="#ff0000" />
```

Define your style to inherit from `Widget.AppCompat.DropDownItem.Spinner` so that it will inherit the correct spacing for dropdown items.  Otherwise, you may notice the dropdown items are too closely spaced or not enough margin to the left-hand side:

```xml
<style name="spinner_dropdown_style_theme1" parent="Widget.AppCompat.DropDownItem.Spinner">
     <item name="android:textColor">@android:color/white</item>
     <item name="android:background">#507B91</item>
</style>
```

You then bind the string array of items to the layout:

```java
ArrayAdapter adapter = ArrayAdapter.createFromResource(this, R.array.planets_array, R.layout.spinner_item);
spinner.setAdapter(adapter);
```

See [this link](http://www.broculos.net/2013/09/how-to-change-spinner-text-size-color.html) for more details.
Check out the [Spinners](http://developer.android.com/guide/topics/ui/controls/spinner.html) guide for more details. 

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
        R.array.planets_array, android.R.layout.simple_spinner_dropdown_item);
// Specify the layout to use when the list of choices appears
adapter.setDropDownViewResource(android.R.layout.simple_spinner_custom_layout);
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

