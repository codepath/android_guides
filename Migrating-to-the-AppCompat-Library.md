## Overview

The [AppCompat](https://developer.android.com/tools/support-library/features.html#v7) support library enables the use of the ActionBar and Material Design specific implementations such as [Toolbar](https://developer.android.com/reference/android/support/v7/widget/Toolbar.html) for older devices down to Android v2.1.   Currently, new projects created through Android Studio incorporate this library by default.  You can confirm by looking at the `build.gradle` file to see the AppCompat library being set:

```gradle
dependencies {
    compile 'com.android.support:appcompat-v7:22.2.1'
}
```

Note that the AppCompat library has an implicit dependency on the support-v4 library.  The support-v4 declaration however does not necessarily need to be declared.

## Search and replacing changes

Older projects may not include this library, so migrating requires changing the theme references and many of the main imports described in this [blog post](http://android-developers.blogspot.com/2014/10/appcompat-v21-material-design-for-pre.html).  Because the support class declarations are not compatible with the standard Android ones, you need to make sure you are using the imports from the support library entirely.  Otherwise your app is likely to crash.

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

#### AlertDialog Changes

Your AlertDialogs should import from the AppCompat support library instead, which takes advantage of the new Material Design theme.

<img src="https://blog.xamarin.com/wp-content/uploads/2015/05/alertdialogs.png"/>

 * `import android.app.AlertDialog` -> `import android.support.v7.app.AlertDialog`

#### Theme XML Changes

If you were migrating from the Holo theme, your new theme  would inherit from `Theme.AppCompat` instead of `android:Theme.Holo.Light`:

```xml
-    <style name="AppTheme" parent="android:Theme.Holo.Light.DarkActionBar">
+    <style name="AppTheme" parent="Theme.AppCompat.Light.DarkActionBar">
```

If you wish to have a style that disables the ActionBar in certain Activity screens but still wish to use many of the ones you custom defined, you can inherit from `AppTheme` and apply the same styles declared in `Theme.AppCompat.NoActionBar`:

```xml
    <style name="AppTheme.NoActionBar">
        <item name="windowActionBar">false</item>
        <item name="windowNoTitle">true</item>
    </style>
```
If you see `AppCompat does not support the current theme features`, it's likely the `windowNoTitle` setting has not been set or is set to false.  There is more strict enforcement on what this value must be set on newer AppCompat libraries.  See this [Stack Overflow](http://stackoverflow.com/questions/29790070/upgraded-to-appcompat-v22-1-0-and-now-getting-illegalargumentexception-appcompa) article for more context.

#### Menu XML Changes

For your menu/ layout files, add an `app:` namespace .  For menu items, the `showAsAction` must be from the `app` namespace instead of `android` namespace.  It is considered a custom attribute of the support library and will no otherwise be processed correctly without making this change.

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

#### Changes to Menu Options

The `MenuItemCompat` helper class has a few static methods that should be used in lieu of `MenuItem`:

```java
  @Override
    public void onCreateOptionsMenu(Menu menu, MenuInflater inflater) {
        inflater.inflate(R.menu.my_menu, menu);
        mSearchView = (SearchView) MenuItemCompat.getActionView(menu.findItem(R.id.contentSearch));
```

#### Changing targetSDKVersion

In addition, setting the `targetSdkVersion` to the latest SDK version ensures that the  AppCompat library will attempt to apply the Material Design assuming the device itself can support it. The support library will still check to see if the minimum SDK version is being used on the device.

```gradle
android {
    targetSdkVersion 21
```

### Known issues

The AppCompat library has issues with Samsung v4.2.2 devices.  See [this issue](https://code.google.com/p/android/issues/detail?id=78377) for more details.

# References

* <http://www.androidauthority.com/goodbye-menu-button-hello-action-bar-48312/>
* <http://android-developers.blogspot.in/2013/08/actionbarcompat-and-io-2013-app-source.html>
* <http://android-developers.blogspot.com/2014/10/appcompat-v21-material-design-for-pre.html>
* <https://chris.banes.me/2015/04/22/support-libraries-v22-1-0/>