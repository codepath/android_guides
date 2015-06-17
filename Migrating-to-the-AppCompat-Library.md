# Overview

The [AppCompat](https://developer.android.com/tools/support-library/features.html#v7) support library enables the use of the ActionBar and Material Design specific implementations for older devices down to Android v2.1.   Currently, new projects created through Android Studio incorporate this library by default.  You can confirm by looking at the `build.gradle` file to see the AppCompat library being set:

```gradle
dependencies {
    compile 'com.android.support:appcompat-v7:22.2.0'
}
```


### Search and replacing changes

Older projects may not include this library, so migrating requires changing the base theme and many of the main imports outlined in this [blog post](http://android-developers.blogspot.com/2014/10/appcompat-v21-material-design-for-pre.html).  Because the class declarations are not compatible, you need to make sure you are using the support library definitions entirely.  Otherwise your app is likely to crash.

The simplest is often to do a search-and-replace to start changing the following statements to start using the support libraries.

#### Activities Changes

 * `import android.app.Activity` -> `import android.support.v7.app.AppCompatActivity`
 * `extends Activity` -> `extends AppCompatActivity`

#### Fragment Changes

 * `extends FragmentActivity` -> `extends AppCompatActivity`
 * `import android.support.v4.app.FragmentActivity` -> `import android.support.v7.app.AppCompatActivity` 
 * `import android.app.Fragment` -> `import android.support.v4.app.Fragment`
 * `getFragmentManager()` -> `getSupportFragmentManager()`

#### ActionBar Changes

 * `import android.app.ActionBar` -> `import android.support.v7.app.ActionBar`
 * `getActionBar()` -> `getSupportActionBar()`

#### Theme XML Changes

If you were migrating from the Holo theme, your new `themes.xml` would inherit from `Theme.AppCompat` instead of `android:Theme.Holo.Light`:

```xml
-    <style name="AppTheme" parent="android:Theme.Holo.Light.DarkActionBar">
+    <style name="AppTheme" parent="Theme.AppCompat.Light.DarkActionBar">
```

#### Menu XML Changes

For your menu/ layout files, add an `app:` namespace .  For menu items, the `showAsAction` must be from the `app` namespace instead of `android` namespace.  It is considered a custom attribute of the support library and not supported in the main Android framework.

```xml
-<menu xmlns:android="http://schemas.android.com/apk/res/android">
+<menu xmlns:android="http://schemas.android.com/apk/res/android" xmlns:app="http://schemas.android.com/apk/res-auto">
 <item android:id="@+id/myMenuItem"
       android:title="@string/select"
-      android:showAsAction="ifRoom"
+      app:showAsAction="ifRoom"
```

If you are using widgets such as the SearchView, you must also use `android.support.v7.widget.SearchView` instead of `android.widget.SearchView`.  Note that the `app` namespace must also be used.

```xml
+<menu xmlns:android="http://schemas.android.com/apk/res/android" xmlns:app="http://schemas.android.com/apk/res-auto">
 
     <item android:id="@+id/contentSearch"
           android:orderInCategory="2"
           android:title="@string/search"
-          android:showAsAction="ifRoom"
-          android:actionViewClass="android.widget.SearchView">
+          app:showAsAction="ifRoom"
+          app:actionViewClass="android.support.v7.widget.SearchView">
```

### Changing targetSDKVersion

In addition, setting the `targetSdkVersion` to the latest SDK version ensures that the  AppCompat library will attempt to apply the Material Design assuming the device itself can support it.  In other words, `targetSdkVersion` only is used as an extra check to determine whether the SDK version should even be supported or not.  The support library will still check to see if the minimum SDK version is being used on the device.

```gradle
android {
    targetSdkVersion 21
```



# References

* <http://www.androidauthority.com/goodbye-menu-button-hello-action-bar-48312/>
* <http://android-developers.blogspot.in/2013/08/actionbarcompat-and-io-2013-app-source.html>
* <http://android-developers.blogspot.com/2014/10/appcompat-v21-material-design-for-pre.html>