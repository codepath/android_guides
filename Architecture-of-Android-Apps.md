## Overview

When first building Android apps, many developers might start by relying on Model View Controller (MVC) patterns and usually end up writing most of the core business logic in activities or fragments.  The challenge is that writing tests that can validate the app's behavior is difficult to do because the code is often so closely tied to the Android framework and the various lifecycle events.  While automated UI testing could be written to validate individual activities or fragments, maintaining and running them over the long-term is often difficult to sustain.

### Architectural Patterns

There are currently three major approaches to architecting your Android apps:

 1. **Standard Android** (Model-View-Controller) - This is the "default" approach with layout files, Activities/Fragments acting as the controller and Models used for data and persistence. With this approach, Activities are in charge of processing the data and updating the views. Activities act like a controller in MVC but with some extra responsibilities that should be part of the view. The problem with this standard architecture is that Activities and Fragments can become quite large and very difficult to test. You can [learn more on this blog post](https://medium.com/upday-devs/android-architecture-patterns-part-1-model-view-controller-3baecef5f2b6).

 2. **Clean Architecture** (Model View Presenter) - When using MVP, Activities and Fragments become part of the view layer and they delegate most of the work to presenter objects. Each Activity has a matching presenter that handles all access to the model. The presenters also notify the Activities when the data is ready to display. You can learn more in the sections below. 

 3. **Data-binding MVVM** (Model-View-ViewModel) - ViewModels retrieve data from the model when requested from the view via the [[Android data binding framework|Applying-Data-Binding-for-Views]]. With this pattern, Activities and Fragments become very lightweight. Moreover, writing unit tests becomes easier because the ViewModels are decoupled from the view. You can [learn more on this blog post](https://medium.com/upday-devs/android-architecture-patterns-part-3-model-view-viewmodel-e7eeee76b73b).

Check this [blog post from Realm](https://realm.io/news/eric-maxwell-mvc-mvp-and-mvvm-on-android/) for a detailed comparison of the options. Refer to [this sample app](https://github.com/ivacf/archi) for an example of each architecture type. 

## Clean Architecture

Clean architecture principles, as espoused by Robert Martin (also known as "Uncle Bob"), attempt to focus the developer on thinking through the app's core functionality.  It does so by separating the architecture of app into three major layers: how the app shows the data to the user (**presentation layer**), what are the core functions of the app (**domain or use case layer**), and how the data can be accessed (**data layer**).  The presentation layer sits as the outermost layer, the domain layer sits in the middle layer, and the data layer resides in the inner layer.  

<img src="https://i.imgur.com/tJxzrx2.png" />

It's important to note that each layer has its own data model, and data can only be exchanged between layers and usually flows only in one direction (i.e. outer to inner, or inner to outer)  Anytime data needs to be exchanged, usually a converter is used to map one layer's model to another.  In this way, a boundary of separation is created that helps limit changes in one layer to cause unintended side effects in other layers.

At the data layer, every source (i.e. file, network, memory) of data should be represented using the repository data pattern.  There are many different ways of implementing this repository pattern, but the ultimate goal is to expose an interface that defines the different queries that need to be performed in order to abstract away the type of data source used.  The underlying implementations can therefore be swapped regardless of the type of data source used.

Clean architecture introduces more abstractions and attempts to apply [single responsibility principles](https://en.wikipedia.org/wiki/Single_responsibility_principle) in Android development.  While there may be concerns about this approach adding more complexity, slow performance, and poor testability, it has been shown to work successfully in production apps (see [this Droidcon talk](https://www.youtube.com/watch?v=-oZswd1j5H0) or [this Droidcon 2016 talk](https://www.youtube.com/watch?v=R89ufpJI3SY)).

### Model View Presenter

In Android app development, the traditional "Model / View / Controller pattern" is often being dropped in preference for the "Model / View / Presenter" pattern. The Model-View-Presenter (MVP) architecture comprises:

 * **Model**: the data layer
 * **View**: the UI layer, displays data received from Presenter, reacts to user input. On Android, we treat Activities, Fragments, and android.view.View as View from MVP.
 * **Presenter**: responds to actions performed on the UI layer, performs tasks on Model objects (using Use Cases), passes results of those tasks to Views.

<img src="https://i.imgur.com/5WLQno7.png" width="500" />

What we want to achieve by using MVP are simpler tasks, smaller objects, and fewer dependencies between Model and Views layers. This, in turns makes our code easier to manage and test. Major differences from MVC include:

* View is more separated from Model. The Presenter is the mediator between Model and View.
* Easier to create unit tests
* Generally there is a one to one mapping between View and Presenter, with the possibility to use multiple Presenters for complex Views

The easiest method for understanding this is multiple specific examples with sample code. Check out these useful resources for an in-depth look:

 * [MVP Architecture in Android](http://macoscope.com/blog/model-view-presenter-architecture-in-android-applications/)
 * [MVP Explained](https://medium.com/upday-devs/android-architecture-patterns-part-2-model-view-presenter-8a6faaae14a5#.u53s2u5gu) with [official sample code](https://github.com/googlesamples/android-architecture/tree/todo-mvp-rxjava/)
 * [MVP Tutorial](https://medium.com/@be.betr.codr/android-mvp-survival-guide-b2094ab79f78#.ee4ajr7pz) with [handy sample code](https://github.com/WillyShakes/NetflixShows)
 * [Avenging Android MVP Article](https://android.jlelse.eu/avenging-android-mvp-23461aebe9b5)
 * [Introduction to MVP](https://code.tutsplus.com/tutorials/an-introduction-to-model-view-presenter-on-android--cms-26162)
 * [Android MVP Basics with Sample](https://android.jlelse.eu/android-mvp-basics-w-sample-app-3698e33ab9db)
 * [Android MVP Pattern, Part 1](http://www.tinmegali.com/en/model-view-presenter-android-part-1/), [2](http://www.tinmegali.com/en/model-view-presenter-mvp-in-android-part-2/), and [3](http://www.tinmegali.com/en/model-view-presenter-mvp-in-android-part-3/). 
 * [Basic MVP Architecture](https://medium.com/mobiwise-blog/android-basic-project-architecture-for-mvp-72f4b33252d0#.dcco0jo19)
 * [Brief MVP Introduction](https://antonioleiva.com/mvp-android/) with [sample code](https://github.com/antoniolg/androidmvp)
 * [Introduction to MVP Library Nucleus](http://konmik.com/post/introduction_to_model_view_presenter_on_android/)
 * [How to Adopt MVP](https://code.tutsplus.com/tutorials/how-to-adopt-model-view-presenter-on-android--cms-26206)
 * [MVP Guidelines](https://medium.com/@cervonefrancesco/model-view-presenter-android-guidelines-94970b430ddf#.uzpd446ez)

### Migrating to Clean Architecture

If you're attempting to migrate towards clean architecture without necessarily rewriting your entire code base, your best approach is to first try to isolate and encapsulate your logic by moving it outside of your Activities or Fragments.  Moving towards the Model-View-Presenter (MVP), which is a popular way of structuring your presentation layer, should probably be your first step.  There is no exact way of implementing this approach, so here are a few recommended links:

* <https://github.com/Arello-Mobile/Moxy/>
* <http://www.tinmegali.com/en/model-view-presenter-android-part-1/>
* <https://medium.com/mobiwise-blog/android-basic-project-architecture-for-mvp-72f4b33252d0>
* <http://antonioleiva.com/mvp-android/>
* <http://thefinestartist.com/android/mvp-pattern>
* <https://www.youtube.com/watch?v=BlkJzgjzL0c>
* <https://github.com/antoniolg/androidmvp>
* <https://speakerdeck.com/alphonzo79/better-android-architecture/>

### Templates

The following template projects are built to act as a starting point for this architecture:

 * <https://github.com/ribot/android-boilerplate>
 * <https://github.com/dmilicic/Android-Clean-Boilerplate>
 * <https://github.com/android10/Android-CleanArchitecture>
 * <https://github.com/googlesamples/android-architecture>

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
* <https://github.com/alphonzo79/AndroidArchitectureExample/>

MVVM Pattern:
* <https://labs.ribot.co.uk/approaching-android-with-mvvm-8ceec02d5442>

Other tutorial articles:

 * [A Simple Android Apps with MVP, Dagger, RxJava, and Retrofit](https://medium.com/@nurrohman/a-simple-android-apps-with-mvp-dagger-rxjava-and-retrofit-4edb214a66d7)
 * [MVP, Dagger 2, RxJava](https://www.captechconsulting.com/blogs/a-mvp-approach-to-lifecycle-safe-requests-with-retrofit-20-and-rxjava)
 * [Dagger 2 and MVP Architecture](https://adityaladwa.wordpress.com/2016/05/11/dagger-2-and-mvp-architecture/)
 * [Offline-First Reactive Android Apps](https://adityaladwa.wordpress.com/2016/10/25/offline-first-reactive-android-apps-repository-pattern-mvp-dagger-2-rxjava-contentprovider/) - Repository Pattern + MVP + Dagger 2 + RxJava + ContentProvider
 * [MVP + Dagger Tutorial](https://hackernoon.com/yet-another-mvp-article-part-2-how-dagger-helps-with-the-project-90d049a45e00)
 * [Dagger 2 and RxJava 1 Tutorial](http://saulmm.github.io/when-Thor-and-Hulk-meet-dagger2-rxjava-1)
