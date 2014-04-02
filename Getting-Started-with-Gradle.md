Getting started with Gradle on the command-line is easy!

## Install Gradle

Download Gradle (see specific links below) and then extract the Gradle distribution to a folder, which we will call GRADLE_HOME. Add GRADLE_HOME/bin to your PATH environment variable.

### For Android Studio

Download **Gradle 1.8** from the [Gradle web site](http://services.gradle.org/distributions/gradle-1.8-bin.zip). At the moment, Android Studio v0.3.0 requires 1.8 now.

### For Command Line with Eclipse

Download **Gradle 1.6** from the [Gradle web site](http://services.gradle.org/distributions/gradle-1.6-bin.zip). At the moment, only Gradle 1.6 works correctly **with Eclipse**. Using Gradle 1.7+ will fail with a cryptic error.

## Define Environment Variables

Define the ANDROID_HOME environment variable which points to your Android SDK.

```bash
// Unix
export ANDROID_HOME=~/android-sdks

// Windows
set ANDROID_HOME=C:\android-sdks
```

Afterwards, you have to configure the GRADLE_HOME environment variable on your path:

```bash
export GRADLE_HOME=/your_gradle_directory
export PATH=$PATH:$GRADLE_HOME/bin
```

Finally, if you are experimenting with different versions of Gradle, remember to source your path definition file (`source ~/.bash_profile` or wherever your PATH was defined) so that $PATH points to the correct Gradle binary and the old one is no longer in your $PATH. Run `echo $PATH` to confirm.

## Install API 17 and Build Tools

In order for Gradle to work, ensure you have the API 17 SDK installed including the **latest Android SDK Platform-tools and Android SDK Build-tools**. Check this in the Android SDK Manager from within Eclipse. 

## Testing Gradle

Finally, you can check your working installation by running:

```
gradle -v
```

##  Gradle Project Setup

To use Gradle in your Android application, you have to create a `build.gradle` file in your application root directory:

```bash
buildscript {
  repositories {
    mavenCentral()
  }

  dependencies {
    classpath 'com.android.tools.build:gradle:0.4'
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

Within the `buildscript` block, the `classpath` definition should also be set to `com.android.tools.build:gradle:0.4+`. This works with Gradle 1.6. Higher classpath values require a later Gradle version.

## Using Gradle

To build the APK, run this command in the root directory:

```
gradle assemble
```

If a build occurs and you see the output:

```
BUILD SUCCESSFUL

Total time: 5.738 secs
```

Then Gradle is successfully set up for your project. If you get an error, try googling the error. Usually the issue is that you need to [install the build tools](http://stackoverflow.com/questions/16619773/failed-to-import-new-gradle-project-failed-to-find-build-tools-revision-17-0-0) or make sure to use Gradle 1.6.

If you have integration tests you want to run, make sure you have an open emulator or connected device and run this command in the root directory:

```
gradle build
```

This will build the apk, then automatically compile and run your integration tests.

If you want to delete old APKs before you re-run either `gradle assemble` (build without tests) or `gradle build` (build with tests), run:

```
gradle clean
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

Creating a build file that includes ActionBarSherlock in your project can be particularly confusing because ActionBarSherlock imports its own version of the Android support package. If you are using com.android.support classes in your own code (for fragments, tab navigation, and so on), and you have a `com.android.support` jar in your `/libs`, you are likely to run into compiler errors when you start to include ABS. 

Here is a basic `build.gradle` file that overcomes this conflict.

Before you create the buildfile, install the Android Support Repository from the SDK Manager. This is under SDK Manager->Extras->Android Support Repository. (See [http://stackoverflow.com/questions/18559660/android-gradle-build-fails-could-not-find-com-google-androidsupport-v4r18](madhead's StackOverflow answer)). At the time of writing, I also had installed Android Support Library for API 19, but simply having the most recent Android Support Library jar will not fix the error.

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

Note how `repositories` is specified with two different values. This seems to be necessary to overcome a missing library on the new mavenCentral() target; if the older Maven URL is omitted, the aar package for ActionBarSherlock will fail to build.

## Resources

Check out the following links for more details:

 * [Automating Android Builds with Gradle](http://paulemtz.blogspot.com/2013/04/automating-android-builds-with-gradle.html)
 * [Building with Gradle](http://www.vogella.com/articles/AndroidBuild/article.html)
 * [Gradle User Guide](http://tools.android.com/tech-docs/new-build-system/user-guide)
 * [Gradle Example Project](https://github.com/pestrada/android-tdd-playground)