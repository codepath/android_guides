[Android Studio](http://developer.android.com/sdk/installing/studio.html) is now the officially supported IDE by Google for Android development.  It currently has an official v1.0 release but continues to be updated frequently (see [recent tools page](http://tools.android.com/recent) for more details).  This program is based on the [JetBrains](http://jetbrains.com) family of IDE's, so if you've used IntelliJ, PyCharms, or RubyMine, you'll find the user experience very familiar.  Fortunately, this IDE is essentially a form of the IntelliJ Community Edition and made free for Android developers.

## Migrating from Eclipse

Android Studio now has the ability to import your Eclipse projects directly.  When you do the import, your Java and XML resource files are relocated to src/main/java and src/main/res respectively.  Gradle looks explicitly in the src/main directory and can compile successfully but will fail to include your .class files if they are not located in these particular directories.

The plugin will also generate a build.gradle.  An example of what the file format is shown in the [[Gradle cliffnotes|Getting-Started-with-Gradle]].

## Using Gradle in Android Studio

When you make changes to the build.gradle, the changes should now be reflected in the IDE.  You may have target SDK versions conflicts (i.e. Facebook SDK targets for API versions above 17), so you may need to resolve your Gradle configurations to be consistent.

Android Studio should update the build.gradle file if you try to add dependencies. Any .jar files in the libs/ directory are already incorporated at compile time.  You can verify this fact by opening the Gradle file for the specific module you're using:

```
dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
}
```
Also, you should be able to do the same with Maven dependencies.  go to Project Structure -> Modules-> Dependencies. Click on + -> Library Dependency and add modules such as "com.loopj.android:android-async-http:1.4.3".   Double-check the build.gradle file to see that this change was added.

Regardless, you should keep track of what gets changed in the build.gradle since that is ultimately the file Android Studio uses to handle dependency management.

## Use Gradle wrapper

[Gradle wrapper](http://developer.android.com/sdk/installing/studio-build.html#gradleWrapper) allows you to seamlessly work with multiple projects that require different versions of Gradle. 

Android Studio should normally handle creating the Gradle wrapper.  If you are interested in how it works under the covers, read through [[Gradle cliffnotes|Getting-Started-with-Gradle]].

## Preferred keyboard shortcuts

It's possible to choose different keyboard shortcuts from common IDEs to continue to be productive in Android Studio. Go to `Apple -> Search for 'shortcuts' -> Select Keymap` -> Choose from drop down.

## Other Notes

* One useful tip -- inside Edit->Run Configurations, make sure to clear LogCat per
run.  It's often easy to get confused of stack traces and this option clears the logging statements between
each test run:

  ![image](https://f.cloud.github.com/assets/326857/1445221/6f620f78-421b-11e3-9708-df6185495289.png)

* By default, Android Studio does not perform auto-imports.  If you want to enable it, go to Preferences->Editor->Auto Import, click the checkbox "Add unambiguous imports on the fly" but make sure to exclude android.R auto imports since it can often cause conflicts with your own resource imports.  You may also wish to Optimize Imports on the Fly, but you can also type Command-Option-O to do it manually:

  ![http://i.imgur.com/UoFj1XY.gif](http://i.imgur.com/UoFj1XY.gif)

* If you're adding libraries that also define R.java resource files (such as a [PullToRefresh library](http://guides.codepath.com/android/Implementing-Pull-to-Refresh) or Google Play Services), you cannot add them as .jar files.  They must be included as dependencies.  If you try to include them as .jar files, you may encounter R definitions not found during execution time.

* Install the GenerateSerialVersionUID to allow you to use the same feature in Eclipse to auto-generate a unique ID for a Serializable class.  

![image](https://cloud.githubusercontent.com/assets/326857/2773890/6abf6e7c-caa9-11e3-9077-b8c25fa0df25.png)

You can then click on Code->Generate to SerialVersionUID:

![image](https://cloud.githubusercontent.com/assets/326857/2773934/76540c56-caaa-11e3-928a-d9698c9c79d4.png)

* If you are using Genymotion as the emulator, you may have issues trying to use other Android SDK tools such as the Hierarchy Viewer unless you go to Genymotion ADB settings and set the path to your SDK directory (i.e. for OSX, /Applications/Android Studio.app/sdk)

![image](http://imgur.com/PhSYmVo.png)