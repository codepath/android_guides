Robolectric is a framework which mocks part of the Android framework contained in the android.jar file and which allows you to run Android tests directly on the JVM with the JUnit 4 framework.

Robolectric is designed to allow you to test Android applications on the JVM. This enables you to run your Android tests in your continuous integration environment without any additional setup and without an emulator running. Because of this Robolectric is **really fast** in comparison to any other testing approach.

Let's take a look at a step-by-step for setting up Robolectric to test your project.

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

## Verify Installation

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