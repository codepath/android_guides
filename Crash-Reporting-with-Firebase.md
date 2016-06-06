# Crash reporting with Firebase

Crash reporting is one of the features of the new Firebase that were announced at Google I/O, 2016. In this guide, you'll learn how to implement crash reporting with the new firebase.

## Setup
  1. To use firebase crash reporting, you have to do the basic setup for Firebase first. Check this [[guide|Building-Data-driven-Apps-with-Firebase]] for basic setup for Firebase.

  2. Next, is to add the Firebase crash Android library to your app module's `build.gradle` file:

```gradle
compile 'com.google.firebase:firebase-crash:9.0.0'
```

At the end of this step, your `build.gradle` file should look like this:

```gradle
apply plugin: 'com.android.application'

android {
  // ...
}

dependencies {
  // ...
  // other libraries in your project
  
  // firebase core library
  compile 'com.google.firebase:firebase-core:9.0.0'
  
  // firebase crash library
  compile 'com.google.firebase:firebase-crash:9.0.0'
}

// Add to the bottom of the file
apply plugin: 'com.google.gms.google-services'
```

## Sending a crash report.
This step assumes you have completed the basic setup for Firebase.

To send a crash report, you need to add a line like:

```java
FirebaseCrash.report(new Exception("My first Firebase non-fatal error on Android"));
```
After a few minutes (about 20 minutes according to the Firebase [docs](https://firebase.google.com/docs/crash/android#set_up_crash_reporting)), the crash will appear on the console dashboard like:

![](https://i.imgur.com/VNM6a4p.png)

From the dashboard shown above, you'll notice that there is something called **"Clusters"**, and basically, that is Firebase helping to group exceptions with similar stack traces.

Unlike many other crash reporting libraries that only require one line to initialize crash reporting all through the app, Firebase doesn't provide such utility methods. 

You may consider adding this block of code in your main application class to handle your uncaught exceptions:

```java
Thread.setDefaultUncaughtExceptionHandler(new Thread.UncaughtExceptionHandler(){
    @Override
    public void uncaughtException(Thread thread, Throwable ex) {
        FirebaseCrash.report(ex);
    }
});
```

## Send custom logs.

You can also send custom logs with Firebase. To send a custom log, you can use the following line:

```java
FirebaseCrash.log("MainActivity started");
```

There's also an option of creating a logcat output. You can do that by using the *`FirebaseCrash.logcat()`* function like:

```java
FirebaseCrash.logcat(Log.DEBUG, "My tag", "My message");
```

## Deobfuscate Proguard labels.

Firebase also allows you to deobfuscate Proguard label. If you have Proguard enabled in your project, a mapping.txt file is generated for the obfuscated labels. 

Check your `<project-root>/<app-module>/build/outputs/mapping/<build-type>/<appname>-proguard-mapping.txt` for the file and upload to Firebase.

An example of the mapping.txt location is:

`app/build/outputs/mapping/debug/app-proguard-mapping.txt`

For more information on Proguard, check the guide on [Configuring Proguard](http://guides.codepath.com/android/Configuring-ProGuard).

To upload the proguard mapping, click on the **"Mapping files"** tab from the dashboard. You'll see a view like this:

![](https://i.imgur.com/yVJp72F.png)


## Issues

Firebase crash reporting is in beta, and work is still going on with it.

Firebase crash reporting seems to be a separate process, as seen below:
![](https://i.imgur.com/FUvytFS.png)

This may give rise to concurrency issues and it's logged as an issue on the [Firebase docs](https://firebase.google.com/docs/crash/android#known-issues).
	
## Attribution

This guide was originally drafted by [Segun Famisa](segunfamisa).

## References and further reading:

  * [https://firebase.google.com/docs/crash/android](https://firebase.google.com/docs/crash/android)
  * [https://firebase.google.com/docs/reference/android/com/google/firebase/crash/package-summary](https://firebase.google.com/docs/reference/android/com/google/firebase/crash/package-summary)
  * [[Configuring-ProGuard]]
