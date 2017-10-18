## Overview

This guide details how to present an Android app to an audience. The easiest way to do this is by connecting your phone to your computer and displaying the screen using Mobizen. First, we need to setup USB debugging and then we need to setup and configure Mobizen. 

## Enable USB Debugging

First to present your phone, we need to enable USB debugging so your phone can interact with your computer in a developer mode. 

[![](https://i.imgur.com/8DQIyWc.png)](https://www.youtube.com/watch?v=Ucs34BkfPB0&feature=youtu.be)

The following steps are needed:

0. (Windows Only) Install [this ADB Driver](http://download.clockworkmod.com/test/UniversalAdbDriverSetup.msi)
1. Plug-in your Android Device to Computer via USB
2. Open the "Settings" App on the Device
3. Scroll down to bottom to find "About phone" item
4. Scroll down to bottom to find "Build number" section
5. Tap on "Build Number" 7 times in quick succession
6. You should see the message "You are now a developer!"
7. Go back to main "Settings" page
8. Scroll down bottom to find "Developer options" item
9. Turn on "USB Debugging" switch and hit "OK"
10. Unplug and re-plug the device
11. Dialog appears "Allow USB Debugging?"
12. Check "Always allow from this computer" and hit "OK"

Watch [this video tutorial](https://www.youtube.com/watch?v=Ucs34BkfPB0&feature=youtu.be) for a visual guide to getting USB debugging enabled.

## Mobizen

[Mobizen](https://www.mobizen.com/?locale=en) is the easiest and most reliable way to present your Android screen onto your computer.

<a href="https://www.mobizen.com/?locale=en" target="_blank"><img src="https://i.imgur.com/F0uSD12.png" height="400" /></a>

There are three steps to using Mobizen:

1. Download and Configure Mobizen on Android Device
2. Download and install Mobizen PC to your desktop
3. Connect to your Android Device from Computer

### 1. Download and Configure Mobizen onto Android Device

First, we need to download and configure Mobizen on our device. Go to the "Play Store" and install one of these Android apps:

 * [Non-Samsung App](https://play.google.com/store/apps/details?id=com.rsupport.mvagent)
 * [Samsung App](https://play.google.com/store/apps/details?id=com.rsupport.mobizen.sec)
 
Open the app up on your phone:

  1. Tap "Start" to begin
  2. Choose an Email Account
  3. Create a Password to Register
  4. Tap "Next" Right Arrow or Swipe a few times
  5. Tap "Start Mobizen" 

You should see the following on your device:

<img src="https://i.imgur.com/ortJlkO.png" height="400" />

Now, let's setup mobizen on our computer.

### 2. Download and install Mobizen PC to your desktop

Visit the [Mobizen Site](https://www.mobizen.com) to download the plug-in onto your computer:

 * [Download the App](https://download.mobizen.com/download/mobizen.pkg)
 * Opening the downloaded package and run the setup wizard
 * Complete installation by following each step

You should see the following on your computer:

<img src="https://i.imgur.com/QdJD6Ke.png" width="500" />

Now, let's connect your device to your computer using mobizen.

### 3. Connect to your Android Device from Computer

Mobizen can connect to your device using either **USB** or **WiFi** connections. Connecting with USB allows for a **much smoother framerate** and is strongly recommended. Follow these steps: 

1. Connect the Android device to the Computer using USB
2. Ensure `USB Debugging` mode is active by checking the notifications screen.
3. Go to `Settings => Developer options` and enable "Stay awake"
4. Go to `Settings => Display => Screen Timeout` and change to "5 minutes"
5. Sign in to [Mobizen](https://www.mobizen.com/?locale=en) with the email and password configured in earlier steps. 
6. Wait until the present dialog appears on the device
7. Tap "Start Now" on the device to display screen on the computer
8. Select "FullScreen" on the menu sidebar

<img src="https://i.imgur.com/Tk8N6LJ.png" height="530" />
&nbsp;
<img src="https://i.imgur.com/5Njpxpn.png" height="530" />

You should be good to go for presenting your screen!

## Alternatives

There are several alternatives to Mobizen for presenting an Android app including:

 * [Using ChromeCast](https://support.google.com/chromecast/answer/6059461?hl=en) - Requires chromecast
 * [Vysor](http://www.vysor.io/) - Chrome Extension
 * [Mirroring360](http://www.mirroring360.com/how-it-works#steps) - Desktop App
 * [Droid@Screen](http://droid-at-screen.org/installation.html) - JAR Java App
