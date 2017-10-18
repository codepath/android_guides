## Overview

The [EditText](http://developer.android.com/reference/android/widget/EditText.html) is the standard text entry widget in Android apps. If the user needs to enter text into an app, this is the primary way for them to do that. 

![EditText](https://i.imgur.com/W3tEdH0.png)

There are many important properties that can be set to customize the behavior of an `EditText`. Several of these are listed below. Check out the [official text fields](http://developer.android.com/guide/topics/ui/controls/text.html#AutoComplete) guide for even more input field details.

## Usage

An `EditText` is added to a layout with all default behaviors with the following XML:

```xml
<EditText
    android:id="@+id/et_simple"
    android:layout_height="wrap_content"
    android:layout_width="match_parent">
</EditText>
```

Note that an [EditText](http://developer.android.com/reference/android/widget/EditText.html) is simply a thin extension of the [[TextView|Working-with-the-TextView]] and inherits all of the same properties.

### Retrieving the Value

Getting the value of the text entered into an EditText is as follows:

```java
EditText simpleEditText = (EditText) findViewById(R.id.et_simple);
String strValue = simpleEditText.getText().toString();
```

### Customizing the Input Type

By default, any text contents within an EditText control is displayed as plain text. By setting `inputType`, we can facilitate input of different types of information, like phone numbers and passwords:

```xml
<EditText
    ...
    android:inputType="phone">
</EditText>
```

Most common input types include: 

| Type               | Description                    |
| -------------      |:-------------:                 |
| textUri            | Text that will be used as a URI                   |
| textEmailAddress   | Text that will be used as an e-mail address       |
| textPersonName     | Text that is the name of a person                 |
| textPassword       | Text that is a password that should be obscured   |
| textVisiblePassword| Text, next button, and no microphone input        |
| number             | A numeric only field                              |
| phone              | For entering a phone number                       |
| date               | For entering a date                               |
| time               | For entering a time                               |
| textMultiLine      | Allow multiple lines of text in the field         |

You can set multiple inputType attributes if needed (separated by '|')

```xml
<EditText
  android:inputType="textCapSentences|textMultiline"
/>
```

You can see a list of [all available input types here](http://developer.android.com/reference/android/widget/TextView.html#attr_android:inputType).

### Further Entry Customization

We might want to limit the entry to a single-line of text (avoid newlines):

```xml
<EditText
  android:maxLines="1"
  android:lines="1"
/>
```

You can limit the characters that can be entered into a field using the digits attribute:

```xml
<EditText
  android:inputType="number"
  android:digits="01"
/>
```

This would restrict the digits entered to just "0" and "1". We might want to limit the total number of characters with:

```xml
<EditText
  android:maxLength="5"
/>
```

Using these properties we can define the expected input behavior for text fields.

### Adjusting Colors

You can adjust the highlight background color of selected text within an `EditText` with the `android:textColorHighlight` property:

```xml
<EditText
    android:textColorHighlight="#7cff88"
/>
```

with a result such as this:

<img src="https://i.imgur.com/cbfm3Fb.png" width="315" />

### Displaying Placeholder Hints

You may want to set the hint for the EditText control to prompt a user for specific input with:

```xml
<EditText
    ...
    android:hint="@string/my_hint">
</EditText>
```

which results in:

![Hints](https://i.imgur.com/b0kKM7g.png)

### Changing the bottom line color

<img src="https://imgur.com/XUWKoju.png"/>

Assuming you are using the AppCompat library, you can override the styles `colorControlNormal`, `colorControlActivated`, and `colorControlHighlight`:

```xml
<style name="Theme.App.Base" parent="Theme.AppCompat.Light.DarkActionBar">
    <item name="colorControlNormal">#d32f2f</item>
    <item name="colorControlActivated">#ff5722</item>
    <item name="colorControlHighlight">#f44336</item>
</style>
```

If you do not see these styles applied within a DialogFragment, there is a [known bug](https://code.google.com/p/android/issues/detail?id=169760#c1) when using the LayoutInflater passed into the [onCreateView()](http://developer.android.com/reference/android/app/Fragment.html#onCreateView(android.view.LayoutInflater, android.view.ViewGroup, android.os.Bundle)) method. 

The issue has already been fixed in the AppCompat v23 library.  See this [[guide|Migrating-to-the-AppCompat-Library#overview]] about how to upgrade. Another temporary workaround is to use the Activity's layout inflater instead of the one passed into the `onCreateView()` method:

```java
  public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        View view = getActivity().getLayoutInflater().inflate(R.layout.dialog_fragment, container);
  }
```

### Listening for EditText Input

Check out the [[basic event listeners|Basic-Event-Listeners#edittext-common-listeners]] cliffnotes for a look at how to listen for changes to an EditText and perform an action when those changes occur.


### Displaying Floating Label Feedback

Traditionally, the `EditText` hides the `hint` message (explained above) after the user starts typing. In addition, any validation error messages had to be managed manually by the developer. 

<img src="https://i.imgur.com/UM7NmiK.gif" alt="floating" width="400" />

Starting with Android M and the [[design support library|Design-Support-Library]], the `TextInputLayout` can be used to setup a floating label to display hints and error messages. First, wrap the `EditText` in a `TextInputLayout`:

```xml
<android.support.design.widget.TextInputLayout
    android:id="@+id/username_text_input_layout"
    android:layout_width="match_parent"
    android:layout_height="wrap_content">

    <EditText
        android:id="@+id/etUsername"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_centerHorizontal="true"
        android:layout_centerVertical="true"
        android:ems="10"
        android:hint="Username" />

</android.support.design.widget.TextInputLayout>
```

Now the hint will automatically begin to float once the `EditText` takes focus as shown below:

<img src="https://i.imgur.com/S456c0X.gif" alt="floating" width="400" /> 

We can also use the `TextInputLayout` to display error messages using the `setError` and `setErrorEnabled` properties in the activity at runtime:

```java
private void setupFloatingLabelError() {
    final TextInputLayout floatingUsernameLabel = (TextInputLayout) findViewById(R.id.username_text_input_layout);
    floatingUsernameLabel.getEditText().addTextChangedListener(new TextWatcher() {
        // ...
        @Override
        public void onTextChanged(CharSequence text, int start, int count, int after) {
            if (text.length() > 0 && text.length() <= 4) {
                floatingUsernameLabel.setError(getString(R.string.username_required));
                floatingUsernameLabel.setErrorEnabled(true);
            } else {
                floatingUsernameLabel.setErrorEnabled(false);
            }
        }
        @Override
        public void beforeTextChanged(CharSequence s, int start, int count, 
                                      int after) {
            // TODO Auto-generated method stub
        }

        @Override
        public void afterTextChanged(Editable s) {

        }
    });
}
```

Here we use the `addTextChangedListener` to watch as the value changes to determine when to display the error message or revert to the hint.

### Adding Character Counting

`TextInputLayout` since the [announcement of support design library v23.1](http://android-developers.blogspot.com/2015/10/android-support-library-231.html?linkId=17977963) also can expose a character counter for an `EditText` defined within it.  The counter will be rendered below the `EditText` and can change colors of both the line and character counter if the maximum number of characters has been exceeded:

<img src="https://imgur.com/eEYwIO3.png"/>

The `TextInputLayout` simply needs to define `app:counterEnabled` and `app:CounterMaxLength` in the XML attributes.  These settings can also be defined dynamically through `setCounterEnabled()` and `setCounterMaxLength()`:

```xml
<android.support.design.widget.TextInputLayout
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    app:counterEnabled="true"
    app:counterMaxLength="10"
    app:counterTextAppearance="@style/counterText"
    app:counterOverflowTextAppearance="@style/counterOverride">
    <EditText
       android:layout_width="wrap_content"
       android:layout_height="wrap_content"
       android:hint="Username"
       android:layout_centerHorizontal="true"
       android:layout_centerVertical="true"
       android:ems="10"
       android:hint="Username" />
</android.support.design.widget.TextInputLayout>
```

### Adding Password Visibility Toggles

**NOTE**: You must have support library 24.2.0 or higher to use this feature.  

<img src="https://imgur.com/33oQgLr.png"/>

If you use an `EditText` with an input password type, you can also enable an icon that can show or hide the entire text using the `passwordToggleEnabled` attribute.    You can also change the default eye icon with  `passwordToggleDrawable` attribute or the color hint using the `passwordToggleTint` attribute.  See the `TextInputLayout` [attributes](https://developer.android.com/reference/android/support/design/widget/TextInputLayout.html#attr_android.support.design:passwordToggleTintMode) for more details.

```xml
<android.support.design.widget.TextInputLayout
        android:id="@+id/username_text_input_layout"
        app:passwordToggleEnabled="true"
        android:layout_width="match_parent"
        android:layout_height="wrap_content">

        <EditText
            android:id="@+id/etUsername"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_centerHorizontal="true"
            android:layout_centerVertical="true"
            android:ems="10"
            android:inputType="textPassword"
            android:hint="Username" />

    </android.support.design.widget.TextInputLayout>
```

### Styling TextInputLayout

Make sure you have the `app` namespace (`xmlns:app="http://schemas.android.com/apk/res-auto"` defined in your outer layout.  You can type `appNS` as a shortcut in Android Studio to be declared.

The hint text can be styled by defining `app:hintTextAppearance`, and the error text can be changed with `app:errorTextAppearance.`  The counter text and overflow text can also have their own text styles by defining `app:counterTextAppearance` and `app:counterOverflowTextAppearance`.  We can use `textColor`, `textSize`, and `fontFamily` to help change the color, size, or font:

```xml
<style name="counterText">
  <item name="android:textColor">#aa5353cc</item>
</style>

<style name="counterOverride">
  <item name="android:textColor">#ff0000</item>
</style>
```

### Providing Auto-complete

Check out the [official text fields](http://developer.android.com/guide/topics/ui/controls/text.html#AutoComplete) guide for a step-by-step on how to setup autocomplete for the entry.


## References

* <http://developer.android.com/guide/topics/ui/controls/text.html>
* <http://code.tutsplus.com/tutorials/android-user-interface-design-edittext-controls--mobile-7183>
* <http://www.codeofaninja.com/2012/01/android-edittext-example.html>
* <http://www.tutorialspoint.com/android/android_edittext_control.htm>
* <http://developer.android.com/reference/android/widget/EditText.html>
* <http://android-developers.blogspot.com/2015/10/android-support-library-231.html?linkId=17977963>
