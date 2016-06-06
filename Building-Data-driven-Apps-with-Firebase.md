# Setting up the new Firebase

At this year's [Google I/O](https://events.google.com/io2016/), Google announced a number of interesting new features in Firebase. The new Firebase is said to be a “Unified app platform for mobile developers” adding new tools to help develop faster, improve app quality, engage users and monetize apps.

For a quick intro to what the new Firebase is about, you can watch [this video](https://www.youtube.com/watch?v=fgT6r4f9Apc&feature=youtu.be).

<p align="center">
	<a href="http://www.youtube.com/watch?feature=player_embedded&v=fgT6r4f9Apc
	" target="_blank"><img src="http://img.youtube.com/vi/fgT6r4f9Apc/0.jpg" 
	alt="Introducing the new Firebase" width="560" height="315" border="10" /></a>
</p>

The new features that are included in this new Firebase include:

1. [[Crash Reporting|Crash-Reporting-with-Firebase]]
2. [[Cloud Messaging|Google-Cloud-Messaging]]
3. Realtime database and storage
4. Analytics.
5. Notifications.
6. Dynamic Links.
7. App Indexing.
8. AdWords.
9. Firebase Invites etc

## Registration
First point to get started with Firebase, is to create an account [here][firebase-console] if you don't have one already.

## Setup

### Prerequisites
Before you can begin to enjoy the new Firebase, please ensure your development machine satisfies these prerequisites:

  * A device running Android 2.3 (Gingerbread) or newer, and Google Play services 9.0.1 or newer
  * The Google Play services SDK from the Android SDK Manager
  * Android Studio 1.5 or higher
  * An Android Studio project and its package name

### Add Firebase to your app
Next step is to add Firebase to your app on the [Firebase console][firebase-console]. Follow the following steps to add firebase to your app.


#### 1. Create project
Navigate to the [console][firebase-console]. If you don't already have an existing Firebase project, click on "**Create new project**" and follow through the steps as shown in the gif below. Else, if you have an exisiting Google project, click on "**Import Google Project**"

![](https://i.imgur.com/knbrZ4X.gif)

#### 2. Add Firebase to your app
Now that you've created the project, in the dashboard, click on "**Add Firebase to your Android app**". 

To proceed, you need at least a package name unique to your app. If you plan on using Dynamic Links, Invites, and Google Sign-In support in Auth, you have to include a debug signing key SHA1 fingerprint.

To see your debug key SHA1 fingerprint, run the following command if you use Linux or Mac:

```shell
keytool -list -v -keystore ~/.android/debug.keystore -alias androiddebugkey -storepass android -keypass android 
```
If you're running Windows, use this command:

```
keytool -list -v -keystore C:\Documents and Settings\[User Name]\.android\debug.keystore -alias androiddebugkey -storepass android -keypass android 
```

The whole process is as shown below:

![](https://i.imgur.com/qf8d55l.gif)


#### 3. Configure your app
Clicking add app will download a `google-services.json` file. You should move this file to your projects app module directory. Usually `<project>/app` directory. Click continue and follow these instructions:  

  1. In your project-level build.gradle file, add the following lines (`<project>/build.gradle`):

```gradle
buildscript {
  dependencies {
    // Add this line
    classpath 'com.google.gms:google-services:3.0.0'
  }
}
```  

  2. In your app-level build.gradle (`<project>/<app-module>/build.gradle`):

```gradle
apply plugin: 'com.android.application'

android {
  // ...
}

dependencies {
  // ...
  compile 'com.google.firebase:firebase-core:9.0.0'
}

// Add to the bottom of the file
apply plugin: 'com.google.gms.google-services'
```


Sync your gradle files with the project and you're ready to go with firebase.


## Attribution

This guide was originally drafted by [Segun Famisa](segunfamisa).

## References and further reading
  1. [https://firebase.google.com/docs/android/setup](https://firebase.google.com/docs/android/setup)

[firebase-console]: https://firebase.google.com/console/