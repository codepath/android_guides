## Overview

At their I/O 2015 conference, Google announced a new [design support library](http://android-developers.blogspot.com/2015/05/android-design-support-library.html), which helps bring a lot of [[material design components|Material-Design-Primer]] including a navigation drawer view, floating labels, floating action buttons, snackbars, and a new framework to tie motion and scroll events.  The library is supported for Android version 2.3 and higher. 

<img src="http://cdn.androidpolice.com/wp-content/uploads/2014/10/nexus2cee_67-351x625.png" width="200" />
<a href="https://github.com/chrisbanes/cheesesquare"><img src="http://i.stack.imgur.com/Wb28n.png" width="200" /></a>
<a href="http://guides.codepath.com/android/Displaying-the-Snackbar"><img src="https://i.imgur.com/JSdKnP2.png" width="200" /></a>

### Features

The support design library has the following key features:

1. [[FloatingActionButton|Floating-Action-Buttons]] - A round button at the bottom right denoting a primary action on your interface. Promoting key actions within a modern material design app.
2. [[TabLayout|Google-Play-Style-Tabs-using-TabLayout]] - An easier way to put tabs around a `ViewPager` which acts as sliding tabs between fragments within an app.
3. [[NavigationView|Fragment-Navigation-Drawer]] - An easier way to provide a modern navigation drawer from the left with a header and a series of navigation items. 
4. [[SnackBar|Displaying-the-Snackbar]] - Shown on the bottom of the screen and contains text with an optional single action. They automatically time out after the given time length by animating off the screen.
5. [[TextInputLayout|Working-with-the-EditText#displaying-floating-label-feedback]] - Float the hint above any text field as the user is entering information and/or add a character counter. 
6. [[CoordinatorLayout|Handling-Scrolls-with-CoordinatorLayout]] - Provides an additional level of control over scroll and touch events between child views.
     * [AppBarLayout](https://developer.android.com/reference/com/google/android/material/appbar/AppBarLayout.html) allows your toolbar and other views to react to scroll events. 
     * [CollapsingToolbarLayout](https://developer.android.com/reference/com/google/android/material/appbar/CollapsingToolbarLayout.html) extend this to allow the toolbar to collapse as the user scrolls through a view.  
     * [[Bottom Sheets|Handling-Scrolls-with-CoordinatorLayout#bottom-sheets]] to expose a sheet of material that slides up from the bottom of the screen.
7. [[PercentRelativeLayout|Constructing-View-Layouts#percentrelativelayout]] and [PercentFrameLayout](https://developer.android.com/reference/androidx/percentlayout/widget/PercentFrameLayout.html) to enable views to occupy [percentage-based dimensions](https://developer.android.com/reference/androidx/percentlayout/widget/PercentRelativeLayout.html).  **Note** this class is deprecated and `ConstraintLayout` should be used for more complex layouts.
8. [[Vector Drawables|Drawables#vector-drawables]] to reduce the need to include images for every density size.
  * Vector drawables are compatible back to Android 2.1 (API 7), but animated vector drawables are only back-ported to Android 3.0 (API 11).
9. [[Animating view hierarchies|View Hierarchy Animations]] using the [Transitions framework](https://developer.android.com/training/transitions/overview.html) down to Android 4.0 (API 14) .  Currently, there is no backported support for activity/fragment transitions used in this API.
10. [[Bottom Navigation Views]] for easily switching from 3 to 5 tabbed items.

### Setup

Make sure your top-level `build.gradle` includes the following:

```gradle
// Top-level build file where you can add configuration options common to all sub-projects/modules.
buildscript {
    repositories {
        jcenter() 
        google()  // new as of Gradle v4.1 (not to be confused the the Android Gradle plugin below)
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:3.4.2'
    }
}
```

There is a new support design library that must be included. This library also depends on AndroidX [AppCompat](https://developer.android.com/jetpack/androidx/releases/appcompat) library to be included.  If you are not currently using this library, check out this [[migration guide|Migrating-to-the-AppCompat-Library]].  In addition, make sure these versions have been updated.  

Update your root `build.gradle` file:

```gradle
android {
   compileSdkVersion 27  
}

ext {
  appCompatVersion = '1.0.0'
  designSupportVersion = '1.0.0'
  recyclerViewVersion = '1.0.0'
}
```

Add these dependencies to your `app/build.gradle` file:

```gradle
dependencies {
    implementation "androidx.appcompat:appcompat:${appCompatVersion}"
    implementation "com.google.android.material:material:${designSupportVersion}"
}
```

If you are using the [[RecyclerView|Using the RecyclerView]], [[CardView|Using the CardView]], or any other external dependencies make sure to update them.  The RecyclerView for instance has features that are used with this new design support library.

```gradle
dependencies {
    implementation "androidx.recyclerview:recyclerview:${recyclerViewVersion}"
}
```

#### Adding Percent Support Library

To add the percent support library, you need to add this statement:  

```gradle
dependencies {
    implementation "androidx.percentlayout:percentlayout:1.0.0"
}
```

#### Adding Vector Drawable Library

Android Studio v1.4 provides backwards support for vector drawables to pre-Lollipop devices by creating the PNG assets automatically at compile time.  The support library eliminates this necessity by providing vector drawable support for older Android versions, but we need to first disable this auto-generation tool by adding these lines to our `app/build.gradle` configuration:

```gradle

// This config only works for Android Gradle Plugin v2.1.0 or higher:
android {  
   defaultConfig {  
     vectorDrawables.useSupportLibrary = true  
    }  
 }  
```

Finally, the libraries need to be added:

```gradle
dependencies {
    implementation "androidx.vectordrawable:vectordrawable:1.0.0" // VectorDrawableCompat 
    implementation "androidx.vectordrawable:vectordrawable-animated:1.0.0" // AnimatedVectorDrawableCompat
}
```

Check out the [[vector drawables guide|Drawables#vector-drawables]] for usage details.

#### Adding Transitions Library

The Transitions API was first introduced in Android 4.4 (KitKat) but now includes back ported support for animating view hierarchies down to Android 4.0.   However, there is no support for activity/fragment transitions currently.  To use this library, you need to add the library explicitly:

```gradle
dependencies {
    implementation "androidx.transition:transition:1.1.0"
}
```

#### Adding Annotations Library

To leverage the [annotations library](http://tools.android.com/tech-docs/support-annotations), you can also explicitly add it to your Gradle file.  Adding the [[AppCompat|Migrating-to-the-AppCompat-Library]] or support design library implicitly adds it:

```gradle
dependencies {
    implementation "androidx.annotation:annotation:1.1.0"
}
```

### Sample Code

Sample code uses outdated Support Library, check [Migration Guide](https://developer.android.com/jetpack/androidx/migrate) if you want to copy the code from samples. If you want to see how to use the various components, check out this [sample code](https://github.com/chrisbanes/cheesesquare).  For an example of the percent support library, see [this sample code](https://github.com/JulienGenoud/android-percent-support-lib-sample).


### Change Log

For changes and latest versions of AndroidX library visit official [Versions page](https://developer.android.com/jetpack/androidx/versions). Remember that AndroidX replaced all old Support Libraries.

## References
* <https://developer.android.com/jetpack/androidx>
* <https://android-developers.googleblog.com/2018/05/hello-world-androidx.html>
* <https://medium.com/mindorks/upgrading-to-material-components-ebc21ac4e95a>
* <https://www.androidauthority.com/android-jetpack-android-support-library-878587/>
* <https://material.io/develop/android/docs/getting-started/>
* <https://www.reddit.com/r/androiddev/comments/a2bikh/should_i_learn_about_androidx/>
* <https://medium.com/android-bites/first-steps-with-the-design-support-library-8dcf06230ddd>
* <http://android-developers.blogspot.com/2015/05/android-design-support-library.html>
* <https://github.com/chrisbanes/cheesesquare>
* <http://hmkcode.com/material-design-app-android-design-support-library-appcompat/>
* <https://medium.com/ribot-labs/exploring-the-new-android-design-support-library-b7cda56d2c32>
* <https://plus.google.com/+AndroidDevelopers/posts/XTtNCPviwpj>
* <https://code.google.com/p/android/issues/list?can=1&q=label%3AVersion-22.2.1>
* <https://plus.google.com/+AndroidDevelopers/posts/RZutBRWN6sH?linkId=17978076>
* <https://medium.com/@chrisbanes/appcompat-v23-2-age-of-the-vectors-91cbafa87c88#.dvbsz7sts>
* <https://plus.google.com/+AndroidDevelopers/posts/iTDmFiGrVne>
* <https://www.reddit.com/r/androiddev/comments/4y70e7/android_support_library_v242_released/>
