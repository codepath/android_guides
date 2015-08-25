## Overview

Espresso is an instrumentation test framework...
 
## Setup

### Prerequisites
* Android Studio 1.2+
* Android Gradle Plugin 1.2.3+
* Gradle 2.2.1+

Note: Espresso can also be configured with Android Studio 1.1, but the setup requires some additional configuration. Keep in mind that unit testing was still considered an [experimental feature](http://tools.android.com/tech-docs/unit-testing-support) in Android Studio 1.1.

### Android Studio Configuration
1. The first thing we should do is change to the `Project` perspective in the `Project Window`. This will show us a full view of everything contained in the project. The default setting (the `Android` perspective) hides certain folders:

    ![Imgur](https://i.imgur.com/OWun9AJ.png)

2. Next, open the `Build Variants` window and make sure the `Test Artifact` is set to `Android Instrumentation Tests`. Without this, our tests won't be included in the build.

    ![Imgur](http://i.imgur.com/twnVGE1.png)
 
3. Make sure you have an `app/src/androidTest/java` folder. This is the default location for instrumentation tests.

    ![Imgur](http://i.imgur.com/go4sEXZ.png)

4. You'll also need to make sure you have the `Android Support Repository` version 15+ installed.

    ![Imgur](http://i.imgur.com/Rj2aprX.png)

5. It's recommended to turn off system animations on the device or emulator we will be using. Since Espresso is a UI testing framework, system animations can introduce flakiness in our tests. Under `Settings` => `Developer options` disable the following 3 settings and restart the device:

    ![Imgur](http://i.imgur.com/S9gvBnH.png)

6. Finally, we need to pull in the Espresso dependencies and set the test runner in our **_app_** build.gradle:
```gradle
// build.gradle
...
android {
    ...
    defaultConfig {
        ...
        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
    }
}

dependencies {
    androidTestCompile 'com.android.support.test.espresso:espresso-core:2.2'
    androidTestCompile 'com.android.support.test:runner:0.3'
}
```
That's all the setup needed. Now let's move on to writing some actual tests.

## Creating a Simple Espresso Test

TODO ...

## Running Espresso Tests

TODO ...

## References

* <https://code.google.com/p/android-test-kit>
