## Introduction

The core of navigation flows in Android apps centers around the Activity and the Intent. In this guide, we are going to review tasks, navigation, and how to manipulate these when navigating between activities.  

This is typically used in cases where we need to exert more fine-tuned control over the behavior of the task stack which determines the "history" of our application.

**Note:** To skip to changing the behavior of the task stack, jump right to the [[Usage|Navigation-and-Task-Stacks#usage]] section of this document.

## Conceptual Overview

While the concept of using intents to launch activities should already be familiar to you, let's take a second to explore how launching new activities works and how this navigation is managed. 

As we navigate around our app (or even between apps), Android is maintaining a **task stack** which tracks each step in the user's history. A **task** is a collection of activities that users interact with when performing a certain job. The activities are arranged in a stack (the "back stack"), in the order in which each activity is opened.  By default, your activities in your app are organized as part of the same **task affinity group**.

### Understanding the Task Stack

When an application is launched from the Home Screen, the application comes to the foreground and the "stack" for that application becomes active. If no task stack exists for this application (first launch recently), then a new task is created and the "main" activity of the app becomes the first (root) item in the stack.

Each time an activity is launched in your app, that activity is pushed onto the top of the stack and becomes active. The previous activity remains in the stack but it is stopped. However, despite being stopped the system retains the state of the Activity UI automatically.

![Task Stack](https://developer.android.com/images/fundamentals/diagram_backstack.png)

If the user hits "back", the current activity is popped from the top of the stack (and destroyed) and the previous activity resumes with the previous UI state restored. If the user continues to press Back, then each activity in the stack is popped off to reveal the previous one, until the user returns to whichever task was happening before this task stack began.

### Task Stacks Everywhere

A "task" is a collection of activities stemming from a "root" application. This group is cohesive and when the user hits the home screen, the entire "task" is moved to the background until the stack is re-activated. While in the background, all the activities in the task are stopped, but the back stack for the task remains intact.

![MultiTasking](https://developer.android.com/images/fundamentals/diagram_multitasking.png)

If the user were to open another separate application now, that app would have an entirely separate task stack that would come to the foreground. This means that **multiple discrete stacks** frequently are stored by the system. There might be several task stacks in the background and when the user re-enters the related application, that stack will reappear in the foreground with states restored.

### Multiple Activity Instances

This process of multiple task stacks is just the beginning. The real point of understanding comes from exploring what happens when we launch the same activity multiple times. Because the activities in the back stack are never rearranged, every time an activity is started via an intent, a brand new instance of that activity is **created and pushed onto the stack**.

This means that if you have an activity (say the timeline for twitter), and you launch the timeline activity multiple times using different intents, then **multiple instances of that activity** will be tracked in the stack. 

![Multiple Activity](https://developer.android.com/images/fundamentals/diagram_multiple_instances.png)

This gets even more complex when you consider you might launch the same Activity from multiple __different__ task stacks. For example, think about how many apps might launch the browser and how many instances of a browser you might have across all task stacks.

By default, if the user navigates backward using the Back button, each instance of the activity is revealed in the order in which they were opened (each with their own UI state). 

See [Tasks and Back Stack](https://developer.android.com/guide/components/tasks-and-back-stack.html) official guide for the source of the above content and additional details.

## Affinity Groups

Activities within your application all belong to the same affinity group unless you define one differently using the `taskAffinity` XML property:

```xml
<activity android:name=".activities.MyActivity"
android:taskAffinity="com.codepath.myapp">
```

You can group activities together in this way to dismiss them all at once:

```java
ActivityCompat.finishAffinity(this); // equivalent to finish(true); on Android devices API > 4
```
```kotlin
ActivityCompat.finishAffinity(this) // equivalent to finish(true); on Android devices API > 4
```

When using `finishAffinity()`, you may notice that the activity can still show up in the [Most Recents](https://developer.android.com/guide/components/recents.html) screen.  You can use the `android:autoRemoveFromRecents` flag to be true to remove it, which is normally set to false by default.  You can also use `finishAndRemoveTask()` to accomplish the same effect.

```xml
<activity android:name=".MyActivity"
android:autoRemoveFromRecents="true"
android:taskAffinity="com.codepath.myapp"/>
```

## Usage

The behavior described above works automatically and without any special configuration from the developer. The default behavior is to just place all activities started in succession in the same task and in a "last in, first out" stack.

However, there are cases where this behavior may not suit your needs. In the event that you want to tune or tweak this behavior, we need to understand how to change the behavior of our task stack. For example, we might want to:

 * Have an activity in your application to begin a new task when it is started
 * Want to bring forward an existing instance of an activity (rather than create a new one)
 * Want to clear the history of the back stack at a particular point

In order to change this behavior, there are two approaches available:

 * **Activity Properties** - Setting properties on the activity within the manifest that changes the task stack behavior.
 * **Intent Flags** - Adding flags to an intent in order to change the task stack behavior for that launch.

If you always want an Activity to show a particular kind of behaviour then you will set the property in the `<activity>` element corresponding to that Activity in the manifest.

If you only want to exclude it from recent tasks under certain one-off conditions then you set the flag in the Intent and not the manifest.

### Configuring Activity Properties

The first approach to modifying the task stack is to set properties on the [<activity>](https://developer.android.com/guide/topics/manifest/activity-element.html) element within the `AndroidManifest.xml`. 

#### Launch Modes

To tweak the behavior of the Activity in the manifest, we want to set the `launchMode` attribute. This looks like:

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.todoapp"
    … >
    <application
        android:icon="@drawable/ic_launcher"
        android:label="@string/app_name"
        ... >
        <activity
            android:name="com.example.todoapp.TodoActivity"
            android:label="@string/app_name" 
            android:launchMode="singleTop">
        </activity>
    </application>
</manifest>
```

Notice there we have specified the `launchMode` as "singleTop". There are four different "launch modes" to choose from that alter the task stack behaviors for that activity:

 * `standard` - The default behavior. Creates a new instance of the activity in the task.
 * `singleTop` - Reuse an activity instance if already at the top of the stack; otherwise create new instance.
 * `singleTask` - Reuse an activity instance if exists in an existing stack; otherwise create in a new task stack.
 * `singleInstance` - Same as `singleTask` but no other activities are ever inserted into the created task stack.

See the [official guide for stacks](https://developer.android.com/guide/components/tasks-and-back-stack.html#ManifestForTasks) for a more detailed explanation. This table may help clarify the different launch modes:

| Mode           | Default       | Instantiation      | New Task on Launch | Allow other activities within Task |
| --------       | ------------  |------------------| -------------     | --------------------------        |
| standard       | Yes           | Everytime an intent is created, a new instance is created. Also instances can be member of multiple tasks and more than one instance in a Task. | No. Open in the same Task that originated the intent | Yes    |
| singleTop      | No            | Exactly like standard but if the activity is at the top of the Task stack then it uses the existing instance.       | No. Open in the same Task that originated the intent      |    Yes      |
| singleTask     | No   | Single instance   |   Yes. Always a Root Task.      |       Yes          |
| singleInstance | No    | Single instance     |    Yes. Always a Root Task.   |    Never. Always the only activity in the task  |

It might be useful to understand when various modes might be appropriate in an application. The following provides examples:

 * `singleTask` - There is only one `BrowserActivity` at a time and it doesn't become part of tasks that send it intents to open web pages. While it might return to whatever most recently launched it when you hit back it is actually fixed at the bottom of its own task activity stack. 
 * `singleTop` - While there can be multiple instances of the `BrowserBookmarksPage` if there is already one at the top of the task's activity stack it will be reused. This way you only have to hit back once to return to the browser if the bookmarks activity is started multiple times.
 * `singleInstance` - Only one `AlarmAlert` activity at a time and it is always its own task. Anything it might launch (if anything) becomes a part of a separate task stack.

Check out the [understanding launch modes](https://www.mobomo.com/2011/06/android-understanding-activity-launchmode/) guide for more detailed examples. Also check out [this blog post](https://blog.akquinet.de/2011/02/25/android-activities-and-tasks-series-%E2%80%93-activity%C2%A0attributes/) for another explanation.

#### Configuring No History

There are additional properties as well that can be set on an activity. For example, if an activity should not be added to the task stack at all, simply set the [noHistory](https://developer.android.com/guide/topics/manifest/activity-element.html#nohist) flag within the manifest:

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.todoapp"
    … >
    <application
        android:icon="@drawable/ic_launcher"
        android:label="@string/app_name"
        ... >
        <activity
            android:name="com.example.todoapp.TodoActivity"
            android:label="@string/app_name" 
            android:noHistory="true">
        </activity>
    </application>
</manifest>
```

Once this is set, that activity will never be a part of the stack even when launched. You might also want to check out the [taskAffinity](https://developer.android.com/guide/topics/manifest/activity-element.html#aff) property as well as the [allowTaskReparenting](https://developer.android.com/guide/topics/manifest/activity-element.html#reparent) properties for more fine tuned control over the activity's behavior in the task stack.

### Configuring Intent Flags

The other approach to modifying the task stack is to [set flags](https://developer.android.com/reference/android/content/Intent.html#FLAG_ACTIVITY_NEW_TASK) within an intent when starting an Activity.

You can alter the task stack behavior by appending various flags when launching an intent. For example, to clear the task stack when starting a new activity, we can do:

```java
Intent i = new Intent(this, SecondActivity.this);
// Add flags into the intent to alter behavior
i.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK |
Intent.FLAG_ACTIVITY_CLEAR_TASK);
// Execute as normal
startActivity(i);
```
```kotlin
val i = Intent(this@MainActivity, SecondActivity::class.java)
// Add flags into the intent to alter behavior
i.flags = Intent.FLAG_ACTIVITY_NEW_TASK or
        Intent.FLAG_ACTIVITY_CLEAR_TASK
// Execute as normal
startActivity(i)
```

The above can be very useful for login activities. The flags launch a new activity such that when the user presses "back", the previous activity would not show up. The following other flags are available to be set on the intent:

* `FLAG_ACTIVITY_CLEAR_TASK` - Clear any existing tasks on this stack before starting the activity.
* `FLAG_ACTIVITY_NEW_TASK` - Start the activity in a new task or reuse an existing task tied to that activity.
* `FLAG_ACTIVITY_SINGLE_TOP` - If the activity being started is the same as the current activity, then reuses the existing instance which receives a call to [onNewIntent()](https://developer.android.com/reference/android/app/Activity.html#onNewIntent\(android.content.Intent\))
* `FLAG_ACTIVITY_CLEAR_TOP` - If the activity being started is already running in the current task, delivers us back to the existing instance and clears the stack.

There are almost a dozen other flags you can set as well, for example `FLAG_ACTIVITY_BROUGHT_TO_FRONT`, `FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS`, and `FLAG_ACTIVITY_MULTIPLE_TASK`.  For a more comprehensive list, check out [this article](https://blog.akquinet.de/2010/04/15/android-activites-and-tasks-series-intent-flags/). 

See the [official guide for stacks](https://developer.android.com/guide/components/tasks-and-back-stack.html#IntentFlagsForTasks) for a more detailed explanation.

## Providing Proper Navigation

Check out the excellent [Providing Back Navigation](https://developer.android.com/training/implementing-navigation/temporal.html) and [Providing Up Navigation](https://developer.android.com/training/implementing-navigation/ancestral.html) guides for details on how to ensure you properly provide both of these navigation directions.

## References

 * <https://developer.android.com/guide/components/tasks-and-back-stack.html>
 * <https://developer.android.com/training/implementing-navigation/temporal.html>
 * <https://www.peachpit.com/articles/article.aspx?p=1874864>
 * <https://developer.android.com/reference/android/content/Intent.html#FLAG_ACTIVITY_NO_HISTORY>
 * <https://developer.android.com/guide/topics/manifest/activity-element.html#lmode>
 * <https://www.mobomo.com/2011/06/android-understanding-activity-launchmode/>
 * <https://blog.akquinet.de/2011/02/25/android-activities-and-tasks-series-activity-attributes//> 
 * <https://www.sitepoint.com/activities-tasks-and-intents-oh-my/>
 * <https://www.inkling.com/read/programming-android-mednieks-1st/chapter-11/a-flowing-and-intuitive-user>
 * <https://inthecheesefactory.com/blog/understand-android-activity-launchmode/en>