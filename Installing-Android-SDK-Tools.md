## Installing the Android SDK (Automated Way)

You can use [Jake Wharton's SDK Manager](https://github.com/JakeWharton/sdk-manager-plugin) to manage all missing SDK dependencies.  These definitions should be declared before the regular `com.android.application` plugin is applied.  If you have [multiple subprojects](http://gradle.org/docs/current/userguide/multi_project_builds.html) used in your app, make sure every `build.gradle` has this dependency included.

```gradle
buildscript {
  repositories {
    mavenCentral()
  }
  dependencies {
    classpath 'com.android.tools.build:gradle:0.12.+'
    classpath 'com.jakewharton.sdkmanager:gradle-plugin:0.12.+'
  }
}

apply plugin: 'android-sdk-manager' // run before com.android.application
apply plugin: 'com.android.application'

// optionally including an emulator
sdkManager {
  emulatorVersion 'android-19'
  emulatorArchitecture 'armeabi-v7a' // optional, defaults to arm
}
```

## Installing the Android SDK (Manual Way)

Download the Android SDK without Eclipse bundled. Go to [Android SDK](http://developer.android.com/sdk/index.html) and copy the URL for the **SDK Tools Only** download that's appropriate for your build machine OS.

![List of Android SDK downloads from developers.android.com](https://dl.dropboxusercontent.com/u/10808663/gradle_jenkins_android/sdk_downloads.png)

Use `wget` with the correct SDK URL:

    $ wget http://dl.google.com/android/android-sdk_r22.6.2-macosx.zip

Unzip and place the contents within your home directory. The directory names can be anything you like, but I like the following setup.  The Android SDK is in `/Users/ciandroid/src/`.

 ![Directory structure on the build server](https://dl.dropboxusercontent.com/u/10808663/gradle_jenkins_android/directories_on_build_server.png)

Now it's time to set your build environment's `PATH` variable and other variables that will be use to locate Android.

Edit your `.bash_profile` file. If you're not using bash, edit the right config file for your environment.

    # Android 
    export ANDROID_HOME=/Users/ciandroid/android-sdk-macosx
    export PATH=$PATH:$ANDROID_HOME/tools

 
Save and quit. Reload `.bash_profile`:

    $ source ~./bash_profile

### Installing via the GUI
 
![Android SDK manager on build machine](https://dl.dropboxusercontent.com/u/10808663/gradle_jenkins_android/android_sdk_manager.png)

At the prompt, type `android` and hit Enter to launch the Android SDK Manager in a window. If this doesn't work, your `PATH` variable has not been set up with the Android SDK location.   

You will want to install the same Android SDK packages on your build machine as you did to get Gradle running locally. Before you begin, take a look at the `build.gradle` file in your project.

#### Packages to install

Here are the SDK package names you'll definitely wish to select:

  * Android SDK Tools (latest version)
  * Android SDK Platform-tools (latest version)
  * Android SDK Build-tools (latest version)
  * Android SDK Build-tools for the version of Android that you listed in the `build.gradle` file as the `android: buildToolsVersion` target. If your `build.gradle` says 
```gradle
    android {
        buildToolsVersion '17'
        ...
    }
```

then make sure to download that API version in the Android SDK Manager. 

  * Android SDK set for the API levels that you named in the `android: compileSdkVersion` section of your `build.gradle` file.

You will also want to download the extras:
  * Android Support Repository
  * Android Support Library


## Downloading the SDK from the Command Line

You can also download the SDK packages using the command line with the `--no-ui` parameter.

```
android update sdk --no-ui --all
```

If you want to be selective about installing, you can use `android list` to view all the packages and apply the `--filter` option for selective installs:

```
sudo android update sdk --no-ui --filter platform-tools,tools
```

There is currently no filter to install the build tools directly.  See this [ticket](https://code.google.com/p/android/issues/detail?id=78765) for more information.

## Installing the Android SDK (via Chef)

Finally, if you intend to incorporate Android builds into your continuous integration setup, you may want to use [Chef](https://learn.chef.io/) or Puppet to help automate this process.  These configuration management tools help automate the process of managing machine configurations in Amazon, Rackspace, Linode, or any cloud-based hosting service.  

For Chef, you can download the [chef-android-sdk](https://github.com/gildegoma/chef-android-sdk) recipe.  It has a few dependencies, including Ark and 7-zip. This recipe should install the latest Android SDK tools and setup the environment variables in `/etc/profile.d/android-sdk.sh`.  

```bash
wget https://github.com/gildegoma/chef-android-sdk/archive/master.zip
wget https://github.com/burtlo/ark/archive/master.zip
wget https://github.com/sneal/7-zip/archive/master.zip
```

Assuming you install chef-android-sdk, this recipe should install the latest Android SDK tools and setup the environment variables in `/etc/profile.d/android-sdk.sh`.  You will want to insert this statement before running any command-line statements described in the previous section.

```bash
source /etc/profile.d/android-sdk.sh
sudo android update sdk --no-ui --filter platform-tools,tools
```

You should also run this shell script before running any Gradle commands:
```bash
source /etc/profile.d/android-sdk.sh
./gradlew build
```
