## Overview

Many Android applications require the use of an embedded interactive map. On Android, this embedded map is part of the [Google Play Services SDK](http://developer.android.com/google/play-services/setup.html) which is a Google add-on pack for Android enabling all sorts of extra features around gaming, messaging, billing, location, etc.

In this guide, we will walk you through the step by step process of getting an embedded Google Map working within an Android emulator.

### Download Google Play Services

First, let's download and setup the Google Play Services SDK. Open `Android Studio` ⇒ `Tools` ⇒ `Android` ⇒ `Android SDK Manager` and check whether you have already downloaded "Google Play services" or not under Extras section. If not, select "Google Play services" and install the package.

![Play Services](http://i.imgur.com/vvmOHCj.png)

### Retrieve your SHA-1 Fingerprint

Open your terminal and execute the following command to generate SHA-1 fingerprint necessary to get your API key from Google.

On Windows:

```
"C:\Program Files (x86)\Java\jdk1.7.0_60\bin\keytool.exe" -list -v -keystore "%USERPROFILE%\.android\debug.keystore" -alias androiddebugkey -storepass android -keypass android
```

or, depending on the installation location of your JDK:

```
"C:\Program Files\Java\jre7\bin\keytool.exe" -list -v -keystore "%USERPROFILE%\.android\debug.keystore" -alias androiddebugkey -storepass android -keypass android
```

On Mac or Linux:

```
keytool -list -v -keystore ~/.android/debug.keystore -alias androiddebugkey -storepass android -keypass android
```

In the output you can see SHA 1 finger print:

![Fingerprint](http://www.androidhive.info/wp-content/uploads/2013/08/android-google-maps-sha-1-fingerprint.png)

### Register for an API Key

Now open [Google API Console](https://cloud.google.com/console), create and select a project, select "APIs & Auth" on left side and turn on **Google Maps Android API v2** toggle switch.

![Services](http://imgur.com/ZzUOYtA.png)

Now select "Credentials" on left side and on the right side click on "Create New Key" =>  "Android Key"

![Access](http://www.androidhive.info/wp-content/uploads/2013/08/google-console-creating-a-key.png)

It will popup a window asking the SHA1 and package name. Enter your `SHA 1` and your `android project package name` separated by semicolon ; and click on create. The format is:

`<YOURSHA1KEY>;com.example.mapdemo`

For example you might enter:

```
BE:03:E1:44:39:7B:E8:17:02:9F:7F:B7:98:82:EA:DF:84:D0:FB:6A;com.example.mapdemo
```

**Note the API Key** for use in the next step while setting up the map demo app:

![Key](http://www.androidhive.info/wp-content/uploads/2013/08/google-console-android-maps-api-key1.png)

### Setup x86-based Emulator

**Note:** As of March 6th, 2014, we can download the [Google APIs x86](http://software.intel.com/en-us/blogs/2014/03/06/now-available-android-sdk-x86-system-image-with-google-apis) image to test maps. If you want to set the GPS coordinates of the emulator and you're using Android Studio, go to `Tools`->`Android`->`Android Device Monitor`->`Emulator Control`, then type one in Location Controls and hit "Send". Otherwise, for using the Genymotion emulator, see below. Note that **Genymotion is much harder to setup**, but is a much faster and more powerful Android emulator that more closely mirrors a device.

The first step to getting Google Maps working on your emulator is to download a third-party emulator called [Genymotion](http://www.genymotion.com/). The reason for this is that the official emulator does a terrible job of supporting the Google Play Services. While it is possible to get the Intel HAXM fast emulator working with the Play Services SDK, at the moment it's far more trouble then it's worth. 

### Installing Genymotion

See the instructions to [[setup Genymotion|Genymotion 2.0 Emulators-with-Google-Play-support]].  It includes directions for setting Google Play Services and GPS too.  You will need to setup both in order to setup the map demo in the next section.

### Import Maps Demo

Once we have our Genymotion emulator properly setup, let's import the [maps demo application](https://github.com/codepath/android-google-maps-demo) so we can use this to verify if maps are showing up correctly. 

1. Download the [Maps Demo](https://github.com/codepath/android-google-maps-demo/archive/master.zip) application and extract the zip file.
2. Run "File...Import Project...", select the root `build.gradle` and hit "Finish". Refer to this [[importing projects guide|Getting-Started-with-Gradle#importing-existing-android-studio-projects]] to load the project properly.
3. Expand `MapDemo` application and open up the "AndroidManifest.xml"

Fill in your API Key into the meta data for `com.google.android.maps.v2.API_KEY` within the **application node** in the `AndroidManifest.xml`:

```xml
<application
  ...>
  <activity 
     ...>
  </activity>

  <meta-data android:name="com.google.android.gms.version" android:value="@integer/google_play_services_version" />
  <meta-data
     android:name="com.google.android.maps.v2.API_KEY"
     android:value="YOUR-KEY-HERE" />

</application>
```

#### Include the Google Play Services Library

In Android Studio, you need to install the "Google Repository" through the Android Studio SDK Manager. Gradle will pull the Google Play Services library from there.

![Google Repository](http://i.imgur.com/azsdjaz.png)

Now we want to run the map demo, and if everything went well we should see:

<img src="http://i.imgur.com/3KFfS9G.png" width="350" />

**Note:** If you don't, you may have not properly installed the Google Play services on the emulator (see instructions in genymotion setup above) or you may need to update the Google Play services on your emulator by following the instructions given in the app.

### Conclusion

At this point, you should have the Google map displaying in your sample application. If you don't, try restarting the emulator and uninstalling / reinstalling the map demo application. Eventually you will see the maps if you registered your key properly.

For more information including how to use the maps, check out the source of the [[Maps Usage Cliffnotes|Google-Maps-API-v2-Usage]] as well as this article on [androidhive](http://www.androidhive.info/2013/08/android-working-with-google-maps-v2/).

### Troubleshooting

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

 ![](http://i.imgur.com/V6EzQGV.png)

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

This handy [playback-gps](https://github.com/galens/playback-gpx) script allows a developer to simulate traveling a specific route on genymotion. Use this for testing and for presentations if you need to demonstrate how the map works with location in real-time.

## References

* <http://www.androidhive.info/2013/08/android-working-with-google-maps-v2/>
* <http://www.vogella.com/articles/AndroidGoogleMaps/article.html>
* <http://www.genymotion.com/>