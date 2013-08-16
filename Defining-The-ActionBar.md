## Overview

The ActionBar is a consistent navigation element that is standard throughout modern Android applications. The ActionBar can consist of:

 * An application icon
 * An application or activity-specific title
 * Primary action buttons for an activity
 * Consistent navigation (including tabbed UI)

Important to note that prior to 3.0, there was no ActionBar although there is a compatibility library that comes bundled that provides limited ActionBar functionality to older versions. There is also a very popular library called [ActionBarSherlock](http://actionbarsherlock.com/) which provides much better compatibility for older versions (including support for tabbed interfaces).

## ActionBar Basics

Every application unless otherwise specified has an ActionBar by default. The ActionBar by default just has the application icon and an activity title. 

In any Java activity, you can call `getActionBar()` to retrieve a reference to the [ActionBar](http://developer.android.com/reference/android/app/ActionBar.html) and modify or access properties:

```java
ActionBar actionBar = getSupportActionBar();
String title = actionBar.getTitle();
actionBar.hide();
```

### Adding Action Items

When you want to add primary actions to the ActionBar, you simply need to add the items to the activity context menu and if properly specified, they will automatically appear at the top.

An activity populates the action bar in its onCreateOptionsMenu() method:

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
        android:id="@+id/item1"
        android:icon="@drawable/ic_compose"
        android:showAsAction="ifRoom"
        android:title="Compose">
    </item>
    <item
        android:id="@+id/item2"
        android:icon="@drawable/ic_profile"
        android:onClick="onProfileView"
        android:showAsAction="ifRoom|withText"
        android:title="Profile">
    </item>
</menu>
```

Notice that to request that an item appear directly in the action bar as an action button, include `showAsAction="ifRoom"` in the `<item>` tag. If there's not enough room for the item in the action bar, it will appear in the action overflow. If `withText` is specified as well (as in the second item), the text will be displayed with the icon.

### Handling ActionBar Clicks



## References

 * <http://developer.android.com/guide/topics/ui/actionbar.html>
 * <http://developer.android.com/design/patterns/actionbar.html>
 * <http://developer.android.com/reference/android/app/ActionBar.html>
 * <http://www.vogella.com/articles/AndroidActionBar/article.html>
 * <http://jgilfelt.github.io/android-actionbarstylegenerator>