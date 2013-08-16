## Overview

Intent is a powerful concept within the Android universe. An intent is a message that can be thought of as a request that is given to either an activity within your own app, an external application, or a built-in Android service.

Think of an intent as a way for an Activity to communicate with the outside Android world. A few common tasks that an intent might be used for:

 * Take the user to another screen (activity) within your application
 * Take the user to a particular URL within the Android web browser 
 * Take the user to the camera to have them take a picture
 * Initiate a call for the user to a given number

As you can see, the Intent is a core part of user flows in Android development. The [Intent](http://developer.android.com/reference/android/content/Intent.html) object itself is a class that represents a particular "request" including the topic of the request and any request "parameters" which are called the [Bundle](http://developer.android.com/reference/android/os/Bundle.html). 

## Explicit Intents

An "explicit" intent is used to launch other activities within your application. For example, if you the user presses the "compose" button and you want to bring up an activity for them to compose a message, you would launch that second activity using an explicit intent.

Using an intent is as simple as constructing the [Intent](http://developer.android.com/reference/android/content/Intent.html) with the correct parameters and then invoking that intent using the `startActivity` method:

```java
public void launchComposeView() {
  // first parameter is the context, second is the class of the activity to launch
  Intent i = new Intent(this, ActivityTwo.class);
  startActivity(i); 
}
```

## References

 * <http://developer.android.com/guide/components/intents-filters.html>
 * <http://developer.android.com/reference/android/content/Intent.html>
 * <http://www.vogella.com/articles/AndroidIntent/article.html>
 * [Common Implicit Intents](https://github.com/thecodepath/android_guides/wiki/Common-Implicit-Intents)