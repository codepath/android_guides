## Overview

The ActionBar, now known as the [App Bar](https://www.google.com/design/spec/layout/structure.html#structure-app-bar), is a consistent navigation element that is standard throughout modern Android applications. The ActionBar can consist of:

 * An application icon
 * An "upward" navigation to logical parent
 * An application or activity-specific title
 * Primary action icons for an activity
 * Consistent navigation (including navigation drawer)

Important to note is that prior to 3.0, there was no ActionBar. In 2013, Google announced a [support library](http://android-developers.blogspot.com/2013/08/actionbarcompat-and-io-2013-app-source.html) that provides much better compatibility for older versions and support for tabbed interfaces.  Since most of the examples below depend on this support library, make sure to include the [[AppCompat library|Migrating-to-the-AppCompat-Library]].  

## ActionBar Basics

Every application unless otherwise specified has an ActionBar by default. The ActionBar by default now has just the title for the current activity.

![ActionBar](https://i.imgur.com/vAoRWEF.png)

### Changing the ActionBar Title

The ActionBar title displayed at the top of the screen is governed by the `AndroidManifest.xml` file within the `activity` nodes. In the example below, the activity "FirstActivity" will have an ActionBar with the string value of the resource identified by `@string/activity_name`. If the value of that resource is "Foo," the string displayed in the ActionBar for this activity will be "Foo." Note that the `application` node can supply a `android:label` that acts as the default for activities and components with no other specified label.

```xml
<application
        android:icon="@drawable/ic_launcher"
        android:label="@string/app_name"
        android:theme="@style/AppTheme">
        <activity
            android:name="com.codepath.example.simpleapp.FirstActivity"
            android:label="@string/activity_name" >
        </activity>
</application>
```

Change the `android:label` or `android:icon` to modify the ActionBar title or icon for a given activity or for the application as a whole. In any Java activity, you can also call `getSupportActionBar()` to retrieve a reference to the [ActionBar](http://developer.android.com/reference/android/support/v7/app/ActionBar.html) and modify or access any properties of the ActionBar at runtime:

```java
ActionBar actionBar = getSupportActionBar(); // or getActionBar();
getSupportActionBar().setTitle("My new title"); // set the top title
String title = actionBar.getTitle().toString(); // get the title
actionBar.hide(); // or even hide the actionbar
```

You can also change many other properties of the ActionBar not covered here. See the [[Extended ActionBar Guide|Extended-ActionBar-Guide]] for more details.

### Displaying ActionBar Icon

In the new Android 5.0 material design guidelines, the style guidelines have changed to **discourage the use of the icon** in the ActionBar. Although the icon can be added back with:

```java
getSupportActionBar().setDisplayShowHomeEnabled(true);
getSupportActionBar().setLogo(R.mipmap.ic_launcher);
getSupportActionBar().setDisplayUseLogoEnabled(true);
```

The above code results in:

<img src="https://i.imgur.com/gBPAIFu.png" width="400" />

You can read more about this on the [material design guidelines](https://developer.android.com/reference/android/support/v7/widget/Toolbar.html) which state: "The use of application icon plus title as a standard layout is discouraged on API 21 devices and newer." 

### Adding Action Items

When you want to add primary actions to the ActionBar, you add the items to the activity context menu and if properly specified, they will automatically appear at the top right as icons in the ActionBar.

An activity populates the ActionBar from within the `onCreateOptionsMenu()` method:

```java
public class MainActivity extends AppCompatActivity {
    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        // Inflate the menu; this adds items to the action bar if it is present.
        getMenuInflater().inflate(R.menu.menu_main, menu);
        return true;
    }
}
```

Entries in the action bar are typically called actions. Use this method to inflate a menu resource that defines all the action items within a `res/menu/menu_main.xml` file, for example:

```xml
<!-- Menu file for `activity_movies.xml` is located in a file 
     such as `res/menu/menu_movies.xml`  -->
<menu xmlns:android="http://schemas.android.com/apk/res/android"
      xmlns:app="http://schemas.android.com/apk/res-auto">
    <item
        android:id="@+id/miCompose"
        android:icon="@drawable/ic_compose"
        app:showAsAction="ifRoom"
        android:title="Compose">
    </item>
    <item
        android:id="@+id/miProfile"
        android:icon="@drawable/ic_profile"
        app:showAsAction="ifRoom|withText"
        android:title="Profile">
    </item>
</menu>
```

You also should note that the `xmlns:app` namespace must be defined in order to leverage the `showAsAction` option.  The reason is that a [compatibility library](Migrating-to-the-AppCompat-Library#menu-xml-changes) is used to support the `showAsAction="ifRoom"` option.  This option is needed to show the item directly in the action bar as an icon. If there's not enough room for the item in the action bar, it will appear in the action overflow. If `withText` is specified as well (as in the second item), the text will be displayed with the icon.

The above code results in two action icons being displayed:

<img src="https://i.imgur.com/yI4akxQ.png" width="400" />

**Note:** The above code refers to the `@drawable/ic_compose` and `@drawable/ic_profile` resources which would have to exist for this to compile. To generate ActionBar icons, be sure to use the **Asset Studio** in Android Studio. To create a new Android icon set, right click on a `res/drawable` folder and invoke **New -> Image Asset**.

### Handling ActionBar Clicks

There are two ways to handle the click for an ActionBar item. The first approach is you can use the `android:onClick` handler in the menu XML, similar to handling button clicks:

```xml
<menu xmlns:android="http://schemas.android.com/apk/res/android"
      xmlns:app="http://schemas.android.com/apk/res-auto">
    <item
        android:id="@+id/item1"
        android:icon="@drawable/ic_compose"
        android:onClick="onComposeAction"
        app:showAsAction="ifRoom"
        android:title="Compose">
    </item>
</menu>
```

and then define the method `onComposeAction` in the parent activity before attempting to run the application or an exception will be thrown for the missing method:

```java
public class MainActivity extends AppCompatActivity {
  public void onComposeAction(MenuItem mi) {
     // handle click here
  }
}
```

The second approach is to use the `onOptionsItemSelected()` method. Using the MenuItem passed to this method, you can identify the action by calling getItemId(). This returns the unique ID provided by the item tag's id attribute so you can perform the appropriate action:

```java
@Override
public boolean onOptionsItemSelected(MenuItem item) {
    // Handle presses on the action bar items
    switch (item.getItemId()) {
        case R.id.miCompose:
            composeMessage();
            return true;
        case R.id.miProfile:
            showProfileView();
            return true;
        default:
            return super.onOptionsItemSelected(item);
    }
}
```

and then you can handle all the action buttons in this single method.

## Understanding ToolBar

[[ToolBar|Using-the-App-Toolbar]] was introduced in Android Lollipop, API 21 release and is the spiritual successor of the ActionBar. It's a `ViewGroup` that can be placed anywhere in your layout. ToolBar's appearance can be more easily customized than the ActionBar. 

[[<img src="https://i.imgur.com/0auGknf.png" width="500" />|Using-the-App-Toolbar]]

ToolBar works well with apps targeted to API 21 and above. However, Android has updated the AppCompat support libraries so the ToolBar can be used on lower Android OS devices as well. In AppCompat, ToolBar is implemented in the `android.support.v7.widget.Toolbar` class. Refer to the [[ToolBar Guide|Using-the-App-Toolbar]] for more information.

## References

 * <http://developer.android.com/guide/topics/ui/actionbar.html>
 * <http://developer.android.com/design/patterns/actionbar.html>
 * <http://developer.android.com/reference/android/app/ActionBar.html>
 * <http://www.vogella.com/articles/AndroidActionBar/article.html>
 * <http://jgilfelt.github.io/android-actionbarstylegenerator>
 * <http://android-developers.blogspot.com/2014/10/appcompat-v21-material-design-for-pre.html>
