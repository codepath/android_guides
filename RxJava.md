## Overview

RxJava is a library built for Java that is used to help write in a style known as [reactive programming] (https://gist.github.com/staltz/868e7e9bc2a7b8c1f754#what-is-reactive-programming).  Reactive programming often requires a significant mindset shift from traditional approaches because it means thinking of everything as asynchronous data streams.  Not only are mouse clicks asynchronous data streams, but also network calls, database fetches, or cache retrievals could be considered as well.

Imagine trying to perform a series of chained tasks using [[AsyncTask|Creating-and-Executing-Async-Tasks]].  We could implement the dispatching on the `onPostExecute()` of each task, but writing code to dispatch certain tasks depending on certain states will end up cause endless complicated code.  Reactive programming helps provide an abstraction layer that describes how these events should be sequenced.

Reactive programming also provides an abstraction layer to combine, create, or filter them in a way that is easier to understand and maintain for the developer.  

### Advantages

Advantages of using RxJava:

 * Powerful constructs for manipulating streams of data. 
 * Helps control the threading on which the work happens and when the callbacks are fired.

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