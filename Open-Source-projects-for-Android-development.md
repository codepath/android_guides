Here's a documented list of all items that we think should and could be improved.  If you're interested in taking on this project, we would be extremely grateful (and the open source community in general would too).

* Display a logging error at least if there is no Internet permission defined in `AndroidManifest.xml` when trying to use the networking stack layer.  Right now, there is no indicating what's happening in any networking libraries should you forget this Internet permission.

* Fix importing existing Android Studio projects so that you can't blindly import a folder or `app/build.gradle` (https://github.com/codepath/android_guides/wiki/Getting-Started-with-Gradle#importing-existing-android-studio-projects)

* Fix support libraries not to require custom `app` namespaces (i.e. when using the [ActionBar compatibility library](http://guides.codepath.com/android/Defining-The-ActionBar#adding-action-items))