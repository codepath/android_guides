## Overview

Many Android applications require the use of an embedded interactive map. On Android, this embedded map is part of the [Google Play Services SDK](http://developer.android.com/google/play-services/setup.html) which is a Google add-on pack for Android enabling all sorts of extra features around gaming, messaging, billing, location, etc.

In this guide, we will walk you through the step by step process of getting an embedded Google Map working within an Android emulator.

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