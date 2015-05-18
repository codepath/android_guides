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

   Create an test/resources/org.robolectric.Config.properties file.  Note that the directory
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

The [Roboelectric guide](http://robolectric.org/writing-a-test/) is also a useful resource.  Each of your tests must be annotated with a `Test` decorator.

## Running tests

Proceed with the next section about [[Verifying installation|Robolectric-Installation-for-Unit-Testing#verify-installation]].

If you type ./gradlew test from the command line, tests should run correctly.  

## Debugging Tests

If you want to be able to debug your tests inside Android Studio, make sure you are at least using the Android Gradle plug-in v1.1 or above.  In older versions, there were problems being able to run these tests directly from the IDE.

1. Make sure your tests are located in `src/test/java`.   

2. If you are using a Mac, go to `Run` -> `Edit Configurations` -> `Defaults` -> `Junit` and make sure to set $MODULE_DIR$ as the working directory.  There is a known bug in tests not being located unless you set this configuration first.  See the Roboelectric [getting started guide](http://robolectric.org/getting-started/) for more information. 

  <img src="http://robolectric.org/images/android-studio-configure-defaults-4bf48402.png">

3. Select Unit Tests under Build Variants.  If you see this dropdown disabled, make sure to verify what Android Gradle plug-in version youare using.

  <img src="https://camo.githubusercontent.com/cbf79d740e265cc9da9299c2b5f29fc8a63613e7/68747470733a2f2f7777772e657665726e6f74652e636f6d2f73686172642f733331332f73682f35363063346235662d653730622d343830302d623436662d6263313936383631383333382f38396331653734306537313334333136393631613130333032316461663163622f646565702f302f4d794163746976697479546573742e6a6176612d2d2d616e64726f69642d73747564696f2d726f626f6c6563747269632d6578616d706c652d2d2d2d2d2d636f64652d616e64726f69642d73747564696f2d726f626f6c6563747269632d6578616d706c652d2e706e67"/>


4. Make sure you run the test as a JUnit test, not as an Android Application Test.  You can control-click on the file and click on Run.  The icons look different (1st icon is Android test, while the 2nd icon is JUnit.)

   ![](http://i.imgur.com/RDmmdI2.png)


# Setting up for Eclipse

## Download Library JARs

Let's download the [robolectric jar with dependencies](https://www.dropbox.com/s/3xcxxvhviufpblk/robolectric-2.3-with-dependencies.jar) for use and an additional library called [fest-util.jar](https://www.dropbox.com/s/iibi53najlrmd57/fest-util-1.1.6.jar) and then [fest-android.jar](http://repo1.maven.org/maven2/com/squareup/fest-android/1.0.7/fest-android-1.0.7.jar).

## Create Test Folder

We can have our tests live in the main project. First, we want to create a "test" directory at the root of our SimpleApp project. On SimpleApp, choose "New... → Folder" and name the folder "test"

![Screen 0](http://i.imgur.com/TedJHPG.png)

## Create Robolectric Java Test Project

Next, let's create our electric test project for our app. Create a **Java Project** called "SimpleAppElectricTest". Do **not** create an Android project, or an Android JUnit project, but a simple **Java Project** .

![Screen 1](http://i.imgur.com/hkI4xTD.png)

## Configure Source Folders

Click "Next", Expand the SimpleAppElectricTest row and select the "src" row - Click link "Remove source folder 'src' from build path"

![Screen 2](http://i.imgur.com/0oApkjT.png)

Click link "Link additional source" - Browse to and select ".../SimpleApp/test" folder we created earlier.

![Screen 3](http://i.imgur.com/WX5do4S.png)

Click "Finish" on the "Link additional sources" dialog (but keep the new Java project dialog open)

![Screen 4](http://i.imgur.com/0IPqHs2.png)

## Add Project Dependency

Select "Projects" tab at the top of the "New Java Project dialog" - Click "Add..." - Check "SimpleApp" - Click "OK" - Click "Finish" closing the new Java project dialog

![Screen 5](http://i.imgur.com/W1Jb5Qu.png)

## Load Robolectric Library

Create a "lib" folder in the test project with "New...Folder". Move the robolectric jar and the fest-util.jar into the lib folder for the SimpleAppElectricTest project.

![Screen 6](http://i.imgur.com/s3W4aqK.png)

Add the jar to the Java Build Path for the project by right-clicking test project and hover "Build Path" and click "Configure Build Path"

![Screen 7](http://i.imgur.com/7zjgrKe.png)

Select "Add JARs" and select the two jars within the "lib" folder. Hit "OK" to confirm adding these JARs to the build path. Keep the "Libraries" tab open in the "Java Build Path" dialog

![Screen 8](http://i.imgur.com/fHtrhrg.png)
![Screen 9](http://i.imgur.com/hEWC2P1.png)

## Add JUnit library

Select "Add Library" - Select "JUnit" - Click "Next" - Select JUnit library version 4 (Robolectric is not compatible with JUnit 3) - Click "Finish" (keep the Properties dialog for SimpleAppElectricTest open)

![Screen 10](http://i.imgur.com/D5j2NFo.png)

## Add Android Jars

Click "Add External Jars..." - Navigate to <android sdk directory>/platforms/android-8/android.jar - Click "Open" (keep the Properties dialog for SimpleAppElectricTest open)

![Screen 11](http://i.imgur.com/r3xEPrO.png)

Click "Add External Jars..." - Navigate to <android sdk directory>/add-ons/addongoogleapisgoogleinc_8/libs/maps.jar - Click "Open". Click "OK" on the Properties for SimpleAppElectricTest dialog

![Screen 12](http://i.imgur.com/c14ZWyJ.png)

Now we are almost done! Just need to setup the run configuration for our tests.

## Setup Run Configuration

**Note**: Your tests will not run without this step. Your resources will not be found.

Select "Run" → "Run Configurations..."

![Screen 13](http://i.imgur.com/5rkKhrI.png)

Double click "JUnit" (not "Android JUnit Test") - Name: SimpleAppElectricTest - Select the "Run all tests in the selected project, package or source folder:" radio button - Click the "Search" button - Select "SimpleAppElectricTest" - TestRunner: JUnit 4

![Screen 14](http://i.imgur.com/Of55nto.png)

Click on the link "Multiple launchers available Select one..." at the bottom of the dialog - Check the "Use configuration specific settings" box - Select "Eclipse JUnit Launcher" - Click "OK"

![Screen 15](http://i.imgur.com/i0Firbe.png)

Click the "Arguments" tab - Under "Working directory:" select the "Other:" radio button - Click "Workspace..." - Select "SimpleApp" (not "SimpleAppElectricTest", The value inside of 'Other' edit box should be '${workspace_loc:SimpleApp}') - Click "OK" - Click "Close"

![Screen 16](http://i.imgur.com/GYfck1l.png)

# Verify Installation

Create a temporary test to verify that the installation was correct. Create a file in the "test" folder in SimpleAppElectricTest. Create "MyActivityTest" in the "com.codepath.example.simpleapp.test" with the following code:

```java
package com.codepath.example.simpleapp.test;

import static org.hamcrest.CoreMatchers.equalTo;
import static org.junit.Assert.assertThat;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.robolectric.RobolectricTestRunner;

import com.codepath.example.simpleapp.FirstActivity;
import com.codepath.example.simpleapp.R;

@RunWith(RobolectricTestRunner.class)
public class MyActivityTest {

    @Test
    public void shouldHaveHappySmiles() throws Exception {
        String hello = new FirstActivity().getResources().getString(R.string.hello_world);
        assertThat(hello, equalTo("Hello World!"));
    }

}
```

Run the tests by right-clicking on the test project and selecting "Run As..." => "Junit Test"

![Screen 17](http://i.imgur.com/x5Cgs3n.png)

And if that passes green, then Robolectric has been properly installed!