[Android Studio](http://developer.android.com/sdk/installing/studio.html) is now the officially supported IDE by Google for Android development.  This program is based on the [JetBrains](http://jetbrains.com) family of IDE's, so if you've used IntelliJ, PyCharms, or RubyMine, you'll find the user experience very familiar.  Fortunately, this IDE is essentially a form of the IntelliJ Community Edition and made free for Android developers.

## Migrating from Eclipse

Android Studio uses a new build system called [[Gradle|Getting-Started-with-Gradle]].  The recommended approach is to directly import your Eclipse projects.  In Android Studio v1.0, there is an option to "Import Non-Android Studio Project".  Note that you will need to refer to the Eclipse project root directory, which should have immediate subdirectories src/ and res/.  (If Android Studio does not find these directories in the project directory specified, it does not try to perform the migration to Gradle.)

![http://i.imgur.com/gnr8dhW.png](http://i.imgur.com/gnr8dhW.png)

When you do the import, your Java and XML resource files are relocated to src/main/java and src/main/res respectively.  Gradle looks explicitly in the src/main directory and can compile successfully but will fail to include your .class files if they are not located in these particular directories.

The plugin will also generate a build.gradle.  An example of what the file format is shown in the [[Gradle cliffnotes|Getting-Started-with-Gradle]].

## Using Gradle in Android Studio

When you make changes to the build.gradle, the changes should now be reflected in the IDE.  You may have target SDK versions conflicts (i.e. Facebook SDK targets for API versions above 17), so you may need to resolve your Gradle configurations to be consistent.

Android Studio should update the build.gradle file if you try to add dependencies. Any .jar files in the libs/ directory are already incorporated at compile time.  You can verify this fact by opening the Gradle file for the specific module you're using:

```gradle
dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
}
```
Also, you should be able to do the same with Maven dependencies.  go to Project Structure -> Modules-> Dependencies. Click on + -> Library Dependency and add modules such as "com.loopj.android:android-async-http:1.4.6".   Double-check the build.gradle file to see that this change was added.

Regardless, you should keep track of what gets changed in the build.gradle since that is ultimately the file Android Studio uses to handle dependency management.

### Exporting Libraries in Android Studio

In Eclipse, Android libraries could be exported in the `JAR` format and added as libraries in the `libs` folder. With Android Studio, the option to export an Android library as a jar file **has been removed**. JARs had a number of limitations including the **inability to include resource files** within a library. With Android Studio and Gradle, "Android Archives" (or `AAR` files) are now the new binary format for exporting Android libraries. Android Studio provides support for export your Android library as an [aar file](http://tools.android.com/tech-docs/new-build-system/aar-format). See [this stackoverflow post](http://stackoverflow.com/a/17132055/313399) for more details.

If your library is configured as an Android library (i.e. Uses `apply plugin: 'com.android.library'` statement in its `build.gradle` file), an `.aar` will be created automatically when the project is built. It will show up in the `build/outputs/aar` directory in your module's directory. You can choose the "Android Library" type in File > New Module to create a new Android Library. See [this stackoverflow post](http://stackoverflow.com/a/23326397/313399) for how to import local android archives into a project.

### Library Projects in Android Studio

In Android Studio, you can re-create the effects of "library projects" in eclipse where you can "include" one Eclipse project as a library dependency in another project. In Android Studio, this is done with Gradle. First in the **library project**, change the `build.gradle`:

```gradle
apply plugin: 'com.android.library'
```

See [example gradle file](https://github.com/androidsx/hello-android-studio/blob/master/MyLibrary/build.gradle). Next, **in the application** to load the library, change the `build.gradle` file:

```gradle
dependencies {
    compile project(':MyLibrary')
}
```

and in the `iml` file near the bottom in the last `<component>`:

```xml
<orderEntry type="library" name="android-support-v4" level="application" />
<orderEntry type="library" name="MyLibrary.aar" level="project" />
```

See the sample [build.gradle](https://github.com/androidsx/hello-android-studio/blob/master/HelloWorld/build.gradle) and [iml](https://github.com/androidsx/hello-android-studio/blob/master/HelloWorld/HelloWorld.iml#L70-L71) files for reference. That should be it, after syncing Gradle the library should be accessible.

### Use Gradle wrapper

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

  ![http://i.imgur.com/UoFj1XY.gif](http://i.imgur.com/UoFj1XY.gif | height=200px)

* If you're adding libraries that also define R.java resource files (such as a [PullToRefresh library](http://guides.codepath.com/android/Implementing-Pull-to-Refresh) or Google Play Services), you cannot add them as .jar files.  They must be included as dependencies.  If you try to include them as .jar files, you may encounter R definitions not found during execution time.

* Get familiar with the Code->Generate, Code->Reformat, and Code->Optimize Imports options.  The first one will allow you to help generate getter and setter methods on your classes, saving you time from having to write boilerplate code.  The other two options help to tidy up your code.

* Install the GenerateSerialVersionUID to allow you to use the same feature in Eclipse to auto-generate a unique ID for a Serializable class.  

![image](https://cloud.githubusercontent.com/assets/326857/2773890/6abf6e7c-caa9-11e3-9077-b8c25fa0df25.png)

You can then click on Code->Generate to SerialVersionUID:

![image](https://cloud.githubusercontent.com/assets/326857/2773934/76540c56-caaa-11e3-928a-d9698c9c79d4.png)

* If you are using Genymotion as the emulator, you may have issues trying to use other Android SDK tools such as the Hierarchy Viewer unless you go to Genymotion ADB settings and set the path to your SDK directory (i.e. for OSX, ~/Library/Android/sdk)

![image](http://i.imgur.com/iGqP85B.png)

## References

* <http://www.androidsx.com/how-to-link-an-android-library-project-with-gradle-in-android-studio/>