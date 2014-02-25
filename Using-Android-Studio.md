If you find yourself highly dissatisfied with developing with Eclipse (i.e. having to restart whenever recompiles somehow don't work, too many window panes to find, poor XML editing/validation, etc.), you may wish to download a copy of Android Studio (http://developer.android.com/sdk/installing/studio.html).  It currently is alpha and there are updates to the IDE every week, so be forewarned.

## Migrating from Eclipse

Since [version 0.4](http://tools.android.com/recent/androidstudio040released), Android Studio now has the ability to import your Eclipse projects directly.  When you do the import, your Java and XML resource files are relocated to src/main/java and src/main/res respectively.  Gradle looks explicitly in the src/main directory and can compile successfully but will fail to include your .class files if they are not located in these particular directories.

The plugin will also generate a build.gradle.  An example of what the file format is shown at: https://github.com/thecodepath/android_guides/wiki/Getting-Started-with-Gradle

## Dependency Configurations
Android Studio gives you the option of using several different dependency management configurations.  Normally when you import existing projects, it tries looking for Maven or Gradle-based project files (pom.xml or build.gradle respectively).  You cannot switch dependency management systems after you've selected one.

## Using Gradle in Android Studio

As of Android Studio and Gradle actually operate very much independently of each other.  Even if your Android Studio compiles correctly, you may find NoClassDefErrors when it tries to execute the software in your emulator.    When you make changes to the build.gradle, the changes will now be reflected in the UI (previous v0.3 versions did not).

## Other Notes

* One useful tip -- inside Edit->Run Configurations, make sure to clear LogCat per
run.  It's often easy to get confused of stack traces and this option clears the logging statements between
each test run:

  ![image](https://f.cloud.github.com/assets/326857/1445221/6f620f78-421b-11e3-9708-df6185495289.png)

* If you're adding libraries that also define R.java resource files (such as the PullToRefresh library mentioned in http://guides.thecodepath.com/android/Implementing-Pull-to-Refresh or Google Play Services), you cannot add them as .jar files.  They must be included as Modules.  If you try to include them as .jar files, you may encounter R definitions not found during execution time.