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
     * [AppBarLayout](http://developer.android.com/reference/android/support/design/widget/AppBarLayout.html) allows your toolbar and other views to react to scroll events. 
     * [CollapsingToolbarLayout](http://developer.android.com/reference/android/support/design/widget/CollapsingToolbarLayout.html) extend this to allow the toolbar to collapse as the user scrolls through a view.  
     * [[Bottom Sheets|Handling-Scrolls-with-CoordinatorLayout#bottom-sheets]] to expose a sheet of material that slides up from the bottom of the screen.
7. [[PercentRelativeLayout|Constructing-View-Layouts#percentrelativelayout]] and [PercentFrameLayout](https://developer.android.com/reference/android/support/percent/PercentFrameLayout.html) to enable views to occupy [percentage-based dimensions](http://developer.android.com/reference/android/support/percent/PercentRelativeLayout.html).  
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
        maven { url "https://maven.google.com" }  // new as of Google I/O 2017
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:2.3.1'
    }
}
```

There is a new support design library that must be included.   This library also depends on updated versions of the [AppCompat](http://android-developers.blogspot.com/2014/10/appcompat-v21-material-design-for-pre.html) library to be included.  If you are not currently using this library, check out this [[migration guide|Migrating-to-the-AppCompat-Library]].  In addition, make sure these versions have been updated.  

Update your root `build.gradle` file:

```gradle
android {
   compileSdkVersion 26  // usually needs to be consistent with major support libs used, but not necessary
}

ext {
  supportLibVersion = '26.1.0'  // variable that can be referenced to keep support libs consistent
}
```

Add these dependencies to your `app/build.gradle` file:

```gradle
dependencies {
    compile "com.android.support:appcompat-v7:${supportLibVersion}"
    compile "com.android.support:design:${supportLibVersion}"
}
```

If you are using the [[RecyclerView|Using the RecyclerView]], [[CardView|Using the CardView]], or any other [support v7 related](https://developer.android.com/tools/support-library/features.html#v7) libraries you should also upgrade the versions.  The RecyclerView for instance has features that are used with this new design support library.

```gradle
dependencies {
    compile "com.android.support:recyclerview-v7:${supportLibVersion}"
}
```

#### Adding Percent Support Library

To add the percent support library, you need to add this statement:  

```gradle
dependencies {
    compile "com.android.support:percent:${supportLibVersion}"
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
    compile "com.android.support:support-vector-drawable:${supportLibVersion}" // VectorDrawableCompat 
    compile "com.android.support:animated-vector-drawable:${supportLibVersion}" // AnimatedVectorDrawableCompat
}
```

Check out the [[vector drawables guide|Drawables#vector-drawables]] for usage details.

#### Adding Transitions Library

The Transitions API was first introduced in Android 4.4 (KitKat) but now includes back ported support for animating view hierarchies down to Android 4.0.   However, there is no support for activity/fragment transitions currently.  To use this library, you need to add the library explicitly:

```gradle
dependencies {
    compile "com.android.support:transition:${supportLibVersion}"
}
```

#### Adding Annotations Library

To leverage the [annotations library](http://tools.android.com/tech-docs/support-annotations), you can also explicitly add it to your Gradle file.  Adding the [[AppCompat|Migrating-to-the-AppCompat-Library]] or support design library implicitly adds it:

```gradle
dependencies {
    compile "com.android.support:support-annotations:${supportLibVersion}"
}
```

#### Installing the Library

You normally need to open the [SDK Manager](https://developer.android.com/tools/support-library/setup.html) and make sure to download the `Android Support Repository` as well as the latest `Android Support Library`.   However, Android Studio will also show at the bottom any missing libraries and you can click on the `Install repository and sync project`.  The process will only succeed if you specify a valid library and version, but it enables you to upgrade without needing to open the SDK Manager.

<img src="https://imgur.com/GQw6IHt.png"/>

If you are using any type of continuous build system and need to help automate downloading of updates to the support library, you can use [[Jake Wharton's SDK Manager|Installing-Android-SDK-Tools#installing-the-android-sdk-automated-way]] to download the packages for you.  

### Sample Code

If you want to see how to use the various components, check out this [sample code](https://github.com/chrisbanes/cheesesquare).  For an example of the percent support library, see [this sample code](https://github.com/JulienGenoud/android-percent-support-lib-sample).

### Official Source Code

The Android Open Source Project (AOSP) hosts the major release versions for this library, which can be found [here](https://android.googlesource.com/platform/frameworks/support.git/+/master/design/).  For instance, if you are curious about what styles can be overridden for the different components, see this [link](https://android.googlesource.com/platform/frameworks/support.git/+/master/design/res/values/styles.xml).

The latest source code updates for the support library are now always included since v23.1.0 in your SDK directory (i.e. `Library/Android/sdk/extras/android/m2repository/com/android/support/design` for Mac OS X).

### Change Log

#### Changes in Support Library v23

- `Resources.getColor()` has been deprecated.  You must now use `ContextCompat.getColor()` instead.  See this [Stack Overflow article](http://stackoverflow.com/questions/31590714/getcolorint-id-deprecated-on-android-6-0-marshmallow-api-23) or the official [API documentation](http://developer.android.com/reference/android/content/res/Resources.html).

#### Changes in Support Library v23.1 

- `TextInputLayout` and `EditText` now includes the ability to add a character counter.  ([[view guide |Working-with-the-EditText#adding-character-counting]])
 
- A `snap` flag can also be added to the list of scrolling effects declared in `AppBarLayout`.  ([[view guide|Handling-Scrolls-with-CoordinatorLayout#responding-to-scroll-events]])

- A `setOnDragListener()` method has been added to `AppBarLayout`.  ([[view guide|Handling-Scrolls-with-CoordinatorLayout#embedding-google-maps-in-appbarlayout]])

- An `aspectRatio` attribute is now supported in `PercentRelativeLayout`.  ([[view guide|Constructing-View-Layouts#aspect-ratio]])

- A new item animation interface for the `RecyclerView`.  ([[view guide|Using-the-RecyclerView#new-itemanimator-interface]])

- Custom views can be provided for `NavigationView` rows.  ([[view guide|Fragment-Navigation-Drawer#adding-custom-views-to-navigation-drawer]])

#### Changes in Support Library v23.1.1

- NavigationView now contains a `getHeaderView()` method ([[view guide|Fragment-Navigation-Drawer#getting-references-to-the-header]])

#### Changes in Support Library v23.2

- Added support for bottom sheets. ([[view guide|Handling-Scrolls-with-CoordinatorLayout#bottom-sheets]])
- Added setup instructions for vector drawables. ([[view guide|Drawables#vector-drawables]])

#### Changes in Support Library v24.2.0

- `TextInputLayout` and `EditText` now includes the ability to add password visibility toggles.  ([[view guide |Working-with-the-EditText#adding-password-visibility-toggles]])

- Added DiffUtil class for RecyclerView.  ([[view guide|Using-the-RecyclerView#diffing-larger-changes]])

- Support v4 library modules have been broken apart but cannot be used to reduce APK size because the fragment library still depends on all other related modules.  ([[view guide|Migrating-to-the-AppCompat-Library#overview]])

- Transitions API backported to Android 4.0 but does not include support for activity/fragment transitions. ([[view guide|View-Hierarchy-Animations]])

#### Changes in Support Library v25.0.0

- Bottom Navigation Views support added ([[view guide|Bottom Navigation Views]])

## References

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
