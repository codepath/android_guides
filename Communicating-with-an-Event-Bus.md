## Overview

By now, you should be familiar with the various ways the Android framework relies on passing data between the various abstractions used:

 * Activities uses [[the Intent system|Using-Intents-to-Create-Flows]] to pass data between screens.
 * Fragments rely on Bundles to construct fragments, methods by the containing Activity to call, and the listener pattern for the Fragment to fire events (see [[Communicating with Fragments|Creating and Using-Fragments#communicating-with-fragments]])
 * Services rely on Broadcast Managers to send data to Applications ([[Communicating from a Service to an Application|Starting Background Services#communicating-from-a-service-to-an-application]])

### Issues 

One of primary issues of these current approaches is that they can create strong dependencies between each component, making it difficult to change one part of the system without impacting another area.  This [blog post](https://corner.squareup.com/2012/07/otto.html) describes the challenges of creating unmanageable dependencies with the current Android framework.  Instead of encouraging more modular designs, which they are intended to do, the communication patterns in Android can sometimes do the exact opposite.

### How Event Bus Models Helps

In event bus model, there are *publishers* and *subscribers*.  Publishers are responsible for posting events in response to some type of state change, while subscribers respond to these events.  The event acts as an intermediary for exchanging information, isolating and minimizing the dependencies between each side.  In this way, message buses create a communication pipeline that is intended to help make your app more maintainable and scalable.

There are many different libraries which attempt to enable the event bus model, including [EventBus](https://github.com/greenrobot/EventBus), [RxJava](https://github.com/ReactiveX/RxJava), and [Otto](https://github.com/square/otto).   Otto is by far the easiest to start using immediately.  EventBus has a few more advanced features than in Otto described in this [comparison chart](http://timnew.me/blog/2014/09/14/otto-and-android-annotations-compatibility-issue-analysis/), and RxJava enables
a way to declare [sequences of actions](http://www.infoq.com/news/2014/11/android-rxjava-at-soundcloud) to be taken in response to certain events.  

When using an event bus model, there are several consideratons:
  * **Don't assume you should replace every communication pattern with an event bus model**.  If you publish an event meant only for one subscriber and shouldn't trigger changes in other subscribers, rely on 1-to-1 communication patterns such as the Intent system or listener pattern.  For instance, a date picker fragment that could be reused in multiple components of your app should probably require an Activity that implements `OnDateSelected()` method.  Using a event bus model that would publish an event could actually cause unintended changes in other components.  Consider using the design patterns described in this [article](http://timnew.me/blog/2014/12/06/typical-eventbus-design-patterns/).

  * **If you are using publishing or subscribers within an Activity of Fragment they should be registered and unregistered with their lifecycle.**  Otherwise, it's likely you will encounter memory leaks or dangling references that can cause your app to crash.   
  
  * **Be wary about publishing events between Fragments.**  Events cannot be published or received when a Fragment is not running.  If you have a Fragment publishing a message
  to another Fragment that isn't currently executing and then swap one for the other, it's likely the event will not be processed correctly.   The EventBus library has a way
  to replay this event, but the Otto framework does not.
      
## Setting up an Event Bus with Otto

### Setup 

Add this Gradle dependency to your `app/build.gradle` file:

```gradle
dependencies {
    compile 'com.squareup:otto:1.3.8'
}
```
### Implementing PubSub with Otto

### Implementing Events

* https://github.com/square/otto/tree/master/otto-sample

### Replacing LocalBroadcastManager with an Otto

### Updating the UI thread with Otto

* <http://awalkingcity.com/blog/2013/02/26/productive-android-eventbus/>
* <https://github.com/kevintanhongann/EventBusSample/>
* <http://nerds.weddingpartyapp.com/tech/2014/12/24/implementing-an-event-bus-with-rxjava-rxbus/>
* <http://timnew.me/blog/2014/12/06/typical-eventbus-design-patterns/>
* <https://github.com/kaushikgopal/Android-RxJava/>
* <https://gist.github.com/staltz/868e7e9bc2a7b8c1f754/>