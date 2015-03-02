### Installing Genymotion

[Genymotion](http://www.genymotion.com/) is an incredibly fast, memory-efficient VM that runs the Android OS in a more accurate manner than even the official emulator. Many Android developers do all their device testing using this emulator especially when Google Play services is concerned.  In addition, the official Android emulator is plagued with a lot of bugs (i.e. intermittent network loss) that Genymotion is usually a far more reliable option.

To setup your genymotion emulator [sign up](https://cloud.genymotion.com/page/customer/login/?next=/) and follow the [installation guide](https://cloud.genymotion.com/page/doc/):

#### Installation

1. Sign up for an account on the [Genymotion Website](https://cloud.genymotion.com/page/customer/login/?next=/)
2. (Mac or Linux) You **must** [install VirtualBox](https://www.virtualbox.org/wiki/Downloads), a powerful free virtualization software for Genymotion to run.
3. [Download Genymotion Emulator](https://cloud.genymotion.com/page/launchpad/download/) for your platform.
4. Install the Genymotion Emulator
  * Windows: Run the MSI installer
  * Mac: Open the dmg and drag both apps to Applications directory

5. Go to the Genymotion ADB settings and set the path to your SDK directory (i.e. for Mac OSX, the directory is `~/Library/Android/sdk`.  For Windows, all SDK files should be by default in 
`C:\Documents and Settings\<user>\AppData\Local\Android\sdk`)

   ![image](http://i.imgur.com/iGqP85B.png)

6. Install the Genymotion plugin for Android Studio.  Go to Preferences->Plugins, and click Browse repositories. Search for Genymotion and you should find one provided by http://www.genymotion.com.

  ![http://i.imgur.com/AIY7gOS.gif](http://i.imgur.com/AIY7gOS.gif | width=100%)

  After restarting, go to Preferences->Genymotion and setup the location of the Genymotion app.

  ![http://i.imgur.com/0bdrECm.png](http://i.imgur.com/0bdrECm.png)

#### Configuration

1. Run the Genymotion application
2. Sign in and add your first virtual device (Nexus 4 - 4.4.4 - API 19).  
   * You must use an image running 4.4.4.  Older versions may not work.
   * Do **not start your emulator** yet!
3. From **within Android Studio**, click the genymobile icon ![Genymobile](https://cloud.genymotion.com/static/images/doc/genymotion-plugin-eclipse-button.png) and click "Start" on your virtual device.
  * You can start the emulator through the Android Studio plugin, or can launch the Genymotion application separately.
4. Wait for device to boot up into a useable state

**Note:** On Ubuntu/Linux, make sure to [3D acceleration mode](http://imgur.com/Kl9cOmb) by launching VirtualBox and going to `Settings -> Display` to fix. VirtualBox appears to prone to memory leaks, so you may find yourself killing the process from time to time. To avoid large CPU consumption by the compiz window manager and swapping in general, try increasing the video memory allocation and Base Memory (found in `Settings -> System`).

**Note:** Are you getting an error when starting the emulator? `Error Failed to load VMMR0.r0`? Follow the [advice here](https://forums.virtualbox.org/viewtopic.php?f=8&t=40525#p186381). In short, go to [virtual box page](https://www.virtualbox.org/wiki/Downloads) and download and install VirtualBox 4.3.6 Oracle VM VirtualBox Extension Pack.

**Note:** If you get `Failed to load OVI` error when re-adding the emulator, you need to use a new name for the same device. For example, "Nexus 4 - 4.4.4 - API 19" might be called "Nexus 4 - 4.4.4 - API 19 New".

### Setup Google Play Services

1. Download the latest [Google Play Services APK](https://www.mediafire.com/?qbbt4lhyu9q10ix) for 4.4.4 from [Mediafire](https://www.mediafire.com/?qbbt4lhyu9q10ix)
2. Drag and drop the zip file onto the running Genymotion emulator device

   ![Installing Google Apps APK](http://i.imgur.com/PvGjlyo.png)

3. You must **close and restart the emulator** so that Google Play Store can be installed.
4. After restart, open the "Play Store" app on your emulator and **sign in** with a google account.
5. Make sure to update to the latest version of Google Play Services.


### Enable GPS on Emulator

Next, if you **don't have the emulator started yet**, be sure to boot the genymotion emulator from **within the Android Studio plugin**:

![Emulator](http://i.imgur.com/OsGYNpE.png)

Now we need to enable the GPS location on the emulator by **manually selecting a location** on the map:

![Emulator](http://i.imgur.com/oAdAKA0.png)