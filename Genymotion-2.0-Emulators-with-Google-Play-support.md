### Installing Genymotion

[Genymotion](http://www.genymotion.com/) is an incredibly fast, memory-efficient VM that runs the Android OS in a more accurate manner than even the official emulator. Many Android developers do all their device testing using this emulator especially when Google Play services is concerned.  In addition, the official Android emulator is plagued with a lot of bugs (i.e. intermittent network loss) that Genymotion is usually a far more reliable option.

To setup your genymotion emulator [sign up](https://www.genymotion.com/account/login/) and follow the [installation guide](https://docs.genymotion.com/Content/Home.htm).  

#### Installation

1. Sign up for an account on the [Genymotion Website](https://www.genymotion.com/account/login/)
2. Install [the latest VirtualBox](https://www.virtualbox.org/wiki/Downloads), a powerful free virtualization software for Genymotion to run.  
   * If you already have VirtualBox installed, be sure to open and upgrade to the latest version.
3. [Download Genymotion Emulator v2.7.0 or higher](https://www.genymotion.com/download/) for your platform.  
4. Install the Genymotion Emulator
  * Windows: Run the MSI installer
  * Mac: Open the dmg and drag both apps to Applications directory
5. Go to the Genymotion ADB settings and set the path to your SDK directory (i.e. for Mac OSX, the directory is `/Users/[username]/Library/Android/sdk`.  For Windows, all SDK files should be by default in `C:\Documents and Settings\<user>\AppData\Local\Android\sdk`)
   ![image](https://i.imgur.com/iGqP85B.png)
6. (PC's only) You need to reboot and enable Intel Virtualization Technology or Intel VT-x on the BIOS.   Typically you need to reboot and hit `F1`, `Esc`, or `F10` to enter this mode.  See these [instructions](http://www.sysprobs.com/disable-enable-virtualization-technology-bios) for more information.
   <img src="http://cdn.sysprobs.com/wp-content/uploads/2009/10/virt_bios.gif"> 
7. Install the Genymotion plugin for Android Studio.  Go to `Preferences`->`Plugins` and click Browse repositories. Search for Genymotion and you should find one provided by http://www.genymotion.com.

  ![https://i.imgur.com/AIY7gOS.gif](https://i.imgur.com/AIY7gOS.gif | width=100%)

  After restarting, go to `Preferences`->`Genymotion` and setup the location of the Genymotion app.

  ![https://i.imgur.com/0bdrECm.png](https://i.imgur.com/0bdrECm.png)

#### Configuration

1. Run the Genymotion application
2. Sign in and add your first virtual device (i.e. Nexus 4 - 4.4.4 - API 19, Nexus 4 - 5.0.0. - API 21).  
   * You must use an image running 4.3, 4.4.4, or 5.0.0.  Other versions may not work.
   * Do **not start your emulator** yet!
3. From **within Android Studio**, click the genymobile icon ![Genymobile](https://cloud.genymotion.com/static/images/doc/genymotion-plugin-eclipse-button.png) and click "Start" on your virtual device.
  * You can start the emulator through the Android Studio plugin, or can launch the Genymotion application separately.
4. Wait for device to boot up into a useable state

**Note**: if starting Genymotion you see the following error:

```
Unable to start the virtual device
VirtualBox cannot start the virtual device
To find out the cause the problem, start the virtual device from VirtualBox.
```

Try opening VirtualBox and opening the image directly.  If you see an `VERR_PDM_DRIVER_NOT_FOUND` error, try disabling the audio settings for the virtual image in VirtualBox:

1. Select Android VM
2. Click Settings on top.
3. Go to Audio and Uncheck Enable Audio Checkbox

See [this link](http://stackoverflow.com/questions/38275500/genymotion-virtualbox-error) for more information.

**Note:** On Ubuntu/Linux, make sure to [3D acceleration mode](http://imgur.com/Kl9cOmb) by launching VirtualBox and going to `Settings -> Display` to fix. VirtualBox appears to prone to memory leaks, so you may find yourself killing the process from time to time. To avoid large CPU consumption by the compiz window manager and swapping in general, try increasing the video memory allocation and Base Memory (found in `Settings -> System`).

**Note:** Are you getting an error when starting the emulator? `Error Failed to load VMMR0.r0`? Follow the [advice here](https://forums.virtualbox.org/viewtopic.php?f=8&t=40525#p186381). In short, go to [virtual box page](https://www.virtualbox.org/wiki/Downloads) and download and install VirtualBox 4.3.6 Oracle VM VirtualBox Extension Pack.

**Note:** If you get `Failed to load OVI` error when re-adding the emulator, you need to use a new name for the same device. For example, "Nexus 4 - 4.4.4 - API 19" might be called "Nexus 4 - 4.4.4 - API 19 New".

### Setup Google Play Services

**NOTE**: These steps need to be followed only if you want to be able to use Google services such as maps and push messaging on your Genymotion device. For basic testing, these steps can be safely skipped.

Check out [this handy youtube video](https://www.youtube.com/watch?v=UFhStnF42tw) for a guided step-by-step of enabling play services in Genymotion. You may want to use a newer version of these files based on the desired emulator as found below. 

1. Download the [ARM Translation Installer v1.1](http://www14.zippyshare.com/v/44278764/file.html) and drag and drop the zip file onto the running Genymotion emulator device.  The ARM emulator is only needed for apps that trigger a `INSTALL_FAILED_CPU_ABI_INCOMPATIBLE` error.
  * **Note:** If you get `Files successfully copied` message, you need to make sure there **are no spaces in the filename**. Remove any spaces from the name of your zip file before dragging to ensure the file is detected as flashable.

2. You must **close and restart the emulator** fully before continuing.

3. The Google Play APK package needed is specific to the Android emulator version.  You must use the corresponding Google Play Service package.  Otherwise, you may notice problems with Google Play not having Internet connectivity or other strange issues.

| Version     | APK Link                                                                          |
|-------------|:---------------------------------------------------------------------------------:| 406
|Android 6.0  |1) [gapps-L-4-21-15.zip](https://www.androidfilehost.com/?fid=96042739161891406) 2) [benzo-gapps-M-20151011-signed-chroma-r3.zip](https://www.androidfilehost.com/?fid=24052804347835438) (See instructions below)|
|Android 5.0  |[Google Play Services APK](https://www.androidfilehost.com/?fid=95784891001614559) - requires Genymotion 2.4.0+ |
|Android 4.4.4|[Google Play Services APK](https://www.androidfilehost.com/?fid=23501681358544845)             |
|Android 4.3  |[Google Play Services APK](https://www.androidfilehost.com/?fid=23060877490000124)     |

4. Drag and drop the zip file onto the running Genymotion emulator device
   ![Installing Google Apps APK](https://i.imgur.com/PvGjlyo.png)

   **NOTE**: For Android 6.0, you need to first flash the first .ZIP file.  Then you need to sign-in to your Google account and then flash the second file.  See [these instructions](https://z3ntu.github.io/2015/12/10/play-services-with-genymotion.html) for more details.

5. When asked to flash the device, make sure to proceed with the installation.
   - At this point, 'Google Apps Services' will crash frequently with the message "google play services has stopped working".
   - **Note:** If you get `Files successfully copied` message, you need to make sure there **are no spaces in the filename**. Remove any spaces from the name of your zip file before dragging to ensure the file is detected as flashable.

6. You must **close and restart the emulator** so that Google Play Store can be installed.

7. After restart, open the "Play Store" app on your emulator and **sign in** with a google account.
   - If you can't find Google Play, try updating the Google Hangouts app to trigger an update to the Play Store.
   - You may see messages that the Google+ app needs to be updated.  You will need to go to the Google Play Store and click "Update".

8. **Android 5.0.0 only:** If you are using Android 5.0.0, you will be prompted to update to the latest version of Google+ app.  In addition, make sure to upgrade to at least Genymotion 2.4.0.

9. Make sure to update to the latest version of Google Play Services by opening the "Play Store" app and then the "Maps" app to verify play services is running correctly.

**Note:** If you see errors, be sure you installed the correct package above that matches the device version in the emulator. For example, if you install Samsung Galaxy S4 API 18 for Genymotion emulator and install Google Services APK for Android 4.3, Google Play should work without any issue. The important thing is to match the google play API version and Genymotion emulator device version.

### Enable GPS on Emulator

Next, if you **don't have the emulator started yet**, be sure to boot the genymotion emulator from **within the Android Studio plugin**:

![Emulator](https://i.imgur.com/OsGYNpE.png)

Now we need to enable the GPS location on the emulator by **manually selecting a location** on the map:

![Emulator](https://i.imgur.com/oAdAKA0.png)

## Troubleshooting

### Can't run app on Genymotion emulator

If you are encountering issues with a Genymotion device not being detected inside Android Studio, try the following steps. First, **close all emulators and unplug any devices**. Next, open up the Terminal application on your computer. 

```
cd ~/Library/Android/sdk/platform-tools/
./adb kill-server
killall -9 adb
./adb kill-server
```

Open up the Genymotion application and open `Settings => ADB` and then select "Custom SDK tools". Enter your SDK path into the text field: "/Users/[USERNAME]/Library/Android/sdk":

![image](https://i.imgur.com/iGqP85B.png)

Finally, go back to the terminal and run:

```
cd ~/Library/Android/sdk/platform-tools/
./adb start-server
./adb devices
```

Keep running `./adb devices` until you see a device show up in that list:

```
List of devices attached
192.168.57.101:5555	device
```

### Connecting to VPN sites

If you are trying to use a VPN client to connect to an internal web site, open VirtualBox and select the emulator image.  Click on Settings and make sure `Adapter 1` is disabled.   

<img src="http://i.imgur.com/9tFR6We.png"/>

Verify that `Adapter 2` is selected for NAT, which will force connections to be routed through your machine.

<img src="http://i.imgur.com/ZFQMnaE.png"/>


That's it! Try running the app again.