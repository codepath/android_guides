## Overview

Android 7.1 allows you to define shortcuts to specific actions in your app. These shortcuts can be displayed in a supported launcher, such as the one provided with Nexus and Pixel devices. Shortcuts let your users quickly start common or recommended tasks within your app:

<img src="http://i.imgur.com/GRhy6Cx.png" width="300" />

To implement it you have to start with downloading the latest SDK (API level 25). And after updating the SDK you have to update your build.gradle file to point the latest version.

```gradle
android {
    compileSdkVersion 25
    buildToolsVersion "25.0.0"

    defaultConfig {
        applicationId "com.example.app"
        targetSdkVersion 25
        ...
    }
```

## Usage

### Static shortcut

In your app’s manifest file (`AndroidManifest.xml`), find an activity whose intent filters are set to the `android.intent.action.MAIN` action and the `android.intent.category.LAUNCHERcategory`

```xml
<activity
    android:name=".SplashActivity"
    android:screenOrientation="portrait"
    android:theme="@style/AppTheme.BrandedLaunch">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />

        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
    <meta-data android:name="android.app.shortcuts"
        android:resource="@xml/shortcut" />
</activity>
```

Your `shortcut.xml` will be under xml-v25 under res folder and will look like this
```xml
<shortcuts xmlns:android="http://schemas.android.com/apk/res/android">
    <shortcut
        android:enabled="true"
        android:icon="@drawable/icon_name"
        android:shortcutId="shortcut_name"
        android:shortcutShortLabel="@string/name">
        <intent
            android:action="android.intent.action.VIEW"
            android:targetClass=".MainActivity"
            android:targetPackage="com.example.app">
        </intent>
        <categories android:name="android.shortcut.conversation" />
    </shortcut>
</shortcuts>
```
In the above Intent if your Activity is expecting some extras you can always pass it with extra tag like this.
```xml
<intent
    ...
       <extra
             android:name="key"
             android:value="value" />
</intent>
```
![Static Shortcuts](https://i.imgur.com/CZYtNT2.jpg)

So this will create a static shortcut. So we also have Dynamic Shortcuts as well lets see what are they. If you want to always show a shortcut in your app you can go with static shortcut, if you want some shortcuts to be Dynamic like text to a person which is Dynamic not same for all then you have to use Dynamic Shortcuts.

### Dynamic Shortcut

To create a Dynamic Shortcut you will have to use these codes in your activity where you like to initialize the Shortcut.

```java
@TargetApi(25)
private void createShorcut() {
    ShortcutManager sM = getSystemService(ShortcutManager.class);

    Intent intent1 = new Intent(getApplicationContext(), Activity1.class);
    intent1.setAction(Intent.ACTION_VIEW);

    ShortcutInfo shortcut1 = new ShortcutInfo.Builder(this, "shortcut1")
            .setIntent(intent1)
            .setShortLabel(getString(R.string.shortcut1))
            .setLongLabel("Shortcut 1")
            .setShortLabel("This is the shortcut 1")
            .setDisabledMessage("Login to open this")
            .setIcon(Icon.createWithResource(this, R.drawable.shortcut1))
            .build();

    sM.setDynamicShortcuts(Arrays.asList(shortcut1));
}
```

ShortcutManager uses system service so this has to be in an activity and remember this only works on Android 7.1 so it's good to add the annotation `@TargetApi(25)` to this code to avoid compile errors and add version check before calling these methods.

```java
if (Build.VERSION.SDK_INT >= 25) {
    createShorcut();
}
if (Build.VERSION.SDK_INT >= 25) {
    removeShorcuts();
}
```

Also remember the Intent action `setAction();` has to be set.
Remember users can pin the shortcuts as well, so it is dynamic or static they can always pin it, even if you remove dynamic shortcuts users will still have it if pinned. So if you are using Dynamic Shortcuts and you can’t do certain actions you have to disable those shortcuts. And that is where the Disabled messages get displayed.

```java
@TargetApi(25)
private void removeShorcuts() {
    ShortcutManager shortcutManager = getSystemService(ShortcutManager.class);
    shortcutManager.disableShortcuts(Arrays.asList("shortcut1"));
    shortcutManager.removeAllDynamicShortcuts();
}
```
So the method `disableShortcuts(List<String>)` will disable the shortcuts whose id is passed. `removeAllDynamicShortcuts();` will remove all the dynamic shortcuts.
See the Images below for example, left is normal shortcuts for a logged in user and right has two disabled shortcuts for Guest users and when pressed it displays the Disabled messages.

![All shortcuts enabled](https://i.imgur.com/5VTj7RS.jpg)

![Some shortcuts disabled](https://i.imgur.com/OGC9eYk.jpg)


So this is how we use the new feature app shortcuts on Android 7.1, an app can have 5 shortcuts. So you can use `setRank(int)` on `ShortcutInfo` to change the order of the shortcuts.