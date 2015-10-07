## Overview

Chrome custom tabs give apps more control over their web experience, and make transitions between native and web content more seamless without having to resort to a WebView.

Chrome custom tabs allow an app to customize how Chrome looks and feels. An app can change things like:

* Toolbar color
* Enter and exit animations
* Add custom actions to the Chrome toolbar and overflow menu

Chrome custom tabs also allow the developer to pre-start Chrome and pre-fetch content for faster loading.

## Quick Integrating with Chrome Custom Tabs:

1. Add the Android Support Library for Chrome Custom Tabs as a dependency to your gradle build file:

```java
compile 'com.android.support:customtabs:23.0.0+'
```

> The library only works on API 16 (Jelly Bean) an above. If you are supporting previous API, you can add `<uses-sdk tools:overrideLibrary="android.support.customtabs"/>` to your manifest to force its use and check the API version at runtime and just use Chrome Custom tabs if its >= 16

2. Copy the following files from [`GoogleChrome`](https://github.com/GoogleChrome/custom-tabs-client) sample git repo to your project and adjust the package names accordingly:

- [CustomTabActivityHelper.java](https://github.com/GoogleChrome/custom-tabs-client/blob/master/demos/src/main/java/org/chromium/customtabsdemos/CustomTabActivityHelper.java)
- [CustomTabsHelper.java](https://github.com/GoogleChrome/custom-tabs-client/blob/master/shared/src/main/java/org/chromium/customtabsclient/shared/CustomTabsHelper.java)
- [KeepAliveService.java](https://github.com/GoogleChrome/custom-tabs-client/blob/master/shared/src/main/java/org/chromium/customtabsclient/shared/KeepAliveService.java)

3. Use the following method to open a Chrome Custom Tab *if possible*. If the user doesn't have a browser that supports Chrome Custom Tabs, it will open the default browser:

```java
CustomTabsIntent customTabsIntent = new CustomTabsIntent.Builder().build();
CustomTabActivityHelper.openCustomTab(this, customTabsIntent, uri,
        new CustomTabActivityHelper.CustomTabFallback() {
            @Override
            public void openUri(Activity activity, Uri uri) {
                Intent intent = new Intent(Intent.ACTION_VIEW, uri);
                startActivity(intent);
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