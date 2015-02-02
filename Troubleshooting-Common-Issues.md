This page will compile common issues experienced with Android Studio 1.0 or above as they are experienced and recorded. 

## Debugging

If you want to do more in-depth debugging in your code, you can setup breakpoints in your code by clicking on the left side pane and then clicking on Run->Debug.    You can also click on the bug icon if you've enabled the Toolbar (View->Enable Toolbar):

![http://i.imgur.com/zGh8wZ0.png](http://i.imgur.com/zGh8wZ0.png)

## LogCat

Android Studio contains a panel to receive logging messages from the emulator, known as [LogCat](http://developer.android.com/tools/help/logcat.html). If you are not seeing any log messages, click on the Restart icon ![http://imgur.com/kQKr1zv.png](http://imgur.com/kQKr1zv.png).  

![http://i.imgur.com/xP3dQcG.png](http://i.imgur.com/xP3dQcG.png)

## Resetting adb

If you are having issues trying to connect to the emulator or see any type of "Connection refused" errors, you may need to reset the Android Debug Bridge.  You can go to `Tools`->`Android`->`Android Device Monitor`.  Click on the mobile device icon and click on the arrow facing down to find the `Reset adb` option.

![http://i.imgur.com/srLBOMJ.gif](http://i.imgur.com/srLBOMJ.gif)

## Android Studio Issues

* If you decide to rename any of your ID tags in your XML files, you may get "No resource found that matches given name."   You will need to do a Rebuild Project so that the entire resource files can be regenerated and the build/ directories are removed.  Note: Clean Project may not work.

* If you see org.gradle.tooling.GradleConnectionException errors, you may need to install a newer version of JDK (there have been reports of 1.7.0_71 having this issue).  First try to restart the adb server first.

![http://i.imgur.com/1kWwmuh.png](http://i.imgur.com/1kWwmuh.png)

* One of the issues in the new Gradle build system is that you can often get "Multiple dex files define" issues.  If one dependency library already includes an identical set of libraries, then you may have to make changes to your Gradle configurations to avoid this conflict.  For instance, including the Butterknife library with the Parceler library causes multiple declarations of javax.annotation.processing.Processor.  In this case, you have to exclude this conflict:

```gradle
   packagingOptions {

        exclude 'META-INF/services/javax.annotation.processing.Processor'  // butterknife
    }
```

Another error if you attempt to include a library that is a subset of another library.  For instance, suppose we included the Google play-services library but thought we also needed to include it with the play-services-map library.:

```gradle
dependencies {
    compile 'com.google.android.gms:play-services:6.5.+'
    compile 'com.google.android.gms:play-services-maps:6.5.+'
}
```

It turns out that having both is redundant and will cause errors.  It is necessary in this case to remove one or the other, depending on your need to use other Google API libraries.

## Eclipse ADT Issues

For common issues experienced with Eclipse, check the [[Troubleshooting Eclipse Issues]] page instead for a detailed list of common problems.