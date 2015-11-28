## Overview

RxJava is a library used to help write in a style known as [reactive programming] (https://gist.github.com/staltz/868e7e9bc2a7b8c1f754#what-is-reactive-programming).  Reactive programming often requires thinking of everything in the context of processing asynchronous data streams, whether they are network calls, button clicks, or search text changes.  It can considered a huge departure from traditional programming models but became [popular within Netflix](http://www.infoq.com/presentations/rx-service-architecture) in 2014 for abstracting away the complexities of performing dependent API calls.

One of the best introductions about reactive programming is described [here]  (https://gist.github.com/staltz/868e7e9bc2a7b8c1f754).   There is a mental shift that must be made in thinking about how asynchronous data streams can be used to transform into other data streams, filtered into a new stream, or used to create other streams.    It sometimes is helpful to imagine these data streams along a time continuum whereby certain events will emit values (i.e. response from the network).  If these emissions are being monitored, they can be used to create another data stream.  

### Advantages

Here are a few advantages of using RxJava on Android:

 * **Simplifies the ability to chain async operations.**  If you need to make an API call that depends on another API call, you will likely end up implementing this call in the callback of the first one.  In addition, you will need to add logic to check whether the first API call succeeded attempting to make the next API call.  RxJava provides a framework for more clearly expressing these chained dependencies while also avoiding [callback hell](https://www.bignerdranch.com/blog/what-is-functional-reactive-programming/).

 * **Provides a way for declaring how concurrent operations should operate.**  If an error occurs during the background thread of [[AsyncTask|Creating-and-Executing-Async-Tasks]], it can be hard to surface this error on the main thread since the `onPostExecute()` method implicitly expects the result to succeed.   RxJava provides a potentially better solution for AsyncTask that enables errors can be noticed sooner.  In addition, RxJava provides a more flexible way for defining the threading model for both background and callback tasks.

 * **Provides powerful constructs for manipulating streams of data.**   Almost every single type of asynchronous event in Android requires some type of anonymous callback handler.  Within these callback handlers usually requires logic to determine whether the event should be responded.  RxJava helps provide a way to declare this logic more succinctly.

## Setup

Setup your `app/build.gradle`:

```gradle
dependencies {
  compile 'io.reactivex:rxjava:1.0.16'
  compile 'io.reactivex:rxandroid:1.0.1'
}
```

## References

* <https://github.com/ReactiveX/RxJava/wiki/The-RxJava-Android-Module>
* <http://saulmm.github.io/when-Iron-Man-becomes-Reactive-Avengers2/>
* <http://blog.stablekernel.com/replace-asynctask-asynctaskloader-rx-observable-rxjava-android-patterns/>
* <https://www.youtube.com/watch?v=_t06LRX0DV0/>
* <https://vimeo.com/144812843>
