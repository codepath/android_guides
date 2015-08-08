## Overview

This guide focuses on defining custom services including the various mechanisms for sending messages and managing threads to do background work. Android has many different abstractions related to messages and thread management which need to be summarized and understood. These threading abstractions are often related to [defining custom services](http://developer.android.com/guide/components/services.html) because a service by default still runs in the application's main thread and background work must be delegated to threads.

### Understanding the Main Thread

When an application is launched, the system creates a thread of execution for the application, called "main." This thread is very important because it is in charge of dispatching events and rendering the user interface and is usually called the **UI thread**. All components (activities, services, etc) and their executed code run in the same process and are **instantiated by default in the UI thread**. 

<img src="http://i.imgur.com/gOLOBYs.png" width="500" />

Keep in mind that performing long operations such as network access or database queries in the UI thread will block the entire app UI from responding. When the UI thread is blocked, no events can be dispatched, including drawing events. From the user's perspective, the application will appear to freeze. Additionally, keep in mind the **Android UI toolkit is not thread-safe** and as such you must **not manipulate your UI from a background thread**. 

In short, throughout this guide keep in mind two important rules: 

* Do not run long tasks on the main thread (to avoid blocking the UI)
* Do not change the UI at all from a background thread (only the main thread)

Next, let's understand the connection between services and threading.

### Services and Threading

[Services](http://developer.android.com/guide/components/services.html) and [thread management](http://developer.android.com/training/multiple-threads/index.html) are closely related but distinct topics. A service is a component that can run in the background **even when the user is not interacting with your app**. You should only create a service if you **need to perform work while your app isn't open**. 

Thread management is important to understand because a custom service **still runs in your application's main thread by default**. If you create a custom `Service`, then you will still **need to manage the background threads** manually unless the [[IntentService|Starting-Background-Services]] is leveraged.

## Thread Management

As a result of the major problems with blocking the UI thread outlined above, every Android app should **utilize background threads** to perform all long-running tasks such as reading from or writing to the disk or performing network operations. However, there are **several different abstractions for managing threads in the Android framework**. The following table breaks down the most practical options for running background tasks:

| Type            | Description                          | Built On |
| --------------- | -----------------------------------  | -------- |
| `AsyncTask`     |  Sequentially runs short tasks updating the UI | `HandlerThread` |
| `HandlerThread` | Sequentially runs tasks on a single thread | `Handler`, `Looper` |
| `ThreadPoolExecutor` | Concurrently runs tasks using a thread pool | `Executor`, `ExecutorService` |

<img src="http://i.imgur.com/VekPWIr.png" width="500" />

### Using an AsyncTask

[[AsyncTask|Creating-and-Executing-Async-Tasks]] allows for short sequential tasks to be performed within an activity context in the easiest way possible. Often the `AsyncTask` is used to run a background task, report progress and then update the UI thread with the results. `AsyncTask` is designed as a helper class around the [HandlerThread](http://developer.android.com/reference/android/os/HandlerThread.html) which removes the need for a user to be exposed to threads and operations directly.

**Caveats:** `AsyncTask` should be used for executing short operations (a few seconds at the most) in a sequential order. If you need to keep threads running for long periods of time or execute threads in parallel, it is highly recommended you use the [ThreadPoolExecutor](http://developer.android.com/reference/java/util/concurrent/ThreadPoolExecutor.html) instead. If you need to have more control over how you are running sequential background tasks, see the [HandlerThread](http://developer.android.com/reference/android/os/HandlerThread.html) below.

### Using a HandlerThread

[HandlerThread](http://developer.android.com/reference/android/os/HandlerThread.html) is a handy class for starting up a new worker thread that **sequentially runs tasks**. If you need a **single background thread** that starts a loop capable of running code or processing messages as needed, this is the tool for the job. 

The `HandlerThread` is a convenience class that initiates a [Looper](http://developer.android.com/reference/android/os/Looper.html) within a [Thread](http://developer.android.com/reference/java/lang/Thread.html) to process `Runnable` or [Message](http://developer.android.com/reference/android/os/Message.html) objects. Note that a [Handler](http://developer.android.com/reference/android/os/Handler.html) is used to **handle the insertion** of the `Runnable` or `Message` objects into the looper's queue:

```java
// Create a new background thread for processing messages or runnables sequentially
HandlerThread handlerThread = new HandlerThread("HandlerThreadName");
// Starts the background thread
handlerThread.start();
// Create a handler attached to the HandlerThread's Looper
mHandler = new Handler(handlerThread.getLooper()) {
    @Override
    public void handleMessage(Message msg) {
        // Process received messages here!
    }
};
```

Now we can either send a [Message](http://developer.android.com/reference/android/os/Message.html) to pass data or process a [Runnable](http://developer.android.com/reference/java/lang/Runnable.html) to execute code. To execute code on the worker thread through the [handler](http://developer.android.com/reference/android/os/Handler.html):

```java
// Execute the specified code on the worker thread
mHandler.post(new Runnable() {
    @Override
    public void run() {
        // Do something here!
    }
});
```

To send a [message](http://developer.android.com/reference/android/os/Message.html) on the worker thread through the [handler](http://developer.android.com/reference/android/os/Handler.html):
 
```java
// Secure a new message to send
Message message = handler.obtainMessage();
// Create a bundle
Bundle b = new Bundle();
b.putString("message", "foo");
// Attach bundle to the message
message.setData(b);
// Send message through the handler
mHandler.sendMessage(message);
```

The worker thread can be stopped immediately with:

```java
handlerThread.quit(); // On API >= 1
```

On API >= 18, we should use `quitSafely()` instead to finish processing pending messages before shutting down.

**Caveats:** `HandlerThread` is great for running tasks linearly (sequentially) on a thread and affords the developer control over how and when messages and runnables are processed by exposing access to the underlying `Looper` and `Handler`. However, if we need the ability to run tasks concurrently with one another, then we should use a `ThreadPoolExecutor` which helps us to manage a thread pool to execute the tasks in parallel.

### Using a ThreadPoolExecutor

[ThreadPoolExecutor](http://developer.android.com/reference/java/util/concurrent/ThreadPoolExecutor.html) is a great way to **execute parallelized tasks across multiple different threads** within a limited thread pool. If you want to execute work concurrently and retain control over how the work is executed, this is the tool for the job. 

Using a `ThreadPoolExecutor` starts with [constructing a new instance along with many arguments](http://developer.android.com/reference/java/util/concurrent/ThreadPoolExecutor.html#ThreadPoolExecutor\(int, int, long, java.util.concurrent.TimeUnit, java.util.concurrent.BlockingQueue<java.lang.Runnable>\)) configuring the thread pool:

```java
// Determine the number of cores on the device
int NUMBER_OF_CORES = Runtime.getRuntime().availableProcessors();
// Construct thread pool passing in configuration options
// int minPoolSize, int maxPoolSize, long keepAliveTime, TimeUnit unit,
// BlockingQueue<Runnable> workQueue
ThreadPoolExecutor executor = new ThreadPoolExecutor(
    NUMBER_OF_CORES*2,
    NUMBER_OF_CORES*2,
    60L,
    TimeUnit.SECONDS,
    new LinkedBlockingQueue<Runnable>()
);
```

Next, we can queue up a `Runnable` code block to execute on a thread in the pool with:

```java
// Executes a task on a thread in the thread pool
executor.execute(new Runnable() {
    public void run() {
        // Do some long running operation in background
        // on a worker thread in the thread pool!
    }
});
```

Threads are used to process each runnable concurrently as the message is received until all threads are busy. If all threads are currently busy, the `Executor` will queue a new task until a thread becomes available. The thread pool can be shutdown any time with the [shutdown](http://developer.android.com/reference/java/util/concurrent/ThreadPoolExecutor.html#shutdown\(\)) command:

```java
executor.shutdown();
```

This will shutdown the executor safely once all runnables have been processed. To shut down the executor immediately, instead use `executor.shutdownNow()`.

Note that the `ThreadPoolExecutor` is incredibly flexible and affords the developer significant control over all aspects of how tasks are executed. For a more comprehensive overview of `ThreadPoolExecutor` and all underlying components, check out [this excellent tutorial](http://codetheory.in/android-java-executor-framework/) by codetheory. See additional options for control by reviewing [this advanced guide](http://thegreyblog.blogspot.com/2011/12/using-threadpoolexecutor-to-parallelize.html). 

### Threading Glossary

All threading management options within Android including `AsyncTask`, `HandlerThread` and `ThreadPoolExecutor` are all built on even more foundational classes that power threads. The following is a glossary of concepts that power threading within the Android OS:

| Name      | Description |
| ----      | ----------- |
| `Runnable`  | Represents code that can be executed on any thread usually scheduled through a handler |
| `Thread`    | Concurrent unit of execution which runs code specified in a `Runnable` |
| `Message`   | Represents data that can be sent or received through a `Handler` |
| `Handler`   | Processes `Runnable` or `Message` objects on a thread. |
| `Looper`    | Loop that processes and sends `Runnable` or `Message` objects to a `Handler` |
| `MessageQueue` | Stores the list of `Runnable` or `Message` objects dispatched by the `Looper` |

Note that often these objects are primarily used within the context of higher-order abstractions such as `AsyncTask`, `HandlerThread` and `ThreadPoolExecutor`. For a more detailed description of these basic building blocks, check out this [excellent post on the subject](http://codetheory.in/android-handlers-runnables-loopers-messagequeue-handlerthread/). 

## Custom Services

### Background Services with IntentService 

In 90% of cases when you need a background service doing a task when your app is closed, you will [[leverage the IntentService|Starting-Background-Services]] as your first tool for the job. However, `IntentService` does have a few limitations. The biggest limitation is that the `IntentService` uses a **single worker thread** to handle start requests **one at a time**. However, as long as you don't require that your service handle multiple requests simultaneously, the `IntentService` is typically the easiest tool for the job.

However, in certain specialized cases where you do need background tasks to be processed in parallel using a concurrent thread pool and as such you cannot use `IntentService` and must extend from [Service](http://developer.android.com/reference/android/app/Service.html) directly. The rest of this guide is focused on that particular use case.

## References

* <http://developer.android.com/guide/components/services.html>
* <http://developer.android.com/guide/components/processes-and-threads.html#Threads>
* <http://developer.android.com/training/multiple-threads/index.html>
* <http://www.techotopia.com/index.php/A_Basic_Overview_of_Android_Threads_and_Thread_handlers>
* <http://codetheory.in/android-handlers-runnables-loopers-messagequeue-handlerthread/>
* <http://stackoverflow.com/questions/7597742/what-is-the-purpose-of-looper-and-how-to-use-it>
* <http://developer.android.com/reference/java/util/concurrent/ExecutorService.html>
* <http://edisontus.blogspot.com/2014/01/simple-guide-to-executorservice.html>
* <http://codetheory.in/android-java-executor-framework/>
* <http://thegreyblog.blogspot.com/2011/12/using-threadpoolexecutor-to-parallelize.html>
* <http://developer.android.com/guide/components/bound-services.html>
* <http://stackoverflow.com/questions/25246185/what-is-more-efficient-broadcast-receiver-or-handler>
* <http://codetheory.in/android-interprocess-communication-ipc-messenger-remote-bound-services/>
* <http://www.intertech.com/Blog/using-localbroadcastmanager-in-service-to-activity-communications/>
* <http://blog.nikitaog.me/2014/10/11/android-looper-handler-handlerthread-i/>
* <http://blog.denevell.org/android-service-handler-tutorial.html>