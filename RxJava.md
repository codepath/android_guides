## Overview

RxJava is a library used to help write in a style known as [reactive programming] (https://gist.github.com/staltz/868e7e9bc2a7b8c1f754#what-is-reactive-programming).  Reactive programming often requires thinking of everything in the context of processing asynchronous data streams, whether they are network calls, button clicks, or search text changes.  By defining what the relationship of these data streams of what they should represent, this approach is considered a huge departure from the traditional imperative programming model.

One of the best introductions about reactive programming is described [here]  (https://gist.github.com/staltz/868e7e9bc2a7b8c1f754).   There is a mental shift that must be made in thinking about how asynchronous data streams can be used to transform into other data streams, filtered into a new stream, or used to create other streams.    It sometimes is helpful to imagine these data streams along a time continuum whereby certain producers emit values and downstream subscribers respond to these changes.

### Advantages

Here are a few advantages of using RxJava on Android:

 * **Simplifies the ability to chain async operations.**  If you need to make an API call that depends on another API call, you will likely end up implementing this call in the callback of the first one.  In addition, you will need to add logic to check whether the first API call succeeded attempting to make the next API call.  RxJava provides a way to avoid [callback hell](https://www.bignerdranch.com/blog/what-is-functional-reactive-programming/).  For this reason, RxJava became [popular within Netflix](http://www.infoq.com/presentations/rx-service-architecture) in 2014 for abstracting away the complexities of performing dependent API calls.

 * **Exposes a more flexible way for declaring how concurrent operations should operate.**  There are also many different issues with [[AsyncTask|Creating-and-Executing-Async-Tasks]] as described in this [blog post](http://blog.danlew.net/2014/06/21/the-hidden-pitfalls-of-asynctask/) that can be easily avoided by using RxJava.   There is also the issue about how errors that occur on the background thread can be surfaced easily with AsyncTask, since errors in `doInBackground()` do not get propagated to the `onPostExecute()` method.  Not only can RxJava enables errors to be propagated, but it also allows for what type of threading model should be used for both background and callback tasks.

 * **Provides powerful constructs for manipulating streams of data.**  With only a few lines of code, we can write code to aggregate data from different web service calls and filter, merge, or sort them with only a few lines of code.

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
