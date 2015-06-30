## Overview

A service is a component which runs in the background, without direct interaction with the user. As the service has no user interface it is not bound to the lifecycle of an activity. Services are used for repetitive and potential long running operations, checking for new data, data processing, indexing content, etc.

The [IntentService](http://developer.android.com/reference/android/app/IntentService.html) class provides a straightforward structure for running an operation on a single background thread. IntentService runs outside the application in a background process, so the process will run even if your application is closed.

A few limitations of an IntentService to be aware of:

 * You cannot affect the user interface from this background service directly
 * Requests are handled on a single worker thread and processes just one request at a time.
 * You cannot easily cancel an intent service once you start one

However, in most cases an IntentService is the preferred way to simple background operations.

### In Comparison to AsyncTask

A common point of confusion is when to use an AsyncTask and when to use an IntentService. An AsyncTask is tightly bound to a particular Activity. In other words, if the Activity is destroyed or the configuration changes then the AsyncTask will not be able to update the UI on completion. For short one-off background tasks **tightly coupled to updating an Activity**, we should use an AsyncTask. A good example is for a several second network request that will populate data into a ListView.

`IntentService` is geared towards longer running tasks that should run in the background, independent of the activity that is currently open and being viewed. The activity can be switched or the app can be paused and the `IntentService` will still continue to run in the background. For longer running tasks that are **independent of a particular Activity**, use IntentService.

### Launchers

Services can be thought of at a high-level as background tasks that run independent of the rest of the app. The services are "launched" or started by a few different types of "triggers". Refer to the following table to better understand the launchers that trigger the start of a service:

| Trigger      | Description   | Example |
| ---------    | ---------     | ------- |
| [[Intent|Starting-Background-Services#executing-the-intentservice]]       | Trigger directly from an activity or fragment after user action | Starts an image upload |
| [[AlarmManager|Starting-Background-Services#using-with-alarmmanager-for-periodic-tasks]] | Trigger at a specified time in the future or at a recurring interval | Poll for new updates |
| [[GCM|Push-Messaging]]          | Trigger when a push message is received through cloud messaging | Chat message received |
| [[BroadcastReceiver|Starting-Background-Services#starting-a-service-at-device-boot]] | Trigger when a particular broadcast message is received | Launch on device bootup |
| [Sensors](https://developer.android.com/training/location/geofencing.html) | Trigger when a particular sensor value is received | Geofencing location update |

Since most developer created services are **short-lived task-based**, they should be running for a finite amount of time after being triggered. Generally speaking, developers should be wary of building extended-run services. 

### Outputs

Remember that a service is **not bound to the Activity and cannot modify views within the UI directly**. Instead, a service tends to have very specific outputs after running that are not directly associated with the UI. Refer to the following table to better understand the outputs created by services:

| Output       | Description   | Example |
| ---------    | ---------     | ------- |
| [[Notifications]] | Creates a dashboard notification to alert the user | New direct message received |
| [[Broadcasts|Starting-Background-Services#communicating-with-a-broadcastreceiver]]   | Triggers a broadcast message to be received | Activity wants to add a new chat message |
| [[SQLite|Local-Databases-with-SQLiteOpenHelper]] | Write data received into the local database | Store new content for querying later |
| [[Files|Persisting-Data-to-the-Device#local-files]] | Cache blob data such as images or json to file | Cache images to be displayed quickly later |
| [[Prefs|Persisting-Data-to-the-Device#shared-preferences]] | Save key-values to shared preferences | Store a flag to display a message on next app open |

Note that we can use [[broadcasts to trigger updates within our app|Starting-Background-Services#communicating-with-a-broadcastreceiver]] while the app is running. In this way, the activity can update the UI accordingly after being told to by a service broadcast.

## Creating an IntentService

First, you define a class within your application that extends `IntentService` and defines the `onHandleIntent` which describes the work to do when this intent is executed:

```java
public class MyTestService extends IntentService {
  // Must create a default constructor
  public MyTestService() {
    // Used to name the worker thread, important only for debugging.
    super("test-service");
  }

  @Override
  public onCreate() {
     super.onCreate() // if you override onCreate(), make sure to call super().
     // If a Context object is needed, call getApplicationContext() here.
  }

  @Override
  protected void onHandleIntent(Intent intent) {
    // This describes what will happen when service is triggered
  }
}
```

Now we can use this service within our application.

### Registering the IntentService

Each service needs to be **registered in the manifest** for your app:

```xml
<application
        android:icon="@drawable/icon"
        android:label="@string/app_name">

        <service
          android:name=".MyTestService"
          android:exported="false"/>
<application/>
```

Notice that we specify this in the manifest file with the `name` and `exported` properties set. `exported` determines whether or not the service can be executed by other applications.

## Executing the IntentService

Once we have defined the service, let's take a look at how to trigger the service and pass the service data. This is done using the same Intent system we are already familiar with. We simply create an intent like normal specifying the IntentService to execute:

```java
public class MainActivity extends Activity { 
    public void onStartService(View v) {
        // Construct our Intent specifying the Service
        Intent i = new Intent(this, MyTestService.class);
        // Add extras to the bundle
        i.putExtra("foo", "bar");
        // Start the service
        startService(i);
    }
}
```

You can start the `IntentService` from any Activity or Fragment at any time during your application. Once you call `startService()`, the `IntentService` does the work defined in its `onHandleIntent()` method, and then stops itself.

## Communicating from a Service to an Application

The next step is to be able to communicate data from the `IntentService` back to the Application. This allows the application to act based on the results of the `IntentService`. This is done using one of two approaches:

 * [ResultReceiver](http://developer.android.com/reference/android/os/ResultReceiver.html) - Generic callback interface for sending results between service and activity. If your service **only needs to connect with its parent application** in a single place, use this approach.
 * [BroadcastReceiver](http://developer.android.com/reference/android/content/BroadcastReceiver.html) - Used to create a generic broadcast event which can then be picked up by any application. If your service **needs to communicate with multiple components** that will to listen for communication, use this approach.

### Communicating with a ResultReceiver

In many cases, an `IntentService` only needs to communicate with the activity or application that spawns it. If this is the case, where only the parent application needs to receive data, then let's take a look at a simple way to communicate using a `ResultReceiver`. First, let's define our `ResultReceiver` which should manage communication via method callbacks:

```java
// Defines a generic receiver used to pass data to Activity from a Service
public class MyTestReceiver extends ResultReceiver {
  private Receiver receiver;

  // Constructor takes a handler
  public MyTestReceiver(Handler handler) {
      super(handler);
  }

  // Setter for assigning the receiver
  public void setReceiver(Receiver receiver) {
      this.receiver = receiver;
  }

  // Defines our event interface for communication
  public interface Receiver {
      public void onReceiveResult(int resultCode, Bundle resultData);
  }

  // Delegate method which passes the result to the receiver if the receiver has been assigned
  @Override
  protected void onReceiveResult(int resultCode, Bundle resultData) {
      if (receiver != null) {
        receiver.onReceiveResult(resultCode, resultData);
      }
  }
}
```

This class is a simple intermediary that can then be used to trigger callbacks from the service in order to pass events to the parent Activity. This is useful when you want to act on the result of the service. Next, when we want to trigger the service to start, we just need to pass the IntentService a reference to the receiver and then setup a receiver callback:

```java
public class MainActivity extends Activity {
  public MyTestReceiver receiverForTest;

  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    setupServiceReceiver();
  }

  // Starts the IntentService
  public void onStartService() {
    Intent i = new Intent(this, MyTestService.class);
    i.putExtra("foo", "bar");
    i.putExtra("receiver", receiverForTest);
    startService(i);
  }

  // Setup the callback for when data is received from the service
  public void setupServiceReceiver() {
    receiverForTest = new MyTestReceiver(new Handler());
    // This is where we specify what happens when data is received from the service
    receiverForTest.setReceiver(new MyTestReceiver.Receiver() {
      @Override
      public void onReceiveResult(int resultCode, Bundle resultData) {
        if (resultCode == RESULT_OK) {
          String resultValue = resultData.getString("resultValue");
          Toast.makeText(MainActivity.this, resultValue, Toast.LENGTH_SHORT).show();
        }
      }
    });
  }
}
```

Now that we have created the Receiver and defined a Receiver callback in the Activity, we can now freely send message to our Activity at any time within the Service by accessing the Receiver:

```java
public class MyTestService extends IntentService {
  public MyTestService() {
    super("test-service");
  }

  @Override
  protected void onHandleIntent(Intent intent) {
    // Extract the receiver passed into the service
    ResultReceiver rec = intent.getParcelableExtra("receiver");
    // Extract additional values from the bundle
    String val = intent.getStringExtra("foo");
    // To send a message to the Activity, create a pass a Bundle
    Bundle bundle = new Bundle();
    bundle.putString("resultValue", "My Result Value. Passed in: " + val);
    // Here we call send passing a resultCode and the bundle of extras
    rec.send(Activity.RESULT_OK, bundle);
  }
}
```

Calling `res.send` will trigger the `onReceiveResult` callback to be called within our Activity and the return value will be displayed in the toast in this case.

### Communicating with a BroadcastReceiver

Using a `ResultReceiver` from above to communicate is not always the right approach. There are several issues with the `ResultReceiver` approach including the fact that if the app quits, then the receiver will not work when the app is relaunched. The receiver also requires each and every activity that wants to receive messages to have a reference to the receiver object passed into the service.

Instead, in many cases we might want one application to be able to pick up IntentService messages even after it has been fully relaunched or we want multiple applications to be able to receive the messages from the service. In these cases, we should use a `BroadcastReceiver` instead. For this example we are using the [LocalBroadcastManager](http://developer.android.com/reference/android/support/v4/content/LocalBroadcastManager.html) which only allows our app to communicate internally between Service and Activity. Let's see how to publish local broadcast messages within the service:

```java
public class MyTestService extends IntentService {
  public static final String ACTION = "com.codepath.example.servicesdemo.MyTestService";

  public MyTestService() {
    super("test-service");
  }

  @Override
  protected void onHandleIntent(Intent intent) {
      // Fetch data passed into the intent on start
      String val = intent.getStringExtra("foo");
      // Construct an Intent tying it to the ACTION (arbitrary event namespace)
      Intent in = new Intent(ACTION);
      // Put extras into the intent as usual
      in.putExtra("resultCode", Activity.RESULT_OK);
      in.putExtra("resultValue", "My Result Value. Passed in: " + val);
      // Fire the broadcast with intent packaged
      LocalBroadcastManager.getInstance(this).sendBroadcast(in);
      // or sendBroadcast(in) for a normal broadcast;
  }
}
```

This service is now sending this broadcast message to any application that wants to listen for these messages based on the `ACTION` namespace. Next, we need to construct a new `BroadcastReceiver`, register to listen and define the `onReceive` method to handle the messages within our Activity:

```java
public class MainActivity extends Activity {
    // ...onCreate...

    // Launching the service
    public void onStartService(View v) {
      Intent i = new Intent(this, MyTestService.class);
      i.putExtra("foo", "bar");
      startService(i);
    }

    @Override
    protected void onResume() {
        super.onResume();
        // Register for the particular broadcast based on ACTION string
        IntentFilter filter = new IntentFilter(MyTestService.ACTION);
        LocalBroadcastManager.getInstance(this).registerReceiver(testReceiver, filter);
        // or `registerReceiver(testReceiver, filter)` for a normal broadcast
    }

    @Override
    protected void onPause() {
        super.onPause();
        // Unregister the listener when the application is paused
        LocalBroadcastManager.getInstance(this).unregisterReceiver(testReceiver);
        // or `unregisterReceiver(testReceiver)` for a normal broadcast
    }

    // Define the callback for what to do when data is received
    private BroadcastReceiver testReceiver = new BroadcastReceiver() {
        @Override
        public void onReceive(Context context, Intent intent) {
            int resultCode = intent.getIntExtra("resultCode", RESULT_CANCELED);
            if (resultCode == RESULT_OK) {
                String resultValue = intent.getStringExtra("resultValue");
                Toast.makeText(MainActivity.this, resultValue, Toast.LENGTH_SHORT).show();
            }
        }
    };
}
```

Keep in mind that any application and any activity can "listen" for the messages using this same approach. This is what makes the `BroadcastReceiver` a more powerful approach for communication between services and activities. See the official tutorial for [reporting status from an IntentService](http://developer.android.com/training/run-background-service/report-status.html) for more details.

## Networking with IntentService

If you intend to perform networking within the IntentService, keep in mind that you do not necessarily need to be concerned about blocking the primary thread. The service is already running in the background so you will want to **avoid executing AsyncTasks within a Service**. Instead, for simple operations, you can send networking requests synchronously. For example, when using an IntentService with the [Android Async HTTP library](https://github.com/loopj/android-async-http), you need to use the synchronous client `SyncHttpClient` instead of the default asynchronous version:

```java
public class NetworkedIntentService extends IntentService {
   private AsyncHttpClient aClient = new SyncHttpClient();

    @Override
    protected void onHandleIntent(Intent intent) {
       // Send synchronous request
       aClient.get(this, someUrlHere, new AsyncHttpResponseHandler() {
           // ... onSuccess here
       }
    }
}
```



See [this service example](https://github.com/loopj/android-async-http/blob/master/sample/src/main/java/com/loopj/android/http/sample/services/ExampleIntentService.java) for a more complete example. If you try to send asynchronous requests, you will get errors about the thread no longer exists since the service will terminate before the network requests complete.

## Using with AlarmManager for Periodic Tasks

Suppose we need to set periodically executing background tasks. For example, we want to be able to check for new emails or content from a server every 15 minutes even if our application isn't running. This is useful for apps like email clients, news readers, instant messaging clients, et al. In this case, we don't necessarily need a long running task that runs forever. That would take drain battery life significantly and isn't what we want anyways.

For most of these common cases (checking for new data), what we really want to do is setup a **scheduler that triggers a background service at a regular interval** of our choosing. The best way to achieve this is to use an `IntentService` in conjunction with the [AlarmManager](http://developer.android.com/reference/android/app/AlarmManager.html). First, we want to define the `IntentService` to have periodically execute:

```java
public class MyTestService extends IntentService {
    public MyTestService() {
       super("MyTestService");
    }

    @Override
    protected void onHandleIntent(Intent intent) {
       // Do the task here
       Log.i("MyTestService", "Service running");
    }
}
```

Now we want to setup a way to execute this periodically at a specified interval. Enter the `AlarmManager` to execute a periodic task by firing a `BroadcastIntent`. First though, let's define the `BroadcastReceiver` that will be executed by the alarm and will launch our `IntentService`:

```java
public class MyAlarmReceiver extends BroadcastReceiver {
  public static final int REQUEST_CODE = 12345;
  public static final String ACTION = "com.codepath.example.servicesdemo.alarm";

  // Triggered by the Alarm periodically (starts the service to run task)
  @Override
  public void onReceive(Context context, Intent intent) {
    Intent i = new Intent(context, MyTestService.class);
    i.putExtra("foo", "bar");
    context.startService(i);
  }
}
```

Now we have our `IntentService` task defined and our receiver that will be setup to periodically execute in order to trigger the service. Next, let's register both our `IntentService` and `MyAlarmReceiver` in the `AndroidManifest.xml`.  

```xml
<receiver
    android:name=".MyAlarmReceiver"
    android:process=":remote" >
</receiver>

<service
    android:name=".MyTestService"
    android:exported="false" />
```
(Note that we need to define `android:process=":remote"` so that the BroadcastReceiver will run in a separate process so that it will continue to stay alive if the app has closed.  See this [Stack Overflow post](http://stackoverflow.com/questions/4311069/should-i-use-android-process-remote-in-my-receiver) for more details.)

Finally, we need to actually start the periodic alarm that will trigger the receiver. Let's do this in our Activity:

```java
public class MainActivity extends Activity {
  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    scheduleAlarm();
  }

  public void scheduleAlarm() {
    // Construct an intent that will execute the AlarmReceiver
    Intent intent = new Intent(getApplicationContext(), MyAlarmReceiver.class);
    // Create a PendingIntent to be triggered when the alarm goes off
    final PendingIntent pIntent = PendingIntent.getBroadcast(this, MyAlarmReceiver.REQUEST_CODE,
        intent, PendingIntent.FLAG_UPDATE_CURRENT);
    // Setup periodic alarm every 5 seconds
    long firstMillis = System.currentTimeMillis(); // first run of alarm is immediate
    int intervalMillis = 5000; // 5 seconds
    AlarmManager alarm = (AlarmManager) this.getSystemService(Context.ALARM_SERVICE);
    alarm.setInexactRepeating(AlarmManager.RTC_WAKEUP, firstMillis, intervalMillis, pIntent);
  }
}
```

This will cause the alarm to trigger immediately and then fire every 5 seconds from that point forward. Each time the alarm fires, the `MyAlarmReceiver` broadcast intent is triggered which starts up the `IntentService`. The `PendingIntent.FLAG_UPDATE_CURRENT` flag ensures that if the alarm fires very quickly, that the events will replace each other rather than stack up. Similarly, if we ever want to cancel the alarm, we can do:

```java
public void cancelAlarm() {
  Intent intent = new Intent(getApplicationContext(), MyAlarmReceiver.class);
  final PendingIntent pIntent = PendingIntent.getBroadcast(this, MyAlarmReceiver.REQUEST_CODE,
      intent, PendingIntent.FLAG_UPDATE_CURRENT);
  AlarmManager alarm = (AlarmManager) this.getSystemService(Context.ALARM_SERVICE);
  alarm.cancel(pIntent);
}
```

You can see a more detailed example [here](http://www.jeevanreddy.in/2012/05/scheduling-background-tasks-using-alarm.html) or [here](http://nerdwin15.com/2013/04/android-creating-an-alarm-with-alarmmanager/). For a more detailed example that includes starting the alarm when the phone boots up, check out [this blog post](http://dhimitraq.wordpress.com/2012/11/27/using-intentservice-with-alarmmanager-to-schedule-alarms/).

## Starting a Service at Device Boot

In certain cases, we might want a service to start **right after the device boots up**. This is a specific case of a broader trigger of launching a service when a particular broadcast is received by your application. To start a service when a broadcast (such as boot message) is received, we can start by adding the necessary permissions to receive this message in our manifest `AndroidManifest.xml` in the `<manifest>` element:

```xml
<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" />
<uses-permission android:name="android.permission.WAKE_LOCK" />
```

We need to link this boot message with a particular broadcast receiver which will receive and processes the "boot" message issued by the phone. Second, let's define our broadcast receiver class as extending from [WakefulBroadcastReceiver](https://developer.android.com/reference/android/support/v4/content/WakefulBroadcastReceiver.html) which ensures the device will stay awake until service has been started:

```java
package com.codepath.example;

// WakefulBroadcastReceiver ensures the device does not go back to sleep 
// during the startup of the service
public class BootBroadcastReceiver extends WakefulBroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        // Launch the specified service when this message is received
        Intent startServiceIntent = new Intent(context, MyTestService.class);
        context.startService(startServiceIntent);
    }
}
```

Now that we've created the receiver to start our service, within our manifest `AndroidManifest.xml` in the `<application>` element, we need to add our broadcast receiver specifying a fully qualified path:

```xml
<receiver android:name="com.codepath.example.BootBroadcastReceiver">  
    <intent-filter>  
        <action android:name="android.intent.action.BOOT_COMPLETED" />  
    </intent-filter>  
</receiver>
```

This registers the receiver and applies the `BOOT_COMPLETED` message which ensures the receiver is launched when the device boots up. The boot message is received and the "wakeful" receiver launches the service. Don't forget to **release the wake lock** within `onHandleIntent` so the device can go back to sleep after the service is launched:

```java
public class MyTestService extends IntentService {
  // ...
  @Override
  protected void onHandleIntent(Intent intent) {
      // Release the wake lock provided by the WakefulBroadcastReceiver.
      BootBroadcastReceiver.completeWakefulIntent(intent);
  }
}
```

With this completed, our service will start automatically whenever the device boots!

## Custom Services

In 90% of cases when you need a background service, you will grab `IntentService` as your tool. However, `IntentService` does have a few limitations. The biggest limitation is that the `IntentService` uses a single worker thread to handle start requests **one at a time**. Therefore, as long as you don't require that your service handle multiple requests simultaneously, the `IntentService` is the best tool for the job.

However, in certain specialized cases where you do need the requests to be processed in parallel, you cannot use `IntentService` and instead might want to extend from [Service](http://developer.android.com/reference/android/app/Service.html) directly. See the [Services guide](http://developer.android.com/guide/components/services.html) for more details on the basics of the `Service` and it's underlying behavior.

Now that the base `Service` actually runs in the same process as the application in which it is declared and in the **main UI thread** of that application. To avoid impacting application performance, you have to manually manage your own threads for the Service.

## Custom Threading

In cases where we want to avoid services altogether, you might want to consider `Runnable` as a last resort. In 99% of cases, you will probably use a `Service` or `IntentService` but in that last 1% of cases, check [this article about threads](http://www.techotopia.com/index.php/A_Basic_Overview_of_Android_Threads_and_Thread_handlers) can get you started. In nearly all cases, the `IntentService` is the simplest and most robust option for background processing. Make sure you have a very good reason for implementing your own threading. Otherwise, fall back to the higher-level built in mechanisms provided by the Android framework.

## Concluding Background Services

This guide has shown you the basics of using an IntentService and communicating between a service and an Activity. For a more complete example of an `IntentService` in action, check out [this tutorial on peachpit](http://www.peachpit.com/articles/article.aspx?p=1823692&seqNum=4).

## References

* <http://corner.squareup.com/2013/12/android-main-thread-2.html>
* <http://codetheory.in/android-pending-intents/>