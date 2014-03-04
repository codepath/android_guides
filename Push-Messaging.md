## Overview

This guide will show you how to configure an Android app to send and receive push notifications. 

### Sending Push Notifications

Check out [this official tutorial](https://parse.com/tutorials/android-push-notifications) for a step-by-step guide to sending simple push notifications. Full source code can be [found on Github](https://github.com/ParsePlatform/PushTutorial). For full details and options, check out the [official parse Push guide](https://parse.com/docs/push_guide#setup/Android). 

### Receiving Push Notifications

#### Push Invoking Activity

Check out the [Push Guide](https://parse.com/docs/push_guide#receiving/Android) for a basic overview of receiving Push messages within an Android app. There are two basic ways to receive push notifications. If you are using [advanced targeting](https://parse.com/docs/push_guide#sending-queries/Android), you can have Parse trigger a particular activity to launch. This is pretty straightforward but you can use a more custom approach if you need more flexibility.

#### Push Broadcast Receiver

Receiving push notifications sent with Parse can be done using the [BroadcastReceiver](http://developer.android.com/reference/android/content/BroadcastReceiver.html) to manage the incoming push events. Any activity can register to handle these broadcast events. If you want to implement the push notification receiver, then check out [this excellent tutorial](http://ahirazitai.blogspot.in/2013/05/push-notification.html). Full source code for that tutorial can be [found on github](https://github.com/ahiraz/pushNotificationDemo). 

### Communicating with an Activity

Often you might want your BroadcastReceiver to communicate with a host activity. There can be a bit of confusion around how to access an activity from the receiver. The easiest way to communicate between an activity and a receiver is to **have the receiver be defined as an inner class** within the activity [as explained here](http://stackoverflow.com/a/10218242) in detail. Once you've defined the receiver that way, accessing the internal state of the Activity is easy.

### Source Code

We have a full demo of Parse Push sending and receiving which [can be found on Github](https://github.com/thecodepath/ParsePushNotificationExample/tree/master/src/com/test). Check out [MyCustomReceiver](https://github.com/thecodepath/ParsePushNotificationExample/blob/master/src/com/test/MyCustomReceiver.java) and [MainActivity](https://github.com/thecodepath/ParsePushNotificationExample/blob/master/src/com/test/MainActivity.java). 

### Gotchas

A few quick things to help make implementing push notifications easier:

 * Make sure to register for the broadcasts using the [LocalBroadcastManager](http://developer.android.com/reference/android/support/v4/content/LocalBroadcastManager.html) with the `LocalBroadcastManager.getInstance(this).registerReceiver` method. 
 * If you need to communicate between a receiver and then a second activity that is not currently in the foreground, you may consider **persisting the notification** to disk in SQLite. This way we can easily access that from another Activity once that activity switches to the foreground.

## References

* <http://developer.android.com/google/gcm/index.html>
* <https://pushbots.com/>
* <https://parse.com/products/push>
* <http://www.push-notification.org/>
* <https://mixpanel.com/docs/people-analytics/android-push>
* <http://aws.amazon.com/sns/>