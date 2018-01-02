## Attributions

This guide is **not original content**, instead this post is an amalgamation of numerous other guides discussing the process of learning Android. This guide currently incorporates content from the following list of sources:

* This guide was originally adapted from this [reddit post](http://www.reddit.com/r/Android/comments/1roag7/i_developed_an_app_and_created_a_starting_guide/)  and [this document](https://docs.google.com/document/d/19uj9TS0Hzaxyv8IM1D20d23GdU9aAkXo5BYfL15VG0I/edit) which is the original source. The original content was created by [jokerbrb](http://www.reddit.com/user/jokerbrb), developer of [The Talker App](https://play.google.com/store/apps/details?id=com.thetalkerapp.main). 

* [This other reddit post](http://www.reddit.com/r/Android/comments/1w3woc/a_step_by_step_guide_about_how_to_get_started_and/) by [santaschesthairs](http://www.reddit.com/user/santaschesthairs). The information has been formatted, cleaned up and made available here. 

* [HalfApp's "How to get started programming Android apps"](http://halfapp.com/blog/get-started-programming-android-apps/) blog post has been integrated into this article as well. We've also added resources of our own here including Java resources from [this gist](https://gist.github.com/nesquena/6136ca4c461a9309e579) and links that we recommend to people when first learning Android.

* [HawkBlue](http://www.reddit.com/user/Hawk_Blue)'s post about [his Android journey](http://www.reddit.com/r/Android/comments/2dofph/just_published_my_first_app_a_comprehensive_guide/) has been incorporated as well. You can check out his published [GPA Calculator](https://play.google.com/store/apps/details?id=com.praneeth.gpacalculator&hl=en) on the Play Store.

Thanks to these original content authors for helping us put together this useful resource.

## High Level Guide

* **Installation** - If you have never done Android development, first setup your Android environment by following these [Android Setup Slides](https://docs.google.com/presentation/d/1iD0sMc-qIG80yZ1AQfDU5nxSAl3Xe4nx-2W_g9yzMSM/edit#slide=id.p)
* **Programming** - If you are starting with no programming knowledge, we recommend you learn Java first (see resources in next section)
* **First App** - Consider starting by building a simple [todo app](http://goo.gl/pBKfYP) by following these slides. Alternately try this [video tutorial](http://www.youtube.com/watch?v=Z149x12sXsw) that will guide you through the basics of an Android app. I recommend that you not only watch the video but repeat what he is doing simultaneously.
- **Finding Solutions** - Learn how to search for solutions before you ask questions; check sources like [StackOverflow](http://stackoverflow.com/) and the [cliffnotes](http://guides.codepath.com/android) for answers. Learn how to [[debug your code|Debugging-Exceptions-within-your-App]]. This means using **LogCat** and reading stack traces, also learning [how to use breakpoints](http://guides.codepath.com/android/Debugging-Exceptions-within-your-App#investigation-methodologies).
* **Reference Book** - A great companion book is [The Busy Coder's Guide to Android Development](http://commonsware.com/Android/). If you don't want or can't buy the book, consider looking at its many code examples [available for free](https://github.com/commonsguy/cw-omnibus/)

## Learning to Program with Java

If you never programmed at all before or if you are interested in starting with the basics, spend the first few months just on learning Java. Learn the syntax and understand how everything works. You’ll need to be able to create classes, create and call methods, use interfaces as well as know how inheritance works, before you can go to the next step. These are the basics of Java, and you’ll use them extensively when developing Android apps. Helpful resources for learning Java:

* [Start with these Java Interactive Guides](https://www.ktbyte.com/java-tutorial/book#chapters)
* [CodePath Java for Android Slides](https://www.dropbox.com/s/4u8jjlgtrm65a8q/Introduction%20to%20Java%20%28prequel%20for%20Android%29.pdf)
* [Programming by doing Java](http://programmingbydoing.com/) - Great step-by-step guide
* [Head First Java Book](http://www.amazon.com/Head-First-Java-2nd-Edition/dp/0596009208)
* [The Java help subreddit](http://www.reddit.com/r/javahelp)
* [The official Oracle Java tutorials](http://docs.oracle.com/javase/tutorial/getStarted/index.html)
* [TutsPlus Java Overview](http://mobile.tutsplus.com/tutorials/android/java-tutorial/)
* [Learn Java Online](http://www.javastring.org/) - Interactive Java Tutorial
* [Free Java Book](http://java2s.com/Book/Java/CatalogJava.htm) - Solid online book
* [Udemy Free Java Course](https://www.udemy.com/java-tutorial/) - Free videos to learn Java
* [CodingBat Java Exercises](http://codingbat.com/) - Exercises to practice Java
* [Best Ways to Learn Java Overview](http://www.onvard.com/tracks/best-way-to-learn-java) - Overview article
* [Java Language Fundamentals](http://en.wikibooks.org/wiki/Java_Programming/Language_Fundamentals)
* [Java: A Beginners Guide](http://www.amazon.com/Java-Beginners-Guide-Herbert-Schildt/dp/0071809252)
* [TutorialsPoint Java Guide](http://www.tutorialspoint.com/java/java_basic_syntax.htm)
* [Java tutorial](https://hackr.io/tutorials/learn-java) - A programming community & a great place to find the best online programming courses and tutorials.

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
 * [Documentation for Twitter API](https://dev.twitter.com/overview/api)
 * [JSON Overview Tutorial](http://www.copterlabs.com/blog/json-what-it-is-how-it-works-how-to-use-it/)
 * [Parsing JSON in Java](http://www.tutorialspoint.com/json/json_java_example.htm)

You also need to know how to understand XML since a great deal of Android development takes place in XML files:

 * [XML Basics](http://www.xmlnews.org/docs/xml-basics.html)
 * [W3Schools XML Tutorial](http://www.w3schools.com/xml/xml_whatis.asp)

Once you understand Java, OOP (Classes, Inheritance, etc), XML and APIs with JSON, you now have the proper context to enter the world of Android development.

## Practical Tips for Android Development

Understanding the concepts above is a good thing because it allows you search for practical ways to achieve something. Almost everything you can think of doing has already been tried and documented by someone else. You just need to know how to find it. Which leads me to my next set of things you may consider to learn first:

* **Learn how to search** -- We cannot stress this enough. If you are stuck or in doubt, search for your question on Google. And most of all: Search before you ask.
* **Learn how to debug your code** There is a surprisingly high number of developers who do not debug their code at all and can't really [understand their code's log](http://stackoverflow.com/questions/6065258/how-to-interpret-logcat). Refer to our comprehensive [[debugging your exceptions|Debugging-Exceptions-within-your-App]] to understand how to investigate and solve your own issues.

## Designing Android Apps

In terms of designing a good looking app, here are some links that will help you start looking at ideas and acquiring design assets like icons. 

### Assets

There are various sites that provide icons, colors or other assets for your apps:

 * [MaterialDesignIcons](http://materialdesignicons.com/) - Good icons for material designed apps
 * [IconFinder](https://www.iconfinder.com/) - Excellent source of free to use graphics
 * [iconmonstr](http://iconmonstr.com/) - Great source of icons with customized shapes and colors
 * [NounProject](http://thenounproject.com/) - Another source of free icons and graphics
 * [Icon Generators](http://romannurik.github.io/AndroidAssetStudio/) - Easily generate icons for use in your apps
 * [MaterialPalette](http://www.materialpalette.com/) - Easy color palette selection for your apps. 
 * [Android Cheatsheet](http://petrnohejl.github.io/Android-Cheatsheet-For-Graphic-Designers/) - Great summary of design specs and dimensions

### Principles

There are various guides that provide insight into the design principles for Android:

 * [Android Design Principles](http://developer.android.com/design/get-started/principles.html)
 * [Common Mistakes while Porting](http://www.androiduipatterns.com/2012/05/common-mistakes-in-porting-ios-apps-to.html) - Highlights some common mistakes people make while porting their iOS app to Android.
 * [[Material Design Primer|Material-Design-Primer]] - Walks you through the various elements of a modern material design for Android. "Material Design" is the latest approach to designing beautiful Android apps. 

### Inspiration

There are various sites that provide inspiration by showcasing beautiful modern app design:

 * [UI-Oh-My](http://uiohmy.com/) - Great resource for seeing screenshots of apps.
 * [inspired-ui](http://android.inspired-ui.com/) - Endless list of beautiful UI design
 * [mobilepatterns](https://www.mobile-patterns.com/android) - Common screen patterns neatly organized
 * [pttrns](http://pttrns.com/android-patterns) - List of android screens in categories
 * [androidniceties](http://androidniceties.tumblr.com/) - Blog of beautiful interfaces
 * [androidux](http://androidux.com/) - Explores common patterns for UX
 * [androidpatterns](http://www.androidpatterns.com/) - UI patterns organized by questions, such as "How can I let users manage their accounts?"
  
## Beginning Android Resources

With the basics in mind, it is time to start coding your first Android app. To begin you need to [setup Android Studio and the Android SDK](http://goo.gl/X2SVFR). The Android SDK is actually a bundle of helpful tools consisting of Android libraries, an emulator, a debugger, and documentation. It also gives you a framework of Java classes and methods that all Android devices are able to use, the core Android library. You'll have to update this library as new versions come out, but more on that later. The whole SDK is packaged inside Android Studio, allowing you to only worry about your code and how devices implement it.

When learning Android it’s not about learning how to code, it’s more about understanding the way that Android works. That means that you’ll spend the majority of the time learning about [[Activity Lifecycle|Activity-Lifecycle]], [[Fragments|Creating-and-Using-Fragments]], [[ListViews|Using-an-ArrayAdapter-with-ListView]], [[Intents|Using-Intents-to-Create-Flows]], and other important Android specific concepts as opposed to algorithms or data structures. 

### Courses

The following online courses teach Android through structured lessons and quizzes:

* [Udacity Android Development for Beginners](https://www.udacity.com/course/android-development-for-beginners--ud837) (Beginner) - This course is designed for students who are new to programming, and want to learn how to build Android apps. You don’t need any programming experience to take this course. 

* [Developing Android Apps by Google](https://www.udacity.com/course/ud853) (Intermediate) - Udacity course created by Google that teaches the core concepts involved in developing Android apps through videos and course work. 

* [Programming Android Applications on Coursera](https://www.coursera.org/course/android) - Coursera online course from the University of Maryland.

* [Building Mobile Applications Course](http://cs76.tv/2012/spring/#about,lectures) - Courseware including videos and slides with high-level overview of Android development.

* [Google Android Interactive Codelabs](https://codelabs.developers.google.com/?cat=Android) - Hands-on step-by-step labs supported by Google.

### Guides

The following guides act as excellent references while exploring Android:

* [CodePath Android Cliffnotes](http://guides.codepath.com/android#getting-started) are the guides that you are reading right now! Check out the link to see the various "getting started" resources and hopefully you'll find them as an invaluable resource.

* [The Android official training guides](http://developer.android.com/training/index.html) are a good place to start. The [Building Your First App](http://developer.android.com/training/basics/firstapp/index.html) lesson is very easy to follow  and already gives you a good understanding of some key concepts of the Android SDK.

* [Vogella Android Tutorials](http://www.vogella.com/tutorials/Android/article.html) - These are awesome free tutorials for most common Android topics. 

* [Google Android Glossary](https://developers.google.com/android/for-all/vocab-words/) - Common Android terms defined in a glossary site with visual diagrams to help reinforce concepts.

* [Common tasks](http://developer.android.com/guide/faq/commontasks.html) is a useful list from Google of typical things you can do in your app with links to explanations on how to do them. 

### Tutorials

The following tutorials attempt to explain aspects of Android development step-by-step:

* [Google Codelabs for Android Tutorial](https://codelabs.developers.google.com/?cat=Android) - Interesting interactive tutorial for learning Android step-by-step.

* [Android UI Tutorial: Layouts and Animations](https://www.codementor.io/android/tutorial/android-ui-layouts-animations-mirror) - a quick live coding of Android UI creations and shows how to use the different layouts, Views (TextView, ListView, ImageView, GridView, RecyclerView) and Motions (Property Animation, drawable Animation) by live coding. 

### Video Resources

The following video lessons present on the basics of Android development:

* [The Android Development Tutorial](http://www.youtube.com/watch?v=Z149x12sXsw) by Derek Banas is great for those who prefer video lessons. It has 25 video lessons in total ranging from 10 to 30 minutes each. Note that he teaches in Eclipse instead of Android Studio.

### Apps

The following apps can be a useful examples when getting started:

* [[Open-source Apps|Sample-Android-Apps]] provides a list of the open-source sample apps that can be used as a resource when first getting started on Android. These sample apps can be a great help in understanding how all the pieces fit together.

* [DevAppsDirect](https://play.google.com/store/apps/details?id=com.inappsquared.devappsdirect&hl=en) is another great app you can get from the Play Store. While it also won't teach you how to develop an app, it will show you what is available out there. The app maintains a list of open source libraries you can use in your project for a variety of purposes. Knowing what you can reuse will save you a lot of time in the future.

### Books

The following books can act as a complement to the resources above:

* [Android Programming: Pushing the Limits](http://www.amazon.com/Android-Programming-Pushing-Erik-Hellman/dp/1118717376) is a fairly good book to check out.

* [The Busy Coder's Guide to Android Development](http://commonsware.com/Android/) is comprehensive with its over 2,400 pages but starts with the basics. It also has "do-it-yourself" tutorials to help you retain what you are learning. The book is a bit expensive but it comes with a one-year subscription to keep it updated during the period, as the author is constantly adding to the text. 
    * All code examples are free and [can be found here](https://github.com/commonsguy/cw-omnibus/). Even if you don't buy the book, consider browsing through some of the examples there to learn how other programmers do things. Finally, Mark Murphy, the author of the book, is helpful whether you contact him by e-mail or on the [StackOverflow website](http://stackoverflow.com/). Check out [his profile](http://stackoverflow.com/users/115145/commonsware).

## Key Concepts

There are several key concepts in Android that can cause confusion at first. Descriptions and further reading is listed below.

### Configuration Changes

A configuration change is when the app is re-created on screen. The most common of these is caused by orientation changes. Whenever there is an orientation change, your activity needs to be destroyed and recreated to address the changes in layout. This means you need to [[handle this recreation process yourself|Handling-Configuration-Changes]], making sure your app doesn't crash. A lot of beginners (myself included) consider simply disabling these changes but this is considered a bad practice. Besides, even if you do disable orientation changes, there are other things that can cause configuration changes that you need to handle anyway. 

Refer to [[our guide on configuration changes|Handling-Configuration-Changes]]. In addition, here are two good posts discussing this further: <http://stackoverflow.com/a/582585/362298> and <http://stackoverflow.com/questions/1111980/how-to-handle-screen-orientation-change-when-progress-dialog-and-background-thre>

To force yourself to catch problems sooner than later, consider the following tip from a previous [post here on Reddit](http://www.reddit.com/r/androiddev/comments/19143c/is_your_app_stateful_test_your_might/). You can enable a developer setting to not keep activities, so that they are always destroyed and recreated. 
  
And more specifically, here are two examples of dealing with this problem when using a FragmentPagerAdapter, a common use case:  

 - <http://stackoverflow.com/questions/17629463/fragmentpageradapter-how-to-handle-orientation-changes> 
 - <http://stackoverflow.com/questions/18425646/fragmentpageradapter-not-restoring-fragments-upon-orientation-change>

In a more abstract sense but still related to the topic, [an old post on avoiding memory leaks](http://www.curious-creature.org/2008/12/18/avoid-memory-leaks-on-android/ ) is probably worthy of your time as well. This was written by Romain Guy, a Google employee very active in the Android community. It is certainly worth [following him on Google+](https://plus.google.com/+RomainGuy).

### Parcelable

You can pass Java objects from one activity to another in a `Bundle` if your class implements the [Parcelable interface](http://developer.android.com/reference/android/os/Parcelable.html). Parcelables were designed specifically for performance and should almost always be used instead of Serializable. Refer to [[our guide on Parcelables|Using-Parcelable]] for a detailed overview of how to use them. 

In addition, here is a [good page explaining how to use it](http://shri.blog.kraya.co.uk/2010/04/26/android-parcel-data-to-pass-between-activities-using-parcelable-classes/). You can also have inheritance while using it without adding too much of an overhead to the children [like this post](http://stackoverflow.com/questions/20018095/parcelable-inheritance-issue-with-getcreator-of-heir-class/20018463#20018463)
  
One thing you MUST always keep in mind is the order with which you write to the parcel and read from it. That order must be consistent. Otherwise you will start getting very crypt error messages which are very hard to debug.

### Threading

Android is full of features to help you deal with threads. This is a very important aspect of Android development because your app has to give snappy responses to user interaction. All your heavy work, such as database operations and network access, needs to be done in a separate thread so that the app does not appear frozen. The best place to start is learning how to [[start background intent services|Starting-Background-Services]] and how to [[run AsyncTasks to perform quick background operations|Creating-and-Executing-Async-Tasks]]
 
It is important to know when to use a `Service`, a `Thread`, an `IntentService` or an `AsyncTask`. Learn about them, check examples, and make sure you use the right constructs whenever appropriate. For details, refer to our [[detailed threading and services guide|Managing-Threads-and-Custom-Services]] for a comprehensive look at threading in Android. 

### Broadcast Receivers

Broadcast receivers are a great way to have your asynchronous tasks and services (refer to the previous topic above) communicate with the main thread or to receive push notifications from your phone. It is a powerful feature to understand and use. Refer to our [[guide on communicating with services|Starting-Background-Services#communicating-with-a-broadcastreceiver]] for a detailed look. 

Consider reading about it in the [official documentation](http://developer.android.com/reference/android/content/BroadcastReceiver.html). Also, refer again to the list of [common tasks](http://developer.android.com/guide/faq/commontasks.html) to get to know how to use them.

## Wrapping Up

Again please note this guide above was originally posted [on reddit](http://www.reddit.com/r/Android/comments/1roag7/i_developed_an_app_and_created_a_starting_guide/) and was adapted here for wider access. For more details, check out the [original document](https://docs.google.com/document/d/19uj9TS0Hzaxyv8IM1D20d23GdU9aAkXo5BYfL15VG0I/edit).

## References

* <http://www.reddit.com/r/Android/comments/1roag7/i_developed_an_app_and_created_a_starting_guide/>
* <http://halfapp.com/blog/get-started-programming-android-apps/>
* <http://www.reddit.com/r/Android/comments/1w3woc/a_step_by_step_guide_about_how_to_get_started_and/>
* <http://www.reddit.com/r/Android/comments/2dofph/just_published_my_first_app_a_comprehensive_guide/>