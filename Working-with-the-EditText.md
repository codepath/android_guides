## Overview

The [EditText](http://developer.android.com/reference/android/widget/EditText.html) is the standard text entry widget in Android apps. If the user needs to enter text into an app, this is the primary way for them to do that. 

![EditText](http://i.imgur.com/W3tEdH0.png)

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
| number             | A numeric only field                              |
| phone              | For entering a phone number                       |
| date               | For entering a date                               |
| time               | For entering a time                               |
| textMultiLine      | Allow multiple lines of text in the field         |

You can see a list of [all available input types here](http://developer.android.com/reference/android/widget/TextView.html#attr_android:inputType).

### Further Entry Customization

We might want to limit the entry to a single-line of text (avoid newlines):

```xml
<EditText
  android:singleLine="true"
  android:lines="1"
/>
```

You might also want to use the capitalize attribute to automatically capitalize each sentence, word or character:

```xml
<EditText
  android:capitalize="sentences"
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

<img src="http://i.imgur.com/cbfm3Fb.png" width="315" />

### Displaying Placeholder Hints

You may want to set the hint for the EditText control to prompt a user for specific input with:

```xml
<EditText
    ...
    android:hint="@string/my_hint">
</EditText>
```

which results in:

![Hints](http://i.imgur.com/b0kKM7g.png)

### Listening for EditText Input

Check out the [[basic event listeners|Basic-Event-Listeners#edittext-common-listeners]] cliffnotes for a look at how to listen for changes to an EditText and perform an action when those changes occur.

### Providing Auto-complete

Check out the [official text fields](http://developer.android.com/guide/topics/ui/controls/text.html#AutoComplete) guide for a step-by-step on how to setup autocomplete for the entry.

## References

* <http://developer.android.com/guide/topics/ui/controls/text.html>
* <http://code.tutsplus.com/tutorials/android-user-interface-design-edittext-controls--mobile-7183>
* <http://www.codeofaninja.com/2012/01/android-edittext-example.html>
* <http://www.tutorialspoint.com/android/android_edittext_control.htm>
* <http://developer.android.com/reference/android/widget/EditText.html>