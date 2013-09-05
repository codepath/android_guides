## Overview

Android has support for many different input controls for accepting input from the user. The common input controls include:

 * [Buttons](http://developer.android.com/guide/topics/ui/controls/button.html)
 * [Text Fields](http://developer.android.com/guide/topics/ui/controls/text.html)
 * [Checkboxes](http://developer.android.com/guide/topics/ui/controls/checkbox.html)
 * [Radio Buttons](http://developer.android.com/guide/topics/ui/controls/radiobutton.html)
 * [Toggle Buttons](http://developer.android.com/guide/topics/ui/controls/togglebutton.html)
 * [Spinners](http://developer.android.com/guide/topics/ui/controls/spinner.html)
 * [Date and Time Pickers](http://developer.android.com/guide/topics/ui/controls/pickers.html)

Adding an input control to your UI is as simple as adding an XML element to your XML layout.

![Imgur](http://i.imgur.com/OWGeaH9.png)

### Checkboxes

Checkboxes allow the user to select one or more options from a set. Typically, you should present each checkbox option in a vertical list. Add them to the view:

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent">
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

Using this pattern you can manage the checkbox events and take the user input.

### Radio Buttons

Radio buttons allow the user to select one option from a set. If it's not necessary to show all options side-by-side, use a spinner instead.

To create each radio button option, create a RadioButton in your layout. However, because radio buttons are mutually exclusive, you must group them together inside a RadioGroup. By grouping them together, the system ensures that only one radio button can be selected at a time.

```xml
<?xml version="1.0" encoding="utf-8"?>
<RadioGroup xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="fill_parent"
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

Get the selected item out a spinner using:

```java
String value = spinner.getSelectedItem().toString();
```

Setting spinner item based on value:

```java
public void setSpinnerToValue(Spinner spinner, String value) {
	int index = 0;
	SpinnerAdapter adapter = spinner.getAdapter();
	for (int i = 0; i < adapter.getCount(); i++) {
		if (adapter.getItem(i).equals(value)) {
			index = i;
		}
	}
	spinner.setSelection(index);
}
```

You can also load a spinner using a dynamic source of options, check out the [Spinners](http://developer.android.com/guide/topics/ui/controls/spinner.html) guide for more details. 

## References

 * <http://developer.android.com/guide/topics/ui/controls.html>