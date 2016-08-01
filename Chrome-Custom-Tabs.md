## Overview

[Chrome custom tabs](https://developer.chrome.com/multidevice/android/customtabs) give apps more control over their web experience, and make transitions between native and web content more seamless without having to resort to a [[WebView|Working with the WebView]].

Chrome custom tabs allow an app to customize how Chrome looks and feels. An app can change things like:

* Toolbar color
* Enter and exit animations
* Add custom actions to the Chrome toolbar and overflow menu

Chrome custom tabs also allow the developer to pre-start Chrome and pre-fetch content for faster loading.

See [this README](https://github.com/GoogleChrome/custom-tabs-client/blob/master/Using.md) more ways to use Chrome Custom Tabs.

## Setup

You will need to have the Chrome app installed on your phone.  If you are using an emulator, you must setup [[Google Play Services|Genymotion-2.0-Emulators-with-Google-Play-support#setup-google-play-services]] and install the Chrome app though the Play store.

Add the Android Support Library for Chrome Custom Tabs as a dependency to your gradle build file:

```java
compile 'com.android.support:customtabs:24.0.0+'
```

The library only works on API 16 (Jelly Bean) an above. If you are supporting previous API, you can add `<uses-sdk tools:overrideLibrary="android.support.customtabs"/>` to your manifest to force its use and check the API version at runtime and just use Chrome Custom tabs if its >= 16

The most basic example to launch a Chrome tab is through a custom intent as shown below: 

```java
String url = "https://www.codepath.com/";
CustomTabsIntent.Builder builder = new CustomTabsIntent.Builder();
// set toolbar color and/or setting custom actions before invoking build()
CustomTabsIntent customTabsIntent = builder.build();
customTabsIntent.launchUrl(this, Uri.parse(url));
```

If you do not have Chrome installed, the intent will launch the default browser installed on the device.  The `CustomTabsIntent` simply launches an [[implicit intent|Common-Implicit-Intents]] (`android.intent.action.VIEW`) and passes an extra data in the intent (i.e. `android.support.customtabs.extra.SESSION` and `android.support.customtabs.extra.TOOLBAR_COLOR`) that gets ignored if the default browser cannot process this information.

### Setting Toolbar color

If you wish to set the toolbar color, you can use the `setToolbarColor()` method in the builder class:

```java
CustomTabsIntent.Builder builder = new CustomTabsIntent.Builder();
// set toolbar color
builder.setToolbarColor(ContextCompat.getColor(this, R.color.colorAccent));
```

Normally, `context.getResources().getColor())` can be used, but in Android API 23 this method has been [deprecated](http://stackoverflow.com/questions/31590714/getcolorint-id-deprecated-on-android-6-0-marshmallow-api-23).   For this reason, see this guide for how to include the [[design support library|Design Support Library]] to leverage a new `ContextCompat` API.  

### Enable pre-starting and pre-fetching

Chrome custom tabs also allow the developer to pre-start Chrome and pre-fetch content for faster loading.

Copy the following files from [`GoogleChrome`](https://github.com/GoogleChrome/custom-tabs-client) sample git repo to your project and adjust the package names accordingly:

- [CustomTabActivityHelper.java](https://github.com/GoogleChrome/custom-tabs-client/blob/master/demos/src/main/java/org/chromium/customtabsdemos/CustomTabActivityHelper.java)
- [CustomTabsHelper.java](https://github.com/GoogleChrome/custom-tabs-client/blob/master/shared/src/main/java/org/chromium/customtabsclient/shared/CustomTabsHelper.java)
- [KeepAliveService.java](https://github.com/GoogleChrome/custom-tabs-client/blob/master/shared/src/main/java/org/chromium/customtabsclient/shared/KeepAliveService.java)
- [ServiceConnection.java](https://github.com/GoogleChrome/custom-tabs-client/blob/master/shared/src/main/java/org/chromium/customtabsclient/shared/ServiceConnection.java)
- [ServiceConnectionCallback.java](https://github.com/GoogleChrome/custom-tabs-client/blob/master/shared/src/main/java/org/chromium/customtabsclient/shared/ServiceConnectionCallback.java)

Use the following method to open a Chrome Custom Tab *if possible*. If the user doesn't have a browser that supports Chrome Custom Tabs, it will open the default browser:

```java
CustomTabsIntent customTabsIntent = new CustomTabsIntent.Builder().build();
CustomTabActivityHelper.openCustomTab(this, customTabsIntent, uri,
        new CustomTabActivityHelper.CustomTabFallback() {
            @Override
            public void openUri(Activity activity, Uri uri) {
                Intent intent = new Intent(Intent.ACTION_VIEW, uri);
                activity.startActivity(intent);
            }
        });
```

## Next Steps

The above quick integration example will open your Uri on a Chrome Custom Tab without warming up, pre-fetching or UI customizations.

You can find an example on how to connect to the Chrome Custom Tabs service to use warm-up and pre-fetching on the [ServiceConnectionActivity](https://github.com/GoogleChrome/custom-tabs-client/blob/master/demos/src/main/java/org/chromium/customtabsdemos/ServiceConnectionActivity.java) sample from the Google Chrome Team.

For more information on possible UI customizations, check the [CustomUIActivity](https://github.com/GoogleChrome/custom-tabs-client/blob/master/demos/src/main/java/org/chromium/customtabsdemos/CustomUIActivity.java) sample from the Google Chrome Team.

## References

* <https://developer.chrome.com/multidevice/android/customtabs>
* <https://github.com/GoogleChrome/custom-tabs-client>