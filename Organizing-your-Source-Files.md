## Overview

Android applications should always be [neatly organized with a clear folder structure](http://blog.smartlogicsolutions.com/2013/07/09/organizing-your-android-development-code-structure/) that makes your code easy to read. The way to do this is to logically group related things together into sub-packages within your application. Make sure every app has the following subpackages:

* `com.example.myapp.activities` - Contains all activities
* `com.example.myapp.adapters` - Contains all custom adapters
* `com.example.myapp.models`   - Contains all our data models
* `com.example.myapp.fragments` - Contains all fragments
* `com.example.myapp.helpers` - Contains all helpers (custom code that supports the app).
* `com.example.myapp.interfaces` - Contains all interfaces

Keeping these folders in each app means that code is logically organized and scanning the code is a pleasant experience.

## References

* <http://blog.smartlogic.io/blog/2013/07/09/organizing-your-android-development-code-structure>