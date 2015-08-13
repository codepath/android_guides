## Overview

Snackbars are shown on the bottom of the screen and contain text with an optional single action. They automatically fade out after enough time similar to a toast. Snackbars can be swiped away by the user or contain other actions making them more powerful [[than simple toasts|Displaying-Toasts]]. However, the API is very familiar.

<img src="https://i.imgur.com/JSdKnP2.png" width="200" />

Note the snackbar at the bottom with embedded action. See [this design guide](http://www.google.com/design/spec/components/snackbars-toasts.html) or more details.

### Simple Snackbar

Make sure to follow the [[Design Support Library]] setup instructions first.

Create a snackbar using `make`, setting an optional action and then call `.show()`: 

```java
Snackbar.make(parentLayout, R.string.snackbar_text, Snackbar.LENGTH_LONG)
  .setAction(R.string.snackbar_action, myOnClickListener)
  .show(); // Don’t forget to show!
```

Note the use of a `View` as the first parameter to `make()` which the snackbar uses as the parent layout used for positioning.  Snackbar will try to walk up the view hierarchy searching for either a CoordinatorLayout or the top-most container layout, whichever comes first.  Adding a CoordinatorLayout in the view hierarchy is helpful in cases where the floating action buttons needs to moved to make room for displaying the Snackbar as discussed in [this guide](http://guides.codepath.com/android/Handling-Scrolls-with-CoordinatorLayout#floating-action-buttons-and-snackbars).

In a [recent update](https://plus.google.com/+AndroidDevelopers/posts/XTtNCPviwpj) of the support library, you can now specify `LENGTH_INDEFINITE` that will continue to show the Snackbar until it is dismissed or another one is shown:

```java
Snackbar.make(parentLayout, R.string.snackbar_text, Snackbar.LENGTH_INDEFINITE).show();
```

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