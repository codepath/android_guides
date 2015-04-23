Robolectric is a framework which mocks part of the Android framework contained in the android.jar file and which allows you to run Android tests directly on the JVM with the JUnit 4 framework.

Robolectric is designed to allow you to test Android applications on the JVM. This enables you to run your Android tests in your continuous integration environment without any additional setup and without an emulator running. Because of this Robolectric is **really fast** in comparison to any other testing approach.

Let's take a look at a step-by-step for setting up Robolectric to test your project.

# Setting up for Android Studio

## Configure build.gradle

   You'll need to setup your build.gradle configuration to include the Robolectric Gradle plugin:

   ```
   buildscript {
      repositories {
         mavenCentral()
      } 
      dependencies {
         classpath 'com.android.tools.build:gradle:0.12.+'
         classpath 'org.robolectric:robolectric-gradle-plugin:1.0.1'
      }
   }

   apply plugin: 'org.robolectric'
   ``` 

   Within your dependencies, you need to include the following defines. The exclude modules
   are intended to remove duplicate dependency definitions (template borrowed from 
   https://github.com/robolectric/deckard-gradle/blob/master/README.md).  

   For instance, the support-v4.jar will likely to be specified as a compilation dependency already. Roboelectric also specifies it as a dependency and will trigger a (Lcom/example/android/app/FragmentLayoutSupport; had used a different Landroid/support/v4/app/FragmentActivity; during pre-verification) error if it is not excluded.

```gradle
dependencies {
  compile fileTree(dir: 'libs', include: ['*.jar'])
  compile 'com.android.support:support-v4:22.1.0'

  androidTestCompile 'org.hamcrest:hamcrest-integration:1.3'
  androidTestCompile 'org.hamcrest:hamcrest-core:1.3'
  androidTestCompile 'org.hamcrest:hamcrest-library:1.3'

  androidTestCompile('junit:junit:4.+') {
      exclude module: 'hamcrest-core'
  }
  androidTestCompile('org.robolectric:robolectric:2.4') {
      exclude module: 'classworlds'
      exclude module: 'commons-logging'
      exclude module: 'httpclient'
      exclude module: 'maven-artifact'
      exclude module: 'maven-artifact-manager'
      exclude module: 'maven-error-diagnostics'
      exclude module: 'maven-model'
      exclude module: 'maven-project'
      exclude module: 'maven-settings'
      exclude module: 'plexus-container-default'
      exclude module: 'plexus-interpolation'
      exclude module: 'plexus-utils'
      exclude module: 'support-v4'
      exclude module: 'wagon-file'
      exclude module: 'wagon-http-lightweight'
      exclude module: 'wagon-provider-api'
  }
}
```

## Creating resource directory

   Create an androidTest/resources/org.robolectric.Config.properties file.  Note that the directory
   needs to be resources/ (not to be confused with /res directory used to store your layout files).   

   ```
   # Robolectric doesn't know how to support SDK 19 yet.
   emulateSdk=18
   ```

## Running tests

Proceed with the next section about [[Verifying installation|Robolectric-Installation-for-Unit-Testing#verify-installation]].

If you type ./gradlew test from the command line, tests should run correctly.  

## Debugging Tests

If you want to be able to debug your tests inside Android Studio, here is what you need to do.

1. Make sure you run the test as a JUnit test, not as an Android Application Test.  You can control-click on the file and click on Run.  The icons look different (1st icon is Android test, while the 2nd icon is JUnit.)

   ![](http://i.imgur.com/RDmmdI2.png)

2. When you run a JUnit test with Android Studio, you will see the dreaded `Stub!` exception:

   ```
      java.lang.RuntimeException: Stub!
          at junit.runner.BaseTestRunner.<init>(BaseTestRunner.java:5)
          at junit.textui.TestRunner.<init>(TestRunner.java:54)
          at junit.textui.TestRunner.<init>(TestRunner.java:48)
          at junit.textui.TestRunner.<init>(TestRunner.java:41)
   ```

   Each time you modify/update build.gradle (or restart Android Studio), you must edit app/app.iml      
   and move the orderEntry for the Android SDK to the end. 

   ```
    	    	<orderEntry type="library" exported="" scope="TEST" name="wagon-provider-api-1.0-beta-6" level="project" />
    	    	<orderEntry type="library" exported="" scope="TEST" name="xercesMinimal-1.9.6.2" level="project" />
    	    	<orderEntry type="jdk" jdkName="Android API 19 Platform" jdkType="Android SDK" />			    		<---make sure this is the last orderEntry
    		</component>
    	</module>
   ```

   For Android Studio, dependency ordering is currently not modifiable via any GUI (Google took   
this feature out in subsequent releases).  Therefore, you must modify the project iml file directly as such and reload the project.  Hopefully this issue can get fixed soon but for now you have to move things around manually.

3. Executing tests as JUnit means they will be generated into a separate directory, so Android Studio won't know how to find the generated class files.  Go to Run -> Edit Configuration -> Defaults -> JUnit:

   1. Start an unit test.  Assuming you did step 1, it will report the missing class.
   2. Click on the output from the terminal and copy the entire --classpath parameter.
   3. Edit classpath and add /[projectPath]/build/test-classes/debug (i.e. /Users/rhu/projects/codepath-twitter-client/app/build/test-classes)

   ![image](http://imgur.com/EQdtUck.png)

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