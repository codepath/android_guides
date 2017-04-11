## Overview

This guide will show you how to configure an Android app to send and receive push notifications. There are several approaches to sending push notifications. The two most common are:

 - [Parse Push](#parse-push) - Push wrapper around GCM that makes push simple and easy
 - [Google Cloud Messaging](#google-cloud-messaging) - The foundation of push messaging in Android

## Parse Push

First approach is to use the Parse Push service to send and receive push notifications.  You will first need to setup a Parse server by following our [configuring parse server guide](https://guides.codepath.com/android/Configuring-a-Parse-Server) to get things started.

### Setup

Start by checking out our [push notifications setup guide for Parse](https://guides.codepath.com/android/Push-Notifications-Setup-for-Parse) to get things started.  Use the [push notifications client demo](https://github.com/codepath/ParsePushNotificationExample) as a reference as well.

### Sending Push Notifications

Because of the implicit [security issues](https://github.com/ParsePlatform/parse-server/issues/396#issuecomment-183792657) with allowing push notifications to be sent through Android or iOS directly to other devices, this feature is disabled.  Normally in hosted Parse you can toggle an option to override this security restriction.  For open source Parse, you must implement pre-defined code written in JavaScript that can be called by the clients to execute, otherwise known as [Parse Cloud]( http://blog.parse.com/announcements/pushing-from-the-javascript-sdk-and-cloud-code/).

Your Java client should call this function:

```java
HashMap<String, String> test = new HashMap<>();
test.put("message", "testing");
test.put("customData", "abc");
ParseCloud.callFunctionInBackground("pushChannelTest", test);
```

You then need to implement a custom Parse function on your cloud server:

```javascript
Parse.Cloud.define('pushChannelTest', function(request, response) {

  // request has 2 parameters: params passed by the client and the authorized user
  var params = request.params;
  var user = request.user;

  var message = params.message;
  var customData = params.customData;

  // use to custom tweak whatever payload you wish to send
  var pushQuery = new Parse.Query(Parse.Installation);
  pushQuery.equalTo("deviceType", "android");

  var payload = { "alert": message,
                  "customdata": customData
                };
  };

  // Note that useMasterKey is necessary for Push notifications to succeed.

  Parse.Push.send({
  where: pushQuery,      // for sending to a specific channel
  data: payload,
  }, { success: function() {
     console.log("#### PUSH OK");
  }, error: function(error) {
     console.log("#### PUSH ERROR" + error.message);
  }, useMasterKey: true});

  response.success('success');
});
```

### Receiving Push Notifications

Check out the [Receiving Push Guide](http://docs.parseplatform.org/android/guide/#receiving-pushes) for a basic overview of receiving Push messages within an Android app. There are two basic ways to receive push notifications.

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

#### Creating Dashboard Notifications

By default, Parse will create dashboard notifications based on certain details of the push message that are sent if you specify the "alert" and "title" properties in the notification. In the event that you want to manually manage the notifications, all you have to do is avoid sending the `alert` or `title` so parse doesn't create the notification for you.

Details of setting up custom push notifications can be found in the [[Notifications|Notifications#creating-a-notification]] guide for a detailed look at creating and managing these notices. See also how to [group notifications](http://developer.android.com/training/notify-user/managing.html) by modifying existing notices. This is useful if you want to avoid stacking a bunch of notifications and instead group messages or only show the latest one.

##### Launching an Activity

When working with push messages, often the notification will launch an activity or the activity may even be launched by the broadcast receiver directly. In these cases, we might want to ensure that the activity launched is the same activity that is already running rather a new instance. To achieve this, we can use [[launch modes or intent flags|Navigation-and-Task-Stacks#launch-modes]] such as `singleTop` to ensure the same activity instance is presented.

##### Checking App State when Receiving a Broadcast

In certain cases when receiving a push, you want to **update an activity only if the activity is on the screen**. Otherwise, if the activity is not on screen then you want to [[create a notification|Notifications]].

There are two approaches to this: either use `ActivityManager` to check if the activity is running or use ordered broadcasts to override the receiver when the activity is running. Both possible solutions to this are [outlined in this post](http://stackoverflow.com/a/18311830/313399) with a [code sample here](http://stackoverflow.com/a/15949723/313399).

### Source Code

We have a full demo of Parse Push sending and receiving which [can be found on Github](https://github.com/codepath/ParsePushNotificationExample/tree/master/app/src/main/java/com/test). Check out [MyCustomReceiver](https://github.com/codepath/ParsePushNotificationExample/tree/master/app/src/main/java/com/test/MyCustomReceiver.java) and [MainActivity](https://github.com/codepath/ParsePushNotificationExample/tree/master/app/src/main/java/com/test/MainActivity.java).

### Gotchas

A few quick things to help make implementing push notifications easier:

 * Review the official [parse push troubleshooting guide](http://docs.parseplatform.org/android/guide/#troubleshooting).
 * Make sure to register for the broadcasts using the [LocalBroadcastManager](http://developer.android.com/reference/android/support/v4/content/LocalBroadcastManager.html) with the `LocalBroadcastManager.getInstance(this).registerReceiver` method.
 * If you need to communicate between a receiver and then a second activity that is not currently in the foreground, you may consider **persisting the notification** to disk in SQLite. This way we can easily access that from another Activity once that activity switches to the foreground.

## Google Cloud Messaging

Second approach is the more manual way using GCM. Google Cloud Messaging for Android (GCM) is a service that allows you to send data from your server to your users' Android-powered device and also to receive messages from devices on the same connection.  Beneath the surface, Parse implements push notifications for Android with GCM.

Read our [[Google Cloud Messaging]] guide for specific implementation details.

## References

* [Google I/O 2012 Intro Talk](https://www.youtube.com/watch?v=YoaP6hcDctM)
* <http://developer.android.com/google/gcm/index.html>
* <https://pushbots.com/>
* <http://www.push-notification.org/>
* <https://mixpanel.com/docs/people-analytics/android-push>
* <http://aws.amazon.com/sns/>
