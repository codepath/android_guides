## Attribution

This guide was adapted from this [reddit post](http://www.reddit.com/r/Android/comments/1roag7/i_developed_an_app_and_created_a_starting_guide/)  and [this document](https://docs.google.com/document/d/19uj9TS0Hzaxyv8IM1D20d23GdU9aAkXo5BYfL15VG0I/edit) which is the original source. The original content was created by [jokerbrb](http://www.reddit.com/user/jokerbrb). The information has been formatted and made available here.

## High Level Guide

* If you are starting from scratch, first get the [Android SDK here](http://developer.android.com/sdk/index.html)
* If you are really starting from scratch, you should learn Java first (see resources below)
* Consider starting with this [video tutorial](http://www.youtube.com/watch?v=Z149x12sXsw) - This is a great series that will guide you through the basics of an Android app. I recommend that you not only watch the video but repeat what he is doing simultaneously.
* A great companion book is [The Busy Coder's Guide to Android Development](http://commonsware.com/Android/). If you don't want or can't buy the book, consider looking at its many code examples [available for free](https://github.com/commonsguy/cw-omnibus/)
- Learn how to search for solutions before you ask questions; check sources like [StackOverflow](http://stackoverflow.com/) and the [cliffnotes](http://guides.thecodepath.com/android) for answers.
* Learn how to debug your code -- This means using **LogCat** and reading stack traces, also learning how to use breakpoints
* Your friends can be a great source of help and feedback. Ask kindly, do not harass them. 
* Unless your app is the next big thing since sliced bread, do not bother too much with announcing it before it's ready. There are too many "pre-apps" out there, nobody cares.

## Starting from the Beginning

If you never programmed at all before or if you are interested in some basic information, consider looking at these first:

* [Java Language Fundamentals](http://en.wikibooks.org/wiki/Java_Programming/Language_Fundamentals)
* [Dealing with Exceptions](http://en.wikibooks.org/wiki/Java_Programming/Exceptions)
* [Understanding Threads and Runnables](http://en.wikibooks.org/wiki/Java_Programming/Threads_and_Runnables)
* [Using Interfaces](http://en.wikibooks.org/wiki/Java_Programming/Interfaces)
* [Using Event Listeners](http://docs.oracle.com/javase/tutorial/uiswing/events/intro.html)
* [Creating your own Callbacks and Listeners](http://stackoverflow.com/a/1477229/362298)

Understanding the concepts is a good thing because it lets you search for practical ways to achieve something. Almost everything you can think of doing has already been tried and documented by someone else. You just need to know how to find it. Which leads me to my next set of things you may consider to learn first:

* **Learn how to search** -- I cannot stress this enough. Seriously, just Google it. If you are stuck or in doubt, search for your question. And most of all: Search before you ask.
* **Learn how to debug your code** There is a surprisingly high number of learners who do not debug their code at all and can't really [understand their code's log](http://stackoverflow.com/questions/6065258/how-to-interpret-logcat). If you are using Eclipse,  [check this out] (http://stackoverflow.com/questions/8551818/how-to-debug-android-application-line-by-line-using-eclipse)

Speaking of Eclipse, the IDE most people still use to program for Android, whether you want to use it or not is up to you. Bear in mind the following though:

  * The alternative IDE most people talk about, Android Studio (which is based on IntelliJ), is still in an **early access** preview. The official Android page has the [following warning about it](http://developer.android.com/sdk/installing/studio.html): "Android Studio is currently available as an early access preview. Several features are either incomplete or not yet implemented and you may encounter bugs."
  * Most tutorials and documentation for beginners are still based on the use of Eclipse. All the resources I talk about here which are connected to an IDE, refer to Eclipse.
  
## Beginning Android Resources

With the basics in mind, it is time to start coding your first Android app. To begin, download and install the [Android Developer Tools] (http://developer.android.com/sdk/index.html). Then consider the following tutorials and resources:

* [The Android official training guides](http://developer.android.com/training/index.html) are a good place to start. The [Building Your First App](http://developer.android.com/training/basics/firstapp/index.html) lesson is very easy to follow  and already gives you a good understanding of some key concepts of the Android SDK.

* [The Android Development Tutorial](http://www.youtube.com/watch?v=Z149x12sXsw) by Derek Banas is great for those of you who prefer video lessons. It has 25 video lessons in total ranging from 10 to 30 minutes each. I used the first lessons to get a feel of the development process, specially to understand layouts.
  
* [Common tasks](http://developer.android.com/guide/faq/commontasks.html) are a useful list of typical things you can do in your app and how to develop them.
  
* [Android Guides](https://github.com/thecodepath/android_guides/wiki) give you even more explained code recipes and examples on how to build most common things in an Android app. I wish this was published when I started learning. It would have certainly helped.
  
* [Tasker](https://play.google.com/store/apps/details?id=net.dinglisch.android.taskerm&hl=en) is a great app available on the Google Play Store. You probably have heard of it. This app won't teach you anything about how to develop Android apps but it will show you what an Android app can do with your phone. I learned a lot from it. Whenever I realised I wanted to do something specific, I knew I could do it because I had done it before on Tasker.
  
* [DevAppsDirect](https://play.google.com/store/apps/details?id=com.inappsquared.devappsdirect&hl=en) is another great app you can get from the Play Store. While it also won't teach you how to develop an app, it will show you what is available out there. The app maintains a list of open source libraries you can use in your project for a variety of purposes. Knowing what you can reuse will save you a lot of time in the future.

In addition, my favorite book is by far [The Busy Coder's Guide to Android Development](http://commonsware.com/Android/). The book is comprehensive with its over 2,400 pages but starts with the basics, explaining the concepts. It also has those "do-it-yourself" tutorials to help you retain what you are learning. The book is a bit expensive but it comes with a one-year subscription to keep it updated during the period.

All code examples are free and can be found here:  https://github.com/commonsguy/cw-omnibus/ Even if you don't buy the book, consider browsing through some of the examples there to learn how other programmers do things. Finally, Mark Murphy, the author of the book, is helpful whether you contact him by e-mail or on the [Stackoverflow website](http://stackoverflow.com/). Check out [his profile](http://stackoverflow.com/users/115145/commonsware). He is in the all time top-10 ranking list there.

## Wrapping Up

Again this guide above was originally posted [on reddit](http://www.reddit.com/r/Android/comments/1roag7/i_developed_an_app_and_created_a_starting_guide/) and was adapted here for wider access.