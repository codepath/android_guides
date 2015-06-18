## Overview

This guide will show you how to configure an Android app to send and receive push notifications. There are several approaches to sending push notifications. The two most common are:

 - [Parse Push](#parse-push) - Push wrapper around GCM that makes push simple and easy
 - [Google Cloud Messaging](#google-cloud-messaging) - The foundation of push messaging in Android

## Parse Push

First approach is to use the [Parse Push](https://parse.com/tutorials/android-push-notifications) service to send and receive push notifications.

### Setup

Check out [this official tutorial](https://parse.com/tutorials/android-push-notifications) for a step-by-step guide to setting up push notifications with Parse. You can also review this [Parse Setup Guide](https://parse.com/apps/quickstart#parse_push/android/native) for full instructions.  

### Sending Push Notifications

Check out [this official tutorial](https://parse.com/tutorials/android-push-notifications) for a step-by-step guide to sending simple push notifications.   

Make sure to go to the `Settings` -> `Push` section and toggle on `Client push enabled`.  Otherwise, you may notice test push notifications work but sending between Android devices doesn't work.

<img src="http://i.imgur.com/2zrp2KB.png"/>

```java
ParsePush push = new ParsePush();
push.setChannel("mychannel");
push.setMessage("this is my message");
push.sendInBackground();
```

For additional details and options, check out the [official parse Push guide](https://parse.com/docs/android/guide#push-notifications-sending-pushes-to-channels). Full source code can be [found on Github](https://github.com/ParsePlatform/PushTutorial). 

**Running into issues?** Check out the [push troubleshooting guide](https://parse.com/docs/android/guide#push-notifications-troubleshooting). Also compare your app with this [sample reference app](https://github.com/codepath/ParsePushNotificationExample/tree/master/app/src/main/java/com/test).

### Receiving Push Notifications

Check out the [Receiving Push Guide](https://parse.com/docs/android/guide#push-notifications-receiving-pushes) for a basic overview of receiving Push messages within an Android app. There are two basic ways to receive push notifications.

#### Basic Push Invoking Activity

Normally when receiving a push, Parse will open the default activity that is designated as the main launcher in your `AndroidManifest.xml` file, but you can have Parse trigger a different activity to launch.  You can change this behavior by subclassing `ParshPushBroadcastReceiver` and overriding the `getActivity()` method to returning the Activity you prefer to launch instead:

```java
import android.app.Activity;

public class YourBroadcastReceiver extends ParsePushBroadcastReceiver {

  ....
  protected Class<? extends Activity> getActivity(Context context, Intent intent) {
    return YourActivity.class; // the activity that shows up 
  }
  ....

}
```

You will also need to modify the default Parse Broadcast Receiver normally used and replace with this custom class name.

```xml
<receiver
    android:name="your.package.name.YourBroadcastReceiver"
    android:exported="false" >
    <intent-filter>
        <action android:name="com.parse.push.intent.RECEIVE" />
        <action android:name="com.parse.push.intent.DELETE" />
        <action android:name="com.parse.push.intent.OPEN" />
    </intent-filter>
</receiver>
```

This way is pretty straightforward to bringing up an activity when a push is received but you can use a more custom approach if you need more flexibility with how a notification is handled.

#### Custom Push Broadcast Receiver

Receiving push notifications sent with Parse can also be done using the [BroadcastReceiver](http://developer.android.com/reference/android/content/BroadcastReceiver.html) and implementing the `onReceive()` method to manage incoming push events. Any activity can register to handle these broadcast events. 

If you want to implement the push notification receiver, then check out [this tutorial](http://ahirazitai.blogspot.in/2013/05/push-notification.html). Full source code for this tutorial can be [found on github](https://github.com/codepath/ParsePushNotificationExample/blob/master/app/src/main/java/com/test/MyCustomReceiver.java). 

Once a `BroadcastReceiver` is listening for messages, there are a few actions that are commonly taken once a push is received:

 * [[Create a notification|Notifications#creating-a-notification]]
 * [[Update an Activity|Starting-Background-Services#communicating-from-a-service-to-an-application]]
 * [Launch new activity](https://github.com/codepath/ParsePushNotificationExample/blob/master/app/src/main/java/com/test/MyCustomReceiver.java#L69)

You can review examples of these [outlined in this more elaborate code sample](https://github.com/codepath/ParsePushNotificationExample/blob/master/app/src/main/java/com/test/MyCustomReceiver.java). 

In certain cases when receiving a push, you want to **update an activity only if the activity is on the screen**. Otherwise, you want to raise a notification. The solutions to this are [outlined in this post](http://stackoverflow.com/a/18311830/313399) with a [code sample here](http://stackoverflow.com/a/15949723/313399).

#### Creating Dashboard Notifications

By default, Parse will create dashboard notifications based on certain details of the push message that are sent if you specify the "alert" and "title" properties in the notification. In the event that you want to manually manage the notifications, all you have to do is avoid sending the `alert` or `title` so parse doesn't create the notification for you. 

Details of setting up custom push notifications can be [found in this guide](https://www.parse.com/questions/update-notification-in-android). Check out the [[Notifications|Notifications#creating-a-notification]] guide for a detailed look at creating and managing these notices. See also how to [group notifications](http://developer.android.com/training/notify-user/managing.html) by modifying existing notices. This is useful if you want to avoid stacking a bunch of notifications and instead group messages or only show the latest one.

##### Launching an Activity

When working with push messages, often the notification will launch an activity or the activity may even be launched by the broadcast receiver directly. In these cases, we might want to ensure that the activity launched is the same activity that is already running rather a new instance. To achieve this, we can use [launch modes or intent flags](http://guides.codepath.com/android/Navigation-and-Task-Stacks#launch-modes) such as `singleTop` to ensure the same activity instance is presented.

### Source Code

We have a full demo of Parse Push sending and receiving which [can be found on Github](https://github.com/codepath/ParsePushNotificationExample/tree/master/app/src/main/java/com/test). Check out [MyCustomReceiver](https://github.com/codepath/ParsePushNotificationExample/tree/master/app/src/main/java/com/test/MyCustomReceiver.java) and [MainActivity](https://github.com/codepath/ParsePushNotificationExample/tree/master/app/src/main/java/com/test/MainActivity.java). 

### Gotchas

A few quick things to help make implementing push notifications easier:

 * Review the official [parse push troubleshooting guide](https://parse.com/docs/android/guide#push-notifications-troubleshooting).
 * Make sure to register for the broadcasts using the [LocalBroadcastManager](http://developer.android.com/reference/android/support/v4/content/LocalBroadcastManager.html) with the `LocalBroadcastManager.getInstance(this).registerReceiver` method. 
 * If you need to communicate between a receiver and then a second activity that is not currently in the foreground, you may consider **persisting the notification** to disk in SQLite. This way we can easily access that from another Activity once that activity switches to the foreground.

## Google Cloud Messaging

Second approach is the more manual one using GCM. Google Cloud Messaging for Android (GCM) is a service that allows you to send data from your server to your users' Android-powered device, and also to receive messages from devices on the same connection. 

A GCM implementation includes a Google-provided connection server, a 3rd-party app server that interacts with the connection server, and a GCM-enabled client app running on an Android device:

![GCM Arch](http://i.imgur.com/9XzwPqc.png)

In order to use GCM, we need to go through the following steps:

 1. Register with Google Console and enable GCM
    - Obtain Sender ID (Project Number)
    - Obtain Server API Key
 2. Integrate GCM into our Android app
    - Register in app for a GCM Registration ID
    - Transmit the Registration ID (Device ID) to our web server
    - Register a GCM Receiver to handle incoming messages
 3. Develop HTTP Server with GCM endpoints
    - Endpoint for registering a user with a device ID
    - Endpoint for sending a push notification to a specified set of device IDs

Read our [[Google Cloud Messaging]] cliffnotes for specific implementation details. Check out these external links below for reference:

 * [Google GCM Overview](http://developer.android.com/google/gcm/gs.html)
    * [Implementing GCM Server](http://developer.android.com/google/gcm/server.html)
 * [AndroidHive GCM Tutorial](http://www.androidhive.info/2012/10/android-push-notifications-using-google-cloud-messaging-gcm-php-and-mysql/)
 * <https://developers.google.com/cloud/samples/mbs/android/enable_push>
 * <http://javapapers.com/android/google-cloud-messaging-gcm-for-android-and-push-notifications/>
 * <http://hmkcode.com/android-google-cloud-messaging-tutorial/>

## References

* [Google I/O 2012 Intro Talk](https://www.youtube.com/watch?v=YoaP6hcDctM)
* <http://developer.android.com/google/gcm/index.html>
* <https://pushbots.com/>
* <https://parse.com/products/push>
* <http://www.push-notification.org/>
* <https://mixpanel.com/docs/people-analytics/android-push>
* <http://aws.amazon.com/sns/>