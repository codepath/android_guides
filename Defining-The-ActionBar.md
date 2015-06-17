## Overview

The ActionBar is a consistent navigation element that is standard throughout modern Android applications. The ActionBar can consist of:

 * An application icon
 * An application or activity-specific title
 * Primary action buttons for an activity
 * Consistent navigation (including tabbed UI)


Important to note that prior to 3.0, there was no ActionBar. In 2013, Google announced a [support library](http://android-developers.blogspot.com/2013/08/actionbarcompat-and-io-2013-app-source.html) that provides much better compatibility for older versions and support for tabbed interfaces.  Since most of the examples below depend on this support library, make sure to include the [[AppCompat library|Migrating-to-the-AppCompat-Library]].  

## ActionBar Basics

Every application unless otherwise specified has an ActionBar by default. The ActionBar by default just has the application icon and an activity title.

![ActionBar](http://i.imgur.com/DighSEo.png)

### Changing the ActionBar Icon or Title

The ActionBar icon and title displayed at the top of the screen is governed by the `AndroidManifest.xml` file within the `activity` nodes. In the example below, the activity "FirstActivity" will have an ActionBar with the string value of the resource identified by `@string/activity_name`. If the value of that resource is "Foo," the string displayed in the ActionBar for this activity will be "Foo." Note that the `application` node can supply a `android:label` that acts as the default for activities and components with no other specified label.

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

Change the `android:label` or `android:icon` to modify the ActionBar icon or title for a given activity or for the application as a whole. In any Java activity, you can also call `getSupportActionBar()` to retrieve a reference to the [ActionBar](http://developer.android.com/reference/android/support/v7/app/ActionBar.html) and modify or access any properties of the ActionBar at runtime:

```java
ActionBar actionBar = getSupportActionBar(); // or getActionBar();
String title = actionBar.getTitle().toString();
actionBar.hide();
```

You can also change many other properties of the ActionBar not covered here. See the [[Extended ActionBar Guide|Extended-ActionBar-Guide]] for more details.

### Displaying ActionBar Icon

In the new Android 5.0 material design guidelines, the style guidelines have changed to **discourage the use of the icon** in the ActionBar. Although the icon can be added back with:

```java
getSupportActionBar().setDisplayShowHomeEnabled(true);
getSupportActionBar().setLogo(R.drawable.ic_launcher);
getSupportActionBar().setDisplayUseLogoEnabled(true);
```

You can read more about this on the [material design guidelines](https://developer.android.com/reference/android/support/v7/widget/Toolbar.html) which state: "The use of application icon plus title as a standard layout is discouraged on API 21 devices and newer." 

### Adding Action Items

When you want to add primary actions to the ActionBar, you add the items to the activity context menu and if properly specified, they will automatically appear at the top.

An activity populates the action bar in its `onCreateOptionsMenu()` method:

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

Entries in the action bar are typically called actions. Use this method to inflate a menu resource that defines all the action items within a `res/menu` xml file, for example:

```xml
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

Notice that to request that an item appear directly in the action bar as an action button, include `showAsAction="ifRoom"` in the `<item>` tag. If there's not enough room for the item in the action bar, it will appear in the action overflow. If `withText` is specified as well (as in the second item), the text will be displayed with the icon.

**Note:** The above code refers to the `@drawable/ic_compose` resource which would have to exist for this to compile. To generate ActionBar icons, be sure to use the **Asset Studio** in Android Studio. To create a new Android icon set, right click on a `res\drawable` folder and invoke **New->Image Asset**.

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

## ToolBar Basics
ToolBar was introduced in Android Lollipop, API 21 release and is a complete replacement to ActionBar. It's a ViewGroup so you can place it anywhere in your layout. ToolBar also provides greater control to customize its appearance for the same reason.

ToolBar works well with apps targeted to API 21 and above. However, Android has updated the AppCompat support libraries so the ToolBar can be used on lower Android OS devices as well. In AppCompat, ToolBar is implemented in the `android.support.v7.widget.Toolbar` class.

There are two ways to use Toolbar:
 * Use a Toolbar as an Action Bar when you want to use the existing Action Bar facilities (such as menu inflation and selection, ActionBarDrawerToggle, and so on) but want to have more control over its appearance.
 * Use a standalone Toolbar when you want to use the pattern in your app for situations that an Action Bar would not support; for example, showing multiple toolbars on the screen, spanning only part of the width, and so on.

### Using ToolBar as ActionBar

To use Toolbar as an ActionBar, first ensure the AppCompat-v7 support library is added to your application `build.gradle` (Module:app) file:

```gradle
dependencies {
  compile fileTree(dir: 'libs', include: ['*.jar'])
  compile 'com.android.support:appcompat-v7:22.2.0+'
}
```

Second, let's disable the theme-provided ActionBar. The easiest way is to have your theme extend from `Theme.AppCompat.NoActionBar` (or its light variant) in `styles.xml` file.

```xml
<resources>
  <!-- Base application theme. -->
  <style name="AppTheme" parent="Theme.AppCompat.Light.NoActionBar">
  </style>
</resources>
```

Now you need to add `Toolbar` to your Activity layout file. One of the biggest advantages of using the Toolbar widget is that you can place the view anywhere in your layout. Below it's placed at the top of the LinearLayout, like the standard ActionBar, but it can be contained in any other type of layout as well.

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <android.support.v7.widget.Toolbar
      android:id="@+id/toolbar"
      android:minHeight="?attr/actionBarSize"  
      android:layout_width="match_parent"
      android:layout_height="wrap_content"
      android:background="?attr/colorPrimary">
    </android.support.v7.widget.Toolbar>

    ...

</LinearLayout>
```
As Toolbar is just a `ViewGroup`, it can be styled and positioned like any other view. Then in your Activity or Fragment, set the Toolbar to act as your ActionBar by using the `setSupportActionBar(Toolbar)` method.  

*Note:* When using the support library, make sure that you are importing `android.support.v7.widget.Toolbar` and not `android.widget.Toolbar`.

```java
import android.support.v7.app.AppCompatActivity;
import android.support.v7.widget.Toolbar;

public class MyActivity extends AppCompatActivity{

  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_my);

    // Set a ToolBar to replace the ActionBar.
    Toolbar toolbar = (Toolbar) findViewById(R.id.toolbar);
    setSupportActionBar(toolbar);
  }
}
```

From this point on, all menu items are displayed in your Toolbar, populated via the standard options menu callbacks.

## References

 * <http://developer.android.com/guide/topics/ui/actionbar.html>
 * <http://developer.android.com/design/patterns/actionbar.html>
 * <http://developer.android.com/reference/android/app/ActionBar.html>
 * <http://www.vogella.com/articles/AndroidActionBar/article.html>
 * <http://jgilfelt.github.io/android-actionbarstylegenerator>
 * <http://android-developers.blogspot.com/2014/10/appcompat-v21-material-design-for-pre.html>