### Intro to Gradle

Gradle is a powerful new dependency management system for Java.   It provides the ability to provide more custom build logic than predecessors such as [Maven](http://maven.apache.org) or [Ant](http://ant.apache.org).

Android Studio uses an [Android Gradle plugin](https://developer.android.com/tools/building/plugin-for-gradle.html) that handles most of the integration with Gradle.   This plugin allows you to build different versions of your app (i.e. paid and unpaid, custom builds for different Android devices) while still leveraging the same code base.   It also enables signing your APK and code obfuscation support too, as well as [many other features](https://developer.android.com/tools/building/plugin-for-gradle.html).

### Importing existing Android Studio projects

When importing existing Android Studio projects, you should make sure to select the `build.gradle` in the top-level directory.  If you choose this option, you should see the `Use default gradle wrapper` option exposed and will be able to import the project properly.  If you select on a folder or even the `app/build.gradle` file, you are likely to import a blank project and will need to redo the import.

![](https://i.imgur.com/joPKoTk.gif)

If you are interested in what is the Gradle wrapper and why it's helpful to have, read the section below.

### Intro to the Gradle wrapper

When you setup a project in Android Studio, it automatically generates several files for you that helps to allow people to build your code without needing to install Gradle ahead of time, which is useful especially when needing to do continuous integration testing.

By checking in the gradlew and gradlew.bat files into your source code repository, other people on Unix and Windows platforms do not need to go through the process of manually downloading Gradle or installing Android Studio.

The `gradle-wrapper.properties` file, which is created when you first create an Android Studio project, determines what version of Gradle to use:

```
#Wed Apr 10 15:27:10 PDT 2013
distributionBase=GRADLE_USER_HOME
distributionPath=wrapper/dists
zipStoreBase=GRADLE_USER_HOME
zipStorePath=wrapper/dists
distributionUrl=http\://services.gradle.org/distributions/gradle-4.2-all.zip
```

#### Upgrading Gradle

If you wish to upgrade Gradle versions, you need to update the `distributionUrl` accordingly in `gradle-wrapper.properties`:

```
#Wed Apr 10 15:27:10 PDT 2013
distributionUrl=http\://services.gradle.org/distributions/gradle-4.2-all.zip
```

You can check [this link](https://services.gradle.org/distributions) to find the latest version.  Note that the Gradle releases are independent from the Android Gradle plugin, which is defined in your `build.gradle` config.

Gradle will use this configuration to see if the version has already been installed.  If not, it will be downloaded and stored in a separate directory.  (For Unix machines, the various Gradle downloads will live in ~/.gradle/wrapper.)

In addition, you will need to setup the Android Gradle plugin by setting your `build.gradle` to have a dependency:

```gradle
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:2.3.0'
    }
}
```

If you are using the `android-apt` plugin, Gradle v2.3.0 now supports the `annotationProcessor` syntax natively.  See [here](https://bitbucket.org/hvisser/android-apt/wiki/Migration) for migration details.  Basically the process involves:

* Remove the android-apt plugin from your build scripts
* Change all apt, androidTestApt and testApt dependencies to `annotationProcessor`, `androidTestAnnotationProcessor`, and `testAnnotationProcessor` respectively.

Google maintains the Android Gradle plugin and you can find the latest version at http://tools.android.com/tech-docs/new-build-system.  Both Gradle and the Android Studio plugin are constantly evolving, so you check the site to see what versions of Gradle are supported for which plugin.  

Also, keep in mind that the Android Gradle plugin finds your SDK by what is defined in the local.properties file.  You should already find this local.properties file in your project directory:

```
sdk.dir=/Applications/Android Studio.app/sdk
```

If you don't want to set local.properties, you can also define the ANDROID_HOME environment variable which points to your Android SDK.

```bash
// Unix
export ANDROID_HOME=~/android-sdks

// Windows
set ANDROID_HOME=C:\android-sdks
```

### Analyzing Dependencies with Gradle Wrapper

Once your Gradle wrapper is set up, you can use its `androidDependencies` command to generate a diagram of your app's dependencies on the SDK and support SDK, broken out by each library that you use.

```
$ ./gradlew androidDependencies

:app:androidDependencies
alpha
+--- LOCAL: KISSmetricsSDK.jar
+--- com.android.support:support-v4:23.0.1
+--- com.android.support:recyclerview-v7:23.0.1
|    \--- com.android.support:support-v4:23.0.1
+--- net.danlew:android.joda:2.3.3
+--- com.google.android.gms:play-services-ads:7.5.0
|    +--- com.google.android.gms:play-services-base:7.5.0
|    |    \--- com.android.support:support-v4:23.0.1
|    \--- com.google.android.gms:play-services-analytics:7.5.0
|         \--- com.google.android.gms:play-services-base:7.5.0
|              \--- com.android.support:support-v4:23.0.1
+--- com.facebook.android:facebook:3.20.0
|    \--- com.android.support:support-v4:23.0.1
+--- com.couchbase.lite:couchbase-lite-android:1.0.3.1
+--- com.android.support:appcompat-v7:22.1.1
|    \--- com.android.support:support-v4:23.0.1
+--- com.getbase:floatingactionbutton:1.8.0
\--- com.android.support:multidex:1.0.0
``` 

## Testing Gradle

Finally, you can check your working installation by running:

```
./gradlew -v
```

If you are using Windows, you will be using gradlew.bat instead.

```
./gradlew.bat -v
```

## Using Gradle

To build the APK, run this command in the root directory of your project:

```
./gradlew assemble
```

If a build occurs and you see the output:

```
BUILD SUCCESSFUL

Total time: 5.738 secs
```

Then Gradle is successfully set up for your project. If you get an error, try googling the error. Usually the issue is that you need to [install the build tools](http://stackoverflow.com/questions/16619773/failed-to-import-new-gradle-project-failed-to-find-build-tools-revision-17-0-0).

If you have integration tests you want to run, make sure you have an open emulator or connected device and run this command in the root directory:

```
./gradlew build
```

This will build the apk, then automatically compile and run your integration tests.

If you want to delete old APKs before you re-run either `./gradlew` (build without tests) or `./gradlew build` (build with tests), run:

```
./gradlew clean
```

## Declaring Dependencies

To add dependencies to your project, you need to modify the `build.gradle` file and add extra lines configuring the packages you require. For example, for certain Google or Android, dependencies look like:

```gradle
android {
  ...
  dependencies {
      // Google Play Services
      compile 'com.google.android.gms:play-services:6.5.+'

      // Support Libraries
      compile 'com.android.support:support-v4:22.2.1'
      compile 'com.android.support:appcompat-v7:22.2.+'
      compile 'com.android.support:gridlayout-v7:22.2.+'
      compile 'com.android.support:support-v7-mediarouter:22.2.0+'
      compile 'com.android.support:support-v13:22.2.0+'

      // Note: these libraries require the "Google Repository" and "Android Repository"
      //       to be installed via the SDK manager.
  }
}
```

You can also add dependencies based on the [Maven Central Repository](http://search.maven.org/). The best tool for finding packages is actually the [Gradle Please](http://gradleplease.appspot.com/) utility that takes care of helping you locate the correct package and version to add to your gradle file for any library:

<a href="http://gradleplease.appspot.com#picasso"><img src="https://i.imgur.com/PmeUfhE.png" title="Gradle Please Utility" /></a>

### How to setup the Gradle wrapper

If you are planning to setup your own project without Android Studio (i.e. via Eclipse), the current recommendation is to setup the Gradle wrapper (http://www.gradle.org/docs/current/userguide/gradle_wrapper.html).  The reason is that other people trying to use your project do not need to install Gradle themselves once you've generated the files needed to bootstrap the process.  

To generate this initial set of files (Android Studio will automatically handle this work for you), you need to add these lines to your build.gradle file: 

```gradle
task wrapper(type: Wrapper) {
    gradleVersion = '2.2.1'
}
```

Then run:

```
gradle wrapper 
```

The files below will be generated.  Similarly, Android Studio will create the same directory structure when the "wrapper" option is used :

```
  gradlew
  gradlew.bat
  gradle/wrapper/
    gradle-wrapper.jar
    gradle-wrapper.properties
```

## How to add local dependencies

Normally most dependencies can be retrieved remotely from a Maven or Gradle repository hosted at [Maven Central](http://search.maven.org/) or [BinTray](https://bintray.com/bintray/jcenter).  If you wish to include a local dependency, you can copy the source to the project's root directory.  For instance, suppose you wanted to fork a copy of the [material intro](https://github.com/HeinrichReimer/material-intro) repoistory.  You could copy this source into your main project as follows:

```
Project
  |--build.gradle
  |--settings.gradle
  |--material-intro
  |    |--build.gradle
```

You need to modify the `settings.gradle` file to specify that what proejcts are available.  Note that this new directory prefixed with a ":" operator:

Next, go to your `app/build.gradle` and make sure to specify the `compile project(':material-intro:')` is included:

```gradle
dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    compile project(':material-intro') // add this line
 }
```

For more information, see the documentation about [multi-project builds](https://docs.gradle.org/current/userguide/userguide_single.html#intro_multi_project_builds).

## Speeding up Gradle Builds

There are several configuration tweaks that can be experimented with to significantly speed up gradle build times. In Android Studio, open the `gradle.properties` file in the project and add:

```
#Enable daemon
org.gradle.daemon=true

# Try and findout the best heap size for your project build.
org.gradle.jvmargs=-Xmx2048m -XX:MaxPermSize=512m -XX:+HeapDumpOnOutOfMemoryError -Dfile.encoding=UTF-8

# Modularise your project and enable parallel build
org.gradle.parallel=true

# Enable configure on demand.
org.gradle.configureondemand=true
```

On average, this will cause a 65% decrease in your build times. A few additional resources are included below:

* [How to decrease your gradle build time](https://medium.com/@kevalpatel2106/how-to-decrease-your-gradle-build-time-by-65-310b572b0c43)
* [How I save 5h/week on Gradle builds](https://medium.com/@cesarmcferreira/speeding-up-gradle-builds-619c442113cb)
* [Speeding up Gradle builds](http://kevinpelgrims.com/blog/2015/06/11/speeding-up-your-gradle-builds/)
* [Speed Up Gradle Build In Android Studio](https://medium.com/@101/speed-up-gradle-build-in-android-studio-80a5f74ac9ed)

## Customizing Android Builds

Gradle provides enormous flexibility in enabling you to customize your builds for paid/unpaid versions, debug/production releases, and use different signing keys.  Take a look at [this guide](http://toastdroid.com/2014/03/28/customizing-your-build-with-gradle/) for more information.

## Resources

Check out the following links for more details:

 * [Automating Android Builds with Gradle](http://paulemtz.blogspot.com/2013/04/automating-android-builds-with-gradle.html)
 * [Building with Gradle](http://www.vogella.com/articles/AndroidBuild/article.html)
 * [Gradle User Guide](http://tools.android.com/tech-docs/new-build-system/user-guide)
 * [Gradle Example Project](https://github.com/pestrada/android-tdd-playground)
 * [Building Android Projects with Gradle](https://spring.io/guides/gs/gradle-android/) This resource covers creating a Gradle wrapper for use with Android Studio.
 * [Gradle wrapper](http://www.gradle.org/docs/current/userguide/gradle_wrapper.html) More background information of the Gradle wrapper.