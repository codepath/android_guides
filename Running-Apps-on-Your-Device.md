## Overview

When building an Android app, it's important that you always test your application on a real device in addition to emulators. This page describes how to set up your development environment and Android-powered device for testing and debugging on the device.

If you want an ideal SIM-unlocked phone to test on, then you might [consider a Nexus phone](https://play.google.com/store/devices). 

## Connect your Phone to Computer

Plug in your device to your computer with a USB cable. If you're developing on Windows, you might need to install this [universal ADB USB driver](http://download.clockworkmod.com/test/UniversalAdbDriverSetup.msi) or find your [specific USB driver for your device](https://developer.android.com/tools/extras/oem-usb.html).

## Enable USB Debugging

The next step is to enable USB debugging so your phone can interact with your computer in a developer mode. 

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
12. Check "Always allow from this computer" and then hit "OK"

Watch [this video tutorial](https://www.youtube.com/watch?v=Ucs34BkfPB0&feature=youtu.be) for a visual guide to getting USB debugging enabled.

## Running your App

Now, we can launch apps from Android Studio onto our device:

1. Select one of your projects and click "Run" from the toolbar.
2. In the "Choose Device" window that appears, select the "Choose a running device" radio button, select the device, and click OK.

Once Gradle finishes building, Android Studio should install the app on your connected device and start it.

## Troubleshooting

Not seeing your device in the "Choose Device" window? Try the following:

 * Unplug your device from the USB port on the computer 
 * Restart the device by powering off and back on
 * Verify that `Settings => Developer options => USB Debugging` is enabled
 * Quit and re-launch Android Studio
 * Force [restart ADB](https://www.youtube.com/watch?v=FtdydFDBuQo) from the "Android Device Monitor"
 * Plug your device back into the USB port on the computer 
 * Unlock the device and press "OK" on any dialog displayed

Now the phone should work as a debugging device as expected!

**Still Not Working?**

If after plugging the device into the computer and you don't see any message about authorizing the device, then you may need to purchase another USB cable. Not all USB cables are **enabled for data transfer**. If there's a chance that your cable may be a charging only cable, you can purchase a [USB-C cable for Nexus 6P](https://www.amazon.com/Nekteck-resistor-Charging-Reversible-Macbook/dp/B00VIWE1ZY) or the [micro-USB cable for Nexus 6 and prior](https://www.amazon.com/Mediabridge-USB-2-0-High-Speed-30-004-06B/dp/B004GF8TIK).

## References

* <https://developer.android.com/studio/run/device.html>
* <https://developer.android.com/training/basics/firstapp/running-app.html>
