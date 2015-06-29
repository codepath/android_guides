## Overview

By now, you should be familiar with the various ways the Android framework relies on passing data between the various abstractions used:

 * Activities uses [[Intents|Using-Intents-to-Create-Flows]] to pass data between screens.
 * Fragments use [[multiple ways|Creating and Using-Fragments#communicating-with-fragments]] to communicate: Bundles to construct fragments, methods by the containing Activity to call, and the listener pattern for the Fragment to fire events.
 * Services rely on [[Broadcast Managers|Starting-Background-Services#communicating-from-a-service-to-an-application]] to send data to Applications.

One of primary issues of these current approaches is that they can create strong dependencies between each component, making it difficult to change one part of the system without impacting another area.  This [blog post](https://corner.squareup.com/2012/07/otto.html) describes the challenges of creating unmanageable dependencies with the current Android framework.  Instead of encouraging more modular designs, the communication patterns can sometimes do the exact opposite.

Publish/subscribe models try to avoid this tight integration by relying an event bus model.  In this type of model, there are publishers and subscribers.  Publishers are responsible for posting events in response to some type of state change, while subscribers respond to these events.  The event acts as an intermediary for exchanging information, isolating and minimizing the dependencies between each side.  In this way, message buses create a communication pipeline that is intended to help make your app more maintainable and scalable.

There are many different libraries which attempt to enable the event bus model, including [EventBus](https://github.com/greenrobot/EventBus), [RxJava](https://github.com/ReactiveX/RxJava), and [Otto](https://github.com/square/otto).   Otto is by far the easiest to start using immediately.  EventBus has a few more advanced features than in Otto described in this [comparison chart](http://timnew.me/blog/2014/09/14/otto-and-android-annotations-compatibility-issue-analysis/), and RxJava enables
a way to declare [sequences of actions](http://www.infoq.com/news/2014/11/android-rxjava-at-soundcloud) to be taken in response to certain events.  

### Considerations

When using an event bus model, there are several considerations:
  * *Don't assume you should replace every communication pattern with an event bus model*.  If you publish an event meant only for one subscriber that shouldn't trigger changes in other subscribers, rely on 1-to-1 communication patterns such as the [[Intents|Using Intents to Create Flows]] or [[listener pattern|Creating Custom Listeners]].  For instance, a date picker fragment that could be reused in multiple components of your app should probably expose a listener interface since using a event bus model could cause this event to be published to many components waiting to respond to changes in the date selection.    Consider the design patterns described in this [article](http://timnew.me/blog/2014/12/06/typical-eventbus-design-patterns/) too.

  * *If you are using publishing or subscribers within an Activity of Fragment they should be registered and unregistered with their lifecycle.*  Otherwise, it's likely you will encounter memory leaks or dangling references that can cause your app to crash.

  * *Be wary about publishing events between Fragments.*  Events cannot be published or received when a Fragment is not running.  If you have a Fragment publishing a message to another Fragment that isn't currently executing and then swap one for the other, it's likely the event will not be processed correctly.   The EventBus library has a way
  to replay this event, but the Otto framework does not.


## Setting up an Event Bus with Otto

Add this Gradle dependency to your `app/build.gradle` file:

```gradle
dependencies {
    compile 'com.squareup:otto:1.3.8'
}
```

### Replacing Local Broadcast Managers with Otto

One primary use case that the event bus model works well is replacing it lieu of the [[LocalBroadcastManager|Starting-Background-Services#communicating-with-a-broadcastreceiver]] when communicating from a service to the application.   One advantage is that you can simplify a lot of code that normally is used to serialize or deserialize the data by avoiding the need to pass Intents.

### Creating a Bus

The first step we need to do is create a Singleton bus class.   This shared bus will be used by your application
to publish and subscribe to events.  

```java
public final class BusProvider {
    private static final Bus BUS = new Bus();

    public static Bus getInstance() {
        return BUS;
    }

    private BusProvider() {
        // No instances.
    }
}
```

Since services normally run on separate threads from the main UI thread, we must also ensure that the events are published on the main thread by subscribers to handle.  We expose some extra functionality to provide the ability for services to explicitly publish on the main thread:

```java
    private static final Handler mainThread = new Handler(Looper.getMainLooper());

    public static void postOnMain(final Object event) {
        if (Looper.myLooper() == Looper.getMainLooper()) {
            getInstance().post(event);
        } else {
            mainThread.post(new Runnable() {
                @Override
                public void run() {
                    getInstance().post(event);

                }
            });
        }
    }
```

### Registering on the Bus

If you wish to subscribe Activities and Fragments to the event bus, it should be done with respect to the lifecycle
of these objects.  The reason is that when an event is published, Otto will try to find all the registered subscribers.
If there is a subscriber attached to the Activity currently not running, there is likely to be a stale reference
that causes your app to crash.

```java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onPause() {
        super.onPause();
        BusProvider.getInstance().unregister(this);
    }

    @Override
    protected void onResume() {
        super.onResume();
        BusProvider.getInstance().register(this);
    }
```

### Creating an Event

Create a new Event from a standard Java class.   You can define any set of members variables.  For now, we
create a class to accept a result code and a String values.

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
      BusProvider.postOnMain(new IntentServiceResult(Activity.RESULT_OK, "done!!"));
  }
```

### Creating a subscriber

We simply need to annotate a method that has this specific event type with the `@Subscribe` decorator.  The method must also take in the parameter of the event for which it is listening.  In this case, it's `IntentServiceResult`.

Assuming the Activity is registered on the bus, Otto will look for any subscribers when this specific event is published.  We can display the result value to verify that the data passed is correct.

```java
   public class MainActivity extends ActionBarActivity {

   @Subscribe public void doThis(IntentServiceResult intentServiceResult) {
       Toast.makeText(this, intentServiceResult.getResultValue(), Toast.LENGTH_SHORT).show();
   }
```

### Using Otto with Dagger (TODO)

## References

* <http://awalkingcity.com/blog/2013/02/26/productive-android-eventbus/>
* <https://github.com/kevintanhongann/EventBusSample/>
* <http://nerds.weddingpartyapp.com/tech/2014/12/24/implementing-an-event-bus-with-rxjava-rxbus/>
* <http://timnew.me/blog/2014/12/06/typical-eventbus-design-patterns/>
* <https://github.com/kaushikgopal/Android-RxJava/>
* <https://gist.github.com/staltz/868e7e9bc2a7b8c1f754/>
* <https://github.com/square/otto/tree/master/otto-sample/>