The Android system shows an on-screen keyboard, known as a soft input method, when a text field in your UI receives focus. To provide the best user experience, you can specify characteristics about the type of input you expect (such as whether it's a phone number or email address) and how the input method should behave (such as whether it performs auto-correct for spelling mistakes).

<img src="https://i.imgur.com/vIMWbX9.png" width="400" />

## Displaying the Soft Keyboard

### AVD Manager

By default, the soft keyboard may not appear on the emulator. If you want to test with the soft keyboard, be sure to open up the Android Virtual Device Manager (`Tools => Android => AVD Manager`) and **uncheck** "Enable Keyboard Input" for your emulator. 

<img src="https://i.imgur.com/hlPAVYG.png" width="500" /> 

Now restart the emulator. See [these screenshots](https://imgur.com/a/kf1s9) for a visual reference.

### Genymotion

<img src="https://imgur.com/SIIyVMc.png">

If you are using [[Genymotion|Genymotion-2.0-Emulators-with-Google-Play-support]], you need to click on the wrench icon (<img src="https://imgur.com/HRxr9Sm.png"/>) on the emulator image and check `Use virtual keyboard for text input` before starting the emulator.

<img src="https://imgur.com/xNxupXW.png"/>

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

## Configuring the Soft Keyboard Mode

The soft keyboard can be configured for each activity within the `AndroidManifest.xml` file using the [android:windowSoftInputMode](http://developer.android.com/guide/topics/manifest/activity-element.html#wsoft) attribute to adjust both default visibility and also how the keyboard affects the UI when displayed. 

### Showing the Keyboard when Activity Starts

Although Android gives focus to the first text field in your layout when the activity starts, it does not show the soft keyboard. To show the keyboard when your activity starts, add the [android:windowSoftInputMode](http://developer.android.com/guide/topics/manifest/activity-element.html#wsoft) attribute to the `<activity>` element with the `"stateVisible"` value within the Android manifest. Check out [this guide](http://developer.android.com/training/keyboard-input/visibility.html#ShowOnStart) for more details. Within the `AndroidManifest.xml` file:

```xml
<activity
    android:name="com.example.myactivity"
    android:windowSoftInputMode="stateVisible" />
```

The options for the mode include two aspects: visibility of the keyboard and adjustment of the UI. Visibility options include `stateUnchanged`, `stateHidden`, `stateVisible` and [several others listed here](http://developer.android.com/guide/topics/manifest/activity-element.html#wsoft).

### Changing UI Reaction

The virtual keyboard reduces the amount of space available for your app's UI. We can also use this same `android:windowSoftInputMode` property within the `<activity>` node to change the way that the soft keyboard displays the view elements when appearing within the `AndroidManifest.xml` file:

```xml
<!-- Configures the UI to be resized to make room for the keyboard -->
<activity
    android:name="com.example.myactivity"
    android:windowSoftInputMode="adjustResize" />
```

The options for the mode include two aspects: visibility and adjustment. Adjustment options include `adjustResize`, `adjustPan`, and `adjustUnspecified` and are [listed in full here](
http://developer.android.com/guide/topics/manifest/activity-element.html#wsoft). Both visibility and adjustment can be combined with:

```xml
<!-- Configures the keyboard to be visible right away and for UI to be resized when shown -->
<activity
    android:name="com.example.myactivity"
    android:windowSoftInputMode="stateVisible|adjustResize" />
```

See the guide on [keyboard visibility](http://developer.android.com/training/keyboard-input/visibility.html) for more details.

## Troubleshooting

### Toolbar Height Expands on UI Resize

To avoid incorrect `Toolbar` height calculations, you can add `android:fitsSystemWindows="true"` ([learn more](https://medium.com/google-developers/why-would-i-want-to-fitssystemwindows-4e26d9ce1eec)) to the parent layout of the `Toolbar`. In many cases, this should resolve the issue.  

## References

 * <http://developer.android.com/training/keyboard-input/style.html>
 * <http://developer.android.com/training/keyboard-input/navigation.html>
 * <http://developer.android.com/training/keyboard-input/commands.html>
 * <http://developer.android.com/training/keyboard-input/visibility.html>
