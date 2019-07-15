## Overview

The [AppCompat](https://developer.android.com/tools/support-library/features.html#v7) support library enables the use of the ActionBar and Material Design specific implementations such as [Toolbar](https://developer.android.com/reference/android/support/v7/widget/Toolbar.html) for older devices down to Android v2.1. 

**Caution**: Starting with Support Library release 26.0.0 (July 2017), the minimum supported API level across most support libraries has increased to Android 4.0 (API level 14) for most library packages [https://developer.android.com/topic/libraries/support-library/index.html#api-versions](https://developer.android.com/topic/libraries/support-library/index.html#api-versions)

**Android Studio v3.4.1 and higher will now start forcing [AndroidX](https://developer.android.com/jetpack/androidx) namespaces.** See [this document](https://developer.android.com/jetpack/androidx/migrate) for migrating libraries and imports.

```gradle
android {
    compileSdkVersion 29
}

dependencies {
    // See https://developer.android.com/jetpack/androidx/releases/appcompat
    implementation 'androidx.appcompat:appcompat-1.0.2'
}
```

# References

* <http://www.androidauthority.com/goodbye-menu-button-hello-action-bar-48312/>
* <http://android-developers.blogspot.in/2013/08/actionbarcompat-and-io-2013-app-source.html>
* <http://android-developers.blogspot.com/2014/10/appcompat-v21-material-design-for-pre.html>
* <https://chris.banes.me/2015/04/22/support-libraries-v22-1-0/>