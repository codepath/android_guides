## Overview

Often your app will have secret credentials or API keys that you need to have in your app to function but you'd rather not have easily extracted from your app. The following common strategies exist for storing secrets in your source code:

 * [[Embedded in resource file|Storing-Secret-Keys-in-Android#secrets-in-resource-files]]
 * Hidden as constants in source code
 * [Hidden in BuildConfigs](http://www.rainbowbreeze.it/environmental-variables-api-key-and-secret-buildconfig-and-android-studio/)
 * [[Obfuscating with Proguard|Configuring-ProGuard]]
 * [Disguised or Encrypted Strings](https://developer.android.com/training/articles/keystore.html)
 * Hidden in Native Libraries with NDK

However, **none of these strategies will ensure the protection of your keys** and [your secrets aren't safe](https://rammic.github.io/2015/07/28/hiding-secrets-in-android-apps/). The best way to protect secrets is to never reveal them in the code in the first place. Compartmentalizing sensitive information and operations on your own backend server/service should always be your first choice. If you do have to consider a hiding scheme, you should do so with the realization that you can only make the reverse engineering process harder  and may add significant complication to the development, testing, and maintenance of your app in doing so.

The simplest and most straightforward approach is outlined below which is to **store your secrets within a resource file**. Keep in mind that most of the other more complex approaches above are at best only marginally more secure.

## Secrets in Resource Files

The simplest approach for storing secrets in to keep them as resource files that are simply not checked into source control. Start by creating a resource file for your secrets called `res/values/secrets.xml` with:

```xml
<!-- Inside of `res/values/secrets.xml` -->
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string name="parse_application_id">xxxxxx</string>
    <string name="parse_client_secret">yyyyyy</string>
    <string name="google_maps_api_key">zzzzzz</string>
</resources>
```

Once these keys are in the file, Android will automatically merge it into your resources, where you can access them exactly as you would your normal strings. You can access the secret values with:

```java
Parse.initialize(
    this, 
    getString(R.string.parse_application_id),
    getString(R.string.parse_client_secret)
);
```

If you need your keys in another XML file such as in `AndroidManifest.xml`, you can just use the XML notation for accessing string resources:

```xml
<meta-data
    android:name="com.google.android.maps.v2.API_KEY"
    android:value="@string/google_maps_api_key"/>
```

Since your secrets are now in an individual file, they're simple to ignore in your source control system (for example, in Git, you would add this to the '.gitignore' file in your repository):

```
echo "**/*/res/values/secrets.xml" > .gitignore
```

This process is not bulletproof. As resources, they are somewhat more vulnerable to decompilation of your application package, and so they are discoverable if somebody really wants to know them. This solution does, however, prevent your secrets just sitting in plaintext in source control waiting for someone to use, and also has the advantage of being simple to use, leveraging Android's resource management system, and requiring no extra libraries.

## Resources

* <https://joshmcarthur.com/2014/02/16/keeping-secrets-in-an-android-application.html>
* <https://rammic.github.io/2015/07/28/hiding-secrets-in-android-apps/>