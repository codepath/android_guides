## Overview

Many Android applications require the use of an embedded interactive map. On Android, this embedded map is part of the [Google Play Services SDK](http://developer.android.com/google/play-services/setup.html) which is a Google add-on pack for Android enabling all sorts of extra features around gaming, messaging, billing, location, etc.

In this guide, we will walk you through the step by step process of getting an embedded Google Map working within an Android emulator.

### Import Maps Demo

1. Download the [Maps Demo Repo](https://github.com/codepath/android-google-maps-demo/archive/master.zip) ([GitHub link](https://github.com/codepath/android-google-maps-demo/))
2. Run `File` ⇒ `Open`, select `android-google-maps-demo-master`
3. Verify that you have the Google Maven repository defined in your root `build.gradle` file:
      ```gradle
      allprojects {
        repositories {
          jcenter()
          maven { url "https://maven.google.com" }
        }
      }
      ```
4. Verify sure you have these dependencies listed in your `app/build.gradle` file:
      ```gradle
      dependencies {
          compile 'com.google.android.gms:play-services-maps:11.4.2'
          compile 'com.google.android.gms:play-services-location:11.4.2'
      }
      ```
5. Make sure to select `Build => Clean project` and then `Build => Re-build project` to make any issues with `MapDemoActivityPermissionsDispatcher` in `MapDemoActivity.java` clear up the errors.

If Gradle does not find the Play Services Gradle packages, make sure you followed step 2 and verified that the Google Maven repository has been added.

Next, we need to get ourselves a maps API key from Google to put into our `AndroidManifest.xml`.

### Get API Key

Navigate to [Google Maps Android API](https://developers.google.com/maps/documentation/android-api/) and select `Get A Key`

<img src="https://i.imgur.com/6PhVvQU.png" width="500" />

Select `Continue` and then `Create a new project`

<img src="https://i.imgur.com/kRJDzk3.png" width="650" />

Now you want to skip optional section on restricting usage and click "Create". At this stage you should have your working `API Key`:

<img src="https://i.imgur.com/mGbpWdN.png" width="650" />

Copy your API Key into the `res/values/strings.xml` file for the `google_maps_api_key`:

```java
<resources>
    <string name="google_maps_api_key">YOUR-API-KEY here</string>
</resources>
```

The meta data for `com.google.android.maps.v2.API_KEY` within the **application node** in the `AndroidManifest.xml` for your app should reference this string resource:

```xml
<application
  ...>
  <activity 
     ...>
  </activity>

  <meta-data android:name="com.google.android.gms.version" android:value="@integer/google_play_services_version" />

  <meta-data android:name="com.google.android.maps.v2.API_KEY"
android:value="@string/google_maps_api_key" />

</application>
```

## Installing on a Device

If you want to run the app **using a physical device**, then check out [[this guide on running apps using your device|Running-Apps-on-Your-Device]] and then skip the rest of this section and go directly to "Run the App" section below! You only need this section **if you want to run maps on an emulator**. 

If you are using Genymotion then see the section below to make sure **you've setup Google Play services**. If you want to use the official emulator, see the "Setup x86-based Emulator" section below. 

### Installing on Genymotion

See the instructions to [[setup Genymotion|Genymotion 2.0 Emulators-with-Google-Play-support]].  It includes directions for setting Google Play Services and GPS too.  You will need to setup both in order to setup the map demo.

### Setup x86-based Emulator

**Note:** Skip this step unless you can't run maps on Genymotion or a physical device. 

For the respective Android API version you are targeting, go to `Tools` -> `Android SDK` and make sure to download the respective `Google Play Intel x86 Atom System Message` by clicking on the `Show Package Details` checkbox:

<img src="https://imgur.com/0iQA2ji.png">

If you forget to follow this step, you may see error messages such as `Google Play services out of date.  Requires 11400000 but found 10932470` or similar error messages.

Open `Android Studio` ⇒ `Tools` ⇒ `Android` ⇒ `Android AVD Manager` and `Create a Virtual Device...`

Choose only an AVD emulator that has `Play Store` support listed.  You can see the icon that only the Nexus 5x and Nexus 5 emulators provide support:

<img src="https://imgur.com/Ab0tldo.png">

Select `Nexus 5X` ⇒ `Show downloadable system images` and select `Oreo, API Level 26, x86_64, Android 6.0` **(with Google APIs)**

<img src="https://i.imgur.com/gfSSoDB.png" width="650" alt="Emulator" />

You may be prompted to update your Google Play Services.  If you are running an AVD emulator that supports Google Play, you will be asked to sign-in with a Google account.  If you do not see any window launching or see an error such as `android.content.ActivityNotFoundException: No Activity found to handle Intent { act=android.intent.action.VIEW dat=market://details?id=com.google.android.gms`, then chances are you not using an emulator with Google Play support.

<img src="https://imgur.com/zoY1t32.png"/>

If sign-in is successful, you should see Google Play services updating:

<img src="https://imgur.com/p4wCfuA.png"/>

> ***Note:** Make sure to install `HAXM` if Android Studio prompts you to do so.

This emulator is now configured to run Google Play Services. When you start this emulator, you may notice some loss of crispness, especially on text. This happens because the emulator launches to fill the window and may result in a non-power of 2 scaling. To fix this, you can edit the AVD and select a scale of `2dp = 1px on screen` and go into advanced settings and turn off `Enable Device Frame`.

<img src="https://i.imgur.com/XOpvoci.png" width="650" alt="FixDisplay" />

## Enabling Location on Emulator

If you are using a physical device, you can **skip this section**. You only need this section **if you want to run maps and location on an emulator**. 

### Genymotion GPS

Note that you **need to set the location on your emulator** either through Genymotion or on the official emulator. On Genymotion, we can click the GPS icon on the right-hand sidebar and then enable GPS:

<img src="https://i.imgur.com/xha71ga.png" width="350" />

You can enter any lat / lng i.e `37.4810289, -122.1543292`. With GPS enabled, you can now close the dialog and use the "current location" marker on the app to move the map to your device's configured location.

### Official Emulator

If using **the official Google emulator**, we can update the "current location" for the device by clicking the "..." icon (<img src="https://i.imgur.com/ncXZJxz.png" width="30"/>) and then selecting the "Location" tab on the left sidebar:

<img src="https://i.imgur.com/hfZZBQF.png" width="650" alt="Android device monitor" />

You can enter any lat / lng i.e `37.4810289, -122.1543292`.

**Make sure to click the Send** button to report the GPS location.  After updating this location, you can use the "current location" marker on the app to move the map to your device's configured location.

## Run the App

Before running our app, let's **open up the "official Maps" app** on the device or emulator to make sure the current location has been configured properly:

<img src="https://i.imgur.com/2IWY4wF.png" width="350" />
&nbsp;
<img src="https://i.imgur.com/170keQM.png" width="350" />

Now we can finally run our own maps app demo, and if everything goes well we should see:

<img src="https://i.imgur.com/PQJhTf7.png" width="400" alt="Demo app" />

**Note:** If you don't see the "Maps" app or the map, you may have not properly installed the Google Play services on the emulator (see instructions in genymotion setup above) or you may need to update the Google Play services on your emulator by following the instructions given in the app.

### Conclusion

At this point, you should have the Google map displaying in your sample application. If you don't, try restarting the emulator and uninstalling / reinstalling the map demo application. Eventually you will see the maps if you registered your key properly.

For more information including how to use the maps, check out the source of the [[Maps Usage Cliffnotes|Google-Maps-API-v2-Usage]] as well as this article on [androidhive](http://www.androidhive.info/2013/08/android-working-with-google-maps-v2/).

## Troubleshooting

Use this checklist for troubleshooting below:

**Trouble launching Genymotion?**

 * Did you install [VirtualBox](https://www.virtualbox.org/wiki/Downloads) first. Run VirtualBox to ensure that it was installed correctly.
 * Did you properly move Genymotion into the Applications folder?
 * Launch VirtualBox, then verify the emulator is listed and ensure it is in a **powered off** state
 * Open Genymotion app and verify the emulator is listed there and no errors are shown
 * Restart Android Studio and try to launch the emulator again through Android Studio plugin
 * Restart your computer and try to launch the emulator again through Android Studio plugin

**Seeing `INSTALL_FAILED_MISSING_SHARED_LIBRARY` when trying to run my app?**

Did you download the latest Google Play Services APK?  See the [[google play genymotion section|Genymotion-2.0-Emulators-with-Google-Play-support#setup-google-play-services]]. Make sure to follow the instructions to reboot, sign-in with your Google Account, and upgrade to the latest Play Services.

**Trouble seeing the map in the demo app?**

 You are likely to see an error message such as the following in your LogCat:

 ![](https://i.imgur.com/V6EzQGV.png)

 If so, make sure to check the following:

   * Does the Android Key on the Google API Console **match the package namespace of the maps demo app** i.e `<YOURSHA1KEY>;com.example.mapdemo`?  The error messages should include the Key and package namespace you should be using.

   * Did you enter the correct API Key into the `AndroidManifest.xml` as explained in the map demo setup?
   * Did you enable the "Google Maps Android API v2" toggle switch on the Google API Console under "APIs & Auth" tab => APIs?

Other things to check:

   * Did you enable GPS location for the emulator and set a location by going to the map?
   * Did you try running "Play Store" on your emulator and sign in with a Google account?
   * Uninstall the "Map Demo" from the emulator first, then re-install
   * Restart Android Studio and try to launch the app again 
   * Restart your computer and try to launch the app again

Hopefully with these troubleshooting steps you have gotten things working!

### Map Access Across Computers

Often when collaborating on a project with others, you need to have the maps work across multiple computers. The problem is that the map key fingerprint is different from computer to computer and thus by default the maps will only work on the computer that was used to generate the key.

The simplest fix is described in detail within [this stack overflow post](http://stackoverflow.com/a/9653946/313399) but in short you can get the `debug.keystore` from one of the team members, check that into git and then instruct other team members to replace their `debug.keystore` file with the one from repository. See also [this link](http://groups.google.com/group/android-developers/browse_thread/thread/c9051635ab37f252) and [this guide](http://developer.android.com/guide/publishing/app-signing.html#debugmode). 

**Note:** After you copy the debug.keystore be sure to do the following before trying to run the app

1.  Clean your project
2.  Uninstall your app

### Simulating GPS Locations

This handy [playback-gps](https://github.com/galens/playback-gpx) script allows a developer to simulate traveling a specific route on genymotion. Use this for testing and for presentations if you need to demonstrate how the map works with location in real-time. Additional resources related to mocking GPS locations on Android device or emulator:

 * [Simulate GPS Locations with Plugin](http://www.jesusamieiro.com/android-studio-simulate-multiple-gps-points-with-mock-location-plugin/) - Using Android Studio Mock Location plugin to simulate locations. 
 * [Lockito](https://play.google.com/store/apps/details?id=fr.dvilleneuve.lockito) - Allows you to make your phone follow a fake itinerary, with total control over the speed and GPS signal accuracy. This can be recorded using [GPS Logger](https://play.google.com/store/apps/details?id=com.mendhak.gpslogger&hl=en_GB) You can also simulate a static location.
 * [Fake GPS location app](https://play.google.com/store/apps/details?id=com.lexa.fakegps&hl=en) - Teleport your phone to any place in the world with two clicks! This app sets up fake GPS location so every other app in your phone belives you are there!
 * [FakeLocationProvider](https://gist.github.com/iutinvg/4671582) - Sample code for faking locations provider for Android. Useful for demoing tracking related applications.
 * [android-gps-emulator](https://github.com/dpdearing/android-gps-emulator) - GPS location emulator for changing/setting/simulating the GPS location of the Android emulator through a simple map-based interface.


## References

* <http://www.androidhive.info/2013/08/android-working-with-google-maps-v2/>
* <http://www.vogella.com/articles/AndroidGoogleMaps/article.html>
* <http://www.genymotion.com/>
