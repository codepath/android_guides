## Overview

Google provides an [Android Testing framework](http://developer.android.com/tools/testing/testing_android.html) that is part of the Android SDK and is built on top of standard JUnit testing extended with a instrumentation framework and Android-specific testing classes.

Note: **You must be running at least version 1.1.0 of the Android plug-in for Gradle, since unit testing with Android Studio was only recently supported.**  More information can be found [here](http://tools.android.com/tech-docs/unit-testing-support).  

```gradle
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:1.3.1'
    }
}
```

Tests in Android are organized into their own projects. A test project is a directory in which you create a test application that contains unit test source files that run on top of the Android SDK.  In a default Gradle project, the source code for your app is typically located in the `app/src/main/java` folder.  The source code for your local unit tests would then be placed under a `app/src/test/java/folder`.  See the [setup instructions](http://developer.android.com/training/testing/unit-testing/local-unit-tests.html#setup) from the official Android documentations for more information.

The Android testing API also provides hooks into the Android component and application life cycle. These hooks are called the instrumentation API and allow your tests to control the life cycle events. The Android instrumentation API allows you to run the test project and the normal Android project in the same process so that the test project can call methods of the Android project directly.  These tests should be placed in `app/src/androidTest/java/folder` instead of `app/src/test/java/folder`.

#### Creating a Test Project

Let's try testing a very simple application called `SimpleApp`. This app is just two activities. The first (`FirstActivity`) has a text field and a button. When you type in the text field and hit the button, a `SecondActivity` appears that displays the text entered.

<img src="https://i.imgur.com/BhD9S8n.png" width="430" alt="Screen 1" />
&nbsp;
<img src="https://i.imgur.com/YOssiuC.png" width="430" alt="Screen 2" />

Let's take a look at how to test this very simple application using the Android testing framework. First, let's create a new Test Project called "SimpleAppTest".

![Test Project](https://i.imgur.com/7APmagr.png)

Select the "Test Target" as SimpleApp. Now we have our brand new test project for writing out app test cases. If we want to embed the test project within our existing application within a folder, check out [this step-by-step guide](https://web.archive.org/web/20140117014928/http://jonblack.org/2012/11/24/creating-an-android-test-project-within-a-project/). 

#### Unit Testing

Unit testing is about testing a particular component (i.e activity) in isolation of other components. These tests tend to be simpler and faster than integration tests. 

Now, let's test the basic behavior of our application. Create a new test class called `FirstActivityUnitTest` extending from superclass `android.test.ActivityUnitTestCase`. The overall structure of a `ActivityUnitTestCase` is:

```java
// Import the resource object constant from our app for easy access
import com.codepath.example.simpleapp.R;

public class FirstActivityUnitTest extends
    android.test.ActivityUnitTestCase<FirstActivity> {

  private FirstActivity activity;

  public FirstActivityUnitTest() {
    super(FirstActivity.class);
  }

  @Override
  protected void setUp() throws Exception {
    super.setUp();
    Intent intent = new Intent(getInstrumentation().getTargetContext(),
        FirstActivity.class);
    startActivity(intent, null, null);
    activity = getActivity();
  }

  @SmallTest
  public void testSomething() {
    // assertions here
  }

  @Override
  protected void tearDown() throws Exception {
    super.tearDown();
  }
}
```

Now, we want to test the basic functions of the first activity. Namely, that I can enter a text value into the field and then that value will be properly passed to the `SecondActivity` through an intent. Let's add those basic tests:

```java
public class FirstActivityUnitTest extends
    android.test.ActivityUnitTestCase<FirstActivity> {

  // ...

  // Sanity check for the layout
  @SmallTest
  public void testLayoutExists() {
    // Verifies the button and text field exist
    assertNotNull(activity.findViewById(R.id.btnLaunch));
    assertNotNull(activity.findViewById(R.id.etResult));
    // Verifies the text of the button
    Button view = (Button) activity.findViewById(R.id.btnLaunch);
    assertEquals("Incorrect label of the button", "Launch", view.getText());
  }

  // Validate the intent is fired on button press with correct result from
  // text field
  @SmallTest
  public void testIntentTriggerViaOnClick() {
    String fieldValue = "Testing Text";
    // Set a value into the text field
    EditText etResult = (EditText) activity.findViewById(R.id.etResult);
    etResult.setText(fieldValue);
    // Verify button exists on screen
    Button btnLaunch = (Button) activity.findViewById(R.id.btnLaunch);
    assertNotNull("Button should not be null", btnLaunch);
    // Trigger a click on the button
    btnLaunch.performClick();
    // Verify the intent was started with correct result extra
    Intent triggeredIntent = getStartedActivityIntent();
    assertNotNull("Intent should have triggered after button press",
        triggeredIntent);
    String data = triggeredIntent.getExtras().getString("result");
    assertEquals("Incorrect result data passed via the intent",
        "Testing Text", data);
  }
}
```

You may notice the use of the `@SmallTest` annotation which is used to indicate the "scale" of each test method and categorize them into groups. This allows us to run only certain groups of tests instead of the entire suite to allow for faster targeted test executions. Google has a [blog post](http://googletesting.blogspot.com/2010/12/test-sizes.html) explaining the rules of thumb for organizing tests:

[![Size table](https://i.imgur.com/dwvkQI2.png)](http://googletesting.blogspot.com/2010/12/test-sizes.html)

The full source code for that file can be [found here](https://gist.github.com/nesquena/7f38c84891abe7528991). This is the basic structure of an Activity unit test. 

![Run Tests](https://i.imgur.com/k6LjpW9.png)

Now, in Eclipse we can right-click and select "Run As..." and then select "Android JUnit Test" and the tests will execute within the test runner. The tests should both pass and the "test bar" should be green.

#### Functional Testing

Functional testing is about testing a series of components (i.e activity) and their interactions with one another. These tests tend to be more complex and slower than unit tests.

For completeness, let's also check out how to write functional tests. The last test was a simple "unit test" that checked the first activity. Let's now write an integration test that will test the flow of both activities and check the text is properly displayed on the second activity. Create a new test class called `SimpleActivityFunctionalTest` extending from superclass `android.test.ActivityInstrumentationTestCase2`. The overall structure of a `SimpleActivityFunctionalTest` is:


```java
// Import the resource object constant from our app for easy access
import com.codepath.example.simpleapp.R;

public class SimpleActivityFunctionalTest extends
    ActivityInstrumentationTestCase2<FirstActivity> {

  private FirstActivity activity;

  public SimpleActivityFunctionalTest() {
    super(FirstActivity.class);
  }

  @Override
  protected void setUp() throws Exception {
    super.setUp();
    setActivityInitialTouchMode(false);
    activity = getActivity();
  }

  @SmallTest
  public void testStartSecondActivity() throws Exception {
      // Assertions in here
  }

  @Override
  protected void tearDown() throws Exception {
    super.tearDown();
  }
}
```
Let's add a simple end-to-end test for passing data into the TextView of the second activity:

```java
public class SimpleActivityFunctionalTest extends
    ActivityInstrumentationTestCase2<FirstActivity> {
  
  // ...
  
  @MediumTest
  public void testStartSecondActivity() throws Exception {		
      final String fieldValue = "Testing Text";
	  
      // Set a value into the text field
      final EditText etResult = (EditText) activity.findViewById(R.id.etResult);
      activity.runOnUiThread(new Runnable() {
          @Override
          public void run() {
               etResult.setText(fieldValue);
          }
      });
      
      // Add monitor to check for the second activity
      ActivityMonitor monitor = getInstrumentation().addMonitor(
          SecondActivity.class.getName(), null, false);
      
      // find button and click it
      Button btnLaunch = (Button) activity.findViewById(R.id.btnLaunch);
      TouchUtils.clickView(this, btnLaunch);
      
      // Wait 2 seconds for the start of the activity
      SecondActivity secondActivity = (SecondActivity) monitor
          .waitForActivityWithTimeout(2000);
      assertNotNull(secondActivity);
      
      // Search for the textView
      TextView textView = (TextView) secondActivity
          .findViewById(R.id.tvResult);
      
      // check that the TextView is on the screen
      ViewAsserts.assertOnScreen(secondActivity.getWindow().getDecorView(),
          textView);
      
      // Validate the text on the TextView
      assertEquals("Text should be the field value", fieldValue, textView
          .getText().toString());
      
      // Press back to return to original activity
      // We have to manage the initial position within the emulator
      this.sendKeys(KeyEvent.KEYCODE_BACK);
  }
	
  // ...
}
```

The full functional test source code can be [found here](https://gist.github.com/nesquena/0c5be91e4cd7698cf3be). At this point we have tested the functionality of our basic application and we have explored both unit testing and functional testing approaches using the built-in Android Testing Framework.

![Run Tests](https://i.imgur.com/k6LjpW9.png)

Now, in Eclipse we can right-click and select "Run As..." and then select "Android JUnit Test" and the tests will execute within the test runner. The tests should both pass and the "test bar" should be green.

#### More Details

You can check out the [Testing API](http://developer.android.com/tools/testing/testing_android.html#TestAPI) on the official docs. To understand how to test activities in particular, check out the [Activity Testing Guide](http://developer.android.com/tools/testing/activity_testing.html) including the `ActivityInstrumentationTestCase2` class. You can also read about how to do testing [within the Eclipse IDE](http://developer.android.com/tools/testing/testing_eclipse.html).

## References

* <http://developer.android.com/tools/testing/index.html>
* <http://developer.android.com/tools/testing/testing_android.html>
* <http://developer.android.com/tools/testing/testing_eclipse.html>
* <http://developer.android.com/tools/testing/activity_testing.html>
* <http://www.vogella.com/articles/AndroidTesting/article.html>
* <http://corner.squareup.com/2013/04/the-resurrection-of-testing-for-android.html>
* <http://www.guru99.com/why-android-testing.html>