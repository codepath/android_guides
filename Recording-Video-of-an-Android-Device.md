## Overview

This guide is about how to record a video from a device. This can be done two ways:

1. Using Android Studio
2. Directly through ADB

### 1. Using Android Studio

Open Android Studio. First, go to your menu and `View` -> `Tool Windows` -> `Logcat`.  To take a video recording of your app, see [these instructions](https://developer.android.com/studio/debug/am-video).

### 2. Directly through ADB

1. Connect your device to the computer using USB
2. Open up the terminal and find the path to `adb` executable on your machine. 
3. Run the `adb shell screenrecord` to start recording

  ```
  cd /Users/myusername/Library/Android/sdk/platform-tools
  ./adb shell screenrecord /sdcard/myapp.mp4
  ```
4. Press `Ctrl-C` to stop recording when done
5. Copy the video from the device to your computer:

  ```
  ./adb pull /sdcard/myapp.mp4 \local\path
  ```

Additional resources and options can be reviewed:

 * [Recording configuration](https://www.webniraj.com/2013/11/07/screen-recording-on-nexus-5-using-android-4-4-kitkat-and-android-sdk/) and this [CNET screen recording post](http://www.cnet.com/how-to/how-to-record-your-screen-on-android-4-4-kitkat/)
 * [Copying Files to Computer](http://www.herongyang.com/Android/adb-push-and-pull-Command.html)

With this, you can now easily record videos directly on an Android device.

## Recording with an Emulator

Alternatively, apps can be recorded using the official or [genymotion emulators](Genymotion-2.0-Emulators-with-Google-Play-support). The easiest way to record your screen on OSX is to use the Quicktime Player:

1. Launch Quicktime Player
2. Select `File => New Screen Recording`
3. Click "Down Arrow" next to Record to Configure Microphone
4. Drag to Select Emulator Bounds
5. When done, hit "Stop" icon in Status Bar at the Top
6. Download and use [Handbrake](https://handbrake.fr/) to compress the video.

## References

* <http://stackoverflow.com/a/31547690/313399>
* <https://www.webniraj.com/2013/11/07/screen-recording-on-nexus-5-using-android-4-4-kitkat-and-android-sdk/>
* <http://www.cnet.com/how-to/how-to-record-your-screen-on-android-4-4-kitkat/>
* <http://www.herongyang.com/Android/adb-push-and-pull-Command.html>