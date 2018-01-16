## Overview

A Context provides access to information about the application state.  It provides Activities, Fragments, and Services access to resource files, images, [[themes/styles|Styles-and-Themes]], and external directory locations.  It also enables access to Android's built-in services, such as those used for layout inflation, keyboard, and finding content providers.

In many cases when the "context is required", we simply need to **pass in the instance of the current activity**. In situations where we are inside objects created by the activity such as adapters or fragments, we need to pass in the activity instance into those objects. In situations where we are outside of an activity (in an application or service), we can use the "application" context instead.

## What is a Context used for?

Below are a few use cases that require a Context object.

### Explicitly starting a component

```java
// Provide context if MyActivity is an internal activity.
Intent intent = new Intent(context, MyActivity.class);
startActivity(intent);
```

When explicitly starting a component two pieces of information are required:
* package name, which identifies the application that contains the component.
* fully-qualified Java class name for the component.

If starting an internal component, the context can be passed in since the package name of the current application can be extracted through `context.getPackageName()`.

### Creating a View

```java
TextView textView = new TextView(context);
```

Contexts contain the following information that views require:
* device screen size and dimensions for converting dp,sp to pixels
* styled attributes
* activity reference for onClick attributes

### Inflating an XML Layout file

We use the context to fetch the `LayoutInflater` in order to inflate an XML layout into memory:

```java
// A context is required when creating views.
LayoutInflater inflater = LayoutInflater.from(context);
inflater.inflate(R.layout.my_layout, parent);
```

### Sending a local broadcast

We use the context to fetch the `LocalBroadcastManager` when sending out or registering a receiver for a broadcast:

```java
// The context contains a reference to the main Looper, 
// which manages the queue for the application's main thread.
Intent broadcastIntent = new Intent("custom-action");
LocalBroadcastManager.getInstance(context).sendBroadcast(broadcastIntent);
```

### Retrieving a System Service

To send notifications from an application, the `NotificationManager` system service is required:

```java
// Context objects are able to fetch or start system services.
NotificationManager notificationManager = 
    (NotificationManager) getSystemService(NOTIFICATION_SERVICE);

int notificationId = 1;

// Context is required to construct RemoteViews
Notification.Builder builder = 
    new Notification.Builder(context).setContentTitle("custom title");

notificationManager.notify(notificationId, builder.build());
```

Refer to the **[list of all available system services](http://developer.android.com/reference/android/content/Context.html#getSystemService(java.lang.String))** that can be retrieved through a Context.

## Application vs Activity Context

While themes and styles are usually applied at the application level, they can be also specified at the Activity level.  In this way, the activity can have a different set of themes or styles than the rest of the application (e.g. if the ActionBar needs to be disabled for certain pages).   You will notice in the `AndroidManifest.xml` file that there is typically an `android:theme` for the application.  We can also specify a different one for an Activity:

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

For this reason, it's important to know that there is an Application Context and an Activity Context, which last for the duration of their respective lifecycle.  Most [[Views|Defining-Views-and-their-Attributes]] should be passed an Activity Context in order to gain access to what themes, styles, dimensions should be applied.   If no theme is specified explicitly for the Activity, the default is to use the one specified for the application.

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

When using anonymous functions when implementing listeners, the keyword `this` in Java applies to the most immediate class being declared.  In these cases, the outer class `MainActivity` has to be specified to refer to the Activity instance.  

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
    }
```

### Adapters


#### Array Adapter

When constructing adapters for a [[ListView|Using-an-ArrayAdapter-with-ListView#defining-the-adapter]], typically `getContext()` is usually called during the layout inflation process.  This method usually uses the context that instantiated the ArrayAdapter:

```java
     if (convertView == null) {
        convertView = 
            LayoutInflater
                .from(getContext())
                .inflate(R.layout.item_user, parent, false);
     }
```

If you instantiate the ArrayAdapter with the application context, however, you are likely to notice that the themes/styles are not being applied.  For this reason, make sure you are passing in the Activity context in these cases.

#### RecyclerView Adapter
```java
public class MyRecyclerAdapter extends RecyclerView.Adapter<MyRecyclerAdapter.ViewHolder> {

    @Override 
    public ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        View v = 
            LayoutInflater
                .from(parent.getContext())
                .inflate(R.layout.itemLayout, parent, false);
        
        return new ViewHolder(v);
    }

    @Override
    public void onBindViewHolder(ViewHolder viewHolder, int i) {
        // If a context is needed, it can be retrieved 
        // from the ViewHolder's root view.
        Context context = viewHolder.itemView.getContext();

        // Dynamically add a view using the context provided.
        if(i == 0) {
            TextView tvMessage = new TextView(context);
            tvMessage.setText("Only displayed for the first item.")

            viewHolder.customViewGroup.addView(tvMessage);
        }
    }

   public static class ViewHolder extends RecyclerView.ViewHolder {
       public FrameLayout customViewGroup;

       public ViewHolder(view imageView) {
           // Very important to call the parent constructor
           // as this ensures that the imageView field is populated.
           super(imageView);
           
           // Perform other view lookups.
           customViewGroup = (FrameLayout) imageView.findById(R.id.customViewGroup);
       }
   }
}
```

Whereas an `ArrayAdapter` requires a context to be passed into it's constructor, a `RecyclerView.Adapter` does not.  Instead, the correct context can be inferred from the parent view when inflation is necessary.

The associated `RecyclerView` always passes itself as the parent view into the `RecyclerView.Adapter.onCreateViewHolder()` call.

If a context is needed outside of the `onCreateViewHolder()` method, as long as there's a `ViewHolder` instance available, a context can be retrieved via: `viewHolder.itemView.getContext()`.  `itemView` is a public, non-null, and `final` field in the base ViewHolder class.

### Avoiding memory leaks

The Application Context is typically used when singleton instances need to be created, such as a custom manager class that requires Context information to gain access to system services but gets reused across multiple Activities.  Since retaining a reference to the Activity context would cause memory not to be reclaimed after it is no longer running, it's important to use the Application Context instead.

In the example below, if the context being stored is an Activity or Service and it's destroyed by the Android system, it won't be able to be garbage collected because the CustomManager class holds a static reference to it.

```java
public class CustomManager {
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