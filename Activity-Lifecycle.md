## Background

As a user navigates throughout an app, Android maintains the visited activities in a stack, with the currently visible activity always placed at the top of the stack. 

At any point in time a particular activity can be in one of the following 4 states:

| Activity State | Description |
| -------------- |-------------|
| Running | Activity is visible and interacting with the user |
| Paused | Activity is still visible, but no longer interacting with the user |
| Stopped | Activity is no longer visible |
| Killed | Activity has been killed by the system (low memory) or its `finish()` method has been called |

## Activity Lifecycle

The following diagram shows the important state paths of an Activity. The square rectangles represent callback methods you can implement to perform operations when the Activity moves between states. These are described further in the table below the diagram.

![Imgur](https://i.imgur.com/aUd1KA1.png)

| Lifecycle Method | Description | Common Uses  |
| ------------- |-------------| -----|
| `onCreate()` | The activity is starting (but not visible to the user) | Most of the activity initialization code goes here. This is where you [setContentView()](http://developer.android.com/reference/android/app/Activity.html#setContentView(int)) for the activity, initialize views, set up any adapters, etc. |
| `onStart()` | The activity is now visible (but not ready for user interaction) | This lifecycle method isn't used much, but can come in handy to register a BroadcastReceiver to monitor for changes that impact the UI (since the UI is now visible to the user).  |
| `onResume()` | The activity is now in the foreground and ready for user interaction | This is a good place to start animations, open exclusive-access devices like the camera, etc. |
| `onPause()` | Counterpart to `onResume()`. The activity is about to go into the background and has stopped interacting with the user. This can happen when another activity is launched in front of the current activity. | It's common to undo anything that was done in onResume() and to save any global state (such as writing to a file). |
| `onStop()` | Counterpart to `onStart()`. The activity is no longer visible to the user. | It's common to undo anything that was done in `onStart()`. |
| `onDestroy()` | Counterpart to `onCreate(...)`. This can be triggered because `finish()` was called on the activity or the system needed to free up some memory. |  It's common to do any cleanup here. For example, if the activity has a thread running in the background to download data from the network, it may create that thread in onCreate() and then stop the thread here in onDestroy() |
| `onRestart()` | Called when the activity has been stopped, before it is started again | It isn't very common to need to implement this callback.    |

### Handling Configuration Changes

The Activity lifecycle is especially important because whenever an activity leaves the screen, **the activity can be destroyed**. When an activity is destroyed, when the user returns to the activity, the activity will be re-created and the lifecycle methods will be called again. Activities are also **re-created whenever the orientation changes** (i.e the screen is rotated). In order to ensure your application is robust to recreation amongst a wide variety of situations, be sure to review the [[handling configuration changes|Handling-Configuration-Changes]].

### Calling the super class

When overriding any of the methods, you may need to call the superclass implementation.  The rule of thumb is that during initialization, you should always call the superclass first:

```java
public void onCreate() {
   super.onCreate();
   // do work after super class function
   // setContentView(R.layout.main);
}
```

During de-initialization, you should do the work first before calling the super class:

```java
public void onPause() {
   // do work here first before super class function
   // LocalBroadcastManager.getInstance(this).unregisterReceiver(mMessageReceiver);
   super.onPause();
}
```

See this [Stack Overflow article](http://stackoverflow.com/questions/16925579/android-implementation-of-lifecycle-methods-can-call-the-superclass-implementati) for more context.

## References

* http://developer.android.com/training/basics/activity-lifecycle/index.html
* https://github.com/xxv/android-lifecycle