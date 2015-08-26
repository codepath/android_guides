## Overview

Espresso is a UI test framework (part of the [Android Testing Support Library](https://developer.android.com/tools/testing-support-library/index.html)) that allows you to create automated UI tests for your Android app. Espresso tests run on actual device or emulator (they are [instrumentation based tests](http://developer.android.com/tools/testing/testing_android.html#Instrumentation)) and behave as if an actual user is using the app (i.e. if a particular view is off screen, the test won't be able to interact with it). 

Espresso's simple and extensible API, automatic synchronization of test actions with the UI of the app under test, and rich failure information make it a great choice for UI testing. In many circles Espresso is considered to be a full replacement for Robotium (see this [stack overflow post](http://stackoverflow.com/a/20487527/5154829) that compares Robotium to Espresso).

## Android Studio Setup

There are several steps needed to setup Espresso with Android Studio:

1. First, let's change to the `Project` perspective in the `Project Window`. This will show us a full view of everything contained in the project. The default setting (the `Android` perspective) hides certain folders:

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

The code below shows a simple Espresso test that enters some text into an EditText and then verifies the text entered. It's based off the standard new project template which has a single `MainActivity` that contains a `TextView` with the text "Hello world!".

1. Add an `EditText` to `MainActivity` that has `id` = `R.id.etInput`.
2. Create a new class `MainActivityInstrumentationTest` inside of the default instrumentation tests directory (`src/androidTest/java`). The best practice is to mimic the same package structure with your tests as your product code. This has the benefit of giving your tests access to `package-private` fields in your product code. For this example, we'll be creating `MainActivityInstrumentationTest` at `src/androidTest/java/com.codepath.espressodemo.MainActivityInstrumentationTest`.

```java
// MainActivityInstrumentationTest.java

import static android.support.test.espresso.Espresso.onView;
import static android.support.test.espresso.action.ViewActions.typeText;
import static android.support.test.espresso.assertion.ViewAssertions.matches;
import static android.support.test.espresso.matcher.ViewMatchers.withId;
import static android.support.test.espresso.matcher.ViewMatchers.withText;

// Tests for MainActivity
public class MainActivityInstrumentationTest {

    // Preferred JUnit 4 mechanism of specifying the activity to be launched before each test
    @Rule
    public ActivityTestRule<MainActivity> activityTestRule =
            new ActivityTestRule<>(MainActivity.class);

    // Looks for an EditText with id = "R.id.etInput"
    // Types the text "Hello" into the EditText
    // Verifies the EditText has text "Hello"
    @Test
    public void validateEditText() {
        onView(withId(R.id.etInput)).perform(typeText("Hello")).check(matches(withText("Hello")));
    }
}
```

When writing Espresso tests, you'll be using a lot of static imports. This makes the code easier to read, but can make it more difficult to understand when you are first learning Espresso. Let's dive into the parts of the above test:

* [Espresso](http://developer.android.com/reference/android/support/test/espresso/Espresso.html) – This is the entry point. Mostly of the time you'll be using `onView(...)` to specify you want to interact with a view.
* [ViewMatchers](http://developer.android.com/reference/android/support/test/espresso/matcher/ViewMatchers.html) – This is how we find views. `ViewMatchers` contains a collection of [hamcrest matchers](https://code.google.com/p/hamcrest/wiki/Tutorial) that allow you to find specific views in your view hierarchy. Above, we've used `withId(R.id.etInput)` to specify we are looking for an `EditText` with `id` = `R.id.etInput`.
* [ViewActions](http://developer.android.com/reference/android/support/test/espresso/action/ViewActions.html) – This is how we interact with views. Above we've used the `typeText(...)` method to type `Hello` into our `EditText`.
* [ViewAssertions](http://developer.android.com/reference/android/support/test/espresso/assertion/ViewAssertions.html) – This is our validation. We use `ViewAssertions` to validate specific properties of views. Most of the time you'll be using `ViewAssertions` that are powered by `ViewMatchers` underneath. In our example above the `withText(...)` method is actually returning a `ViewMatcher` which we've converted into a `ViewAssertion` using the `matches(...)` method.

The standard pattern for an Espresso test is to find a view (`ViewMatchers`), do something to that view (`ViewActions`), and then validate some view properties (`ViewAssertions`). There's a handy [cheat sheet] (https://github.com/googlesamples/android-testing/blob/master/downloads/Espresso%20Cheat%20Sheet%20Master.pdf) that's a great reference to see what's available in each of these classes.

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
* <https://developer.android.com/training/testing/ui-testing/espresso-testing.html>