## Overview

Within an [Android project structure](http://i.imgur.com/KDSWUXt.png), the most frequently edited folders are:

* `src` - Java source files associated with your project. This includes the Activity "controller" files as well as your models and helpers.
* `res` - Resource files associated with your project. All graphics, strings, layouts, and other resource files are stored in the resource file hierarchy under the res directory. 
* `res/layout` - XML layout files that describe the views and layouts for each activity and for partial views such as list items.
* `res/values` - XML files which store various attribute values. These include [[strings.xml|Using-String-Resources]], dimens.xml, [[styles.xml|Styles-and-Themes]], colors.xml, [[themes.xml|Developing-Custom-Themes]], and so on.
* `res/drawable` - Here we store the various density-independent graphic assets used in our application.
* `res/drawable-hdpi` - Series of folders for density specific images to use for various resolutions.

The most frequently edited files are:

* `AndroidManifest.xml` - This is the Android application definition file. It contains information about the Android application such as minimum Android version, permission to access Android device capabilities such as internet access permission, ability to use phone permission, etc.
* `res/layout/activity_foo.xml` - This file describes the layout of the activity's UI. This means the placement of every view object on one app screen.
* `src/.../FooActivity.java` - The Activity "controller" that constructs the activity using the view, and handles all event handling and view logic for one app screen.

Other less edited folders include:

* `gen` - Generated Java code files, this library is for Android internal use only.
* `assets` - Uncompiled source files associated with your project; Rarely used.
* `bin` - Resulting application package files associated with your project once it’s been built.
* `libs` - Contains any secondary libraries (jars) you might want to link to your app.

## References

 * <http://developer.android.com/tools/projects/index.html#ApplicationProjects>
 * <http://www.codeproject.com/Articles/395614/Basic-structure-of-an-Android-project>
 * <http://mobile.tutsplus.com/tutorials/android/android-sdk-app-structure/>