At the Google I/O 2015 conference, Google announced a new [support design library](http://android-developers.blogspot.com/2015/05/android-design-support-library.html), which helps bring a lot of material design components including a navigation drawer view, floating labels, floating action buttons, snackbars, and a new framework to tie motion and scroll events.

### Support Design Library

1. Make sure that you have at least Gradle v1.2.3 supported.  There are several issues with using older versions including some support library widgets fail to render correctly (see  https://code.google.com/p/android/issues/detail?id=170841).  

   ```gradle
      dependencies {
        classpath 'com.android.tools.build:gradle:1.2.3'
      }
   ```
2. There is a new support design library that must be included.   This library also depends on updated versions of the [AppCompat](http://android-developers.blogspot.com/2014/10/appcompat-v21-material-design-for-pre.html) library and support v4 library to be included.  Make sure these versions have been updated too.

   ```gradle
        dependencies {
          compile 'com.android.support:appcompat-v7:22.2.0'
          compile 'com.android.support:support-v4:22.2.0'
          compile 'com.android.support:design:22.2.0'
      }
   ```
   **Note** The [official Google docs](http://developer.android.com/tools/support-library/features.html#design) suggest that the Gradle line is `support-design`, but it   appears to be a typo.  A ticket is filed [here](https://code.google.com/p/android/issues/detail?id=175066).