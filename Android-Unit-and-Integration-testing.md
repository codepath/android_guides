## Overview

Automated Testing is an important topic that helps us ensure quality when building Android apps. There many different testing tools and frameworks we can use while developing Android apps. This guide will take a look at the most popular approaches available.

## Usage

There are three common approaches used for automated Android testing:

 * [Google Android Testing](http://developer.android.com/tools/testing/testing_android.html) - This is the framework included as part of the platform. ([Jump](https://github.com/thecodepath/android_guides/wiki/Android-Unit-and-Integration-testing#android-testing-framework))
 * [Robotium](https://code.google.com/p/robotium/) - Black box integration testing for Android ([Jump](https://github.com/thecodepath/android_guides/wiki/Android-Unit-and-Integration-testing#robotium))
 * [Robolectric](http://robolectric.org/index.html) - Unit tests that run outside the emulator making the tests very fast ([Jump](https://github.com/thecodepath/android_guides/wiki/Android-Unit-and-Integration-testing#robolectric))
 
The first question many developers have is which tools to use in their applications. As everyone will tell you, this depends on your circumstance. However, I think for many apps the right combination is **using both Robolectric and Robotium**.

### Testing Comparisons

Emulator/instrumentation-based tests (including Robotium) are painfully slow but follow the Android API to the letter (except when they randomly fail, which is more often than you'd like).

Robolectric works well for most common testing needs (and dramatically speeds up testing cycles) but does not work for every case and only supports unit tests. Robotium is great for testing the edges of the API that aren't yet simulated by Robolectric, and for testing actual phone behavior more rigorously through complete integration testing.

The primary takeaway is **use Robolectric unit tests as much as possible** since they are the fastest testing suite. When tests fall outside the unit testing scope or the features of Robolectric then we need to rely on **Robotium for complete integration testing**.

### Android Testing Framework

#### Introduction

Before we move on to using those popular third-party libraries, let's take a quick look at the [official Android Testing framework](http://developer.android.com/tools/testing/testing_android.html). This framework is baked into the Android SDK and is built on top of standard JUnit testing extended with a instrumentation framework and Android-specific testing classes. 

Tests in Android are organized into their own projects. A test project is a directory in which you create a test application that contains unit test source files that run ontop of the Android SDK. Often the test project is added as a "tests" folder at the root level of your Android app. 

The Android testing API provides hooks into the Android component and application life cycle. These hooks are called the instrumentation API and allow your tests to control the life cycle events. The Android instrumentation API allows you to run the test project and the normal Android project in the same process so that the test project can call methods of the Android project directly.

#### Unit and Functional Examples

Let's try testing a very simple application called [SimpleApp](https://github.com/thecodepath/android-simple-app). This app is just two activities. The first (`FirstActivity`) has a text field and a button. When you type in the text field and hit the button, a `SecondActivity` appears that displays the text entered.

<img src="http://i.imgur.com/BhD9S8n.png" width="430" alt="Screen 1" />
&nbsp;
<img src="http://i.imgur.com/YOssiuC.png" width="430" alt="Screen 2" />

Let's take a look at how to test this very simple application using the Android testing framework. Make sure to [download SimpleApp](https://github.com/thecodepath/android-simple-app) and import the project into your workspace. First, let's create a new Test Project called "SimpleAppTest".

![Test Project](http://i.imgur.com/7APmagr.png)

Select the "Test Target" as SimpleApp. Now we have our brand new test project for writing out app test cases. Next, let's test the basic behavior of our application. Create a new test class called `FirstActivityUnitTest` extending from superclass `android.test.ActivityUnitTestCase`. The overall structure of a `ActivityUnitTestCase` is:

```java
public class FirstActivityUnitTest extends
    android.test.ActivityUnitTestCase<FirstActivity> {

  private FirstActivity activity;

  public SomeActivityUnitTest() {
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
import com.codepath.example.simpleapp.R;

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

The full source code for that file can be [found here](https://gist.github.com/nesquena/7f38c84891abe7528991). This is the basic structure of an Activity unit test. Now, in Eclipse we can right-click and select "Run As..." and then select "Android JUnit Test" and the tests will execute within the test runner. The tests should both pass and the "test bar" should be green.

For completeness, let's also check out how to write functional tests. The last test was a simple "unit test" that checked the first activity. Let's now write an integration test that will test the flow of both activities and check the text is properly displayed on the second activity. Create a new test class called `SimpleActivityFunctionalTest` extending from superclass `android.test.ActivityInstrumentationTestCase2`. The overall structure of a `SimpleActivityFunctionalTest` is:


```java
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
      
      // Press back and click again
      this.sendKeys(KeyEvent.KEYCODE_BACK);
  }
	
  // ...
}
```

The full functional test source code can be [found here](https://gist.github.com/nesquena/0c5be91e4cd7698cf3be). At this point we have tested the functionality of our basic application and we have explored both unit testing and functional testing approaches using the built-in Android Testing Framework.

#### More Details

You can check out the [Testing API](http://developer.android.com/tools/testing/testing_android.html#TestAPI) on the official docs. To understand how to test activities in particular, check out the [Activity Testing Guide](http://developer.android.com/tools/testing/activity_testing.html) including the `ActivityInstrumentationTestCase2` class. You can also read about how to do testing [within the Eclipse IDE](http://developer.android.com/tools/testing/testing_eclipse.html).

### Robotium

This framework is designed to provide black box tests for Android applications. This means that you test for expected outcomes instead of specific methods. It does require the application to be running in an emulator or device. Robotium **builds off of the core Android integration testing libraries** but provides an "extra" layer on top to make our testing easier with the `Solo` driver. Through the solo object, you can set values in input fields, click on buttons and get results from other UI components. Methods of JUnits Assert class can then be used to check those results.

#### Robotium Example

Let's take a look at writing blackbox integration tests with Robotium for our [SimpleApp](https://github.com/thecodepath/android-simple-app). First, you need to setup a "Test Project" much the same way as you would for the Android Testing Framework above. So check the previous "example" section above and generate a "Test Project" called "SimpleAppTest". Be sure to download the [SimpleApp](https://github.com/thecodepath/android-simple-app) and read what the app does in the previous section testing section as well.

Next, we need to add the [robotium jar](https://code.google.com/p/robotium/downloads/list) to our Test Project. Download the latest "robotium-solo-X.X.jar". You need to add the robotium JAR to the Libraries on the projects Build Path.

![Build Path](http://i.imgur.com/ghMdX4W.png)

Now, within the integration test case, in the setUp method of your testing class, you can create a Solo object and specify the Activity to be started. The Solo object can then be used in your testXXX methods to get and set values in/from UI components and click buttons and such. A skeleton integration testing class with Robotium looks like:

```java
public class RobotiumActivityFunctionalTest extends
    ActivityInstrumentationTestCase2<FirstActivity> {
  // Solo Robotium helper object
  private Solo solo;

  public RobotiumActivityFunctionalTest() {
    super(FirstActivity.class);
  }

  // Setup test case with solo
  @Override
  protected void setUp() throws Exception {
    solo = new Solo(getInstrumentation(), getActivity());
  }

  public void testSomething() {
    // Test something here
  }

  // Finalize solo object
  @Override
  public void tearDown() throws Exception {
    try {
      solo.finalize();
    } catch (Throwable e) {
      e.printStackTrace();
    }
    getActivity().finish();
    super.tearDown();
  }
}
```

Now, let's write the tests for the basic behavior of our simple app which is to allow the user to enter text, hit the "Launch" button and then view that text in a second activity:

```java
public void testStartSecondActivity() throws Exception {		
  final String fieldValue = "Testing Text";
  
  // Set a value into the text field
  solo.enterText((EditText) solo.getView(R.id.etResult), fieldValue);
  
  // find button and click it
  solo.clickOnButton("Launch");
  // or solo.clickOnView(solo.getView(R.id.btnLaunch));

  // Wait 2 seconds for the start of the activity
  solo.waitForActivity(SecondActivity.class, 2000);
  solo.assertCurrentActivity("Should be second activity", SecondActivity.class);
  // Search for the textView
  TextView textView = (TextView) solo.getView(R.id.tvResult);
  
  // Validate the text on the TextView
  assertEquals("Text should be the field value", fieldValue, 
      textView.getText().toString());
}
```

The full source for this test can be [found here](https://gist.github.com/nesquena/631575e209e2cd165719). Compare this with the official Android testing and notice how much cleaner and clearer tests are with Robotium. 

#### Solo Actions

There are many actions which you can take during tests with Robotium via the `Solo` object including these which are the most common:

| Method            | Description                                                           |
| -------------     |-------------                                                          |
| getView(int id)   | Searches for the view with the specified ID in the current activity   |
| assertCurrentActivity(text, Activity.class) | Ensure that the current activity equals the second parameter  |   
| getCurrentActivity() | Searches for the current activity and returns      |
| waitForText(text) | waits for a text on the screen, default timeout 5 secs |
| clickOnButton(text) | clicks on a button with the "text" text | 
| clickOnText(text) | Search for text in the current user interface and clicks on it |
| enterText(editText, text) | Enters text in the specified EditText |
| searchText(text) | Searches for a text in the current user interface, return true if found |
| searchButton(text) | Searches for a button with the text in the current user interface |
| goBack() | Presses the back button |
| takeScreenshot() | saves a screenshot on the device |
| waitForActivity(SecondActivity.class, 2000) | Waits for an Activity matching the specified name. |
| waitForText(text, 2000) | Waits for the specified text to appear. |
| waitForView(R.id.view) | Waits for a View matching the specified resource id. |

For more details, make sure to check the [Solo documentation](http://robotium.googlecode.com/svn/doc/com/jayway/android/robotium/solo/Solo.html) to see which actions you can take using Robotium. 

### Robolectric

Robolectric is a framework which mocks part of the Android framework contained in the android.jar file and which allows you to run Android tests directly on the JVM with the JUnit 4 framework.

Robolectric is designed to allow you to test Android applications on the JVM. This enables you to run your Android tests in your continuous integration environment without any additional setup and without an emulator running. Because of this Robolectric is **really fast** in comparison to any other testing approach.

Unfortunately, Robolectric doesn't support every situation so when you cannot achieve the test using this approach, fall back to a full Robotium integration test for more complex situations or multi-activity scenarios. 

#### Robolectric Example

Let's take a look at testing [SimpleApp](https://github.com/thecodepath/android-simple-app) with Robolectric. Keep in mind that Robolectric is unique in that the tests **do not require an emulator** to be running. First, we need to setup our project for Robolectric tests using our [step-by-step Robolectric installation guide](https://github.com/thecodepath/android_guides/wiki/Robolectric-Installation-for-Unit-Testing).

With the Robolectric test project setup, we can now create a unit test for our Activity by creating a new class in the "test" folder of our `SimpleAppElectricTest` called "FirstActivityTest". The rough skeleton of an activity test with Robolectric is:

```java
import static org.junit.Assert.fail;

import org.junit.After;
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.robolectric.RobolectricTestRunner;

import com.codepath.example.simpleapp.R;
import com.codepath.example.simpleapp.FirstActivity;


@RunWith(RobolectricTestRunner.class)
public class FirstActivityTest {

    @Before
    public void setUp() throws Exception {
    }

    @After
    public void tearDown() throws Exception {
    }

    @Test
    public void testSomething() {
        fail("not implemented");
    }

}
```
Now, we can write ourselves unit tests for a particular Activity using the Robolectric test file and "shadow" objects. We want to verify that the correct text is passed as the result into the intent when the launch button is pressed:

```java
import static org.hamcrest.CoreMatchers.*;
import static org.junit.Assert.*;
import static org.robolectric.Robolectric.shadowOf;

import org.junit.After;
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.robolectric.Robolectric;
import org.robolectric.RobolectricTestRunner;

import android.content.Intent;
import android.widget.Button;
import android.widget.EditText;

import com.codepath.example.simpleapp.FirstActivity;
import com.codepath.example.simpleapp.R;
import com.codepath.example.simpleapp.SecondActivity;

@RunWith(RobolectricTestRunner.class)
public class FirstActivityTest {
  private Button btnLaunch;
  private EditText etResult;
  private FirstActivity activity;

  @Before
  public void setup() throws Exception{
    activity = Robolectric.buildActivity(FirstActivity.class).create().visible().get();
    btnLaunch = (Button) activity.findViewById(R.id.btnLaunch);
    etResult = (EditText) activity.findViewById(R.id.etResult);
  }

  // Sanity check for the layout
  @Test
  public void shouldHaveButtonThatSaysAudit() throws Exception{
    // Verifies the button and text field exist
    assertThat(btnLaunch, notNullValue());
    assertNotNull(etResult);
    // Verifies the text of the button
    assertThat((String) btnLaunch.getText(), equalTo("Launch"));
  }

  // Validate the intent is fired on button press with correct result
  @Test
  public void pressingLaunchButtonForSecondActivity() throws Exception {
    String resultValue = "Testing Text";
    // Set a value into the text field
    etResult.setText(resultValue);
    // Verify button exists on screen
    assertNotNull(btnLaunch);
    // Trigger a click on the button
    btnLaunch.performClick();
    // Construct Shadow versions of Activity and Intent
    Intent startedIntent = shadowOf(activity).getNextStartedActivity();
    // Verify the intent was started with correct result extra
    assertThat(resultValue, equalTo(startedIntent.getStringExtra("result")));
    assertThat(startedIntent.getComponent().getClassName(), equalTo(SecondActivity.class.getName()));
  }

  @After
  public void tearDown() throws Exception {
    // Nothing Here
  }

}
```

Now, if we right-click and select "Run As...JUnit Test", the tests should pass green because the behavior for our SimpleApp is correct. Check out a richer set of examples in the [Robolectric Sample](https://github.com/robolectric/RobolectricSample/tree/master/src/test/java/com/pivotallabs) code.

#### Shadow Objects

The Robolectric testing framework provides "shadow objects" that override certain aspects of the Android SDK. These objects allow the code being tested to execute outside of the Android environment. At times, it's useful to manipulate or access these shadow objects to achieve some expected result.

```java
ShadowActivity shadowActivity = shadowOf(activity);
Intent startedIntent = shadowActivity.getNextStartedActivity();
ShadowIntent shadowIntent = shadowOf(startedIntent);
```

We can get the "shadow" of almost any Android object for richer testing. For the full documentation of Shadow objects and more, check out the [Robolectric JavaDocs](http://robolectric.org/javadoc/index.html?overview-summary.html).

#### Hamcrest Core Matchers

Instead of the typical JUnit assertions such as `assertEqual`, we can also use the "hamcrest" matchers to leverage `assertThat` with certain "core matchers" such as:

```java
// Not Null
assertThat(btnLaunch, notNullValue());
// Equals
assertThat((String) btnLaunch.getText(), equalTo("Launch"));
// All Conditions
assertThat("myValue", allOf(startsWith("my"), containsString("Val")))
// Any Conditions
assertThat("myValue", anyOf(startsWith("foo"), containsString("Val")))
// Both
assertThat("fab", both(containsString("a")).and(containsString("b")))
// Kind Of
assertThat(activity, isA(FirstActivity.class))
// Array Inclusion
assertThat(Arrays.asList("foo", "bar"), hasItem("bar"))
// IsNull
assertThat(activity, is(nullValue())
```

Check out a full list of core matchers for use with `assertThat` in the [hamcrest docs](http://hamcrest.org/JavaHamcrest/javadoc/1.3/org/hamcrest/CoreMatchers.html).

#### Related Libraries

In addition to Robolectric, you may also want to check out the following related libraries:

 * [Spoon](http://square.github.io/spoon/) - Distributes instrumentation test execution and displays results from all devices.
 * [Fest-Android](http://square.github.io/fest-android/) - A set of FEST assertions geared toward testing Android.

Check [this excellent square blog post](http://corner.squareup.com/2013/04/the-resurrection-of-testing-for-android.html) for an overview of the use of these tools. This [Big Nerd Ranch Testing](http://blog.bignerdranch.com/2583-testing-the-android-way/) post covers these same libraries as well. Robolectric combined with Fest-Android makes for an optimal unit testing syntax.

## Libraries

* [Robolectric](http://robolectric.org/index.html)
* [Robotium](https://code.google.com/p/robotium/)

## References

 * <http://developer.android.com/tools/testing/index.html>
 * <http://developer.android.com/tools/testing/testing_android.html>
 * <http://developer.android.com/tools/testing/testing_eclipse.html>
 * <http://developer.android.com/tools/testing/activity_testing.html>
 * <http://www.vogella.com/articles/AndroidTesting/article.html>
 * <http://corner.squareup.com/2013/04/the-resurrection-of-testing-for-android.html>
 * <https://code.google.com/p/robotium/wiki/Getting_Started>
 * <http://www.guru99.com/why-android-testing.html>
 * <http://blog.bignerdranch.com/2583-testing-the-android-way/>
 * <http://pivotal.github.io/robolectric/>
 * <http://corner.squareup.com/2013/05/robolectric-two-point-oh.html>
 * <http://www.techrepublic.com/blog/android-app-builder/unit-testing-android-apps-is-easy-with-robolectric/>
 * <https://code.google.com/p/android-mock/wiki/UsingAndroidMock>
 * <http://www.vogella.com/articles/Robotium/article.html>
 * <http://jayaruwanmannapperuma.wordpress.com/2012/03/01/robotium-for-android/>