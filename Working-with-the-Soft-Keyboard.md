The Android system shows an on-screen keyboard, known as a soft input method, when a text field in your UI receives focus. To provide the best user experience, you can specify characteristics about the type of input you expect (such as whether it's a phone number or email address) and how the input method should behave (such as whether it performs auto-correct for spelling mistakes).

**Note:** By default, the soft keyboard may not appear on the emulator. If you want to test with the soft keyboard, be sure to open up AVD (Window => Android Virtual Device Manager) and set "Hardware Keyboard Present" to **false** for your emulator. See [this screenshot](http://imgur.com/a/kf1s9) for a visual reference.

## Showing Soft Keyboard Programmatically 

The following code will reveal the soft keyboard focused on a specified view:

```java
public void showSoftKeyboard(View view){
    if(view.requestFocus()){
        InputMethodManager imm =(InputMethodManager) getSystemService(Context.INPUT_METHOD_SERVICE);
        imm.showSoftInput(view,InputMethodManager.SHOW_IMPLICIT);
    }
}
```

## Hiding the Soft Keyboard Programmatically

You can force Android to hide the virtual keyboard using the [InputMethodManager](http://developer.android.com/reference/android/view/inputmethod/InputMethodManager.html), calling [hideSoftInputFromWindow](http://developer.android.com/reference/android/view/inputmethod/InputMethodManager.html#hideSoftInputFromWindow%28android.os.IBinder,%20int%29), passing in the token of the window containing your edit field.

```java
public void hideSoftKeyboard(View view){
  InputMethodManager imm =(InputMethodManager)getSystemService(Context.INPUT_METHOD_SERVICE);
  imm.hideSoftInputFromWindow(view.getWindowToken(), 0);
}
```

This will force the keyboard to be hidden in all situations. 

## Adding a "Done" Key

In the keyboard, you can hide the "Next" key and add "Done" instead by adding the following to the `imeOptions` for the EditText view:

```xml
<EditText
  android:imeOptions="actionDone">
</EditText>
```

or in Java:

```java
myEditText.setImeOptions(EditorInfo.IME_ACTION_DONE);
```

See the [EditText documentation](http://developer.android.com/reference/android/widget/TextView.html#attr_android%3aimeActionLabel) for a more detailed look at `imeOptions`.

## Showing the Keyboard when Activity Starts

Although Android gives focus to the first text field in your layout when the activity starts, it does not show the soft keyboard. To show the keyboard when your activity starts, add the [android:windowSoftInputMode](http://developer.android.com/guide/topics/manifest/activity-element.html#wsoft) attribute to the `<activity>` element with the `"stateVisible"` value within the Android manifest. Check out [this guide](http://developer.android.com/training/keyboard-input/visibility.html#ShowOnStart) for more details.

```xml
<activity
    android:name="com.example.myactivity"
    android:windowSoftInputMode="stateVisible" />
```

We can also use this to change the way that the soft keyboard displaces the view when it appears as well with:

```xml
<activity
    android:name="com.example.myactivity"
    android:windowSoftInputMode="stateVisible|adjustResize" />
```

See the guide on [keyboard visibility](http://developer.android.com/training/keyboard-input/visibility.html) for more details.

## References

 * <http://developer.android.com/training/keyboard-input/style.html>
 * <http://developer.android.com/training/keyboard-input/navigation.html>
 * <http://developer.android.com/training/keyboard-input/commands.html>
 * <http://developer.android.com/training/keyboard-input/visibility.html>