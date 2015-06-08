## Overview

Snackbars are shown on the bottom of the screen and contain text with an optional single action. They automatically fade out after enough time similar to a toast. Snackbars can be swiped away by the user or contain other actions actions making them more powerful [[than simple toasts|Displaying-Toasts]]. However, the API is very familiar.

<img src="http://i.imgur.com/JSdKnP2.png" width="200" />

Note the snackbar at the bottom with embedded action. See [this design guide](http://www.google.com/design/spec/components/snackbars-toasts.html) or more details.

### Simple Snackbar

Make sure to follow the [[Design Support Library]] setup instructions first.

Create a snackbar using `make`, setting an optional action and then call `.show()`: 

```java
Snackbar.make(parentLayout, R.string.snackbar_text, Snackbar.LENGTH_LONG)
  .setAction(R.string.snackbar_action, myOnClickListener)
  .show(); // Don’t forget to show!
```

Note the use of a `View` as the first parameter to `make()` which the snackbar uses as the parent layout used for positioning. 

### Configuration Options

Additional options can be used to configure the snackbar such a `setActionTextColor` and `setDuration`:

```java
Snackbar.make(parentLayout, R.string.snackbar_text, Snackbar.LENGTH_LONG)
 .setAction(R.string.snackbar_action, myOnClickListener)
 .setActionTextColor(R.color.green)
 .setDuration(3000).show();
```

That's all!

## References

* <http://android-developers.blogspot.com/2015/05/android-design-support-library.html>
* <http://developer.android.com/reference/android/support/design/widget/Snackbar.html>
* <http://www.google.com/design/spec/components/snackbars-toasts.html>