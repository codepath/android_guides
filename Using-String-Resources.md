## Overview

In Android, almost everything is a **resource**. [Defining resources](http://developer.android.com/guide/topics/resources/providing-resources.html) that you can then [access in your app](http://developer.android.com/guide/topics/resources/accessing-resources.html) is an essential part of Android development.

Resources are used for anything from defining colors, images, layouts, menus, and string values. The value of this is that nothing is hardcoded. Everything is defined in these resource files and then can be referenced within your application's code. The simplest of these resources and the most common is using **string resources** to allow for flexible, localized text. 

### Types of Resources

The following are the most common types of resources within Android apps:

| Name                 | Folder    | Description                                     |
| ----                 | ------    | -----------                                     |
| Property Animations  | animator/ | XML files that define property animations.      |
| Tween Animations     | anim/     | XML files that define tween animations.         |
| Drawables            | drawable/ | Bitmap files or XML files that act as graphics  | 
| Layout               | layout/   | XML files that define a user interface layout   |
| Menu                 | menu/     | XML files that define menus or action bar items |
| Values               | values/   | XML files with values such as strings, integers, and colors. |

For the full list, check out the [Providing a Resource](http://developer.android.com/guide/topics/resources/providing-resources.html) guide.

### Defining a String Resource

For every piece of text you want to display within your application (i.e the label of a button, or the text inside a TextView), you should first define the text in the `res/values/strings.xml` file. Each entry is a key (representing the id of the text) and a value (the text itself). For example, if I want a button to display "Submit", I might add the following string resource to `res/values/strings.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string name="hello">Hello!</string>
    <string name="submit_label">Submit</string>
</resources>
```

Now if I ever reference the string resource for `submit_label`, the default will be for "Submit" to be displayed. Later though, you could create **qualified resource files** that change this value for different countries or between devices. We can also store more complex strings (with html or special characters) by using CDATA to escape the string such as:

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
  <string name="feedback_label">
  <![CDATA[
    Please <a href="http://highlight.com">let us know</a> if you have feedback on this or if 
    you would like to log in with another identity service. Thanks! This is a longer string!  
  ]]>
  </string>
</resources>
```

### Referencing a Resource

Now that we have defined our string resource, we can access that resource in either our Java code or our XML layouts at any time. To access, the resource in the XML Layout file, simply use the `@` syntax used to access any resource:

```xml
<Button
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:text="@string/submit_label" />
```

To access the resource directly in your Java code, simply use the `getResources.getString` or `getString` methods to access the value given the resource id:

```java
var submitText = getResources().getString(R.string.submit_label)
```

And the string value will be retrieved. This similar pattern works for almost any resource from images (drawables) to colors. The `getResources()` method returns a [Resources](http://developer.android.com/reference/android/content/res/Resources.html) object with many resource fetching methods. Each resource is defined within different folders and files within the `res` directory depending on their type.

### Dynamic Resource Retrieval

In certain cases, you might want to dynamically retrieve resources using just the key name rather than by "hardcoding" the resource id. For example, suppose I wanted to retrieve the "submit_label" string based on just that key name alone. This can be achieved using the `getIdentifier` method within an Activity:

```java
public String getStringValue(String key) {
    // Retrieve the resource id
    String packageName = getBaseContext().getPackageName();
    Resources resources = getBaseContext().getResources();
    int stringId = resources.getIdentifier(key, "string", packageName);
    if (stringId == 0) { return null; }
    // Return the string value based on the res id
    return resources.getString(stringId);
}
```

Now you can reference your string resources dynamically using:

```java
public String myKey = "submit_label"; // Maps to R.string.submit_label
public String myStringValue = getStringValue(myKey); // Returns string text
```

This can be done similarly for other types of resources as well. For example, for dynamically retrieving a view by a string ID:

```java
// getViewByStringId("tvTest");
public View getViewByStringId(String id) {
    // Retrieve the resource id
    String packageName = getBaseContext().getPackageName();
    Resources resources = getBaseContext().getResources();
    int viewId = resources.getIdentifier(id, "id", packageName);
    if (viewId == 0) { return null; }
    // Return the string value based on the res id
    return findViewById(viewId);
}
```

Check out the [getResources](http://developer.android.com/reference/android/content/res/Resources.html) object and <a href="http://developer.android.com/reference/android/content/res/Resources.html#getIdentifier(java.lang.String, java.lang.String, java.lang.String)">getIdentifier</a> for more details.

## References

 * <http://developer.android.com/guide/topics/resources/string-resource.html>
 * <http://developer.android.com/guide/topics/resources/accessing-resources.html>
 * <http://mobile.tutsplus.com/tutorials/android/android-string/>
 * <http://developer.android.com/guide/topics/resources/providing-resources.html>