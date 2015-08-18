# Overview

This framework is designed to provide black box tests for Android applications. This means that you test for expected outcomes instead of specific methods. It does require the application to be running in an emulator or device. Robotium **builds off of the core Android integration testing libraries** but provides an "extra" layer on top to make our testing easier with the `Solo` driver. Through the solo object, you can set values in input fields, click on buttons and get results from other UI components. Methods of JUnits Assert class can then be used to check those results.

## Robotium Example

Let's take a look at writing blackbox integration tests with Robotium. First, you need to setup a "Test Project" much the same way as you would for the Android Testing Framework.

Next, we need to add the [robotium jar](https://code.google.com/p/robotium/downloads/list) to our Test Project. Download the latest "robotium-solo-X.X.jar". You need to add the robotium JAR to the Libraries on the projects Build Path.

![Build Path](https://i.imgur.com/ghMdX4W.png)

Now, within the integration test case, in the setUp method of your testing class, you can create a Solo object and specify the Activity to be started. The Solo object can then be used in your testXXX methods to get and set values in/from UI components and click buttons and such. A skeleton integration testing class with Robotium looks like:

```java
// Import the resource object constant from our app for easy access
import com.codepath.example.simpleapp.R;

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
  
  @SmallTest
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
@MediumTest
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

  // Return to the original activity
  // We have to manage the initial position within the emulator
  solo.goBack();
}
```

The full source for this test can be [found here](https://gist.github.com/nesquena/631575e209e2cd165719). Compare this with the official Android testing and notice how much cleaner and clearer tests are with Robotium. 

![Run Tests](https://i.imgur.com/k6LjpW9.png)

Now, in Eclipse we can right-click and select "Run As..." and then select "Android JUnit Test" and the tests will execute within the test runner. The tests should both pass and the "test bar" should be green.

## Solo Actions

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

For more details, make sure to check the [Robotium documentation](https://code.google.com/p/robotium/wiki/Getting_Started). 

## References

* <http://www.vogella.com/articles/Robotium/article.html>
* <http://jayaruwanmannapperuma.wordpress.com/2012/03/01/robotium-for-android/>
* <https://code.google.com/p/robotium/wiki/Getting_Started>
* <http://www.guru99.com/why-android-testing.html>