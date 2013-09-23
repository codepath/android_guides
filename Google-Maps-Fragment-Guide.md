## Overview

Many Android applications require the use of an embedded interactive map. On Android, this embedded map is part of the [Google Play Services SDK](http://developer.android.com/google/play-services/setup.html) which is a Google add-on pack for Android enabling all sorts of extra features around gaming, messaging, billing, location, etc.

In this guide, we will walk you through the step by step process of getting an embedded Google Map working within an Android emulator.

### Download Google Play Services

First, let's download and setup the Google Play Services SDK. Open **Eclipse ⇒ Windows ⇒ Android SDK** Manager and check whether you have already downloaded Google Play Services or not under Extras section. If not select play services and install the package.

![Play Services](http://www.androidhive.info/wp-content/uploads/2013/08/downloading-google-play-services-sdk.png)

### Install Google Play Services

After downloading play services we need to import it to Eclipse which will be used as a library for our maps project.

1. In Eclipse goto **File ⇒ Import ⇒ Android ⇒ Existing Android Code Into Workspace**

2. Click on Browse and select Google Play Services project from your android sdk folder. You can locate play services library project from
<your-android-sdk-path>\extras\google\google_play_services\libproject\google-play-services_lib

3. Importantly while importing check Copy projects into workspace option as shown in the below image.

![Google Play Load](http://www.androidhive.info/wp-content/uploads/2013/08/importing-google-play-services.png)

### Register for API Key



### Import Maps Demo

Now that we have our Genymotion emulator properly setup, let's import the [demo application](https://github.com/thecodepath/android-google-maps-demo) we can use to verify if maps are showing up correctly. 

1. Download the [Maps Demo](https://github.com/thecodepath/android-google-maps-demo/archive/master.zip) application and extract the files.
2. Run "File...Import...Existing Android Code Into Workspace" and hit "Finish"
3. Expand MapDemo application and open up the "AndroidManifest.xml"

Enter your API Key into the meta data for `com.google.android.maps.v2.API_KEY`:

```xml
<meta-data
   android:name="com.google.android.maps.v2.API_KEY"
   android:value="YOUR-KEY-HERE" />
```

### Genymotion

The first step to getting Google Maps working on your emulator is to download a third-party emulator called [Genymotion](http://www.genymotion.com/). The reason for this is that the official emulator does a terrible job of supporting the Google Play Services. While it is possible to get the Intel HAXM fast emulator working with the Play Services SDK, at the moment it's far more trouble then it's worth. 

[Genymotion](http://www.genymotion.com/) is an incredibly fast, memory-efficient VM that runs the Android OS in a more accurate manner than even the official emulator. Many Android developers do all their device testing using this emulator especially when Google Play services is concerned.

To setup your genymotion emulator [sign up](https://cloud.genymotion.com/page/customer/login/?next=/) and follow the [installation guide](https://cloud.genymotion.com/page/doc/):

1. Sign up for an account on the [genymotion website](https://cloud.genymotion.com/page/customer/login/?next=/)
2. (Mac or Linux) [Install VirtualBox](https://www.virtualbox.org/wiki/Downloads), a powerful free virtualization software.
3. [Download Genymotion Emulator](https://cloud.genymotion.com/page/launchpad/download/) for your platform.
4. Install the Genymotion Emulator
   * Windows: Run the msi installer
   * Mac: Open dmg and drag apps to Applications directory
5. Run Genymotion application
6. Sign in and add your first virtual device (Nexus S - 4.2.2 with Google Apps - API 17)
7. Install the Eclipse Plugin
   * Go to the "Help/Install New Software..." menu
   * Add a new software site: Genymobile - http://plugins.genymotion.com/eclipse
   * Check all genymobile entries
   * Accept licenses and install
   * Restart eclipse
8. Click the genymobile icon ![Genymobile](https://cloud.genymotion.com/static/images/doc/genymotion-plugin-eclipse-button.png) and click "Start" on your virtual device.
9. Wait for device to boot up and then run through initial setup

## References

* <http://www.androidhive.info/2013/08/android-working-with-google-maps-v2/>
* <http://www.vogella.com/articles/AndroidGoogleMaps/article.html>
* <http://www.genymotion.com/>