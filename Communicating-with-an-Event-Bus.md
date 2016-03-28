## Overview

By now, you should be familiar with the various ways the Android framework relies on passing data between the various abstractions used:

 * Activities uses [[Intents|Using-Intents-to-Create-Flows]] to pass data between screens.
 * Fragments use [[multiple ways|Creating and Using-Fragments#communicating-with-fragments]] to communicate: Bundles to construct fragments, methods by the containing Activity to call, and the listener pattern for the Fragment to fire events.
 * Services rely on [[Broadcast Managers|Starting-Background-Services#communicating-from-a-service-to-an-application]] to send data to Applications.

One of primary issues of these current approaches is that they can create strong dependencies between each component, making it difficult to change one part of the system without impacting another area.  This [blog post](https://corner.squareup.com/2012/07/otto.html) describes the challenges of creating unmanageable dependencies with the current Android framework.  Instead of encouraging more modular designs, the communication patterns can sometimes do the exact opposite.

Publish/subscribe models try to avoid this tight integration by relying on an event bus model.  In this type of model, there are publishers and subscribers.  Publishers are responsible for posting events in response to some type of state change, while subscribers respond to these events.  The event acts as an intermediary for exchanging information, isolating and minimizing the dependencies between each side.  In this way, message buses create a communication pipeline that is intended to help make your app more maintainable and scalable.  

One additional benefit of using these pub/sub frameworks is that they help facilitate passing Java objects between Activities, Fragments, or Services.  You don't need to spend time serializing and deserializing data, which can often creates the tight bindings between these components.   It also helps enforce more type-safety across complex Java objects.  

There are many different libraries which attempt to enable the event bus model, including [EventBus](https://github.com/greenrobot/EventBus), [RxJava](https://github.com/ReactiveX/RxJava), and [Otto](https://github.com/square/otto).   Otto has been deprecated in favor of RxJava/RxAndroid approaches.  EventBus has a few more advanced features than in Otto described in this [comparison chart](https://github.com/greenrobot/EventBus/blob/master/COMPARISON.md) and recently has become the more supported Java library.

### Considerations

Event buses are especially helpful include notifying activities or fragments when tasks are completed, such as when a AsyncTask or a Background Service finishes.  When using an event bus model, there are several considerations:

  * *Don't assume you should replace every communication pattern with an event bus model*.  If you publish an event meant only for one subscriber that shouldn't trigger changes in other subscribers, rely on 1-to-1 communication patterns such as the [[Intents|Using Intents to Create Flows]] or [[listener pattern|Creating Custom Listeners]].  For instance, a date picker fragment that could be reused in multiple components of your app should probably expose a listener interface since using a event bus model could cause this event to be published to many components waiting to respond to changes in the date selection.    Consider the design patterns described in this [article](http://timnew.me/blog/2014/12/06/typical-eventbus-design-patterns/) too.

  * *If you are using publishing or subscribers within an Activity of Fragment they should be registered and unregistered with their lifecycle.*  Otherwise, it's likely you will encounter memory leaks or dangling references that can cause your app to crash.

  * *Be wary about publishing events between Fragments.*  Events cannot be published or received when a Fragment is not running.  If you have a Fragment publishing a message to another Fragment that isn't currently executing and then swap one for the other, it's likely the event will not be processed correctly.   The EventBus library has a way to replay this event, but the Otto framework does not.

## Setting up an Event Bus with EventBus

Add this Gradle dependency to your `app/build.gradle` file:

```gradle
dependencies {
    compile 'org.greenrobot:eventbus:3.0.0'
}
```

### Replacing Local Broadcast Managers with EventBus

One primary use case that the event bus model works well is replacing it lieu of the [[LocalBroadcastManager|Starting-Background-Services#communicating-with-a-broadcastreceiver]] when communicating from a service to the application.   One advantage is that you can simplify a lot of code that normally is used to serialize or deserialize the data by avoiding the need to pass Intents.

### Registering on the Bus

If you wish to subscribe Activities and Fragments to the event bus, it should be done with respect to the lifecycle of these objects.  The reason is that when an event is published, EventBus will try to find all the registered subscribers.  If there is a subscriber attached to the Activity currently not running, there is likely to be a stale reference that causes your app to crash.

```java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onPause() {
        super.onPause();
        EventBus.getDefault().unregister(this);
    }

    @Override
    protected void onResume() {
        super.onResume();
        EventBus.getDefault().register(this);
    }
```

### Creating an Event

Create a new Event from a standard Java class.   You can define any set of members variables.  For now, we create a class to accept a result code and a String values.

```java
public class IntentServiceResult {

    int mResult;
    String mResultValue;

    IntentServiceResult(int resultCode, String resultValue) {
        mResult = resultCode;
        mResultValue = resultValue;
    }

    public int getResult() {
        return mResult;
    }

    public String getResultValue() {
        return mResultValue;
    }
}
```

### Creating a publisher

Inside our Intent service, we can should publish the event.  We will simply pass an `OK` result with
a message that will be displayed when the event is received on the Activity.

```java
@Override
protected void onHandleIntent(Intent intent) {

    // do some work
    EventBus.getDefault().post(new IntentServiceResult(Activity.RESULT_OK, "done!!"));
}
```

### Creating a subscriber

We simply need to annotate a method that has this specific event type with the `@Subscribe` decorator.  The method must also take in the parameter of the event for which it is listening.  In this case, it's `IntentServiceResult`.

Because the `IntentService` is executed on a separate thread, the subscribers will normally also execute on the same thread.   If we need to make UI changes on the main thread, we need to be explicit that the actions executed should be done on the main thread.  See [this section](http://greenrobot.org/eventbus/documentation/delivery-threads-threadmode/) for more information. 

```java
public class MainActivity extends AppCompatActivity {

@Subscribe(threadMode = ThreadMode.MAIN)
public void doThis(IntentServiceResult intentServiceResult) {
       Toast.makeText(this, intentServiceResult.getResultValue(), Toast.LENGTH_SHORT).show();
}
```

## References

* <http://awalkingcity.com/blog/2013/02/26/productive-android-eventbus/>
* <https://github.com/kevintanhongann/EventBusSample/>
* <http://nerds.weddingpartyapp.com/tech/2014/12/24/implementing-an-event-bus-with-rxjava-rxbus/>
* <http://timnew.me/blog/2014/12/06/typical-eventbus-design-patterns/>
* <https://github.com/kaushikgopal/Android-RxJava/>
* <https://gist.github.com/staltz/868e7e9bc2a7b8c1f754/>
* <http://www.stevenmarkford.com/passing-objects-between-android-activities/>
* <http://greenrobot.org/eventbus/documentation/>