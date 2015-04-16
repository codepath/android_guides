This page will compile common issues experienced with Android Studio 1.0 or above as they are experienced and recorded. 

## Debugging

If you want to do more in-depth debugging in your code, you can setup breakpoints in your code by clicking on the left side pane and then clicking on `Run`->`Debug`.    You can also click on the bug icon if you've enabled the Toolbar (`View`->`Enable Toolbar`):

![http://i.imgur.com/zGh8wZ0.png](http://i.imgur.com/zGh8wZ0.png)

## LogCat

Android Studio contains a panel to receive logging messages from the emulator, known as [LogCat](http://developer.android.com/tools/help/logcat.html). If you are not seeing any log messages, click on the Restart icon ![http://imgur.com/kQKr1zv.png](http://imgur.com/kQKr1zv.png).  

![http://i.imgur.com/xP3dQcG.png](http://i.imgur.com/xP3dQcG.png)

## Resetting adb

If you are having issues trying to connect to the emulator or see any type of "Connection refused" errors, you may need to reset the Android Debug Bridge.  You can go to `Tools`->`Android`->`Android Device Monitor`.  Click on the mobile device icon and click on the arrow facing down to find the `Reset adb` option.

![http://i.imgur.com/srLBOMJ.gif](http://i.imgur.com/srLBOMJ.gif)

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

### Seeing `Unable to execute dex: method ID` when compiling

This might also show up as `Too many field references: 131000; max is 65536.` in newer build systems. This error occurs when the total number of references within a single bytecode file exceeds the 65,536 method limit. This usually means you have a substantial amount of code or are loading a large number of libraries.

After crossing that limit, we need to adjust our project to use a multidex configuration. To enable this, please review the [official multidex guide](https://developer.android.com/tools/building/multidex.html#mdex-gradle) to adjust your gradle files.

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

If you decide to rename any of your ID tags in your XML files, you may get "No resource found that matches given name." You will need to do a `Rebuild Project` so that the entire resource files can be regenerated and the build/ directories are fully removed.  Note: `Clean Project` may not work.

### Getting "tooling.GradleConnectionException" errors

If you see `org.gradle.tooling.GradleConnectionException` errors, you may need to install a newer version of JDK (there have been reports of 1.7.0_71 having this issue).  First try to restart the adb server first.

![http://i.imgur.com/1kWwmuh.png](http://i.imgur.com/1kWwmuh.png)

### Getting "failed to find Build Tools revision x.x.x"

If you're opening another Android Studio project and the project fails to compile, you may see "failed to find Build Tools revision x.x.x" at the bottom of the screen.  Since this package is constantly being changed, it should be no surprise that other people who have installed Android Studio may have different versions. You can either click the link below to install this specific Build Tools version, or you can modify the build.gradle file to match the version you currently have installed.

![http://i.imgur.com/IsAWMrl.png](http://i.imgur.com/IsAWMrl.png)

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

## Eclipse ADT Issues

For common issues experienced with Eclipse, check the [[Troubleshooting Eclipse Issues]] page instead for a detailed list of common problems.