### For Android Studio

At the moment, Android Studio v0.5.8 requires gradle 1.10 (no lower, no higher).

You do not need to download Gradle manually for Android Studio.  You just need to make sure that you have selected the "Use default gradle wrapper" option for your project.

Android Studio should handle most of the setup work, but if you are interested in how things work, read the section below.

### Using with Eclipse

Eclipse does not have the same type of integration, so you will have to run Gradle from the command-line.  You will also need to download **Gradle 1.6** from the [Gradle web site](http://services.gradle.org/distributions/gradle-1.6-bin.zip). At the moment, only Gradle 1.6 works correctly **with Eclipse**. Using Gradle 1.7+ will fail with a cryptic error.

### How to setup the Gradle wrapper

The current recommendation is to setup the Gradle wrapper (http://www.gradle.org/docs/current/userguide/gradle_wrapper.html).  The reason is that other people trying to use your project do not need to install Gradle themselves once you've generated the files needed to bootstrap the process.  

To generate this initial set of files (Android Studio will automatically do so when you select the wrapper option), you need to add these lines to your build.gradle file: 

```bash
task wrapper(type: Wrapper) {
    gradleVersion = '1.4'
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

The gradle-wrapper.properties file determines what version of Gradle to use:

```
#Wed Apr 10 15:27:10 PDT 2013
distributionBase=GRADLE_USER_HOME
distributionPath=wrapper/dists
zipStoreBase=GRADLE_USER_HOME
zipStorePath=wrapper/dists
distributionUrl=http\://services.gradle.org/distributions/gradle-1.12-all.zip
```

Gradle will use this configuration to see if the version has already been installed.  If not, it will be downloaded and stored in a separate directory.  (For Unix machines, the various Gradle downloads will live in ~/.gradle/wrapper.)

In addition, you will need to setup the Android Gradle plugin by setting your build.gradle to have a dependency:

```bash
buildscript {
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:0.12.+'
    }
}
```

Google maintains this Android plugin and you can find the latest updates at http://tools.android.com/tech-docs/new-build-system.  Both Gradle and the Android Studio plugin are constantly evolving, so you check the site to see what versions of Gradle are supported for which plugin.

Also, keep in mind that the Android Gradle plugin finds your SDK by what is defined in the local.properties file.  You will need to create this local.properties file in your project directory:

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

## Install API 17 and Build Tools

In order for Gradle to work, ensure you have the API 17 SDK installed including the **latest Android SDK Platform-tools and Android SDK Build-tools**. Check this in the Android SDK Manager from within Eclipse. 

## Testing Gradle

Finally, you can check your working installation by running:

```
./gradlew -v
```

If you are using Windows, you will be using gradlew.bat instead.

```
./gradlew -v
```

##  Gradle Project Setup

To use Gradle in your Android application, you have to create a `build.gradle` file in your application root directory:

```bash
buildscript {
  repositories {
    mavenCentral()
  }

  dependencies {
    classpath 'com.android.tools.build:gradle:0.12+'
  }
}

apply plugin: 'android'

android {
  compileSdkVersion 17
  buildToolsVersion '17'

  sourceSets {
    main {
      manifest.srcFile 'AndroidManifest.xml'
      java.srcDirs = ['src']
      resources.srcDirs = ['src']
      renderscript.srcDirs = ['src']
      res.srcDirs = ['res']
      assets.srcDirs = ['assets']
    }

    instrumentTest.setRoot('tests')
    instrumentTest {
      java.srcDirs = ['tests/src']
      res.srcDirs = ['tests/res']
      assets.srcDirs = ['tests/assets']
      resources.srcDirs = ['tests/src']
    }
  }

  dependencies {
    compile fileTree(dir: 'libs', include: '*.jar')
  }
}
```

Note that with Gradle 1.6 as downloaded above, the **SDK version must be specified as 17**. Changing to 18 at this time may break the Gradle build. 

Within the `buildscript` block, the Gradle `classpath` definition should also be set to `com.android.tools.build:gradle:0.4+`. This version works with Gradle 1.6. Higher values require a later Gradle version.

## Using Gradle

To build the APK, run this command in the root directory of your project:

```
gradlew assemble
```

If a build occurs and you see the output:

```
BUILD SUCCESSFUL

Total time: 5.738 secs
```

Then Gradle is successfully set up for your project. If you get an error, try googling the error. Usually the issue is that you need to [install the build tools](http://stackoverflow.com/questions/16619773/failed-to-import-new-gradle-project-failed-to-find-build-tools-revision-17-0-0) or make sure to use Gradle 1.6.

If you have integration tests you want to run, make sure you have an open emulator or connected device and run this command in the root directory:

```
gradlew build
```

This will build the apk, then automatically compile and run your integration tests.

If you want to delete old APKs before you re-run either `gradle assemble` (build without tests) or `gradle build` (build with tests), run:

```
gradlew clean
```

## Declaring Dependencies

To add dependencies to your project, you need to modify the `build.gradle` file and add extra lines configuring the packages you require. For example, for certain Google or Android, dependencies look like:

```bash
android {
  ...
  dependencies {
      // Google Play Services
      compile 'com.google.android.gms:play-services:4.0.30'

      // Support Libraries
      compile 'com.android.support:support-v4:19.0.0'
      compile 'com.android.support:appcompat-v7:19.0.0'
      compile 'com.android.support:gridlayout-v7:19.0.0'
      compile 'com.android.support:support-v7-mediarouter:19.0.0'
      compile 'com.android.support:support-v13:19.0.0'

      // Note: these libraries require the "Google Repository" and "Android Repository"
      //       to be installed via the SDK manager.
  }
}
```

You can also add dependencies based on the [Maven Central Repository](http://search.maven.org/). The best tool for finding packages is actually the [Gradle Please](http://gradleplease.appspot.com/) utility that takes care of helping you locate the correct package and version to add to your gradle file for any library:

<a href="http://gradleplease.appspot.com"><img src="http://i.imgur.com/MT7TbPg.png" title="Gradle Please Utility" /></a>


### Integrating Gradle, ActionBarSherlock, and the Android Support Libraries

Creating a build file that includes ActionBarSherlock in your project can be challenging because ActionBarSherlock imports its own version of the Android support package. If you are using com.android.support classes in your own code (for fragments, tab navigation, and so on), you may need to debug a `Jar mismatch!` compiler error when you start to include ABS. 

Here is a basic `build.gradle` file that overcomes this conflict.

Before you create the build file, install the Android Support Repository from the SDK Manager. This is under SDK Manager->Extras->Android Support Repository. (See [madhead's StackOverflow answer.](http://stackoverflow.com/questions/18559660/android-gradle-build-fails-could-not-find-com-google-androidsupport-v4r18)) At the time of writing, I also had installed Android Support Library for API 19, but simply having a recent Android Support Library jar will not fix the error.

Okay. Now, create `build.gradle` in your Android project:

```
buildscript {
    repositories {
        maven { url 'http://repo1.maven.org/maven2' }
    }

    dependencies {
        classpath 'com.android.tools.build:gradle:0.4+'
    }
}

apply plugin: 'android'

repositories{
    mavenCentral()
}

dependencies {
    compile 'com.actionbarsherlock:actionbarsherlock:4.4.0@aar'
    compile 'com.android.support:support-v4:13.0.+'
}

android {
    compileSdkVersion 17
    buildToolsVersion '17'

    dependencies {
        compile fileTree(dir: 'libs', include: '*.jar')
    }

    sourceSets {
        main {
            manifest.srcFile 'AndroidManifest.xml'
            java.srcDirs = ['src']
            resources.srcDirs = ['src']
            aidl.srcDirs = ['src']
            renderscript.srcDirs = ['src']
            res.srcDirs = ['res']
            assets.srcDirs = ['assets']
        }
    }
}
```

Note how `repositories` is specified with two different values. This seems to be necessary to overcome a missing library on the new mavenCentral() target; if the other Maven URL is omitted, the repository package for ActionBarSherlock will fail to build.

### Loading Dependencies with Gradle and Eclipse

Eclipse does not have great support for dependency loading via Gradle. However, scripts can be written that make this possible. Check out [the solution found on this site](http://www.nodeclipse.org/projects/gradle/android) for a working way to import dependencies into Eclipse. You can also check out [this gist](https://gist.github.com/nesquena/9e261582e55620d5fbd8). 

## Resources

Check out the following links for more details:

 * [Automating Android Builds with Gradle](http://paulemtz.blogspot.com/2013/04/automating-android-builds-with-gradle.html)
 * [Building with Gradle](http://www.vogella.com/articles/AndroidBuild/article.html)
 * [Gradle User Guide](http://tools.android.com/tech-docs/new-build-system/user-guide)
 * [Gradle Example Project](https://github.com/pestrada/android-tdd-playground)
 * [Building Android Projects with Gradle](https://spring.io/guides/gs/gradle-android/) This resource covers creating a Gradle wrapper for use with Android Studio.
 * [Gradle wrapper](http://www.gradle.org/docs/current/userguide/gradle_wrapper.html) More background information of the Gradle wrapper.