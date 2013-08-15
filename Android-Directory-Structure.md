## Overview

Within an Android project, the most frequently edited folders are:

* `src` - Java source files associated with your project. This includes the Activity "controller" files as well as your models and helpers.
* `res` - Resource files associated with your project. All graphics, strings, layouts, and other resource files are stored in the resource file hierarchy under the res directory. 
* `res/layout` - XML layout files that describe the views and layouts for each activity and for partial views such as list items.
* `res/values` - XML files which store various string values (titles, labels, styles).
* `res/drawable` - Here we store the various images used in our application.

The most frequently edited files are:

* `AndroidManifest.xml` - This is the Android application definition file. It contains information about the Android application such as minimum Android version, permission to access Android device capabilities such as internet access permission, ability to use phone permission, etc.
* `res/layout/activity_foo.xml` - This file describes the layout of the page. This means the placement of every view object on one app screen.
* `src/.../FooActivity.java` - The Activity "controller" that constructs the activity using the view, and handles all event handling and view logic for one app screen.

Other less edited folders include:

* `gen` - Generated Java code files, this library is for Android internal use only.
* `assets` - Uncompiled source files associated with your project; Rarely used.
* `bin` - Resulting application package files associated with your project once it’s been built.
* `libs` - Contains any secondary libraries (jars) you might want to link to your app.

## References

 * <http://developer.android.com/tools/projects/index.html#ApplicationProjects>
 * <http://www.codeproject.com/Articles/395614/Basic-structure-of-an-Android-project>