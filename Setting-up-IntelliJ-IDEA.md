Android Studio is a version of JetBrain's [IntelliJ Community Edition](http://www.jetbrains.org/pages/viewpage.action?pageId=983211) specifically tailored for Android development.  Installation instructions for Android Studio can be found [here](http://goo.gl/X2SVFR).

IntelliJ IDEA contains all the features found in Android Studio.  Setup for Android development is, however, a bit more involved since it does not include the Android SDK by default.

## Prerequisites
* Install [Java 7 JDK](http://www.oracle.com/technetwork/java/javase/downloads)
* Install [IntelliJ IDEA](https://www.jetbrains.com/idea/download/)
* [[Install Android SDK Tools|Installing-Android-SDK-Tools]]
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
    * Note: Press `Cmd+Shift+.` to show hidden files in the file chooser dialog.
    * Navigate to the JDK location.  E.g., `/Library/Java/JavaVirtualMachines/jdk1.7.0_79.jdk` on OSX.
    * Select the JDK folder
  * Add Android SDK
    * Click `+ > Android SDK`
    * Note: Press `Cmd+Shift+.` to show hidden files in the file chooser dialog.
    * Navigate to the Android SDK location.  E.g., for Homebrew installations, use `/usr/local/Cellar/android-sdk/<sdk-tools-version>`
    * Select the Android SDK version folder and accept the defaults presented
  * Click **OK**

## Create an Android Project
* On the welcome screen, click **Create new project** or select `File > New Project` from the menu bar.
* Select **Android** from the Project Type List on the left
* Select **Gradle:Android Module** from the options on the right
* ![project type selection](https://raw.githubusercontent.com/codepath/android_guides/master/images/intellij_idea_new_project_type.png)
* On the project details screen, specify:
  * Application name
  * Package name
  * Minimum, Target, Compile SDKs
  * ![project details](https://raw.githubusercontent.com/codepath/android_guides/master/images/intellij_idea_project_details.png)
* Select **Blank Activity** from the template screen.
* Accept or modify the subsequent defaults as necessary.
* When the project is opened, Gradle attempts to pull in any dependencies.  If there are errors:
  * In the error log, click on the recommended resolution.
  * ![missing repository](https://raw.githubusercontent.com/codepath/android_guides/master/images/intellij_idea_missing_dependency.png)

## Deploying and Running on a Genymotion Emulator
* Instructions are similar to [[running Genymotion with Android Studio|Genymotion-2.0-Emulators-with-Google-Play-support]]