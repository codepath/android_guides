## Overview

One of the challenges in writing robust Android apps is the dynamic nature of changing inputs.   In traditional imperative programming models, values have to be explicitly set on variables for them to be updated.  If one dependent value changes, the value will not be updated without adding another line of code.   Consider the following example:

```java
// init variables
int i, j, k; 

// Init inputs
i = 1;
j = 2;

// Set output value
k = i + j;

// Update a dependent value
j = 4;
k = ?  // What should k be?
```

Traditional asynchronous programming approaches tend to rely on callbacks to update these changes, but layering callbacks upon callbacks leads to a problem known as [callback hell](http://callbackhell.com/).  **_Reactive programming_** (see intro [here](https://gist.github.com/staltz/868e7e9bc2a7b8c1f754)) addresses these issues by providing a framework to describe outputs to reflect their changing inputs.  RxJava, which is a port of the [Reactive Extensions](https://msdn.microsoft.com/en-us/data/gg577609.aspx) library from .NET, enables Android apps to be built in this style.

### Advantages

Here are a few advantages of using RxJava on Android:

 * **Simplifies the ability to chain async operations.**  If you need to make an API call that depends on another API call, you will likely end up implementing this call in the callback of the first one.  RxJava provides a way to avoid needing to creating [layers of callbacks](https://www.bignerdranch.com/blog/what-is-functional-reactive-programming/) to address this issue.  For this reason, RxJava became [popular within Netflix](http://www.infoq.com/presentations/rx-service-architecture) in 2014 for abstracting away the complexities of performing dependent API calls.

 * **Exposes a more explicit way for declaring how concurrent operations should operate.**  Although RxJava is  single-threaded by default, RxJava helps enable you to define more explicitly what type of threading models should be used for both background and callback tasks.  Since Android only allows UI updates on the main thread, using RxJava helps make the code more clear about what operations will be done to update the views. 

 * **Surfaces errors sooner.** One issue with [[AsyncTask|Creating-and-Executing-Async-Tasks]] is that errors that occur on the background thread are hard to pass along when updating the UI thread using the `onPostExecute()` method.  In addition, there are limitations in how many AsyncTasks can be dispatched concurrently as described in this [blog post](http://blog.danlew.net/2014/06/21/the-hidden-pitfalls-of-asynctask/).  RxJava provides a way to enable these errors to be surfaced.

 * **Provides powerful constructs for manipulating streams of data.**  With only a few lines of code, we can write code to aggregate data from different web service calls and filter, merge, or sort them.

## Setup

Setup your `app/build.gradle`:

```gradle
dependencies {
  compile 'io.reactivex:rxjava:1.0.16'
  compile 'io.reactivex:rxandroid:1.0.1'
}
```

## Replacing AsyncTask

```java
public Observable<Bitmap> getImageNetworkCall() {
    // Insert network call here!
}

Subscription subscription = getImageNetworkCall()
    .subscribeOn(Schedulers.io())
    .observeOn(AndroidSchedulers.mainThread())
    .subscribe(new Observer<Bitmap>() {

        @Override
        public void onCompleted() {
             // Update user interface if needed
        }

        @Override
        public void onError() {
             // Update user interface to handle error
        }

        @Override
        public void onNext(Bitmap bitmap) {
             // Handle result of network request
        }
});
```

## References

* <https://github.com/ReactiveX/RxJava/wiki/The-RxJava-Android-Module>
* <http://saulmm.github.io/when-Iron-Man-becomes-Reactive-Avengers2/>
* <http://blog.stablekernel.com/replace-asynctask-asynctaskloader-rx-observable-rxjava-android-patterns/>
* <https://www.youtube.com/watch?v=_t06LRX0DV0/>
* <https://vimeo.com/144812843>
* <http://code.hootsuite.com/asynchronous-android-programming-the-good-the-bad-and-the-ugly/>
* <https://www.youtube.com/watch?v=va1d4MqLUGY&feature=youtu.be/>
* <http://www.slideshare.net/TimoTuominen1/rxjava-architectures-on-android-android-livecode-berlin/>
* <https://www.youtube.com/watch?v=va1d4MqLUGY&feature=youtu.be/>