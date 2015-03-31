## Overview

Android applications should always be neatly organized with a clear folder structure that makes your code easy to read. 

There are several best practices for organising your app's package structure.

#### Organize packages by category

[The way to do this]((http://blog.smartlogic.io/2013/07/09/organizing-your-android-development-code-structure)) is to group things together by their category. Each component goes to the corresponding package:

* `com.example.myapp.activities` - Contains all activities
* `com.example.myapp.adapters` - Contains all custom adapters
* `com.example.myapp.models`   - Contains all our data models
* `com.example.myapp.fragments` - Contains all fragments
* `com.example.myapp.helpers` - Contains all helpers (custom code that supports the app).
* `com.example.myapp.interfaces` - Contains all interfaces

Keeping these folders in each app means that code is logically organized and scanning the code is a pleasant experience.

#### Organize packages by application features

Package-by-feature uses packages to [reflect the feature set](http://www.javapractices.com/topic/TopicAction.do?Id=205). Consider the following package structure:

* `com.example.myapp.service.*` - Is a subpackage for all background related service packages/classes
* `com.example.myapp.ui.*` - Is a subpackage for all UI-related packages/classes
* `com.example.myapp.ui.mainscreen` - Contains classes related to some app's Main Screen
* `com.example.myapp.ui.detailsscreen` - Contains classes related to some app's Item Details Screen

This feature allows you to place `DetailsActivity`, `DetailsFragment`, `DetailsListAdapter`, `DetailsItemModel` in one package, which provides comfortable navigation when you're working on "item details" feature.

`DetailsListAdapter` and `DetailsItemModel` classes and/or their properties can be made package-private, and thus not exposed outside of the package. Within the package you may access their properties directly without generating tons of boilerplate "setter" methods.

This can make object creation really simple and intuitive, while objects remain immutable outside the package.


##Conclusion

It is up to you to decide which of the aforementioned approaches suits your project best. 

However, in general Java programming, packaging apps by feature is considered preferable and [makes a lot of sense](http://www.javapractices.com/topic/TopicAction.do?Id=205).

## References

* <http://blog.smartlogic.io/2013/07/09/organizing-your-android-development-code-structure>
* <http://stackoverflow.com/questions/5525872/android-project-package-structure>
* <http://www.javapractices.com/topic/TopicAction.do?Id=205>