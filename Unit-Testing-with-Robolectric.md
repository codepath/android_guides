# Overview

Robolectric is a unit testing framework that allows Android applications to be tested on the JVM without an emulator or device.  Running Android tests on the JVM usually fails because the Android core libraries included with the SDK, specifically the `android.jar` file, only contain stub implementations of the Android classes.  The actual implementations of the core libraries are built directly on the device or emulator, so running tests usually requires one to be active in order to execute.

So for all of us that want faster executing tests on the JVM, Robolectric saves the day. Robolectric provides implementations of the Android SDK by rewriting the Android core libraries using [[shadow classes|Unit-Testing-with-Robolectric#using-shadows]]. This gives us the ability to execute our tests on the JVM and achieve much faster test execution times than if we were running on a device or emulator.
 
## Setup

Recent versions of Android Studio have made setup much easier. We'll walk through these steps before we jump into the actual tests.

### Prerequisites
* Android Studio 1.2+
* Android Gradle Plugin 1.2.3+
* Gradle 2.2.1+

Note: Robolectric can also be configured with Android Studio 1.1, but the setup required the [robolectric gradle plugin](https://github.com/robolectric/robolectric-gradle-plugin/) and some additional configuration.  Unit testing in Android Studio has been supported since v1.2, so it is highly recommended you upgrade to a newer version to use Roboelectric.

### Android Studio Configuration
1. The first thing we should do is change to the `Project` perspective in the `Project Window`. This will show us a full view of everything contained in the project. The default setting (the `Android` perspective) hides certain directories (including the unit tests!):

    ![Imgur](https://i.imgur.com/OWun9AJ.png)

2. Next, open the `Build Variants` window and set the `Test Artifact` to `Unit Tests`. Without this, our unit tests won't be included in the build. Note that this is enabled by default in [Android studio version 2+](http://stackoverflow.com/a/35708264/4252667).

    ![Imgur](https://i.imgur.com/sStdOnI.png)
 
3. Creating a new project doesn't automatically create a directory for our unit tests so let's manually add the `src/test/java` directory to the project. This is the default location for unit tests.

    ![Imgur](https://i.imgur.com/zI7xcBx.png)

4. (Mac and Linux users only) => There is a known issue that needs to be addressed so that tests can be located properly. Go to `Run` -> `Edit Configurations` -> `Defaults` -> `Junit` and make sure to set `Working directory:` = `$MODULE_DIR$`.  You can see the Robolectric [getting started guide](http://robolectric.org/getting-started/) for more information. 

    <img src="http://robolectric.org/images/android-studio-configure-defaults-4bf48402.png">

   If you forget to add this statement, you may see error messages such as `java.io.FileNotFoundException: build/intermediates/bundles/debug/AndroidManifest.xml (No such file or directory)`.
5. Then, we just need to pull in Robolectric to our **_app_** build.gradle.

```gradle
dependencies {
    ...
    testCompile 'org.robolectric:robolectric:3.3'
}
```

That's all the setup needed. Now let's move on to writing some actual tests.

## Creating a Simple Robolectric Test

The code below shows a basic Robolectric test that verifies the text inside of a TextView. It's based off the standard new project template which has a single `MainActivity` that contains a `TextView` with the text "Hello world!".

1. Create a new class `MainActivityTest` inside of the unit tests directory (`src/test/java`). The best practice is to mimic the same package structure with your tests as your product code. This has the benefit of giving your tests access to `package-private` fields in your product code. For this example, we'll be creating `MainActivityTest` at `src/test/java/com.codepath.robolectricdemo.MainActivityTest`.
2. The best way to look up a view in the view hierarchy is by using an `id`. Since the "Hello world!" TextView doesn't have an `id`, make sure to give it one before running the test. In the example below, we've called it `tvHelloWorld`.

```java
// MainActivityTest.java

// Static imports for assertion methods
import static junit.framework.Assert.assertNotNull;
import static junit.framework.Assert.assertTrue;

@Config(constants = BuildConfig.class, sdk = Build.VERSION_CODES.LOLLIPOP)
@RunWith(RobolectricTestRunner.class)
public class MainActivityTest {
    private MainActivity activity;
	
    // @Before => JUnit 4 annotation that specifies this method should run before each test is run
    // Useful to do setup for objects that are needed in the test
    @Before
    public void setup() {
        // Convenience method to run MainActivity through the Activity Lifecycle methods:
        // onCreate(...) => onStart() => onPostCreate(...) => onResume()
        activity = Robolectric.setupActivity(MainActivity.class);
    }
	
    // @Test => JUnit 4 annotation specifying this is a test to be run
    // The test simply checks that our TextView exists and has the text "Hello world!"
    @Test
    public void validateTextViewContent() {
        TextView tvHelloWorld = (TextView) activity.findViewById(R.id.tvHelloWorld);
        assertNotNull("TextView could not be found", tvHelloWorld);
        assertTrue("TextView contains incorrect text",
            "Hello world!".equals(tvHelloWorld.getText().toString()));
    }
}
```

Note: Robolectric currently doesn't support API level 22 (Android 5.1).  If your app targets API 22, you will need to specify the `sdk = Build.VERSION_CODES.LOLLIPOP` annotation.

## Running Robolectric Tests

There are 2 ways to run your tests:

1. Run a single test through Android Studio:
  * Right click on the test class and select `Run`:
   
    ![Imgur](https://i.imgur.com/WHmlAaT.png)
  * **Note:** If you are presented with two options as in the diagram below, make sure to select the first one (designating to use the Gradle Test Runner instead of the JUnit Test Runner). Android Studio will cache your selection for future runs.
      
    ![Imgur](https://i.imgur.com/RDmmdI2.png)
      
  * View the results in the `Console` output. You may need to enable `Show Passed` as in the diagram below to see the full results.
      
    ![Imgur](https://i.imgur.com/cv1Aryi.jpg)  
      
2. Run all the tests through Gradle:
  * Open the Gradle window and find `testDebug` under Tasks => verification
  * Right click and select `Run`

    ![Imgur](https://i.imgur.com/6xthcJH.png)

  * This will generate an html test result report at `app/build/reports/tests/debug/index.html`
      
    ![Imgur](https://i.imgur.com/eTBMo82.png)

  * Note: You can also run the tests on the command line using: `./gradlew testDevDebug`

## Using Shadows

Robolectric has an important concept called shadows. Shadows are classes that modify or extend the behavior of classes in the Android SDK.   Most of the Android core views are built with shadow classes to avoid needing the device or an emulator to run.  For a list of all the components that are implicitly mocked when using Roboelectric, see [this link](https://github.com/robolectric/robolectric/tree/master/robolectric-shadows/shadows-core/src/main/java/org/robolectric/shadows).  You can read more about Robolectric's shadows [here](http://robolectric.org/extending/).  

When an Android class is instantiated, Robolectric first looks to see if it has a corresponding shadow class implementation (i.e. a [ShadowTextView](https://github.com/robolectric/robolectric/blob/master/robolectric-shadows/shadows-core/src/main/java/org/robolectric/shadows/ShadowTextView.java) for a TextView), and if it finds one it creates a shadow object to associate with the Android class. Every time a method is invoked on the Android class, Robolectric first invokes the shadow class' corresponding method (if there is one). This gives the shadow classes a chance to maintain and expose extra state that wouldn't be available from just the Android classes. 

### Checking navigation flows

Let's assume our MainActivity has a `Button` that launches a `SecondActivity`. We'd like to validate that clicking on the button launches the correct activity with an automated test.  

**Side Note:** This brings up an important point about Robolectric. Since it's a unit testing framework, a single test only has the capability to work with a particular "unit" (i.e. an activity, a fragment, an adapter, etc). It doesn't have the ability to create integration or end to end tests that span across several activities. If we think about our scenario where we have a button that launches a second activity, Robolectric is only able to validate that the second activity ***would*** have been launched, but not that it is actually launched.

The test below uses shadows to validate that the correct activity is launched when the button is clicked.

```java
@Test
public void secondActivityStartedOnClick() {
    activity.findViewById(R.id.btnLaunchNextActivity).performClick();

    // The intent we expect to be launched when a user clicks on the button
    Intent expectedIntent = new Intent(activity, SecondActivity.class);

    // An Android "Activity" doesn't expose a way to find out about activities it launches
    // Robolectric's "ShadowActivity" keeps track of all launched activities and exposes this information
    // through the "getNextStartedActivity" method.
    ShadowActivity shadowActivity = Shadows.shadowOf(activity);
    Intent actualIntent = shadowActivity.getNextStartedActivity();

    // Determine if two intents are the same for the purposes of intent resolution (filtering).
    // That is, if their action, data, type, class, and categories are the same. This does
    // not compare any extra data included in the intents
    assertTrue(actualIntent.filterEquals(expectedIntent));
}
```

### Custom Shadows

The best way to understand how shadows work is to understand how one is implemented.  Let's use the Bitmap class as an example.  There is an equivalent [ShadowBitmap](https://github.com/robolectric/robolectric/blob/master/robolectric-shadows/shadows-core/src/main/java/org/robolectric/shadows/ShadowBitmap.java) defined in Roboelectric.  Supposed we tried to create a Bitmap image using the `Bitmap.createBitmap`:

```java
  @RunWith(RobolectricTestRunner.class)
  @Config(constants=BuildConfig.class)
  @Test
  public void testBitmapScaling() {
       Bitmap bitmap = Bitmap.createBitmap(200, 200, Bitmap.Config.ARGB_8888);
```

Instead of calling the [actual implementation](https://github.com/android/platform_frameworks_base/blob/master/graphics/java/android/graphics/Bitmap.java#L661-L744), ShadowBitmap's [createBitmap()](https://github.com/robolectric/robolectric/blob/master/robolectric-shadows/shadows-core/src/main/java/org/robolectric/shadows/ShadowBitmap.java#L189-L206) is called instead.  By doing so, the normal routines that depend on Canvas painting are not performed.  

### Other Shadow packages

While Roboelectric provides a base set of shadow classes, there are others that need to be explicitly defined if any of your unit tests depend on them to run.

|Package                             | Shadow equivalent   
|------------------------------------|-------------------------------------
|com.android.support.support-v4      |org.robolectric:shadows-support-v4   
|com.android.support.multidex        |org.robolectric:shadows-multidex     
|com.google.android.gms:play-services|org.robolectric:shadows-play-services
|com.google.android.maps:maps	     |org.robolectric:shadows-maps         
|org.apache.httpcomponents:httpclient|org.robolectric:shadows-httpclient 
  
## Testing the Activity Lifecycle

Dealing with the activity lifecycle is a common source of bugs in Android. Fortunately, Robolectric allows you to test the activity lifecycle. Below you'll see how we've added some activity lifecycle tests to our `MainActivityTest` class.

### Simulating the Full Activity Lifecycle

```java
@Config(constants = BuildConfig.class, sdk = Build.VERSION_CODES.LOLLIPOP)
@RunWith(RobolectricGradleTestRunner.class)
public class MainActivityTest {
    // ActivityController is a Robolectric class that drives the Activity lifecycle
    private ActivityController<MainActivity> controller;
    private MainActivity activity;
	    
    @Before
    public void setUp() {
        // Call the "buildActivity" method so we get an ActivityController which we can use
        // to have more control over the activity lifecycle
        controller = Robolectric.buildActivity(MainActivity.class);
    }
	
    // Activity creation that allows intent extras to be passed in
    private void createWithIntent(String extra) {
        Intent intent = new Intent(RuntimeEnvironment.application, MainActivity.class);
        intent.putExtra("activity_extra", extra);
        activity = controller
                .withIntent(intent)
                .create()
                .start()
                .resume()
                .visible()
                .get();
    }
	
    // Test that simulates the full lifecycle of an activity
    @Test
    public void createsAndDestroysActivity() {
        createWithIntent("my extra_value");

        // ... add assertions ...
    }

    // @After => JUnit 4 annotation that specifies this method should be run after each test
    @After
    public void tearDown() {
        // Destroy activity after every test
        controller
                .pause()
                .stop()
                .destroy();
    }
}
```

### Simulating a Phone Call

```java
@Test
public void pausesAndResumesActivity() {
    createWithIntent("my extra_value");
    controller.pause().resume();

    // ... add assertions ...
}
```

### Simulating Device Rotation

```java
@Test
public void recreatesActivity() {
    Bundle bundle = new Bundle();

    // Destroy the original activity
    controller
        .saveInstanceState(bundle)
        .pause()
        .stop()
        .destroy();

    // Bring up a new activity
    controller = Robolectric.buildActivity(MainActivity.class)
            .create(bundle)
            .start()
            .restoreInstanceState(bundle)
            .resume()
            .visible();
    activity = controller.get();

    // ... add assertions ...
}
```

You can read more about driving the activity lifecycle through Robolectric [here](http://robolectric.org/activity-lifecycle/). 

## Testing Qualified Resources

If you've ever wanted to test something on a tablet or when the language is set to Spanish, Robolectric can help you there. Robolectric has support for specifying resource qualifiers to simulate different configurations for the [alternative resources](http://developer.android.com/guide/topics/resources/providing-resources.html#AlternativeResources) system.

Let's return to the original example where we just have a `TextView` with the text "Hello world!". If we want to validate this is localized properly in Spanish and French, we can annotate each test with the config qualifier and Robolectric will make sure those resources are loaded for our test!

```java
@Test
@Config(qualifiers = "es")
public void localizedSpanishHelloWorld() {
    TextView tvHelloWorld = (TextView)activity.findViewById(R.id.tvHelloWorld);
    assertEquals(tvHelloWorld.getText().toString(), "Hola Mundo!");
}

@Test
@Config(qualifiers = "fr")
public void localizedFrenchHelloWorld() {
    TextView tvHelloWorld = (TextView)activity.findViewById(R.id.tvHelloWorld);
    assertEquals(tvHelloWorld.getText().toString(), "Bonjour le monde!");
}
```

You can read more about Robolectric's support for qualified resources [here](http://robolectric.org/using-qualifiers/).

## Limitations

Roboelectric 3.0 currently does not have the ability to support loading of native libraries when executing tests (noted as an [issue](https://github.com/robolectric/robolectric/issues/1516)).

## References

* <http://robolectric.org>
* <https://github.com/robolectric/robolectric>
* <https://www.bignerdranch.com/blog/triumph-android-studio-1-2-sneaks-in-full-testing-support>
* <https://github.com/mutexkid/android-studio-robolectric-example>
* <http://blog.nikhaldimann.com/2013/10/10/robolectric-2-2-some-pages-from-the-missing-manual>
* <https://corner.squareup.com/2013/04/the-resurrection-of-testing-for-android.html>
* <http://simpleprogrammer.com/2010/07/27/the-best-way-to-unit-test-in-android/>
* <https://youtu.be/f7ihSQ44WO0?t=15m11s>
