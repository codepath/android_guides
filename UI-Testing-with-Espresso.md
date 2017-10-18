## Overview

Espresso is a UI test framework (part of the [Android Testing Support Library](https://developer.android.com/tools/testing-support-library/index.html)) that allows you to create automated UI tests for your Android app. Espresso tests run on actual device or emulator (they are [instrumentation based tests](http://developer.android.com/tools/testing/testing_android.html#Instrumentation)) and behave as if an actual user is using the app (i.e. if a particular view is off screen, the test won't be able to interact with it). 

Espresso's simple and extensible API, automatic synchronization of test actions with the UI of the app under test, and rich failure information make it a great choice for UI testing. In many circles Espresso is considered to be a full replacement for Robotium (see this [stack overflow post](http://stackoverflow.com/a/20487527/5154829) that compares Robotium to Espresso).

## Android Studio Setup

There are several steps needed to setup Espresso with Android Studio:

1. First, let's change to the `Project` perspective in the `Project Window`. This will show us a full view of everything contained in the project. The default setting (the `Android` perspective) hides certain folders:

    ![Imgur](https://i.imgur.com/OWun9AJ.png)
 
2. Make sure you have an `app/src/androidTest/java` folder. This is the default location for instrumentation tests.

    ![Imgur](https://i.imgur.com/go4sEXZ.png)

3. You'll also need to make sure you have the `Android Support Repository` version 15+ installed.

    ![Imgur](https://i.imgur.com/Rj2aprX.png)

4. It's recommended to turn off system animations on the device or emulator we will be using. Since Espresso is a UI testing framework, system animations can introduce flakiness in our tests. Under `Settings` => `Developer options` disable the following 3 settings and restart the device:

    ![Imgur](https://i.imgur.com/S9gvBnH.png)

5. Finally, we need to pull in the Espresso dependencies and set the test runner in our **_app_** build.gradle:
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

The standard pattern for an Espresso test is to find a view (`ViewMatchers`), do something to that view (`ViewActions`), and then validate some view properties (`ViewAssertions`). There's a handy [cheat sheet](https://google.github.io/android-testing-support-library/downloads/espresso-cheat-sheet-2.1.0.pdf) that's a great reference to see what's available in each of these classes.

## Running Espresso Tests

There are 2 ways to run your tests:

1. Run a single test through Android Studio:
  * Right click on the test class and select `Run`:
   
    ![Imgur](https://i.imgur.com/HVntgBc.png)
  * **Note:** If you are presented with two options as in the diagram below, make sure to select the first one (designating to use the Gradle Test Runner instead of the JUnit Test Runner). Android Studio will cache your selection for future runs.
      
    ![Imgur](https://i.imgur.com/RDmmdI2.png)  
  * View the results in the `Console` output. You may need to enable `Show Passed` as in the diagram below to see the full results.
      
    ![Imgur](https://i.imgur.com/oCHth1N.png)      
2. Run all the tests through Gradle:
  * Open the Gradle window and find `connectedDebugAndroidTest` under `Tasks` => `verification`.
  * Right click and select `Run`

    ![Imgur](https://i.imgur.com/rqw0GPb.png)

  * This will generate an html test result report at `app/build/reports/androidTests/connected/index.html`
      
    ![Imgur](https://i.imgur.com/m7KYVYX.png)
  * Note: You can also run the tests on the command line using: `./gradlew connectedDebugAndroidTest`

## Other Espresso Test Scenarios

### Interacting with a ListView

ListView is an `AdapterView`. AdapterViews present a problem when doing UI testing. Since an AdapterView doesn't load all of it's items upfront (only as the user scrolls through the items), AdapterView's don't work well with `onView(...)` since the particular view might not be part of the view hierarchy yet.

Fortunately, Espresso provides an `onData(...)` entry point that makes sure to load the AdapterView item before performing any operations on it.

Let's run through an example of how you might interact with a ListView in Espresso. Our ListView will be a simple list of text strings.

```java
@Test
public void clickOnItemWithTextEqualToTwo() {
    // Find the adapter position to click based on matching the text "two" to the adapter item's text
    onData(allOf(is(instanceOf(String.class)), is("two"))) // Use Hamcrest matchers to match item
        .inAdapterView(withId(R.id.lvItems)) // Specify the explicit id of the ListView
        .perform(click()); // Standard ViewAction
}
```

Alternately, if we know the position of the particular item, we can directly specify the position instead of using a data `Matcher` to find it:

```java
@Test
public void clickOnItemWithTextEqualToTwo() {
    // Directly specify the position in the adapter to click on
    onData(anything()) // We are using the position so don't need to specify a data matcher
        .inAdapterView(withId(R.id.lvItems)) // Specify the explicit id of the ListView
        .atPosition(1) // Explicitly specify the adapter item to use
        .perform(click()); // Standard ViewAction
}
```

### Interacting with a RecyclerView

Unfortunately, RecyclerView is not an `AdapterView` so we can't use `onData(...)` for a RecyclerView, but Espresso does support RecyclerView in the [`android.support.test.espresso.contrib`](http://developer.android.com/reference/android/support/test/espresso/contrib/package-summary.html) package. 

Let's first pull the package into our **_app_** build.gradle:

```gradle
// build.gradle
dependencies {
    ...
    androidTestCompile('com.android.support.test.espresso:espresso-contrib:2.2') {
        // Necessary to avoid version conflicts
        exclude group: 'com.android.support', module: 'appcompat'
        exclude group: 'com.android.support', module: 'support-v4'
        exclude group: 'com.android.support', module: 'support-annotations'
        exclude module: 'recyclerview-v7'
    }
}
```

Now we can use [RecyclerViewActions](http://developer.android.com/reference/android/support/test/espresso/contrib/RecyclerViewActions.html) to interact with our RecyclerView:

```java
// Click on the RecyclerView item at position 2
onView(withId(R.id.rvItems)).perform(RecyclerViewActions.actionOnItemAtPosition(2, click()));
```

You can see all the supported RecyclerViewActions (including how to interact with views inside of the RecyclerView item) [here](http://developer.android.com/reference/android/support/test/espresso/contrib/RecyclerViewActions.html).

### Stubbing out the Camera

A limitation of Espresso is that it's only able to interact with the single app under test. This means that if your app launches another app, your Espresso test code won't be able to interact with the other app at all. When we think about scenarios where an app will launch the camera (i.e. another app) to take a photo and then return the resulting image to the original app, this becomes problematic.

Fortunately, Espresso provides the [`android.support.test.espresso.intent`](http://developer.android.com/reference/android/support/test/espresso/intent/package-summary.html) package to help us mock out the intent. 

Let's assume we have an activity inside of our app that lets the user take a photo:

```java
// CameraActivity.java
static final int REQUEST_IMAGE_CAPTURE = 1;

// Initiate implicit intent to get a photo
private void dispatchTakePictureIntent() {
    Intent takePictureIntent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
    if (takePictureIntent.resolveActivity(getPackageManager()) != null) {
        startActivityForResult(takePictureIntent, REQUEST_IMAGE_CAPTURE);
    }
}

// Get the returned bitmap
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    if (requestCode == REQUEST_IMAGE_CAPTURE && resultCode == RESULT_OK) {
        Bundle extras = data.getExtras();
        Bitmap imageBitmap = (Bitmap) extras.get("data");

        // ... do something with the bitmap ...
    }
}
```

Now assume that we want to test this activity and have an actual image "returned" to our app from the Camera. This is where [Espresso-Intents](http://developer.android.com/reference/android/support/test/espresso/intent/package-summary.html) can help. If we didn't have Espresso-Intents, our app would launch the Camera, but have no way of getting any resulting image back from it.

To start using the intent package, let's first add it to our **_app_** build.gradle:

```gradle
// build.gradle
dependencies {
    ...
    androidTestCompile('com.android.support.test.espresso:espresso-intents:2.2') {
        // Necessary to avoid version conflicts
        exclude group: 'com.android.support', module: 'support-annotations'
    }
}
```

Then, let's build an actual test that shows how to stub out the Camera intent:

```java
// CameraActivityInstrumentationTest.java
public class CameraActivityInstrumentationTest {

    // IntentsTestRule is an extension of ActivityTestRule. IntentsTestRule sets up Espresso-Intents
    // before each Test is executed to allow stubbing and validation of intents.
    @Rule
    public IntentsTestRule<CameraActivity> intentsRule = new IntentsTestRule<>(CameraActivity.class);

    @Test
    public void validateCameraScenario() {
        // Create a bitmap we can use for our simulated camera image
        Bitmap icon = BitmapFactory.decodeResource(
                InstrumentationRegistry.getTargetContext().getResources(),
                R.mipmap.ic_launcher);

        // Build a result to return from the Camera app
        Intent resultData = new Intent();
        resultData.putExtra("data", icon);
        Instrumentation.ActivityResult result = new Instrumentation.ActivityResult(Activity.RESULT_OK, resultData);

        // Stub out the Camera. When an intent is sent to the Camera, this tells Espresso to respond 
        // with the ActivityResult we just created
        intending(toPackage("com.android.camera2")).respondWith(result);

        // Now that we have the stub in place, click on the button in our app that launches into the Camera
        onView(withId(R.id.btnTakePicture)).perform(click());

        // We can also validate that an intent resolving to the "camera" activity has been sent out by our app
        intended(toPackage("com.android.camera2"));

        // ... additional test steps and validation ...
    }
}
```

## References

* <https://code.google.com/p/android-test-kit>
* <https://developer.android.com/training/testing/ui-testing/espresso-testing.html>
* <https://github.com/vgrec/EspressoExamples>
* <https://github.com/designatednerd/Wino>
* <http://chiuki.github.io/advanced-android-espresso/#/>
* <http://www.vogella.com/tutorials/AndroidTestingEspresso/article.html>
