# Robolectric Android Studio Setup

Robolectric is a framework that allows Android applications to be tested on the JVM using JUnit 4 (no emulator or device needed!). More info about robolectric is available [[here|Android-Unit-and-Integration-testing#robolectric]].
  
In this guide, we'll walk through the steps necessary to add Robolectric to an existing project.

## Prerequisites:
* Android Studio 1.2.1.1+
* Android Gradle Plugin 1.2.3+
* Gradle 2.2.1+

Note: Robolectric can also be configured with Android Studio 1.1, but the setup requires the [robolectric gradle plugin](https://github.com/robolectric/robolectric-gradle-plugin/) and some additional configuration. Also, keep in mind that unit testing was still considered an [experimental feature](http://tools.android.com/tech-docs/unit-testing-support) in Android Studio 1.1.

## App build.gradle
First, we need to add the following to the **_app_** build.gradle. Robolectric's latest version is currently `3.0-rc3`.  

  ```gradle
  dependencies {
    ...
    testCompile('org.robolectric:robolectric:3.0-rc3')
  }
   ``` 
## Android Studio Configuration
Next, there are a couple things we need to configure in Android Studio for running unit tests. One very important setting is the `Test Artifact` setting that toggles between `Unit Tests` and `Android Instrumentation Tests`. This setting controls the visibility of the corresponding tests in the project browser.

  <img src="https://camo.githubusercontent.com/cbf79d740e265cc9da9299c2b5f29fc8a63613e7/68747470733a2f2f7777772e657665726e6f74652e636f6d2f73686172642f733331332f73682f35363063346235662d653730622d343830302d623436662d6263313936383631383333382f38396331653734306537313334333136393631613130333032316461663163622f646565702f302f4d794163746976697479546573742e6a6176612d2d2d616e64726f69642d73747564696f2d726f626f6c6563747269632d6578616d706c652d2d2d2d2d2d636f64652d616e64726f69642d73747564696f2d726f626f6c6563747269632d6578616d706c652d2e706e67"/>

For each type of test, Android Studio defaults to looking for tests in the following locations:
* Unit Tests => `src/test/java`
* Android Instrumentation Tests => `src/androidTest/java`

Since we will be creating unit tests, we need our robolectric tests to live in `src/test/java`.  The easiest way to create this folder (if instrumentations tests aren't also needed) is to rename the `androidTest` folder to `test` like so:

![image](http://i.imgur.com/EqGf1UQ.png)

Note: If you are using a Mac, there is a known issue that needs to be addressed so that tests can be located properly. Go to `Run` -> `Edit Configurations` -> `Defaults` -> `Junit` and make sure to set `$MODULE_DIR$` as the working directory.  You can see the Robolectric [getting started guide](http://robolectric.org/getting-started/) for more information. 

<img src="http://robolectric.org/images/android-studio-configure-defaults-4bf48402.png">

That's all the setup needed. Now we can move on to writing a test.

## Creating a Robolectric test

The code below shows a basic Robolectric test that verifies the expected text inside of a TextView. To enable this test:

1. Make sure you have a test file named `MainActivityTest.java` (you can rename `ApplicationTest.java` to `MainActivityTest.java` if you'd like)
2. Make sure your app has an Activity named `MainActivity`
3. Add a TextView to `MainActivity` named `tvHello` containing the text "Hello world!"
4. Copy the code below into `MainActivityTest.java`

```java
import android.widget.TextView;

import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.robolectric.Robolectric;
import org.robolectric.RobolectricGradleTestRunner;
import org.robolectric.annotation.Config;

import static org.junit.Assert.assertNotNull;
import static org.junit.Assert.assertTrue;

@Config(constants = BuildConfig.class, sdk = 21)
@RunWith(RobolectricGradleTestRunner.class)
public class MainActivityTest {
    private MainActivity activity;

    @Before
    public void setup() throws Exception {
        activity = Robolectric.setupActivity(MainActivity.class);
        assertNotNull("MainActivity is not instantiated", activity);
    }

    @Test
    public void validateTextViewContent() throws Exception {
        TextView tvHelloWorld = (TextView) activity.findViewById(R.id.tvHello);
        assertNotNull("TextView could not be found", tvHelloWorld);
        assertTrue("TextView contains incorrect text",
                "Hello world!".equals(tvHelloWorld.getText().toString()));
    }
}
```

Note: Robolectric currently doesn't support API level 22 (Android 5.1).  If your app targets API 22, you will need to specify the `sdk = 21` annotation.

## Running tests

1. Android Studio - Make sure you run the test as a Gradle test (the first item).  The first item is for a Gradle test, while the 2nd item is for JUnit.

![](http://i.imgur.com/RDmmdI2.png)

If all goes well, you will see the passing results in the console. Note: you may need to enable `Show Passed` as in the diagram to see the full results.

![image](http://i.imgur.com/cv1Aryi.jpg)

To run tests on the command line `./gradlew test`

## References

* <http://robolectric.org/>
* <https://github.com/robolectric/robolectric/>
* <https://www.bignerdranch.com/blog/triumph-android-studio-1-2-sneaks-in-full-testing-support/>
* <https://github.com/mutexkid/android-studio-robolectric-example>