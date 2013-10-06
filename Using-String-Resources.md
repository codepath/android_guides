## Overview

In Android, almost everything is a **resource**. [Defining resources](http://developer.android.com/guide/topics/resources/providing-resources.html) that you can then [access in your app](http://developer.android.com/guide/topics/resources/accessing-resources.html) is an essential part of Android development.

Resources are used for anything from defining colors, images, layouts, menus, and string values. The value of this is that nothing is hardcoded. Everything is defined in these resource files and then can be referenced within your application's code. The simplest of these resources and the most common is using **string resources** to allow for flexible, localized text. 

### Defining a String Resource

For every piece of text you want to display within your application (i.e the label of a button, or the text inside a TextView), you should first define the text in the `res/values/strings.xml` file. Each entry is a key (representing the id of the text) and a value (the text itself). For example, if I want a button to display "Submit", I might add the following string resource to `res/values/strings.xml`:

```
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string name="hello">Hello!</string>
    <string name="submit_label">Submit</string>
</resources>
```

Now if I ever reference the string resource for `submit_label`, the default will be for "Submit" to be displayed. Later though, you could create **qualified resource files** that change this value for different countries or between devices.

### Referencing a Resource

Now that we have defined our string resource, we can access that resource in either our Java code or our XML layouts at any time. To access, the resource in the XML Layout file, simply use the `@` syntax used to access any resource:

```xml
<Button
    android:layout_width="fill_parent"
    android:layout_height="wrap_content"
    android:text="@string/submit_label" />
```

To access the resource directly in your Java code, simply use the `getResources.getString` or `getString` methods to access the value given the resource id:

```java
var submitText = getResources().getString(R.string.submit_label)
```

And the string value will be retrieved. This similar pattern works for almost any resource from images (drawables) to colors. Although each resource is defined within different folders and files within the `res` directory.

## References

 * <http://developer.android.com/guide/topics/resources/string-resource.html>
 * <http://developer.android.com/guide/topics/resources/accessing-resources.html>
 * <http://mobile.tutsplus.com/tutorials/android/android-string/>
 * <http://developer.android.com/guide/topics/resources/providing-resources.html>