## Attribution

This guide is **not original content** was adapted from this [reddit post](http://www.reddit.com/r/Android/comments/1roag7/i_developed_an_app_and_created_a_starting_guide/)  and [this document](https://docs.google.com/document/d/19uj9TS0Hzaxyv8IM1D20d23GdU9aAkXo5BYfL15VG0I/edit) which is the original source. The original content was created by [jokerbrb](http://www.reddit.com/user/jokerbrb). The information has been formatted, cleaned up and made available here.

In addition, resources linked from [HalfApp's "How to get started programming Android apps"](http://halfapp.com/blog/get-started-programming-android-apps/) blog post has been integrated into this article as well. We've also added resources of our own here including Java resources from [this gist](https://gist.github.com/nesquena/6136ca4c461a9309e579) and links that we recommend to people when first learning Android.

## High Level Guide

* If you are starting from scratch with no programming knowledge, you should learn Java first (see resources below)
* If you have never done Android development, first setup your Android environment by following these [Android Setup Slides](https://docs.google.com/presentation/d/1JOICG5Ow6QgMBrUKChenvSSxjylCpeQKvfvpPbJMx3M/edit)
* Consider starting with building a simple [todo app](http://goo.gl/pBKfYP) with slides or alternately with this [video tutorial](http://www.youtube.com/watch?v=Z149x12sXsw) - This is a great series that will guide you through the basics of an Android app. I recommend that you not only watch the video but repeat what he is doing simultaneously.
* A great companion book is [The Busy Coder's Guide to Android Development](http://commonsware.com/Android/). If you don't want or can't buy the book, consider looking at its many code examples [available for free](https://github.com/commonsguy/cw-omnibus/)
- Learn how to search for solutions before you ask questions; check sources like [StackOverflow](http://stackoverflow.com/) and the [cliffnotes](http://guides.thecodepath.com/android) for answers.
* Learn how to debug your code -- This means using **LogCat** and reading stack traces, also learning how to use breakpoints

## Learning to Program with Java

If you never programmed at all before or if you are interested in starting with the basics, spend the first few months just on learning Java. Learn the syntax and understand how everything works. You’ll need to be able to create classes, create and call methods, use interfaces as well as know how inheritance works, before you can go to the next step. These are the basics of Java, and you’ll use them extensively when developing Android apps. Helpful resources for learning Java:

* [CodePath Java for Android Slides](https://www.dropbox.com/s/6z8s8tlnnovqeij/Introduction%20to%20Java%20%28prequel%20for%20Android%29.pdf)
* [Programming by doing Java](http://programmingbydoing.com/) - Great step-by-step guide
* [Head First Java Book](http://www.amazon.com/Head-First-Java-2nd-Edition/dp/0596009208)
* [The Java help subreddit](http://www.reddit.com/r/javahelp)
* [The official Oracle Java tutorials](http://docs.oracle.com/javase/tutorial/getStarted/index.html)
* [Learn Java Online](http://www.learnjavaonline.org/) - Interactive Java Tutorial
* [Free Java Book](http://java2s.com/Book/Java/CatalogJava.htm) - Solid online book
* [Udemy Free Java Course](https://www.udemy.com/java-tutorial/) - Free videos to learn Java
* [CodingBat Java Exercises](http://codingbat.com/) - Exercises to practice Java
* [Best Ways to Learn Java Overview](http://www.onvard.com/tracks/best-way-to-learn-java) - Overview article
* [Java Language Fundamentals](http://en.wikibooks.org/wiki/Java_Programming/Language_Fundamentals)

Be sure to check out these particular Java topics as well:

* [Dealing with Exceptions](http://en.wikibooks.org/wiki/Java_Programming/Exceptions)
* [Understanding Threads and Runnables](http://en.wikibooks.org/wiki/Java_Programming/Threads_and_Runnables)
* [Using Interfaces](http://en.wikibooks.org/wiki/Java_Programming/Interfaces)
* [Using Event Listeners](http://docs.oracle.com/javase/tutorial/uiswing/events/intro.html)
* [Creating your own Callbacks and Listeners](http://stackoverflow.com/a/1477229/362298)

## Understanding HTTP and APIs

In addition to understanding Java and OOP, most Android development requires the use of data from a server-side database and as such requires you to have a solid conceptual understanding of RESTful APIs, consuming HTTP endpoints and parsing JSON responses. Here are a few resources to get started:

 * [Beginners Guide to REST](http://www.andrewhavens.com/posts/20/beginners-guide-to-creating-a-rest-api/)
 * [Another Guide to REST](http://net.tutsplus.com/tutorials/other/a-beginners-introduction-to-http-and-rest/)
 * [Documentation for Twitter API](https://dev.twitter.com/docs/api/1.1)
 * [JSON Overview Tutorial](http://www.copterlabs.com/blog/json-what-it-is-how-it-works-how-to-use-it/)
 * [Parsing JSON in Java](http://www.tutorialspoint.com/json/json_java_example.htm)

You also need to know how to understand XML since a great deal of Android development takes place in XML files:

 * [XML Basics](http://www.xmlnews.org/docs/xml-basics.html)
 * [W3Schools XML Tutorial](http://www.w3schools.com/xml/xml_whatis.asp)

Once you understand Java, OOP (Classes, Inheritance, etc), XML and APIs with JSON, you now have the proper context to enter the world of Android development.

## Practical Tips for Android Development

Understanding the concepts above is a good thing because it allows you search for practical ways to achieve something. Almost everything you can think of doing has already been tried and documented by someone else. You just need to know how to find it. Which leads me to my next set of things you may consider to learn first:

* **Learn how to search** -- I cannot stress this enough. Seriously, just Google it. If you are stuck or in doubt, search for your question. And most of all: Search before you ask.
* **Learn how to debug your code** There is a surprisingly high number of learners who do not debug their code at all and can't really [understand their code's log](http://stackoverflow.com/questions/6065258/how-to-interpret-logcat). If you are using Eclipse,  [check this out] (http://stackoverflow.com/questions/8551818/how-to-debug-android-application-line-by-line-using-eclipse)

Speaking of Eclipse, the IDE most people still use to program for Android, whether you want to use it or not is up to you. Bear in mind the following though:

  * The alternative IDE most people talk about, Android Studio (which is based on IntelliJ), is still in an **early access** preview. The official Android page has the [following warning about it](http://developer.android.com/sdk/installing/studio.html): "Android Studio is currently available as an early access preview. Several features are either incomplete or not yet implemented and you may encounter bugs."
  * Most tutorials and documentation for beginners are still based on the use of Eclipse. All the resources I talk about here which are connected to an IDE, refer to Eclipse.
  
## Beginning Android Resources

With the basics in mind, it is time to start coding your first Android app. To begin, download and install the [Android Developer Tools] (http://developer.android.com/sdk/index.html). The Android SDK is actually a bundle of helpful tools consisting of Android libraries, emulator, debugger and documentation. It gives you a framework of Java classes and methods that all Android devices are able to use. The whole SDK is neatly packaged inside Eclipse or Android Studio, allowing you to only worry about your code and how devices implement it.

Android apps are a bit different from ordinary Java applications, because they’re build around Activities and Fragments, which both have lifecycles that determine the state of the app. When learning Android it’s not about learning how to code, it’s more about understanding the way Android works. That means that you’ll spend the majority of the time learning about Activity lifecycles, Fragments, ListViews, Bundles and other important Android concepts.

Consider starting with the following tutorials and resources:

* [The Android official training guides](http://developer.android.com/training/index.html) are a good place to start. The [Building Your First App](http://developer.android.com/training/basics/firstapp/index.html) lesson is very easy to follow  and already gives you a good understanding of some key concepts of the Android SDK.

* [The Android Development Tutorial](http://www.youtube.com/watch?v=Z149x12sXsw) by Derek Banas is great for those of you who prefer video lessons. It has 25 video lessons in total ranging from 10 to 30 minutes each. I used the first lessons to get a feel of the development process, specially to understand layouts.

* [Android Guides](https://github.com/thecodepath/android_guides/wiki) give you even more explained code recipes and examples on how to build most common things in an Android app. I wish this was published when I started learning. It would have certainly helped.

* [Vogella Android Tutorials](http://www.vogella.com/tutorials/Android/article.html) - Awesome free tutorials for most common Android topics. Great as a supplementary resource on top of these [Android Cliffnotes](http://guides.thecodepath.com/android).
  
* [Common tasks](http://developer.android.com/guide/faq/commontasks.html) are a useful list of typical things you can do in your app and how to develop them.
  
* [Tasker](https://play.google.com/store/apps/details?id=net.dinglisch.android.taskerm&hl=en) is a great app available on the Google Play Store. You probably have heard of it. This app won't teach you anything about how to develop Android apps but it will show you what an Android app can do with your phone. I learned a lot from it. Whenever I realised I wanted to do something specific, I knew I could do it because I had done it before on Tasker.
  
* [DevAppsDirect](https://play.google.com/store/apps/details?id=com.inappsquared.devappsdirect&hl=en) is another great app you can get from the Play Store. While it also won't teach you how to develop an app, it will show you what is available out there. The app maintains a list of open source libraries you can use in your project for a variety of purposes. Knowing what you can reuse will save you a lot of time in the future.

In addition, my favorite book is by far [The Busy Coder's Guide to Android Development](http://commonsware.com/Android/). The book is comprehensive with its over 2,400 pages but starts with the basics, explaining the concepts. It also has those "do-it-yourself" tutorials to help you retain what you are learning. The book is a bit expensive but it comes with a one-year subscription to keep it updated during the period.

All code examples are free and [can be found here](https://github.com/commonsguy/cw-omnibus/). Even if you don't buy the book, consider browsing through some of the examples there to learn how other programmers do things. Finally, Mark Murphy, the author of the book, is helpful whether you contact him by e-mail or on the [StackOverflow website](http://stackoverflow.com/). Check out [his profile](http://stackoverflow.com/users/115145/commonsware). He is in the all time top-10 ranking list there.

## Key Concepts

### Configuration Changes

The most common of these is caused by orientation changes. Whenever there is an orientation change, your activity needs to be destroyed and recreated to address the changes in layout. This means you need to handle this recreation process yourself, making sure your app doesn't crash. A lot of beginners (myself included) consider simply disabling these changes but this is consider a bad practice. Besides, even if you do disable orientation changes, there are other things that can cause configuration changes that you need to handle anyway. 

Here are two good posts discussing this further: <http://stackoverflow.com/a/582585/362298> and <http://stackoverflow.com/questions/1111980/how-to-handle-screen-orientation-change-when-progress-dialog-and-background-thre>

To force yourself to catch problems sooner than later, consider the following tip from a previous [post here on Reddit](http://www.reddit.com/r/androiddev/comments/19143c/is_your_app_stateful_test_your_might/). You can enable a developer setting to not keep activities, so that they always get destroyed and recreated. 
  
And more specifically, here are two examples of dealing with this problem when using a FragmentPagerAdapter, a common use case:  

 - <http://stackoverflow.com/questions/17629463/fragmentpageradapter-how-to-handle-orientation-changes> 
 - <http://stackoverflow.com/questions/18425646/fragmentpageradapter-not-restoring-fragments-upon-orientation-change>

In a more abstract sense but still related to the topic, [an old post on avoiding memory leaks](http://www.curious-creature.org/2008/12/18/avoid-memory-leaks-on-android/ ) is probably worthy of your time as well. This was written by Romain Guy, a Google employee very active in the Android community. It is certainly worth [following him on Google+](https://plus.google.com/+RomainGuy).

### Parcelable

You can pass objects from one activity to another in a Bundle if your class implements the [Parcelable interface](http://developer.android.com/reference/android/os/Parcelable.html). Parcelables were designed specifically for performance and should almost always be used instead of Serializable. Here is a [good page explaining how to use it](http://shri.blog.kraya.co.uk/2010/04/26/android-parcel-data-to-pass-between-activities-using-parcelable-classes/). 
  
  You can also have inheritance while using it without adding too much of an overhead to the children [like this post](http://stackoverflow.com/questions/20018095/parcelable-inheritance-issue-with-getcreator-of-heir-class/20018463#20018463)
  
One thing you MUST always keep in mind is the order with which you write to the parcel and read from it. That order must be consistent. Otherwise you will start getting very crypt error messages which are very hard to debug.

### Threading

Android is full of features to help you deal with threads. This is a very important aspect of Android development because your app has to give snappy responses. So all your heavy work such as database operations and network access need to be done in a separate thread.
 
For this reason it is important to know when to use a `Service`, a `Thread`, an `IntentService` or an `AsyncTask`. Learn about to them, check examples, and make sure you use them whenever appropriate. Perhaps the best place to start is this post summarizing your options: http://techtej.blogspot.com.es/2011/03/android-thread-constructspart-4.html You will realize that knowing about callbacks and listeners will be useful here too.

### Broadcast Receivers

Broadcast receivers are a great way to have your asynchronous tasks (refer to the previous topic above) communicate with the main thread or to receive push notifications from your phone. It is a powerful feature to understand and use. 

Consider reading about it in the [official documentation](http://developer.android.com/reference/android/content/BroadcastReceiver.html). Also, refer again to the list of [common tasks](http://developer.android.com/guide/faq/commontasks.html) to get to know how to use them.

### Compatibility

According to the latest statistics I can see in my Developer Console, API 9 and above represents almost 97% of all active Android devices out there. The number of Android tablets is still much lower than that of Android phones. Take this year's Q1 sales for a reference. [28 million Android tablets were sold](http://www.businessinsider.com/android-ahead-of-ios-tablet-market-share-2013-5) as opposed to [156 million Android phones](http://www.gartner.com/newsroom/id/2482816) sold in the same period.
 
Supporting older Android versions turned out to be easier than I thought initially. I am using the [ActionBarSherlock](http://actionbarsherlock.com/) for providing the same UI experience in terms of ActionBar layout. I know now that the official Android Support Library has a similar library called [ActionBarCompat](http://android-developers.blogspot.pt/2013/08/actionbarcompat-and-io-2013-app-source.html) which is probably worth considering to avoid relying too much on third-party libraries.
 
Aside from the ActionBarSherlock, though, most things in the UI can done to all API levels using the [Support Library](http://developer.android.com/tools/support-library/index.html). You just have to make sure to use the right sets of classes. For example, instead of using "android.app.Fragment" when creating a new fragment, you use android.support.v4.app.Fragment instead.

## Wrapping Up

Again please note this guide above was originally posted [on reddit](http://www.reddit.com/r/Android/comments/1roag7/i_developed_an_app_and_created_a_starting_guide/) and was adapted here for wider access. For more details, check out the [original document](https://docs.google.com/document/d/19uj9TS0Hzaxyv8IM1D20d23GdU9aAkXo5BYfL15VG0I/edit).