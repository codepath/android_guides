This page will compile common issues experienced with Android Studio 1.0 or above as they are experienced and recorded. 

## Debugging

If you want to do more in-depth debugging in your code, you can setup breakpoints in your code by clicking on the left side pane and then clicking on `Run`->`Debug`.    You can also click on the bug icon ![https://i.imgur.com/zGh8wZ0.png](https://i.imgur.com/zGh8wZ0.png) if you've enabled the Toolbar (`View`->`Enable Toolbar`): 

Android Studio v1.2 and higher also provides a [built-in decompiler](http://www.androidpolice.com/2015/04/03/android-studio-1-2-reaches-beta-adds-built-in-decompiler-inline-debugger-variables-smarter-formatting-and-so-much-more/).  You can also use `Navigate`->`Declaration` within the IDE and set breakpoints even within classes for which you may not necessarily have the actual source code.  Notice the warning message at the top and an example of a screenshot of setting a breakpoint of a class defined in a JAR file below:

<img src="https://imgur.com/shKOtyh.png"/>

If the debugger isn't working, check the [[guide section below|Troubleshooting-Common-Issues#debugger-isnt-working-disconnected-or-client-not-ready-yet]] to get things running again.

### Network Traffic Inspection

If you wish to inspect network traffic, see [[this guide|Troubleshooting-API-Calls]] on how to troubleshoot API calls.  The networking library you use determines what approach you can use.

### Database Inspection

Also, the [Stetho](http://facebook.github.io/stetho/) project can be used to view your local SQLLite database.  See [[this guide|Debugging-with-Stetho]] for more details.

<img src="http://facebook.github.io/stetho/static/images/inspector-sqlite.png"/>

## LogCat

Android Studio contains a panel to receive logging messages from the emulator, known as [LogCat](http://developer.android.com/tools/help/logcat.html). If you are not seeing any log messages, click on the Restart icon ![https://imgur.com/kQKr1zv.png](https://imgur.com/kQKr1zv.png).  

![https://i.imgur.com/xP3dQcG.png](https://i.imgur.com/xP3dQcG.png)

## Resetting adb

If you are having issues trying to connect to the emulator or see any type of "Connection refused" errors, you may need to reset the Android Debug Bridge.  You can go to `Tools`->`Android`->`Android Device Monitor`.  Click on the mobile device icon and click on the arrow facing down to find the `Reset adb` option.

![https://i.imgur.com/srLBOMJ.gif](https://i.imgur.com/srLBOMJ.gif)

## Virtual Device Manager

### Unable to delete emulator getting "AVD is currently running in the Emulator"

Open the `Tools => Android => AVD Manager` and select virtual device that you want to delete. Click on the down arrow at the end and select the `[Show on Disk]` option which will open the emulator directory. Inside the `[Your Device].avd` folder, locate any `*.lock` files and delete them. You can now delete the emulator. See [this stackoverflow post](http://stackoverflow.com/questions/27005819/cant-delete-avd-from-avd-manager-in-android-studio) for more details. 

## Android Studio Issues

### Android Studio is Crashing or Freezing Up

If Android Studio starts freezing up or crashing even after rebooting the IDE or your computer, your Studio has likely become corrupted. The best way to resolve this is to clear all the caches by removing all the following folders:

```
~/Library/Application Support/AndroidStudio
~/Library/Caches/AndroidStudio
~/Library/Logs/AndroidStudio
~/Library/Preferences/AndroidStudio
```

and then uninstall Android Studio and re-install the latest stable version. This should allow you to boot Android Studio again without errors.

### Android Studio Design Pane isn't loading properly 

If you find yourself opening up a layout file and not seeing the design pane rendering correctly such as:

<img src="https://i.imgur.com/DaArMlC.png" width="300" />

We can try the following steps to get this functioning again:

 * Try changing the API version selected in the dropdown and try a few different versions
 * Click the "refresh" icon at the top right of the design pane
 * Select `File -> Invalidate Caches / Restart` and restart Android Studio

You may need to install the newest version of Android and select that version within the dropdown for the pane to work as expected. 

### Seeing `Unable to execute dex: method ID` when compiling

This might also show up as `Too many field references: 131000; max is 65536.` or `com.android.dex.DexIndexOverflowException: method ID not in [0, 0xffff]: 65536` or `Error:Execution failed for task ':app:dexDebug'` in the build console. This error occurs when the total number of references within a single bytecode file exceeds the 65,536 method limit. This usually means you have a substantial amount of code or are loading a large number of libraries.

If you've crossed this limit, this means you've loaded too many classes usually due to third-party libraries. Often the culprit is the Google Play Services library. Open up your app gradle file and look for this line `compile 'com.google.android.gms:play-services:X.X.X'`. Remove that line and instead [include the services selectively](https://developers.google.com/android/guides/setup#split) as outlined there. For example:

```gradle
dependencies {
   compile 'com.google.android.gms:play-services-maps:8.3.0'
   compile 'com.google.android.gms:play-services-location:8.3.0'
}
``` 

This can greatly reduce the number of classes loaded. If you've crossed the limit even with this change, we need to adjust our project to [use a multidex configuration](https://developer.android.com/tools/building/multidex.html#mdex-gradle) by enabling `multiDexEnabled true` in your gradle configuration and updating the application to extend from `android.support.multidex.MultiDexApplication`.

### Seeing `java.lang.OutOfMemoryError : GC overhead limit` when compiling

You are most likely exhausting the heap size especially during compilation.  Try to add inside this setting in your `app/build.gradle`:

```gradle
android {
   .
   .
   .
   dexOptions {
     javaMaxHeapSize "4g"
   }
}

```

You can also reduce the build time too by setting `incremental` to be true:

```gradle
android {
   dexOptions { 
      incremental true
      javaMaxHeapSize "4g"
   }
}
```

See this [Google discussion](https://groups.google.com/forum/#!topic/adt-dev/r4p-sBLl7DQ) article for more context.

Still not working?  Try to increase the heap size of Android Studio.  

1. Quit Android Studio.
2. Create or edit a [`studio.vmoptions` file](http://tools.android.com/tech-docs/configuration). 
    * On Mac, this file should be in `~/Library/Preferences/AndroidStudio/studio.vmoptions`.
    * On Windows, it should be in `%USERPROFILE%\.AndroidStudio\studio[64].exe.vmoptions`. 

    Increase the maximum memory to 2 Gb and max heap size of 1 Gb.
    ```
     -Xmx2048m
     -XX:MaxPermSize=1024m`
   ```
3. Start Android Studio.

### Getting "No resource found that matches given name."

If you are using [multiple layout folders](http://stackoverflow.com/questions/4930398/can-the-android-layout-folder-contain-subfolders) and decide to rename any of your ID tags in your XML files, you may get "No resource found that matches given name."    There is a current bug in how changes are detected in [nested subfolders](http://stackoverflow.com/a/31187196/961939) so your best option is to not use this approach until the problem is fixed.  Otherwise, you will need to do a `Rebuild Project` so that the entire resource files can be regenerated and the build/ directories are fully removed.  Note: `Clean Project` may not work. 

### Getting "tooling.GradleConnectionException" errors

If you see `org.gradle.tooling.GradleConnectionException` errors, you may need to install a newer version of JDK (there have been reports of 1.7.0_71 having this issue).  First try to restart the adb server first.

![https://i.imgur.com/1kWwmuh.png](https://i.imgur.com/1kWwmuh.png)

### Getting "failed to find Build Tools revision x.x.x"

If you're opening another Android Studio project and the project fails to compile, you may see "failed to find Build Tools revision x.x.x" at the bottom of the screen.  Since this package is constantly being changed, it should be no surprise that other people who have installed Android Studio may have different versions. You can either click the link below to install this specific Build Tools version, or you can modify the build.gradle file to match the version you currently have installed.

![https://i.imgur.com/IsAWMrl.png](https://i.imgur.com/IsAWMrl.png)

### Getting "com.android.dex.DexException: Multiple dex files define" 

One of the issues in the new Gradle build system is that you can often get "Multiple dex files define" issues.  

If a library is included twice as a dependency you will encounter this issue. Review the `libs` folder for JARS and the gradle file at `app/build.gradle` and see if you can identify the library dependency that has been loaded into your application twice.

If one dependency library already includes an identical set of libraries, then you may have to make changes to your Gradle configurations to avoid this conflict.   This problem usually happens when there are multiple third-party libraries integrated into your code base. See [[Must-Have-Libraries#butterknife-and-parceler]] for more information.

Another error if you attempt to include a library that is a subset of another library.  For instance, suppose we included the Google play-services library but thought we also needed to include it with the play-services-map library.:

```gradle
dependencies {
    compile 'com.google.android.gms:play-services:6.5.+'
    compile 'com.google.android.gms:play-services-maps:6.5.+'
}
```

It turns out that having both is redundant and will cause errors.  It is necessary in this case to remove one or the other, depending on your need to use other Google API libraries. See this overview of the [multidex issue on the Android docs](https://developer.android.com/tools/building/multidex.html).

### Seeing `Unsupported major.minor version 52.0` on some plugins

Some Android Studio plugins do not support Java 1.6 anymore, so it's best to confirm what version you are using.  Inside Android Studio, click on `About Android Studio`.  You should see the JRE version listed as 1.x.x below:

<img src="https://imgur.com/Jn2jevR.png"/>

If you have multiple Java versions installed, you should make sure that v1.6 is not being used. Solutions include:

 * Configuring [JDK 8](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html) as your default Java SDK within Studio. See [this stackoverflow post for solutions](http://stackoverflow.com/a/36262436).
 * Reset your Android Studio cache and set correct gradle version as [shown in this post](http://stackoverflow.com/a/37467179).

On OS X machines, you can remove the JDK from being noticed.  You can move it the temporary directory in case other issues are created by this change.

```bash
sudo mv /System/Library/Java/JavaVirtualMachines/1.6.0.jdk /tmp
```

### INSTALL_FAILED_OLDER_SDK error message 

If your `minSdkVersion` is higher than the Android version you are using (i.e. using an emulator that supports API 19 and your target version is for API 23), then you may see an error message that appears similar to the following:

<img src="https://imgur.com/RKlXMGV.png"/>

You will need to either lower the `minSdkVersion` or upgrade to an Android emulator or device that supports the minimum SDK version required.

### Seeing `java.lang.IllegalAccessError: Class ref in pre-verified class resolved to unexpected implementation`

You have a third-party library reference defined twice.  Check your `app/build.gradle` for duplicate libraries (i.e. commons-io library defined for 1.3 and another one using 2.4).

### Debugger Isn't Working: Breakpoint In Thread is Not Hit

You might notice that the debugger does not always work as expected in background threads such as `AsyncTask` or when using networking libraries to send network requests. The debugger breakpoints are a little bit finicky when configured to stop outside the main thread.

In order for some of these breakpoints in the callbacks to work, you have to set them **after the debugger has been initialized** and not before. If you have already set the breakpoints before, then you have to reset them after the debugger starts. One common strategy is to set the breakpoint on the main thread and then, once that has been hit, add the breakpoint on the background thread. 

### Debugger Isn't Working: `Disconnected` or `Client not ready yet`

This is usually a signal that the emulator instance is not properly connected with ADB and Android Studio. There are a few steps to try:

1. First, try uninstalling the app from within the emulator and then restarting the emulator completely. Try debugging again now. 

2. Next, close the emulator again and also restart Android Studio. Start Android Studio and reload the emulator. Then try debugging again. 

3. If you are using official emulator, try using [[Genymotion|Genymotion-2.0-Emulators-with-Google-Play-support]] or vice-versa. Then try debugging again.

4. Open up `Tools => Android => Android SDK Manager` and see if there are any updates to the platform or SDK tools. Update any suggested changes within the manager and then restart your emulator and Android Studio. Then try debugging again.

## Eclipse ADT Issues

For common issues experienced with Eclipse, check the [[Troubleshooting Eclipse Issues]] page instead for a detailed list of common problems.

## References

* <https://jaanus.com/debugging-http-on-an-android-phone-or-tablet-with-charles-proxy-for-fun-and-profit/>
