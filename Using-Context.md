## Overview

Context provides access to information about the application state.  It provides Activities, Fragments, and Services access to resource files, images, themes/styles, and external directory locations.  It also enables access to Android's built-in services, such as those used for layout inflation, keyboard, and finding content providers.

## Application vs Activity Context

While themes and styles are usually applied at the application level, they can be also specified at the Activity level.  In this way, the activity can have a different set of themes or styles than the rest of the application.   You will notice in the `AndroidManifest.xml` file that there is typically an `android:theme` for the application.  We can also specify a different one for an Activity:

```xml
<application
       android:allowBackup="true"
       android:icon="@mipmap/ic_launcher"
       android:label="@string/app_name"
       android:theme="@style/AppTheme" >
       <activity
           android:name=".MainActivity"
           android:label="@string/app_name"
           android:theme="@style/MyCustomTheme">
```

For this reason, it's important to know that there is an Application Context and an Activity Context, which last for the duration of their respective lifecycle.  Most [[Views|Defining-Views-and-their-Attributes]] should be passed an Activity Context in order to gain access to what themes, styles, dimensions should be applied.   If no theme is specified explicitly for the Activity, the system defaults are used.

In most cases, you should use the Activity Context.  Normally, the keyword `this` in Java references the instance of the class and can be used whenever Context is needed inside an Activity.  The example below shows how Toast messages can be displayed using this approach:

```java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);  
        Toast.makeText(this, "hello", Toast.LENGTH_SHORT).show();
    }
}
```

### Anonymous functions

When using anonymous functions when implementing an `OnClick` listener in Java, the keyword `this` in Java applies to the click handler being declared.  In these cases, the outer class `MainActivity` has to specified too to refer to the Activity instance.

```java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {

        TextView tvTest = (TextView) findViewById(R.id.abc);
        tvTest.setOnClickListener(new View.OnClickListener() {
              @Override
              public void onClick(View view) {
                  Toast.makeText(MainActivity.this, "hello", Toast.LENGTH_SHORT).show();
              }
          });
        }
```

### ArrayAdapters

When constructing adapters for a [[ListView|Using-an-ArrayAdapter-with-ListView#defining-the-adapter]] or a [[RecyclerView|Using-the-RecyclerView]], typically `getContext()` is usually called during the layout inflation process.  This method usually uses the context that instantiated the ArrayAdapter:

```java
     if (convertView == null) {
        convertView = LayoutInflater.from(getContext()).inflate(R.layout.item_user, parent, false);
     }
```

If you instantiate the ArrayAdapter however with the application context, you are likely to notice that the themes/styles are not being applied.  For this reason, make sure you are passing in the Activity context in these cases.

### Avoiding memory leaks

The Application Context is typically used when singleton instances need to be created, such as a custom manager class that requires Context information to gain access to system services but gets reused across multiple Activities.  Since retaining a reference to the Activity context would cause memory not to be reclaimed after it is no long running, it's important to use the Application Context instead.

In the example below, if the context being stored is an Activity or Service and it's destroyed by the Android system, it won't be able to be garbage collected because the CustomManager class holds a static reference to it.

```java
pubic class CustomManager {
    private static CustomManager sInstance;

    public static CustomManager getInstance(Context context) {
        if (sInstance == null) {

            // This class will hold a reference to the context
            // until it's unloaded. The context could be an Activity or Service.
            sInstance = new CustomManager(context);
        }

        return sInstance;
    }

    private Context mContext;

    private CustomManager(Context context) {
        mContext = context;
    }
}
```

### Proper storing of a context: use the application context

To avoid memory leaks, never hold a reference to a context beyond its lifecycle.  Check any of your background threads, pending handlers, or inner classes that may be holding onto context objects as well.

The right way is to store the application context in `CustomManager.getInstance()`.  The application context is a singleton and is tied to the lifespan of application's process, making it safe to store a reference to it.

Use the application context when a context reference is needed beyond the lifespan of a component, or if it should be independent of the lifecycle of the context being passed in.

```java
public static CustomManager getInstance(Context context) {
    if (sInstance == null) {

        // When storing a reference to a context, use the application context.
        // Never store the context itself, which could be a component.
        sInstance = new CustomManager(context.getApplicationContext());
    }

    return sInstance;
}
```

# References
* [What is a context - Simple Code Stuffs](http://www.simplecodestuffs.com/what-is-context-in-android/)
* [Context, what context - Possible Mobile](https://possiblemobile.com/2013/06/context/)
* [Context: What, where, & how? - 101 Apps](http://www.101apps.co.za/index.php/articles/all-about-using-android-s-context-class.html)
* [What is a context - Stack Overflow](http://stackoverflow.com/questions/3572463/what-is-context-in-android)
* [Avoiding memory leaks](http://android-developers.blogspot.com.tr/2009/01/avoiding-memory-leaks.html)
