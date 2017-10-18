### Installing Genymotion

[Genymotion](http://www.genymotion.com/) is an incredibly fast, memory-efficient VM that runs the Android OS in a more accurate manner than even the official emulator. Many Android developers do all their device testing using this emulator especially when Google Play services is concerned.  In addition, the official Android emulator is plagued with a lot of bugs (i.e. intermittent network loss) that Genymotion is usually a far more reliable option.

To setup your genymotion emulator [sign up](https://www.genymotion.com/account/login/) and follow the [installation guide](https://docs.genymotion.com/Content/Home.htm).

**Note:** If you already have installed Genymotion on your system then you can skip this steps and go straight to [setup Google Play Services](#setup-google-play-services)  

#### Installation

1. Sign up for an account on the [Genymotion Website](https://www.genymotion.com/account/login/)
2. Install [the latest VirtualBox](https://www.virtualbox.org/wiki/Downloads), a powerful free virtualization software for Genymotion to run.  
   * If you already have VirtualBox installed, be sure to open and upgrade to the latest version.
3. [Download Genymotion Emulator](https://www.genymotion.com/download/) for your platform.  
4. Install the Genymotion Emulator
  * Windows: Run the MSI installer
  * Mac: Open the dmg and drag both apps to Applications directory
5. Go to the Genymotion ADB settings and set the path to your SDK directory (i.e. for Mac OSX, the directory is `/Users/[username]/Library/Android/sdk`.  For Windows, all SDK files should be by default in `C:\Documents and Settings\<user>\AppData\Local\Android\sdk`)
   ![image](https://i.imgur.com/iGqP85B.png)
6. (PC's only) You need to reboot and enable Intel Virtualization Technology or Intel VT-x on the BIOS.   Typically you need to reboot and hit `F1`, `Esc`, or `F10` to enter this mode.  See these [instructions](http://www.sysprobs.com/disable-enable-virtualization-technology-bios) for more information.
   <img src="http://cdn.sysprobs.com/wp-content/uploads/2009/10/virt_bios.gif"> 

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

**Note:** On Ubuntu/Linux, make sure to enable [3D acceleration mode](https://imgur.com/Kl9cOmb) by launching VirtualBox and going to `Settings -> Display` to fix. VirtualBox appears to prone to memory leaks, so you may find yourself killing the process from time to time. To avoid large CPU consumption by the compiz window manager and swapping in general, try increasing the video memory allocation and Base Memory (found in `Settings -> System`).

**Note:** Are you getting an error when starting the emulator? `Error Failed to load VMMR0.r0`? Follow the [advice here](https://forums.virtualbox.org/viewtopic.php?f=8&t=40525#p186381). In short, go to [virtual box page](https://www.virtualbox.org/wiki/Downloads) and download and install VirtualBox 4.3.6 Oracle VM VirtualBox Extension Pack.

**Note:** If you get `Failed to load OVI` error when re-adding the emulator, you need to use a new name for the same device. For example, "Nexus 4 - 4.4.4 - API 19" might be called "Nexus 4 - 4.4.4 - API 19 New".

### Setup Google Play Services

As of version 2.10, setting up [Google Play Services on Genymotion](https://www.genymotion.com/blog/2-10-open-gapps-widget/) is really straightforward. Once in the emulator, a new "Open GApps" widget will be available in your toolbar. Click it and it’ll do the rest. The "Open GApps" widget is available for every device running Android 4.4 or higher (whether new or already created, as long as Genymotion 2.10 is installed).

The Open GApps package implemented is the smallest one (i.e. "pico") as it fits most development and testing needs. If you need access to further Google Apps and Services, simply [visit the Play Store and download them from there](https://play.google.com/store/apps/dev?id=5700313618786177705). If Google Play Services get updated, no need to click the widget again. Updates will be handled directly within your virtual device – just like a real device after all

#### Manual Installation

**IMPORTANT NOTE**: Since version 2.10 Genymotion includes native support for Google Play Services and Google Store. Just open the emulator and click in the "Open GApps" widget showed in the toolbar. More info in [Genymotion's blog](https://www.genymotion.com/blog/2-10-open-gapps-widget/). If you are working with a previous version you can follow the next steps.

**NOTE**: These steps need to be followed only if you are using Genymotion < 2.10 and you want to be able to use Google services such as maps and push messaging on your Genymotion device. For basic testing, these steps can be safely skipped.

Check out [this handy YouTube video](https://www.youtube.com/watch?v=UFhStnF42tw) for a guided step-by-step of enabling play services in Genymotion. You may want to use a newer version of these files based on the desired emulator as found below. 

1. Download the [opengapps.org](http://opengapps.org/) image corresponding to your Android version. Make sure to choose **x86** and the **nano** instance.  
      <img src="https://imgur.com/z8O6Txp.png" width="400">

2. Drag and drop the zip file onto the running Genymotion emulator device
   ![Installing Google Apps APK](https://imgur.com/zSPf1Fn.png)

3. When asked to flash the device, make sure to proceed with the installation.

4. You must **close and restart the emulator** so that Google Play Store can be installed.

5. After restart, open the "Play Store" app on your emulator and **sign in** with a google account.

6. Make sure to update to the latest version of Google Play Services by opening the "Play Store" app and then the "Maps" app to verify play services is running correctly.

7. If you are using Android 7.0 (API 24) or higher, **make sure** to install the Chrome app from the Play Store.  The WebView browser that comes with API 24 and higher is not a fully feature compatible one. 

8. Download the [ARM Translation Installer v1.1](http://www14.zippyshare.com/v/44278764/file.html) and drag and drop the zip file onto the running Genymotion emulator device.  The ARM emulator is only needed for apps that trigger a `INSTALL_FAILED_CPU_ABI_INCOMPATIBLE` error.  See [this FAQ](https://www.genymotion.com/help/desktop/faq/) for more context.
   * **Note:** If you get `Files successfully copied` message, you need to make sure there **are no spaces in the filename**. Remove any spaces from the name of your zip file before dragging to ensure the file is detected as flashable.

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

<img src="https://i.imgur.com/9tFR6We.png"/>

Verify that `Adapter 2` is selected for NAT, which will force connections to be routed through your machine.

<img src="https://i.imgur.com/ZFQMnaE.png"/>

That's it! Try running the app again.

### Issuing Manually Setting Up Play Services 

While installing play services, there are a variety of issues that can come up, refer to a few common ones below:

* **Received a `Files successfully copied` message after dragging over the zip.** You need to make sure there **are no spaces in the filename**. Remove any spaces from the name of your zip file before dragging to ensure the file is detected as flashable.

* **Received a message about files being added to the SD card** If you are using a Mac, and get this message while dragging over the file to the emulator, make sure to [configure your app security settings](https://kb.wisc.edu/helpdesk/page.php?id=25443) to allow all applications. 

* **Seeing errors after flashing the emulator?** If you see errors, be sure you installed the correct package above that matches the device version in the emulator. For example, if you install Samsung Galaxy S4 API 18 for Genymotion emulator and install Google Services APK for Android 4.3, Google Play should work without any issue. The important thing is to **match the google play API version and Genymotion emulator device version**.

* **Try rebooting your system.** If all else fails, may find in order for things to take hold properly that you need to install the ARM-translator, and then reboot your computer, and after that, install the Gapps and then reboot a second time. This can sometimes fix issues that are otherwise unresolved. 
