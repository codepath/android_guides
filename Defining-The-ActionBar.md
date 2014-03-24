## Overview

The ActionBar is a consistent navigation element that is standard throughout modern Android applications. The ActionBar can consist of:

 * An application icon
 * An application or activity-specific title
 * Primary action buttons for an activity
 * Consistent navigation (including tabbed UI)

Important to note that prior to 3.0, there was no ActionBar although there is a compatibility library that comes bundled that provides limited ActionBar functionality to older versions. There is also a very popular library called [ActionBarSherlock](http://actionbarsherlock.com/) which provides much better compatibility for older versions (including support for tabbed interfaces).

## ActionBar Basics

Every application unless otherwise specified has an ActionBar by default. The ActionBar by default just has the application icon and an activity title. 

![ActionBar](http://i.imgur.com/DighSEo.png)

### Changing the ActionBar Icon or Title

The ActionBar icon and title displayed at the top of the screen is governed by the `AndroidManifest.xml` file in the `application` and `activity` nodes:

```xml
<application
        android:icon="@drawable/ic_launcher"
        android:label="@string/app_name"
        android:theme="@style/AppTheme">
        <activity
            android:name="com.codepath.example.simpleapp.FirstActivity"
            android:label="@string/app_name" >
        </activity>
</application>
```

Change the `android:label` or `android:icon` to modify the ActionBar icon or title for a given activity or for the application as a whole. In any Java activity, you can call `getActionBar()` to retrieve a reference to the [ActionBar](http://developer.android.com/reference/android/app/ActionBar.html) and modify or access properties:

```java
ActionBar actionBar = getActionBar();
String title = actionBar.getTitle();
actionBar.hide();
```

You can also change many other properties of the ActionBar not covered here. See the [Extended ActionBar Guide](https://github.com/thecodepath/android_guides/wiki/Extended-ActionBar-Guide) for more details.

### Adding Action Items

When you want to add primary actions to the ActionBar, you simply need to add the items to the activity context menu and if properly specified, they will automatically appear at the top.

An activity populates the action bar in its `onCreateOptionsMenu()` method:

```java
public class MainActivity extends Activity {
    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        // Inflate the menu; this adds items to the action bar if it is present.
        getMenuInflater().inflate(R.menu.main, menu);
        return true;
    }
}
```

Entries in the action bar are typically called actions. Use this method to inflate a menu resource that defines all the action items within a `res/menu` xml file, for example: 

```xml
<menu xmlns:android="http://schemas.android.com/apk/res/android" >
    <item
        android:id="@+id/miCompose"
        android:icon="@drawable/ic_compose"
        android:onClick="onComposeAction"
        android:showAsAction="ifRoom"
        android:title="Compose">
    </item>
    <item
        android:id="@+id/miProfile"
        android:icon="@drawable/ic_profile"
        android:onClick="onProfileView"
        android:showAsAction="ifRoom|withText"
        android:title="Profile">
    </item>
</menu>
```

Notice that to request that an item appear directly in the action bar as an action button, include `showAsAction="ifRoom"` in the `<item>` tag. If there's not enough room for the item in the action bar, it will appear in the action overflow. If `withText` is specified as well (as in the second item), the text will be displayed with the icon.

**Note:** To generate ActionBar icons, be sure to use the **IconSet Generator** to create the icons. Follow [this step-by-step guide](http://imgur.com/a/8cmLM) to generate icons by selecting **File => New => Other => Android => Android Icon Set**.

### Handling ActionBar Clicks

There are two ways to handle the click for an ActionBar item. First, you can use the `android:onClick` handler in the menu XML, similar to handling button clicks:

```xml
<menu xmlns:android="http://schemas.android.com/apk/res/android" >
    <item
        android:id="@+id/item1"
        android:icon="@drawable/ic_compose"
        android:onClick="onComposeAction"
        android:showAsAction="ifRoom"
        android:title="Compose">
    </item>
</menu>
```

and then define the method `onComposeAction` in the parent activity:

```java
public class MainActivity extends Activity {
  public void onComposeAction(MenuItem mi) {
     // handle click here
  }
}
```

The second way is to use the `onOptionsItemSelected()` method. Using the MenuItem passed to this method, you can identify the action by calling getItemId(). This returns the unique ID provided by the <item> tag's id attribute so you can perform the appropriate action:

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

## References

 * <http://developer.android.com/guide/topics/ui/actionbar.html>
 * <http://developer.android.com/design/patterns/actionbar.html>
 * <http://developer.android.com/reference/android/app/ActionBar.html>
 * <http://www.vogella.com/articles/AndroidActionBar/article.html>
 * <http://jgilfelt.github.io/android-actionbarstylegenerator>