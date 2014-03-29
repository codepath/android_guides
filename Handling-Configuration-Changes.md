## Overview

There are various situations such as when the screen orientation is rotated where the Activity can actually be destroyed and removed from memory and then recreated from scratch again. In these situations, the best practice is to manage for cases where the Activity is re-created by properly saving and restoring the state.

## Saving and Restoring State

As your activity begins to stop, the system calls `onSaveInstanceState()` so your activity can save state information with a collection of key-value pairs. The default implementation of this method **automatically saves** information about the state of the activity's **view hierarchy**, such as the text in an `EditText` widget or the scroll position of a `ListView`.

To save additional state information for your activity, you must implement `onSaveInstanceState()` and add key-value pairs to the Bundle object. For example:

```java
public class MainActivity extends Activity {
    static final String SOME_VALUE = "int_value";
    static final String SOME_OTHER_VALUE = "string_value";

    @Override
    public void onSaveInstanceState(Bundle savedInstanceState) {
        // Save custom values into the bundle
        savedInstanceState.putInt(SOME_VALUE, someIntValue);
        savedInstanceState.putString(SOME_OTHER_VALUE, someStringValue);
        // Always call the superclass so it can save the view hierarchy state
        super.onSaveInstanceState(savedInstanceState);
    }
}
```

The system will call that method before an Activity is destroyed. Then later the system will call `onRestoreInstanceState` where we can restore state from the bundle:

```java
public void onRestoreInstanceState(Bundle savedInstanceState) {
    // Always call the superclass so it can restore the view hierarchy
    super.onRestoreInstanceState(savedInstanceState);
    // Restore state members from saved instance
    someIntValue = savedInstanceState.getInt(SOME_VALUE);
    someStringValue = savedInstanceState.getString(SOME_OTHER_VALUE);
}
```

Read more on the [Recreating an Activity](http://developer.android.com/training/basics/activity-lifecycle/recreating.html) guide.

## Retaining Fragments

In many cases, we can avoid problems when an Activity is re-created by simply using fragments. If your views and state are within a fragment, we can easily have the fragment be retained when the activity is re-created:

```java
public class RetainedFragment extends Fragment {
    // data object we want to retain
    private MyDataObject data;

    // this method is only called once for this fragment
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        // retain this fragment when activity is re-initialized
        setRetainInstance(true);
    }

    public void setData(MyDataObject data) {
        this.data = data;
    }

    public MyDataObject getData() {
        return data;
    }
}
```

Now you can check to see if the fragment already exists by tag before creating one and the fragment will retain it's state across configuration changes. See the [Handling Runtime Changes](http://developer.android.com/guide/topics/resources/runtime-changes.html#RetainingAnObject) guide for more details.


## Locking Screen Orientation

If you want to lock the screen orientation change of any screen (activity) of your android application you just need to set the `android:screenOrientation` property of an `<activity>` within the `AndroidManifest.xml`:

```xml
<activity
    android:name="com.techblogon.screenorientationexample.MainActivity"
    android:screenOrientation="portrait"
    android:label="@string/app_name" >
    <!-- ... -->
</activity>
```

Now that activity is forced to always be displayed in "portrait" mode. 

## Manually Managing Configuration Changes

If your application doesn't need to update resources during a specific configuration change and you have a performance limitation that requires you to avoid the activity restart, then you can declare that your activity handles the configuration change itself, which prevents the system from restarting your activity. 

However, this technique should be considered **a last resort** when you must avoid restarts due to a configuration change and is not recommended for most applications. To take this approach, we must add the `android:configChanges` node to the activity within the `AndroidManifest.xml`:

```xml
<activity android:name=".MyActivity"
          android:configChanges="orientation|screenSize|keyboardHidden"
          android:label="@string/app_name">
```

Now, when one of these configurations change, the activity does not restart. Instead, the activity receives a call to `onConfigurationChanged()`:

```java
// Within the activity which receives these changes
// Checks the current device orientation, and toasts accordingly
@Override
public void onConfigurationChanged(Configuration newConfig) {
    super.onConfigurationChanged(newConfig);

    // Checks the orientation of the screen
    if (newConfig.orientation == Configuration.ORIENTATION_LANDSCAPE) {
        Toast.makeText(this, "landscape", Toast.LENGTH_SHORT).show();
    } else if (newConfig.orientation == Configuration.ORIENTATION_PORTRAIT){
        Toast.makeText(this, "portrait", Toast.LENGTH_SHORT).show();
    }
}
```

See the [Handling the Change](http://developer.android.com/guide/topics/resources/runtime-changes.html#HandlingTheChange) docs. For more about which configuration changes you can handle in your activity, see the [android:configChanges](http://developer.android.com/guide/topics/manifest/activity-element.html#config) documentation and the [Configuration](http://developer.android.com/reference/android/content/res/Configuration.html) class.

## References

* <http://developer.android.com/guide/topics/resources/runtime-changes.html>
* <http://developer.android.com/training/basics/activity-lifecycle/recreating.html>
* <http://www.vogella.com/tutorials/AndroidLifeCycle/article.html#configurationchange>
* <http://www.androiddesignpatterns.com/2013/04/retaining-objects-across-config-changes.html>
* <http://www.intertech.com/Blog/saving-and-retrieving-android-instance-state-part-1/>
* <http://sunil-android.blogspot.com/2013/03/save-and-restore-instance-state.html>