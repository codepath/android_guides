## Overview

Espresso is an instrumentation test framework...
 
## Android Studio Setup

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
    ...
    androidTestCompile('com.android.support.test.espresso:espresso-core:2.2') {
        // Necessary if your app targets Marshmallow (since Espresso
        // hasn't moved to Marshmallow yet)
        exclude group: 'com.android.support', module: 'support-annotations'
    }
    androidTestCompile('com.android.support.test:runner:0.3') {
        // Necessary if your app targets Marshmallow (since the test runner
        // hasn't moved to Marshmallow yet)
        exclude group: 'com.android.support', module: 'support-annotations'
    }
}
```
That's all the setup needed. Now let's move on to writing some actual tests.

## Creating a Simple Espresso Test

TODO ...

## Running Espresso Tests

There are 2 ways to run your tests:

1. Run a single test through Android Studio:
  * Right click on the test class and select `Run`:
   
    ![Imgur](http://i.imgur.com/HVntgBc.png)
  * **Note:** If you are presented with two options as in the diagram below, make sure to select the first one (designating to use the Gradle Test Runner instead of the JUnit Test Runner). Android Studio will cache your selection for future runs.
      
    ![Imgur](https://i.imgur.com/RDmmdI2.png)
      
  * View the results in the `Console` output. You may need to enable `Show Passed` as in the diagram below to see the full results.
      
    ![Imgur](http://i.imgur.com/oCHth1N.png)
      
2. Run all the tests through Gradle:
  * Open the Gradle window and find `connectedDebugAndroidTest` under `Tasks` => `verification`.
  * Right click and select `Run`

    ![Imgur](http://i.imgur.com/rqw0GPb.png)

  * This will generate an html test result report at `app/build/reports/androidTests/connected/index.html`
      
    ![Imgur](http://i.imgur.com/m7KYVYX.png)

  * Note: You can also run the tests on the command line using: `./gradlew connectedDebugAndroidTest`

## References

* <https://code.google.com/p/android-test-kit>
