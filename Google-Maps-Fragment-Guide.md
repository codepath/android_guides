## Overview

Many Android applications require the use of an embedded interactive map. On Android, this embedded map is part of the [Google Play Services SDK](http://developer.android.com/google/play-services/setup.html) which is a Google add-on pack for Android enabling all sorts of extra features around gaming, messaging, billing, location, etc.

In this guide, we will walk you through the step by step process of getting an embedded Google Map working within an Android emulator.

### Download Google Play Services

First, let's download and setup the Google Play Services SDK. Open **Eclipse ⇒ Windows ⇒ Android SDK** Manager and check whether you have already downloaded "Google Play services" or not under Extras section. If not, select "Google Play services" and install the package.

![Play Services](http://i.imgur.com/3ZJsWEw.png)

### Import Google Play Services

After downloading the play services we need to import the project to Eclipse which will be used as a library for our maps project.

1. In Eclipse goto **File ⇒ Import ⇒ Android ⇒ Existing Android Code Into Workspace**

2. Click on Browse and select "Google Play Services" project from your android sdk folder. You can locate the play services library project from
`<your-android-sdk-path>\extras\google\google_play_services\libproject\google-play-services_lib`

3. Be sure to check "Copy projects into workspace" option as shown in the below image.

![Google Play Load](http://i.imgur.com/EHefxss.png)

### Retrieve your SHA-1 Fingerprint

Open your terminal and execute the following command to generate SHA-1 fingerprint necessary to get your API key from Google.

On Windows:

```
"C:\Program Files (x86)\Java\jdk1.7.0_60\bin\keytool.exe" -list -v -keystore "%USERPROFILE%\.android\debug.keystore" -alias androiddebugkey -storepass android -keypass android
```

On Mac or Linux:

```
keytool -list -v -keystore ~/.android/debug.keystore -alias androiddebugkey -storepass android -keypass android
```

In the output you can see SHA 1 finger print:

![Fingerprint](http://www.androidhive.info/wp-content/uploads/2013/08/android-google-maps-sha-1-fingerprint.png)

### Register for an API Key

Now open [Google API Console](https://cloud.google.com/console), create and select a project, select "APIs & Auth" on left side and turn on Google Maps Android API v2 toggle switch.

![Services](http://i.imgur.com/i4NUoJy.png)

Now select "Credentials" on left side and on the right side click on "Create New Key" =>  "Android Key"

![Access](http://www.androidhive.info/wp-content/uploads/2013/08/google-console-creating-a-key.png)

It will popup a window asking the SHA1 and package name. Enter your `SHA 1` and your `android project package name` separated by semicolon ; and click on create. For now enter:

```
BE:03:E1:44:39:7B:E8:17:02:9F:7F:B7:98:82:EA:DF:84:D0:FB:6A;com.example.mapdemo
```

Note the API Key for use in the next step:

![Key](http://www.androidhive.info/wp-content/uploads/2013/08/google-console-android-maps-api-key1.png)

### Genymotion

The first step to getting Google Maps working on your emulator is to download a third-party emulator called [Genymotion](http://www.genymotion.com/). The reason for this is that the official emulator does a terrible job of supporting the Google Play Services. While it is possible to get the Intel HAXM fast emulator working with the Play Services SDK, at the moment it's far more trouble then it's worth. 

[Genymotion](http://www.genymotion.com/) is an incredibly fast, memory-efficient VM that runs the Android OS in a more accurate manner than even the official emulator. Many Android developers do all their device testing using this emulator especially when Google Play services is concerned.

To setup your genymotion emulator [sign up](https://cloud.genymotion.com/page/customer/login/?next=/) and follow the [installation guide](https://cloud.genymotion.com/page/doc/):

1. Sign up for an account on the [Genymotion Website](https://cloud.genymotion.com/page/customer/login/?next=/)
2. (Mac or Linux) You **must** [install VirtualBox](https://www.virtualbox.org/wiki/Downloads), a powerful free virtualization software for Genymotion to run.
3. [Download Genymotion Emulator](https://cloud.genymotion.com/page/launchpad/download/) for your platform.
4. Install the Genymotion Emulator
  * Windows: Run the MSI installer
  * Mac: Open the dmg and drag both apps to Applications directory
5. Run the Genymotion application
6. Sign in and add your first virtual device (Nexus 4 - 4.3 - API 18)
   * Do **not start your emulator** yet
7. Install the Eclipse Plugin
   * Go to the "Help/Install New Software..." menu
   * Add a new software site: Genymobile - http://plugins.genymotion.com/eclipse
   * Check all genymobile entries
   * Accept licenses and install
   * Restart eclipse
8. Click the genymobile icon ![Genymobile](https://cloud.genymotion.com/static/images/doc/genymotion-plugin-eclipse-button.png) and click "Start" on your virtual device.
  * Make sure to start your emulator **through the eclipse plugin**
9. Wait for device to boot up and then run through the initial setup
10. Download the latest [Google Play Services APK](http://goo.im/gapps/gapps-jb-20130813-signed.zip) for 4.3 from [Rootz Wiki](http://wiki.rootzwiki.com/Google_Apps#Universal_Packages_2)
11. Drag and drop the zip file onto the running Genymotion emulator device
    * Seeing a **crash dialog of Google services** is to be expected
12. **Close and restart the emulator** and Google Play Store should now be installed
13. After restart, open the "Play Store" app on your emulator and **sign in** with a google account

**Note:** On Ubuntu 12.04, make sure to [3D acceleration mode](http://imgur.com/Kl9cOmb) by launching VirtualBox and going to `Settings -> Display` to fix. VirtualBox appears to prone to memory leaks, so you may find yourself killing the process from time to time. To avoid large CPU consumption by the compiz window manager and swapping in general, try increasing the video memory allocation and Base Memory (found in `Settings -> System`).

### Enable GPS on Emulator

Next, if you **don't have the emulator started yet**, be sure to boot the genymotion emulator from **within the Eclipse plugin**:

![Emulator](http://i.imgur.com/OsGYNpE.png)

Now we need to enable the GPS location on the emulator by **manually selecting a location** on the map:

![Emulator](http://i.imgur.com/oAdAKA0.png)

With GPS location enabled, let's now setup our map demo app.

### Import Maps Demo

Once we have our Genymotion emulator properly setup, let's import the [maps demo application](https://github.com/thecodepath/android-google-maps-demo) so we can use this to verify if maps are showing up correctly. 

1. Download the [Maps Demo](https://github.com/thecodepath/android-google-maps-demo/archive/master.zip) application and extract the zip file.
2. Run "File...Import...Existing Android Code Into Workspace", select the project and hit "Finish"
3. Expand MapDemo application and open up the "AndroidManifest.xml"

Fill in your API Key into the meta data for `com.google.android.maps.v2.API_KEY` within the **application node** in the `AndroidManifest.xml`:

```xml
<application
  ...>
  <activity 
     ...>
  </activity>
  <meta-data
     android:name="com.google.android.maps.v2.API_KEY"
     android:value="YOUR-KEY-HERE" />
</application>
```

Now we need to **include Google Play Services project** as a library to use in our demo app. So right click on project and select "Properties". In the Properties window on left side select "Android". On the right you can see a Add button under library section. Click it and select google play services project which we imported previously.

![Library](http://i.imgur.com/9dsbUBP.png)

Now we want to run the map demo, and if everything went well we should see:

<img src="http://i.imgur.com/3KFfS9G.png" width="350" />

**Note:** If you don't, you may have not properly installed the Google Play services on the emulator (see instructions in genymotion setup above) or you may need to update the Google Play services on your emulator by following the instructions given in the app.

### Conclusion

At this point, you should have the Google map displaying in your sample application. If you don't, try restarting the emulator and uninstalling / reinstalling the map demo application. Eventually you will see the maps if you registered your key properly.

For more information including how to use the maps, check out the source of this article on [androidhive](http://www.androidhive.info/2013/08/android-working-with-google-maps-v2/).

### Troubleshooting

Use this checklist for troubleshooting below:

* **Trouble launching Genymotion?**
   * Did you install [VirtualBox](https://www.virtualbox.org/wiki/Downloads) first. Run VirtualBox to ensure that it was installed correctly.
   * Did you properly move Genymotion into the Applications folder?
   * Launch VirtualBox, verify the emulator is listed and ensure it is **powered off**
   * Open Genymotion app and verify the emulator is listed there
   * Restart Eclipse and try to launch the emulator again
   * Restart your computer and try to launch the emulator again
* **Trouble seeing the map in the demo app?**
   * Did you enter the API Key into the `AndroidManifest.xml` as explained in the demo setup?
   * Did you enable "Google Maps Android API v2" toggle switch on the Google API Console under "APIs & Auth"?
   * Did you enable GPS location and set a location on the emulator?
   * Did you download the latest [Google Play Services APK](http://goo.im/gapps/gapps-jb-20130813-signed.zip) and install that onto your emulator? See the genymotion section steps 10-12.
   * Did you try running Play services on your emulator and sign in with a Google account?
   * Uninstall the "Map Demo" from the emulator first, then re-install
   * Restart Eclipse and try to launch the app again 
   * Restart your computer and try to launch the app again

## References

* <http://www.androidhive.info/2013/08/android-working-with-google-maps-v2/>
* <http://www.vogella.com/articles/AndroidGoogleMaps/article.html>
* <http://www.genymotion.com/>