Android Studio is version of IntelliJ specifically tailored Android development.  Installation instructions for Android Studio can be found [here](http://goo.gl/X2SVFR).

IntelliJ IDEA contains all the features found in Android Studio.  Setup for Android development is, however, a bit more involved than Android Studio.

## Prerequisites
* Install [Java 7 JDK](http://www.oracle.com/technetwork/java/javase/downloads)
* Install [IntelliJ IDEA](https://www.jetbrains.com/idea/download/)

## Install the Android SDK Tools
* Option 1: Homebrew
  * [Install Homebrew](http://brew.sh/) - package manager for OSX
  * run `brew install android-sdk`
  * This will install the Android SDK tools in `/usr/local/Cellar/android-sdk/<version number>`
* Option 2: Download and unzip
  * Download the [Android SDK Tools](https://developer.android.com/sdk/installing/index.html?pkg=tools)
  * Unzip files to an easy to access/find location.  The location will be required later E.g., `~/android-sdk/`

## Install Android Platform and Genymotion Emulator
* Install at least one version of the Android Platform
  * At a terminal prompt, issue the following command to open the Android SDK Manager:
    * `<location of Android SDK Tools>/android sdk`
  * Ensure the following are installed:
    * `Tools > Android SDK Tools`
    * `Tools > Android SDK Platform-tools`
    * `Tools > Android SDK Build-tools`
    * One version of the Android Platform.  E.g., `Android 5.1.1 (API 22)`
* [[Install Genymotion|Genymotion-2.0-Emulators-with-Google-Play-support]]

## Create Project in IntelliJ IDEA
* Open IntelliJ IDEA
* Close all open project windows
* The IntelliJ Welcome screen will be displayed
* Click on `Configure > Project Defaults > Project Structure`
* Select `SDK`
* Add Java Development Kit
  * Click `+ > JDK`
  * Navigate to the JDK location.  E.g., `/Library/Java/JavaVirtualMachines/jdk1.7.0_79.jdk` on OSX.
  * Select the JDK folder
* Add Android SDK
  * Click `+ > Android SDK`
  * Navigate to the Android SDK location.  E.g., for Homebrew installations, it will be `/usr/local/Cellar/android-sdk/<sdk-tools-version>`

  



