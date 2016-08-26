## Overview

When first building Android apps, many developers might start by relying on Model View Controller (MVC) patterns and usually end up writing most of the core business logic in activities or fragments.  The challenge is that writing tests that can validate the app's behavior is difficult to do because the code is often so closely tied to the Android framework and the various lifecycle events.  While automated UI testing could be written to validate individual activities or fragments, maintaining and running them over the long-term is often difficult to sustain.

Clean architecture principles, as espoused by Robert Martin (also known as "Uncle Bob"), attempt to focus the developer on thinking through the app's core functionality.  It does so by separating the architecture of app into three major layers: how the app shows the data to the user (**presentation layer**), what the core functions of the app (**domain or use case layer**) are, and how the data can be accessed (**data layer**).  The presentation layer sits as the outermost layer, the domain layer sits in the middle layer, and the data layer resides in the inner layer.  

It's important to note that each layer has its own data model, and data can only be exchanged between layers and usually flows only in one direction (i.e. outer to inner, or inner to outer)  Anytime data needs to be exchanged, usually a converter is used to map one layer's model to another.  In this way, a boundary of separation is created that helps limit changes in one layer to cause unintended side effects in other layers.

## Templates

The following template projects are built to act as a starting point for a preferred architecture:

 * <https://github.com/dmilicic/Android-Clean-Boilerplate>
 * <https://github.com/android10/Android-CleanArchitecture>
 * <https://github.com/googlesamples/android-architecture>
 * <https://github.com/antoniolg/androidmvp>
 * <https://labs.ribot.co.uk/approaching-android-with-mvvm-8ceec02d5442>

## References

Clean architecture:

* <https://medium.com/@dmilicic/a-detailed-guide-on-developing-android-apps-using-the-clean-architecture-pattern-d38d71e94029>
* <https://www.youtube.com/watch?v=BlkJzgjzL0c>
* <http://fernandocejas.com/2014/09/03/architecting-android-the-clean-way/>
* <http://fernandocejas.com/2015/07/18/architecting-android-the-evolution/>
* <https://github.com/dmilicic/android-clean-sample-app>
* <https://speakerdeck.com/romainpiel/ingredients-for-a-healthy-codebase/>
* <http://macoscope.com/blog/model-view-presenter-architecture-in-android-applications/>
* <https://github.com/macoscope/RoomBookerMVP/tree/master/mvp/src/main/java/com/macoscope/>

MVP Pattern:

* <https://github.com/Arello-Mobile/Moxy/>
* <http://www.tinmegali.com/en/model-view-presenter-android-part-1/>
* <https://medium.com/mobiwise-blog/android-basic-project-architecture-for-mvp-72f4b33252d0>
* <http://antonioleiva.com/mvp-android/>
* <https://github.com/konmik/konmik.github.io/wiki/Introduction-to-Model-View-Presenter-on-Android>
* <http://thefinestartist.com/android/mvp-pattern>
* <https://www.youtube.com/watch?v=BlkJzgjzL0c>
