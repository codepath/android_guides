Android Studio is version of IntelliJ specifically tailored for Android development.  Installation instructions for Android Studio can be found [here](http://goo.gl/X2SVFR).

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
  * ![Android SDK Manager](https://raw.githubusercontent.com/codepath/android_guides/master/images/intellij_idea_android_sdk_manager.png)
  * Ensure the following are installed:
    * `Tools > Android SDK Tools`
    * `Tools > Android SDK Platform-tools`
    * `Tools > Android SDK Build-tools`
    * One version of the Android Platform.  E.g., `Android 5.1.1 (API 22)`
  * Install any required components by checking the component and clicking **Install packages**
* [[Install Genymotion|Genymotion-2.0-Emulators-with-Google-Play-support]]

## Configure IntelliJ IDEA
* Open IntelliJ IDEA
* Close all open project windows
* The IntelliJ Welcome screen will be displayed
  * Add required SDKs
  * Click on `Configure > Project Defaults > Project Structure`
    * ![welcome screen](https://raw.githubusercontent.com/codepath/android_guides/master/images/intellij_idea_welcome_screen.png)
  * Select `SDKs`
  * Add Java Development Kit
    * Click `+ > JDK`
      * ![Add JDK](https://raw.githubusercontent.com/codepath/android_guides/master/images/intellij_idea_add_sdk.png)
    * Navigate to the JDK location.  E.g., `/Library/Java/JavaVirtualMachines/jdk1.7.0_79.jdk` on OSX.
    * Select the JDK folder
  * Add Android SDK
    * Click `+ > Android SDK`
    * Navigate to the Android SDK location.  E.g., for Homebrew installations, use `/usr/local/Cellar/android-sdk/<sdk-tools-version>`
    * Select the Android SDK version folder and accept the defaults presented
  * Click **OK**

## Create an Android Project
* On the welcome screen, click **Create new project** or select `File > New Project` from the menu bar.
* Select **Android** from the Project Type List on the left
* Select **Gradle:Android Module** from the options on the right
* ![project type selection](https://github.com/codepath/android_guides/blob/master/images/intellij_idea_new_project_type.png)
* On the project details screen, specify:
  * Application name
  * Package name
  * Minimum, Target, Compile SDKs
  * ![project details](https://github.com/codepath/android_guides/blob/master/images/intellij_idea_project_details.png)
* Select **Blank Activity** from the template screen.
* Accept or modify the subsequent defaults as necessary.
* When the project is opened, Gradle attempts to pull in any dependencies.  If there are errors:
  * In the error log, click on the recommended resolution.
  * ![missing repository](https://raw.githubusercontent.com/codepath/android_guides/master/images/intellij_idea_missing_dependency.png)

## Deploying and Running on a Genymotion Emulator
* Instructions are similar to [[running Genymotion with Android Studio|Genymotion-2.0-Emulators-with-Google-Play-support]]