## Overview

Repeating periodic tasks within an application is a common requirement. This functionality can be used for polling new data from the network, running manual animations, or simply updating the UI. There are at least four ways to run periodic tasks:

1. **Handler** - Execute a `Runnable` task on the UIThread after an optional delay 
2. **ScheduledThreadPoolExecutor** - Execute periodic tasks with a background thread pool
3. **AlarmManager** - Execute any periodic task in the background as a service
4. **TimerTask** - Doesn't run in UIThread and **is not reliable**. Consensus is to [never use TimerTask](http://www.mopri.de/2010/timertask-bad-do-it-the-android-way-use-a-handler/)

Recommended methods are outlined below.

### Handler

Using a handler to execute `Runnable` code involves creating a `Handler` and then "posting" messages to the event message queue on the thread.

<img src="http://i.imgur.com/2vg53fk.png" alt="handler" width="450" />

Using a handler to execute a periodic runnable task is demonstrated below:

```java
// Create the Handler object 
Handler handler = new Handler();
// Define the task to be run here
private Runnable runnableCode = new Runnable() {
    @Override
    public void run() {
      // Do something here
      Log.e("Handlers", "Called");
      // Repeat this runnable code block again every 2 seconds
      handler.postDelayed(runnableCode, 2000);
    }
};
// Execute a runnable task as soon as possible
handler.post(runnableCode);
```

Remove the scheduled execution of a runnable with:

```java
// Removes pending code execution
handler.removeCallbacks(runnableCode);
```

Note that with a `Handler`, the `Runnable` executes in `UIThread` so you can safely update the user interface within the runnable code block. see [this handler post](http://www.mopri.de/2010/timertask-bad-do-it-the-android-way-use-a-handler/) and [this other handler post](http://androidtrainningcenter.blogspot.in/2013/12/handler-vs-timer-fixed-period-execution.html) for reference.

### ScheduledThreadPoolExecutor

A [pool of threads](http://developer.android.com/reference/java/util/concurrent/ScheduledThreadPoolExecutor.html) which can schedule commands to execute periodically in the background. Useful when multiple worker threads are needed but generally not needed. See [this guide on how they work](http://codelatte.wordpress.com/2013/11/13/49/) or [this stackoverflow post](http://stackoverflow.com/a/18606091/313399).

### AlarmManager

This should be used if the periodic tasks need to run in the background even when the app is not in the foreground. This leverages the alarm service on the phone to cause periodic executions of a service which will run continuously until stopped. See the [[AlarmManager section|Starting-Background-Services#using-with-alarmmanager-for-periodic-tasks]] of the services guide for details.

## References

* <http://www.mopri.de/2010/timertask-bad-do-it-the-android-way-use-a-handler/>
* <http://stackoverflow.com/questions/18605403/timertask-vs-thread-sleep-vs-handler-postdelayed-most-accurate-to-call-functio>
* <http://androidtrainningcenter.blogspot.in/2013/12/handler-vs-timer-fixed-period-execution.html>
* <http://stackoverflow.com/questions/8098806/where-do-i-create-and-use-scheduledthreadpoolexecutor-timertask-or-handler>