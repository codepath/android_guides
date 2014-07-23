## Overview

This guide will show you how to configure an Android app to handle real-time messaging such as a chat client or group texting app. There are several approaches you can take to achieve this.

## PubNub

One approach is to use the [PubNub](http://www.pubnub.com/docs/java/android/android-sdk.html) service to send and receive real-time messages. PubNub enables low-latency real-time apps that work across any number of platforms including Android, iOS, and Javascript. 

### Setup

PubNub can be setup very quickly using [this setup guide](http://www.pubnub.com/docs/java/android/tutorial/data-push.html#_step_1_install_the_code) for their Android SDK. In short, their SDK is contained within two JARs that need to be added to your app's `libs` folder. You can also [watch this setup video](https://vimeo.com/95542286) to get a better look.

### Usage

PubNub works by allowing any number of clients to subscribe and send **messages** to any number of different **channels**. All clients subscribed to a channel will get messages sent to that channel in real-time.

![PubNub](http://pubnub.github.io/slides/workshop/pictures/broadcast.png)

To subscribe and send messages to a channel, we leverage the easy-to-use Android SDK. Check out [this step-by-step tutorial](http://www.pubnub.com/docs/java/android/tutorial/data-push.html#_step_2_access_the_api) as well as this [handy quickstart guide](http://www.pubnub.com/docs/java/android/tutorial/quick-start.html). We can also take a look at the [Android PubNub Code](https://github.com/pubnub/java/tree/master/android) which includes a detailed README. A complete example of using PubNub can be [found within this example](https://github.com/pubnub/java/tree/master/android/examples/PubnubExample/src/com/pubnub/examples/pubnubExample10).

If we want to **use PubNub as a service** such that the messages are being received even when the app isn't open, we can check out [this simple example](https://github.com/pubnub/java/tree/master/android/examples/SubscribeAtBoot/src/com/pubnub/examples/subscribeAtBoot) which describes how to setup a `PubNubService` which subscribes to a channel and receives messages starting up as soon as the device boots. This solution is expanded on in [this stackoverflow post](http://stackoverflow.com/a/9608967/313399). 

## Sinch

Check out this [detailed guide](http://sinch.github.io/android-messaging-tutorial/) for creating a real-time messaging client using [Sinch](http://www.sinch.com/docs/android/user-guide/).

## References

* <http://www.pubnub.com/docs/java/android/android-sdk.html>
* <http://www.pubnub.com/docs/java/android/tutorial/quick-start.html>
* <http://www.pubnub.com/docs/java/android/tutorial/data-push.html>
* <http://www.pubnub.com/docs/java/android/overview/data-push.html>
* <http://www.pubnub.com/docs/java/android/api/reference.html>