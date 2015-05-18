Robolectric is a framework which mocks part of the Android framework contained in the android.jar file and which allows you to run Android tests directly on the JVM with the JUnit 4 framework.

Robolectric is designed to allow you to test Android applications on the JVM. This enables you to run your Android tests in your continuous integration environment without any additional setup and without an emulator running. Because of this Robolectric is **really fast** in comparison to any other testing approach.

Let's take a look at a step-by-step for setting up Robolectric to test your project.

# Setting up for Android Studio

## Configure top-level build.gradle

   Make sure to be running at least version 1.1.0 of the Android plug-in for Gradle, since unit testing with Android Studio was only recently supported.  More information can be found [here] (http://tools.android.com/tech-docs/unit-testing-support).  

   ```gradle
   buildscript {
      repositories {
         mavenCentral()
      } 
      dependencies {
         classpath 'com.android.tools.build:gradle:1.2.2'
      }
   }
   ```

Setting up this file in the top level will help ensure that there is only one place where this Android plug-in is defined.  All other projects (i.e. app) will inherit this dependency.

## Configure app/build.gradle

  ```gradle
  dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    testCompile 'junit:junit:4.12'
    testCompile('org.robolectric:robolectric:3.0-rc2') {
           exclude group: 'commons-logging', module: 'commons-logging'
           exclude group: 'org.apache.httpcomponents', module: 'httpclient'
    }
   apply plugin: 'org.robolectric'
   ``` 

   Within your dependencies, you need to include the following defines. The exclude modules
   are intended to remove duplicate dependency definitions (template borrowed from https://github.com/mutexkid/android-studio-robolectric-example).  

## Creating resource directory

   Create an `test/resources/org.robolectric.Config.properties` file.  Note that the directory
   needs to be resources/ (not to be confused with /res directory used to store your layout files).   

   ```
   # Robolectric doesn't know how to support SDK 19 yet.
   emulateSdk=18
   ```

## Creating tests

1. See [this example](https://github.com/mutexkid/android-studio-robolectric-example/blob/master/app/src/test/java/com/example/joshskeen/myapplication/MyActivityTest.java).  Note that your test needs to be annotated with `RoboelectricGradleTestRunner` instead of `RoboelectricTestRunner`.  You also need to annotate the `BuildConfig.class`, which should be created during a Gradle run.
  ```java

     RunWith(RobolectricGradleTestRunner.class)
     @Config(constants = BuildConfig.class)
     public class MyActivityTest {
  ```
The [Roboelectric guide](http://robolectric.org/writing-a-test/) is also a useful resource.  Each of your tests must contain an `@Test` annotation.

2. If you intend to create mock network responses, you will need to place them inside the `src/test/resources/` directory.  You can reference these files by using `getResourceAsStream()` (note the backslash is needed in the front):

 
   ```
      InputStream stream = MyClass.class.getResourceAsStream("/test_data.json");
   ```

Note that while running Gradle at the command-line, your tests may pass successfully.  However, in Android Studio you may notice that null gets returned whenever trying to make this call.  It is documented as a [known issue](http://tools.android.com/knownissues#TOC-JUnit-tests-missing-resources-in-classpath-when-run-from-Studio) and will be fixed in the next version of Android Studio.  

One workaround inspired from this [PasteBin](http://pastebin.com/L6CeCtAp) is to add these lines into your `app/build.gradle` file.  What it does it copy all the files from `test/resources` into the build directory so they can be found during your Android Studio test runs.

```gradle

task copyTestResources(type: Copy) {
        from "${projectDir}/src/test/resources"
        into "${buildDir}/intermediates/classes/test/debug"
    }

assembleDebug.dependsOn(copyTestResources)
```

## Running tests

If you type ./gradlew test from the command line, tests should run correctly.  

## Debugging Tests

If you want to be able to debug your tests inside Android Studio, make sure you are at least using the Android Gradle plug-in v1.1 or above.  In older versions, there were problems being able to run these tests directly from the IDE.

1. Make sure your tests are located in `src/test/java`.   

2. If you are using a Mac, go to `Run` -> `Edit Configurations` -> `Defaults` -> `Junit` and make sure to set $MODULE_DIR$ as the working directory.  There is a known bug in tests not being located unless you set this configuration first.  See the Roboelectric [getting started guide](http://robolectric.org/getting-started/) for more information. 

  <img src="http://robolectric.org/images/android-studio-configure-defaults-4bf48402.png">

3. Select `Unit Tests` under `Build Variants`.  If you see this dropdown disabled, make sure to verify what Android Gradle plug-in version youare using.

  <img src="https://camo.githubusercontent.com/cbf79d740e265cc9da9299c2b5f29fc8a63613e7/68747470733a2f2f7777772e657665726e6f74652e636f6d2f73686172642f733331332f73682f35363063346235662d653730622d343830302d623436662d6263313936383631383333382f38396331653734306537313334333136393631613130333032316461663163622f646565702f302f4d794163746976697479546573742e6a6176612d2d2d616e64726f69642d73747564696f2d726f626f6c6563747269632d6578616d706c652d2d2d2d2d2d636f64652d616e64726f69642d73747564696f2d726f626f6c6563747269632d6578616d706c652d2e706e67"/>


4. Make sure you run the test as a JUnit test, not as an Android Application Test.  You can control-click on the file and click on Run.  The icons look different (1st icon is Android test, while the 2nd icon is JUnit.)

   ![](http://i.imgur.com/RDmmdI2.png)