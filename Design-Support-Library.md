## Overview

At their I/O 2015 conference, Google announced a new [design support library](http://android-developers.blogspot.com/2015/05/android-design-support-library.html), which helps bring a lot of [[material design components|Material-Design-Primer]] including a navigation drawer view, floating labels, floating action buttons, snackbars, and a new framework to tie motion and scroll events.  The library is supported for Android version 2.1 and higher. 

<img src="http://cdn.androidpolice.com/wp-content/uploads/2014/10/nexus2cee_67-351x625.png" width="200" />
<a href="https://github.com/chrisbanes/cheesesquare"><img src="http://i.stack.imgur.com/Wb28n.png" width="200" /></a>
<a href="https://github.com/codepath/android_guides/wiki/Displaying-the-Snackbar"><img src="http://i.imgur.com/JSdKnP2.png" width="200" /></a>

### Setup

Open the [SDK Manager](https://developer.android.com/tools/support-library/setup.html) and make sure to download the `Android Support Repository` as well as the latest `Android Support Library`.  You can also use [[Jake Wharton's SDK Manager|Installing-Android-SDK-Tools#installing-the-android-sdk-automated-way]] to download the packages for you.  Your builds will not find the package unless your first download these dependencies.

Make sure that you have at least the Android Gradle plugin v1.2.3 supported.  There are several issues with using older versions including some support library widgets fail to render correctly (see [issue](https://code.google.com/p/android/issues/detail?id=170841)).  

```gradle
    dependencies {
        classpath 'com.android.tools.build:gradle:1.2.3'
    }
```
There is a new support design library that must be included.   This library also depends on updated versions of the [AppCompat](http://android-developers.blogspot.com/2014/10/appcompat-v21-material-design-for-pre.html) library to be included.  If you are not currently using this library, check out this [migration guide](https://github.com/codepath/android_guides/wiki/Migrating-to-the-AppCompat-Library).  In addition, make sure these versions have been updated.  

   ```gradle
        dependencies {
          compile 'com.android.support:appcompat-v7:22.2.0'
          compile 'com.android.support:design:22.2.0'
      }
   ```

If you are using the [[RecyclerView|Using the RecyclerView]], [[CardView|Using the CardView]], or any other [support v7 related](https://developer.android.com/tools/support-library/features.html#v7) libraries you should also upgrade the versions.  The RecyclerView for instance has features that are used with this new design support library.

```gradle
        dependencies {
          compile 'com.android.support:recyclerview-v7:22.2.0'
      }
   ```

### Features

The support design library has the following key features:

1. [[FloatingActionButton|Floating-Action-Buttons]] - A round button at the bottom right denoting a primary action on your interface. Promoting key actions within a modern material design app.
2. [[TabLayout|Google-Play-Style-Tabs-using-TabLayout]] - An easier way to put tabs around a `ViewPager` which acts as sliding tabs between fragments within an app.
3. [[NavigationView|Fragment-Navigation-Drawer]] - An easier way to provide a modern navigation drawer from the left with a header and a series of navigation items. 
4. [[SnackBar|Displaying-the-Snackbar]] - Shown on the bottom of the screen and contains text with an optional single action. They automatically time out after the given time length by animating off the screen.
5. [[TextInputLayout|Working-with-the-EditText#displaying-floating-label-feedback]] - Float the hint above any text field as the user is entering information. 
6. [[CoordinatorLayout|Handling-Scrolls-with-CoordinatorLayout]] - Provides an additional level of control over scroll and touch events between child views.
     * [AppBarLayout](http://developer.android.com/reference/android/support/design/widget/AppBarLayout.html) allows your toolbar and other views to react to scroll events. 
     * [CollapsingToolbarLayout](http://developer.android.com/reference/android/support/design/widget/CollapsingToolbarLayout.html) extend this to allow the toolbar to collapse as the user scrolls through a view.  

### Sample Code

If you want to see how to use the various components, check out this [sample code](https://github.com/chrisbanes/cheesesquare).

### Official Source Code

The source code for this library can be found [here](https://android.googlesource.com/platform/frameworks/support.git/+/master/design/).  For instance, if you are curious about what styles can be overridden for the different components, see this [link](https://android.googlesource.com/platform/frameworks/support.git/+/master/design/res/values/styles.xml).

Currently, the source code for CoordinatorLayout and AppBarLayout do not appear to be publicly available.  Assuming you are using Android Studio v1.2 or higher, the decompiler code should enable you to understand better what's happening.

## References

* <https://medium.com/android-bites/first-steps-with-the-design-support-library-8dcf06230ddd>
* <http://android-developers.blogspot.com/2015/05/android-design-support-library.html>
* <https://github.com/chrisbanes/cheesesquare>
* <http://hmkcode.com/material-design-app-android-design-support-library-appcompat/>
* <https://medium.com/ribot-labs/exploring-the-new-android-design-support-library-b7cda56d2c32>