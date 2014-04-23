If you find yourself highly dissatisfied with developing with Eclipse (i.e. having to restart whenever recompiles somehow don't work, too many window panes to find, poor XML editing/validation, etc.), you may wish to download a copy of Android Studio (http://developer.android.com/sdk/installing/studio.html).  It currently is alpha and there are updates to the IDE every week, so be forewarned.  It is based on the [JetBrains](http://jetbrains.com) family of IDE's, but made free for Android developers.

## Migrating from Eclipse

Since [version 0.4](http://tools.android.com/recent/androidstudio040released), Android Studio now has the ability to import your Eclipse projects directly.  When you do the import, your Java and XML resource files are relocated to src/main/java and src/main/res respectively.  Gradle looks explicitly in the src/main directory and can compile successfully but will fail to include your .class files if they are not located in these particular directories.

The plugin will also generate a build.gradle.  An example of what the file format is shown at: https://github.com/thecodepath/android_guides/wiki/Getting-Started-with-Gradle

## Dependency Configurations
Android Studio gives you the option of using several different dependency management configurations.  Normally when you import existing projects, it tries looking for Maven or Gradle-based project files (pom.xml or build.gradle respectively).  You cannot switch dependency management systems after you've selected one. 

## Using Gradle in Android Studio

When you make changes to the build.gradle, the changes should now be reflected in the IDE.  You may have target SDK versions conflicts (i.e. Facebook SDK targets for API versions above 17), so you may need to resolve your Gradle configurations to be consistent.

Android Studio should update the build.gradle file if you try to add dependencies.  For instance, if you Ctrl-click on a .jar file inside your libs/ directory, you should be able to "Add as Library".  This will automatically add the entry as a dependency in the build.gradle file.

Also, you should be able to do the same with Maven dependencies.  go to Project Structure -> Modules-> Dependencies. Click on + -> Library Dependency and add modules such as "com.loopj.android:android-async-http:1.4.3".   Double-check the build.gradle file to see that this change was added.

Regardless, you should keep track of what gets changed in the build.gradle since that is ultimately the file Android Studio uses to handle dependency management.

## Other Notes

* One useful tip -- inside Edit->Run Configurations, make sure to clear LogCat per
run.  It's often easy to get confused of stack traces and this option clears the logging statements between
each test run:

  ![image](https://f.cloud.github.com/assets/326857/1445221/6f620f78-421b-11e3-9708-df6185495289.png)

* If you're adding libraries that also define R.java resource files (such as the PullToRefresh library mentioned in http://guides.thecodepath.com/android/Implementing-Pull-to-Refresh or Google Play Services), you cannot add them as .jar files.  They must be included as dependencies.  If you try to include them as .jar files, you may encounter R definitions not found during execution time.

* Install the GenerateSerialVersionUID to allow you to use the same feature in Eclipse to auto-generate a unique ID for a Serializable class.  

![image](https://cloud.githubusercontent.com/assets/326857/2773890/6abf6e7c-caa9-11e3-9077-b8c25fa0df25.png)

You can then click on Code->Generate to SerialVersionUID:

![image](https://cloud.githubusercontent.com/assets/326857/2773934/76540c56-caaa-11e3-928a-d9698c9c79d4.png)