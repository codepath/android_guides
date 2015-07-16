## Overview

Context provides an interface to global information about an application environment. It allows access to application-specific resources and classes, as well as up-calls for application-level operations such as launching activities, broadcasting and receiving intents, etc.

## What is a Context

## Why are Contexts important?
### System Resources
### Themes

## Providing Contexts

## Context subclasses
Contexts follow the wrapper/decorator pattern that allow subclasses to add on only the specific feature sets required.

![context subclasses](https://raw.githubusercontent.com/codepath/android_guides/master/images/context_subclasses.png)
* ContextWrapper
* ContextThemeWrapper
* Application
* Activity
* Service

## Considerations

### Avoiding Memory Leaks
Be careful when storing references to a context instance.

#### Improper storing of a context
In the example below, if the context being stored is an Activity or Service and it's destroyed by the Android system, it won't be able to be garbage collected because the CustomManager class holds a static reference to it.

To avoid memory leaks, never hold a reference to a context beyond its lifespan.

This can also be a problem when background threads, pending handlers, or inner classes hold onto context objects as well.

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
Store the application context in `CustomManager.getInstance()`.  The application context is a singleton and is tied to the lifespan of application's process, making it safe to store a reference to it.

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
