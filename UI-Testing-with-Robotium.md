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

##Robotium in Google's Espresso way

Old example uses [`ActivityInstrumentationTestCase2`]([https://developer.android.com/reference/android/test/ActivityInstrumentationTestCase2.html]), which is already marked as deprecated, so it won't be maintained any more. Android Reference says clearly that

> `**This class was deprecated in API level 24.**
>
> Use `ActivityTestRule` instead. New tests should be written using the `Android Testing Support Library`.

Another reason to change `ActivityInstrumentationTestCase2` into `ActivityTestRule` is a generated boilerplate code used for initialization and finishing tests. Futhermore, let's think for a moment what if instead of 

    @MediumTest
    public void testStartSecondActivity() throws Exception {
         solo.waitForActivity("SecondActivity", 2000); 
         ...
we could write:

    @Test(timeout = 2000)
    public void startMainActivityIsProperlyDisplayed() throws InterruptedException {
        solo.waitForActivity("MainActivity", 1000);
        ...

Before we start our journey with using `ActivityTestRule` with `Robotium` testing we need to need to change our existing configuration. 

For better experience create a new Android `Empty Project` and follows these steps:

####Add [`Android Testing Support Library`](https://developer.android.com/tools/testing-support-library/index.html) dependencies to project's `build.gradle` file:

        androidTestCompile 'com.android.support.test:runner:0.4.1'
        androidTestCompile 'com.android.support.test:rules:0.4.1'
        androidTestCompile 'com.android.support:support-annotations:24.1.1'
        androidTestCompile 'com.jayway.android.robotium:robotium-solo:5.6.1'

**Don't forget about defining your `testInstrumentationRunner` runner! **
**Otherwise your tests won't run. **
Just add this line:

      testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"

Finally your `build.gradle file` should look like below:

        apply plugin: 'com.android.application'
    
        android {
                compileSdkVersion 24
                buildToolsVersion "24.0.1"
    
        defaultConfig {
              applicationId "com.example.piotr.myapplication"
              minSdkVersion 15
              targetSdkVersion 21
              versionCode 1
              versionName "1.0"
              testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
          }
    
           buildTypes {
              release {
                  minifyEnabled false
                  proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
              }
    
           }
        }
    
        dependencies {
               compile fileTree(dir: 'libs', include: ['*.jar'])
               testCompile 'junit:junit:4.12'
               compile 'com.android.support:appcompat-v7:24.1.1'
               compile 'com.android.support:design:24.1.1'
               androidTestCompile 'com.android.support.test:runner:0.4.1'
               androidTestCompile 'com.android.support.test:rules:0.4.1'
               androidTestCompile 'com.android.support:support-annotations:24.1.1'
               compile 'com.jayway.android.robotium:robotium-solo:5.6.1'    
        }

####Define your own `ActivityTestRule`

`Android Testing Support Library` supports creating your own test rules for tests. 

For this example, just create in `androidTest` folder new Java class and call it `MyActivityTestRule`. Then put this code into it:


        @Beta
        public class MyActivityTestRule<T extends Activity> extends UiThreadTestRule {
    
        private static final String TAG = "ActivityInstrumentationRule";
    
        private final Class<T> mActivityClass;
    
        public Instrumentation getInstrumentation() {
            return mInstrumentation;
        }
    
        private Instrumentation mInstrumentation;
    
        private boolean mInitialTouchMode = false;
    
        private boolean mLaunchActivity = false;
    
        private T mActivity;
    
        /**
         * Similar to {@link #MyActivityTestRule(Class, boolean, boolean)} but with "touch mode" disabled.
         *
         * @param activityClass    The activity under test. This must be a class in the instrumentation
         *                         targetPackage specified in the AndroidManifest.xml
         * @see MyActivityTestRule#MyActivityTestRule(Class, boolean, boolean)
         */
        public MyActivityTestRule(Class<T> activityClass) {
            this(activityClass, false);
        }
    
        /**
         * Similar to {@link #MyActivityTestRule(Class, boolean, boolean)} but defaults to launch the
         * activity under test once per
         * <a href="http://junit.org/javadoc/latest/org/junit/Test.html"><code>Test</code></a> method.
         * It is launched before the first
         * <a href="http://junit.sourceforge.net/javadoc/org/junit/Before.html"><code>Before</code></a>
         * method, and terminated after the last
         * <a href="http://junit.sourceforge.net/javadoc/org/junit/After.html"><code>After</code></a>
         * method.
         *
         * @param activityClass    The activity under test. This must be a class in the instrumentation
         *                         targetPackage specified in the AndroidManifest.xml
         * @param initialTouchMode true if the Activity should be placed into "touch mode" when started
         * @see MyActivityTestRule#MyActivityTestRule(Class, boolean, boolean)
         */
        public MyActivityTestRule(Class<T> activityClass, boolean initialTouchMode) {
            this(activityClass, initialTouchMode, true);
        }
    
        /**
         * Creates an {@link MyActivityTestRule} for the Activity under test.
         *
         * @param activityClass    The activity under test. This must be a class in the instrumentation
         *                         targetPackage specified in the AndroidManifest.xml
         * @param initialTouchMode true if the Activity should be placed into "touch mode" when started
         * @param launchActivity   true if the Activity should be launched once per
         *                         <a href="http://junit.org/javadoc/latest/org/junit/Test.html">
         *                         <code>Test</code></a> method. It will be launched before the first
         *                         <a href="http://junit.sourceforge.net/javadoc/org/junit/Before.html">
         *                         <code>Before</code></a> method, and terminated after the last
         *                         <a href="http://junit.sourceforge.net/javadoc/org/junit/After.html">
         *                         <code>After</code></a> method.
         */
        public MyActivityTestRule(Class<T> activityClass, boolean initialTouchMode,
                                  boolean launchActivity) {
            mActivityClass = activityClass;
            mInitialTouchMode = initialTouchMode;
            mLaunchActivity = launchActivity;
            mInstrumentation = InstrumentationRegistry.getInstrumentation();
        }
    
        /**
         * Override this method to set up Intent as if supplied to
         * {@link android.content.Context#startActivity}.
         * <p>
         * The default Intent (if this method returns null or is not overwritten) is:
         * action = {@link Intent#ACTION_MAIN}
         * flags = {@link Intent#FLAG_ACTIVITY_NEW_TASK}
         * All other intent fields are null or empty.
         *
         * @return The Intent as if supplied to {@link android.content.Context#startActivity}.
         */
        protected Intent getActivityIntent() {
            return new Intent(Intent.ACTION_MAIN);
        }
    
        /**
         * Override this method to execute any code that should run before your {@link Activity} is
         * created and launched.
         * This method is called before each test method, including any method annotated with
         * <a href="http://junit.sourceforge.net/javadoc/org/junit/Before.html"><code>Before</code></a>.
         */
        protected void beforeActivityLaunched() {
            // empty by default
        }
    
        /**
         * Override this method to execute any code that should run after your {@link Activity} is
         * launched, but before any test code is run including any method annotated with
         * <a href="http://junit.sourceforge.net/javadoc/org/junit/Before.html"><code>Before</code></a>.
         * <p>
         * Prefer
         * <a href="http://junit.sourceforge.net/javadoc/org/junit/Before.html"><code>Before</code></a>
         * over this method. This method should usually not be overwritten directly in tests and only be
         * used by subclasses of MyActivityTestRule to get notified when the activity is created and
         * visible but test runs.
         */
        protected void afterActivityLaunched() {
            // empty by default
        }
    
        /**
         * Override this method to execute any code that should run after your {@link Activity} is
         * finished.
         * This method is called after each test method, including any method annotated with
         * <a href="http://junit.sourceforge.net/javadoc/org/junit/After.html"><code>After</code></a>.
         */
        protected void afterActivityFinished() {
            // empty by default
        }
    
        /**
         * @return The activity under test.
         */
        public T getActivity() {
            if (mActivity == null) {
                Log.w(TAG, "Activity wasn't created yet");
            }
            return mActivity;
        }
    
        @Override
        public Statement apply(final Statement base, Description description) {
            return new ActivityStatement(super.apply(base, description));
        }
    
        /**
         * Launches the Activity under test.
         * <p>
         * Don't call this method directly, unless you explicitly requested not to launch the Activity
         * manually using the launchActivity flag in
         * {@link MyActivityTestRule#MyActivityTestRule(Class, boolean, boolean)}.
         * <p>
         * Usage:
         * <pre>
         *    &#064;Test
         *    public void customIntentToStartActivity() {
         *        Intent intent = new Intent(Intent.ACTION_PICK);
         *        mActivity = mActivityRule.launchActivity(intent);
         *    }
         * </pre>
         * @param startIntent The Intent that will be used to start the Activity under test. If
         *                    {@code startIntent} is null, the Intent returned by
         *                    {@link MyActivityTestRule#getActivityIntent()} is used.
         * @return the Activity launched by this rule.
         * @see MyActivityTestRule#getActivityIntent()
         */
        public T launchActivity(@Nullable Intent startIntent) {
            // set initial touch mode
            mInstrumentation.setInTouchMode(mInitialTouchMode);
    
            final String targetPackage = mInstrumentation.getTargetContext().getPackageName();
            // inject custom intent, if provided
            if (null == startIntent) {
                startIntent = getActivityIntent();
                if (null == startIntent) {
                    Log.w(TAG, "getActivityIntent() returned null using default: " +
                            "Intent(Intent.ACTION_MAIN)");
                    startIntent = new Intent(Intent.ACTION_MAIN);
                }
            }
            startIntent.setClassName(targetPackage, mActivityClass.getName());
            startIntent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
            Log.d(TAG, String.format("Launching activity %s",
                    mActivityClass.getName()));
    
            beforeActivityLaunched();
            // The following cast is correct because the activity we're creating is of the same type as
            // the one passed in
            mActivity = mActivityClass.cast(mInstrumentation.startActivitySync(startIntent));
    
            mInstrumentation.waitForIdleSync();
    
            afterActivityLaunched();
            return mActivity;
        }
    
        // Visible for testing
        void setInstrumentation(Instrumentation instrumentation) {
            mInstrumentation = checkNotNull(instrumentation, "instrumentation cannot be null!");
        }
    
        void finishActivity() {
            if (mActivity != null) {
                mActivity.finish();
                mActivity = null;
            }
        }
    
        /**
         * <a href="http://junit.org/apidocs/org/junit/runners/model/Statement.html">
         * <code>Statement</code></a> that finishes the activity after the test was executed
         */
        private class ActivityStatement extends Statement {
    
            private final Statement mBase;
    
            public ActivityStatement(Statement base) {
                mBase = base;
            }
    
            @Override
            public void evaluate() throws Throwable {
                try {
                    if (mLaunchActivity) {
                        mActivity = launchActivity(getActivityIntent());
                    }
                    mBase.evaluate();
                } finally {
                    finishActivity();
                    afterActivityFinished();
                }
            }
        }
    }

This file is a standard `ActivityTestRule` with some additional getters, which we will use later in our test class.

> **NOTE:** Whole code of this is available here: https://github.com/piotrek1543/robotium-showcase

####Create your first automation testing file

Now we have all needed configuration to start test coding using `Robotium`. 

Just create a new Java class and write the code below:

        @RunWith(AndroidJUnit4.class) 
        public class MainActivityTest {

        private Solo solo;
    
        private static final String MAIN_ACTIVITY = MainActivity.class.getSimpleName();
    
        @Rule
        public MyActivityTestRule<MainActivity> mActivityRule = new MyActivityTestRule<>(MainActivity.class);
    
        @Before
        public void setUp() throws Exception {
            solo = new Solo(mActivityRule.getInstrumentation(), mActivityRule.getActivity());
        }
    
        @Test(timeout = 5000)
        public void checkIfMainActivityIsProperlyDisplayed() throws Exception {
            solo.waitForActivity("MainActivity", 2000);
            solo.assertCurrentActivity(mActivityRule.getActivity().getString(
                    R.string.error_no_class_def_found, MAIN_ACTIVITY), MAIN_ACTIVITY);
            solo.getText("Hello World").isShown();
    
          }
        }
    
To run tests, **right-click** on the name of class or method and select `Run `[nameOfTest]``

As you may notice there are some new annotations to remember:
  *  `@RunWith()` - it describes which runner we would use in our test class.
  *  `@Rule` - it defines where your actual `TestRule` is defined
  *  `@Test` - thanks to this annotation, you can name your test method as you would like to, no more `testSomethingToDo`

## References

* <http://www.vogella.com/articles/Robotium/article.html>
* <http://jayaruwanmannapperuma.wordpress.com/2012/03/01/robotium-for-android/>
* <https://code.google.com/p/robotium/wiki/Getting_Started>
* <http://www.guru99.com/why-android-testing.html>
* <http://wiebe-elsinga.com/blog/whats-new-in-android-testing>
* <https://github.com/piotrek1543/robotium-showcase>
* <http://stackoverflow.com/questions/38783594/use-test-annotation-and-activitytestrule-with-robotium-espresso-way>