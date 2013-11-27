If you find yourself highly dissatisfied with developing with Eclipse (i.e. having to restart whenever recompiles somehow don't work, too many window panes to find, poor XML editing/validation, etc.), you may wish to download a copy of Android Studio (http://developer.android.com/sdk/installing/studio.html).  It currently is alpha and there are updates to the IDE every week, so be forewarned.

## Migrating from Eclipse

Check out http://developer.android.com/sdk/installing/migrate.html.   The plugin attempts to export your Eclipse project to Gradle.  What it essentially does is relocates your Java and XML resource files to src/main/java and src/main/res respectively.  Gradle looks explicitly in the src/main directory and can compile successfully but fail to include your .class files if they are not located in these particular directories (see section about using Gradle in Android Studio)

The plugin will also generate a build.gradle.  An example of what the file format is shown at: https://github.com/thecodepath/android_guides/wiki/Getting-Started-with-Gradle

## Dependency Configurations
Android Studio gives you the option of using several different dependency management configurations.  Normally when you import existing projects, it tries looking for Maven or Gradle-based project files (pom.xml or build.gradle respectively).  You cannot switch dependency management systems after you've selected one.

There is one more way which isn't well documented, which is to choose neither option and rely on manual configuration within Android Studio to define the dependencies yourself.  Currently Gradle inside Android Studio is very slow (though there are parallel and daemon modes being added to speed things up), so for smaller projects you may actually find it easier to take advantage of Android Studio's Project Structure to define your dependencies.  

## Using Gradle in Android Studio

Currently, Android Studio and Gradle actually operate very much independently of each other.  Even if your Android Studio compiles correctly, you may find NoClassDefErrors when it tries to execute the software in your emulator.    Hopefully by the time Android Studio v1.0.0 there will be more UI to manage the dependencies in Gradle, but currently everything must be done by tweaking build.gradle files.

## Manual dependency management

1. If you want to use manual dependency management, reimport your project into Android Studio.  Do not choose either Gradle or Maven to build.

   ![image](https://f.cloud.github.com/assets/326857/1445194/eba67782-421a-11e3-98aa-9d25a9e7e11a.png)

2. Suppose you want to have your Google Play Services and Facebook SDK defined as dependencies.  You need to use the Create Module from Existing Sources and mark the src directory as a "Source directory" (i.e. see https://developers.facebook.com/docs/getting-started/facebook-sdk-for-android-using-android-studio/3.0/) These modules will be used to define your dependencies in the next step.

3. Go to Project Structure->Modules and under your ProjectName, add the Google Play and Facebook module as a dependency to your module (look for the + sign at the right side):

   ![image](https://f.cloud.github.com/assets/326857/1445028/307d4ec0-4217-11e3-8417-181d0db54f69.png)

   You should see under your main project that Google Play Services + Facebook added and the Export button clicked:

   ![image](https://f.cloud.github.com/assets/326857/1445499/431196bc-4222-11e3-8f80-bbc3bb3afe73.png)

3. Make sure to mark the Google Play Services and Facebook SDK as a Library module.  Otherwise, you may get NoClassDefError exceptions when running. 

   ![image](https://f.cloud.github.com/assets/326857/1460057/438638aa-440c-11e3-8f3b-05e8b21ece58.png)

## Other Notes

* Verify that you can traverse the libs/ dir (there should be + signs).

  ![image](https://f.cloud.github.com/assets/326857/1445048/777ae8b4-4217-11e3-9ec0-29b0031527ac.png)

If not, make sure to Add as Library (right-click)

* One useful tip -- inside Edit->Run Configurations, make sure to clear LogCat per
run.  It's often easy to get confused of stack traces:

  ![image](https://f.cloud.github.com/assets/326857/1445221/6f620f78-421b-11e3-9708-df6185495289.png)

* If you're adding libraries that also define R.java resource files (such as the PullToRefresh library mentioned in http://guides.thecodepath.com/android/Implementing-Pull-to-Refresh or Google Play Services), you cannot add them as .jar files.  They must be included as Modules.  If you try to include them as .jar files, you may encounter R definitions not found during execution time.