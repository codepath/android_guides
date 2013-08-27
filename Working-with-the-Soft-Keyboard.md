The Android system shows an on-screen keyboard, known as a soft input method, when a text field in your UI receives focus. To provide the best user experience, you can specify characteristics about the type of input you expect (such as whether it's a phone number or email address) and how the input method should behave (such as whether it performs auto-correct for spelling mistakes).

## Showing the Keyboard when Activity Starts

Although Android gives focus to the first text field in your layout when the activity starts, it does not show the input method. To show the input method when your activity starts, add the [android:windowSoftInputMode](http://developer.android.com/guide/topics/manifest/activity-element.html#wsoft) attribute to the `<activity>` element with the `"stateVisible"` value within the Android manifest. Check out [this guide](http://developer.android.com/training/keyboard-input/visibility.html#ShowOnStart) for more details.

## Showing Soft Keyboard in Code

```java
public void showSoftKeyboard(View view){
    if(view.requestFocus()){
        InputMethodManager imm =(InputMethodManager)
                getSystemService(Context.INPUT_METHOD_SERVICE);
        imm.showSoftInput(view,InputMethodManager.SHOW_IMPLICIT);
    }
}
```

## Hiding the Soft Keyboard

You can force Android to hide the virtual keyboard using the InputMethodManager, calling hideSoftInputFromWindow, passing in the token of the window containing your edit field.

```java
InputMethodManager imm =(InputMethodManager)getSystemService(Context.INPUT_METHOD_SERVICE);
imm.hideSoftInputFromWindow(myEditText.getWindowToken(),0);
```

This will force the keyboard to be hidden in all situations. 

## Adding a "Done" Key

In the keyboard, you can hide the "Next" key and add "Done" instead by adding the following:


See the EditText documentation for a more detailed look.