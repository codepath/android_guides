## Overview

Android applications should always be neatly organized with a clear folder structure that makes your code easy to read. In addition, proper naming conventions for code and classes are important to ensure your code is clean and maintainable.

### Naming Conventions

Be sure to check out the [Ribot Code and Style Guidelines](https://github.com/ribot/android-guidelines/blob/master/project_and_code_guidelines.md) for a more extensive breakdown of suggested style and naming guidelines.

#### For Java Code

The following naming and casing conventions are important for your Java code:

| Type        | Example                | Description                   | Link |
| ----------- | ------------           | ----------------------------- | ------
| Variable    | `incomeTaxRate`        | All variables should be camel case | [Read More](http://www.oracle.com/technetwork/java/javase/documentation/codeconventions-137265.html#547) |
| Constant    | `DAYS_IN_WEEK`         | All constants should be all uppercase | [Read More](http://www.oracle.com/technetwork/java/javase/documentation/codeconventions-137265.html#1255) |
| Method      | `convertToEuroDollars` | All methods should be camel case | [Read More](http://www.oracle.com/technetwork/java/javase/documentation/codeconventions-135099.html#367) |
| Parameter   | `depositAmount`        | All parameter names should be camel case | [Read More](http://www.oracle.com/technetwork/java/javase/documentation/codeconventions-141270.html#16817) |

See [this naming guide](http://www.oracle.com/technetwork/java/codeconvtoc-136057.html) for more details.

#### For Android Classes

Android classes should be named with a particular convention that makes their purpose clear in the name. For example all activities should end with `Activity` as in `MoviesActivity`. The following are the most important naming conventions:

| Name            | Convention                | Inherits            |
| ----------      | ------------------        | ------------        | 
| Activity        | `CreateTodoItemActivity`  | `AppCompatActivity`, `Activity` |
| List Adapter    | `TodoItemsAdapter`        | `BaseAdapter`, `ArrayAdapter`   |
| Database Helper | `TodoItemsDbHelper`       | `SQLiteOpenHelper`  |
| Network Client  | `TodoItemsClient`         | N/A                 |
| Fragment        | `TodoItemDetailFragment`  | `Fragment`          |
| Service         | `FetchTodoItemService`    | `Service`, `IntentService`  |

Use your best judgement for other types of files. The goal is for any Android-specific classes to be identifiable by the suffix. 

### Android Folder Structure

There are several best practices for organizing your app's package structure.

#### Organize packages by category

[The way to do this](http://blog.smartlogic.io/2013/07/09/organizing-your-android-development-code-structure) is to group things together by their category. Each component goes to the corresponding package:

* `com.example.myapp.activities` - Contains all activities
* `com.example.myapp.adapters` - Contains all custom adapters
* `com.example.myapp.models`   - Contains all our data models
* `com.example.myapp.network` - Contains all networking code
* `com.example.myapp.fragments` - Contains all fragments
* `com.example.myapp.utils` - Contains all helpers supporting code.
* `com.example.myapp.interfaces` - Contains all interfaces

Keeping these folders in each app means that code is logically organized and scanning the code is a pleasant experience. You can see a slight variation on this structure as suggested by [Futurice on their best-practices repo](https://github.com/futurice/android-best-practices#java-packages-architecture).

#### Organize packages by application features

Alternatively, we can package-by-feature rather than layers. This approach uses packages to [reflect the feature set](http://www.javapractices.com/topic/TopicAction.do?Id=205). Consider the following package structure [as outlined in this post](https://medium.com/@cesarmcferreira/package-by-features-not-layers-2d076df1964d#.f7znkie19):

* `com.example.myapp.service.*` - Is a subpackage for all background related service packages/classes
* `com.example.myapp.ui.*` - Is a subpackage for all UI-related packages/classes
* `com.example.myapp.ui.mainscreen` - Contains classes related to some app's Main Screen
* `com.example.myapp.ui.detailsscreen` - Contains classes related to some app's Item Details Screen

This feature allows you to place `DetailsActivity`, `DetailsFragment`, `DetailsListAdapter`, `DetailsItemModel` in one package, which provides comfortable navigation when you're working on "item details" feature.

`DetailsListAdapter` and `DetailsItemModel` classes and/or their properties can be made package-private, and thus not exposed outside of the package. Within the package you may access their properties directly without generating tons of boilerplate "setter" methods.

This can make object creation really simple and intuitive, while objects remain immutable outside the package.

## Organizing Resources

Resources should be split up into the following key files and folders:

| Name         | Path                      | Description |
| --------     | -----------               | ----------- |
| XML Layouts  | `res/layout/`             | This is where we put our XML layout files.     |
| XML Menus    | `res/menu/`               | This is where we put our AppBar menu actions.  |
| Drawables    | `res/drawable`            | This is where we put images and XML drawables. | 
| Colors       | `res/values/colors.xml`   | This is where we put [color definitions](http://developer.android.com/guide/topics/resources/more-resources.html#Color). |
| Dimensions   | `res/values/dimens.xml`   | This is where we put [dimension values](http://developer.android.com/guide/topics/resources/more-resources.html#Dimension). | 
| Strings      | `res/values/strings.xml`  | This is where we put strings.           |
| Styles       | `res/values/styles.xml`   | This is where we put style values.      |

See the [full list of resources here](http://developer.android.com/guide/topics/resources/providing-resources.html#ResourceTypes) and note the following:

 * **Don't hardcode color hex values in the layout**. Instead of hardcoding these values, be sure to [move all colors](http://developer.android.com/guide/topics/resources/more-resources.html#Color) into `res/values/colors.xml` and reference the colors in layouts with `@color/royal_blue`.
 * **Don't hardcode margin / padding dimensions in the layout**. Instead of hardcoding these values, be sure to [move all dimension values](http://developer.android.com/guide/topics/resources/more-resources.html#Dimension) into `res/values/dimens.xml` and reference these in layouts with `@dimen/item_padding_left`.
 * To support multiple devices, we can then use the [alternative resources system](http://guides.codepath.com/android/Understanding-App-Resources#providing-alternate-resources) to provide different colors, strings, dimens, styles, etc based on the device type, screen size, API version and much more. 

Be sure to start properly organizing your resources early on in the development of an application. Be sure to check out the [Ribot Code and Style Guidelines](https://github.com/ribot/android-guidelines/blob/master/project_and_code_guidelines.md) for a more extensive breakdown of suggested style and naming guidelines.

### Organizing Resources into Subfolders

Often there are questions about organizing not just the source files but also better organizing the [application resources](http://guides.codepath.com/android/Understanding-App-Resources). In a modern app, there are often hundreds of different layout files, drawables, styles, etc and by default these are all grouped together in a flat list within a single subdirectory (i.e `res/layout`).

In order to further organize or group your various resources, the best way is to [install the third-party folding-plugin](https://plugins.jetbrains.com/plugin/7876) for Android Studio to create virtual folders

<a href="https://github.com/dmytrodanylyk/folding-plugin"><img src="https://i.imgur.com/fOir80H.jpg" width="600" /></a>

Refer to [stackoverflow post](http://stackoverflow.com/q/4930398/313399) for a discussion of the other options. 

## Conclusion

It is up to you to decide which of the aforementioned approaches suits your project best. 

However, in general Java programming, packaging apps by feature is considered preferable and [makes a lot of sense](http://www.javapractices.com/topic/TopicAction.do?Id=205).

## References

* <http://blog.smartlogic.io/2013/07/09/organizing-your-android-development-code-structure>
* <http://stackoverflow.com/questions/5525872/android-project-package-structure>
* <http://www.javapractices.com/topic/TopicAction.do?Id=205>
