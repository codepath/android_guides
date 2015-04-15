## Overview

There are many third-party libraries for Android but several of them are "must have" libraries that are extremely popular and are often used in almost any Android project. Each has different purposes but all of them make life as a developer much more pleasant. The major libraries are listed below in a few categories.

### Standard Pack

This "standard pack" listed below are libraries that are quite popular, widely applicable and should probably be setup within most Android apps:

| Name            | Description                                                 |  
| ----            | ------------                                                |
| [Retrofit](http://square.github.io/retrofit/) | A type-safe REST client for Android which intelligently maps an API into a client interface using annotations.            |
| [Picasso](http://square.github.io/picasso/) | A powerful image downloading and caching library for Android. |
| [ButterKnife](http://jakewharton.github.io/butterknife/) | Using Java annotations, makes Android development better by simplifying common tasks. |
| [Parceler](https://github.com/johncarl81/parceler) | Android Parcelable made easy through code generation |
| [IcePick](https://github.com/frankiesardo/icepick) | Android Instance State made easy |
| [Crouton](https://github.com/keyboardsurfer/Crouton) | Context-sensitive, configurable alert notices much better than toasts |
| [Hugo](https://github.com/JakeWharton/hugo) | Easier logging using annotations |

Keep in mind that the combination of these libraries may not always play nicely with each other.  The following section highlights some of these issues.

#### Parceler and IcePick

Note that you cannot use IcePick at the current time to save state of Parceler objects.  See this [GitHub issue](https://github.com/frankiesardo/icepick/issues/36) for more context.  You will need to use Parcelable objects with IcePick.

#### ButterKnife and Parceler

Using the Butterknife library with the Parceler library causes multiple declarations of javax.annotation.processing.Processor.  In this case, you have to exclude this conflict in your `app/build.gradle` file:

```gradle
   packagingOptions {

        exclude 'META-INF/services/javax.annotation.processing.Processor'  // butterknife
    }
```

### Convenience

 * [Dagger](http://square.github.io/dagger/) - A fast dependency injector for Android and Java.  See this [video intro](http://www.infoq.com/presentations/Dagger) from Square.
 * [AutoParcel](https://github.com/frankiesardo/auto-parcel) - Port of Google AutoValue for Android with Parcelable generation goodies.
 * [Hugo](https://github.com/JakeWharton/hugo) - Easier logging within your app
 * [Logger](https://github.com/orhanobut/logger) - Much cleaner and easier logcat trace messages
 * [AndroidAnnotations](https://github.com/excilys/androidannotations) - Framework that speeds up Android development. It takes care of the plumbing, and lets you concentrate on what's really important. By simplifying your code, it facilitates its maintenance
 * [RoboGuice](https://github.com/roboguice/roboguice) - Powerful extensions to Android using dependency injection.
 * [Calligraphy](https://github.com/chrisjenx/Calligraphy) - Custom fonts made easy
 * [AndroidViewAnimations](https://github.com/daimajia/AndroidViewAnimations) - Common property animations made easy

### Extensions 

 * [Otto](https://github.com/square/otto) - An enhanced Guava-based event bus with emphasis on Android support
 * [EventBus](https://github.com/greenrobot/EventBus) - Android optimized event bus that simplifies communication between components.
 * [Tape](http://square.github.io/tape/) - Tape is a collection of queue-related classes for Android and Java
 * [RxJava](https://github.com/ReactiveX/RxJava) - Reactive Extensions for the JVM
 * [Priority JobQueue](https://github.com/path/android-priority-jobqueue) - Easier background tasks
 * [ACRA](http://acra.ch/) - Crash reporting made easy and free. Check the [setup instructions](https://github.com/ACRA/acra/wiki/BasicSetup) and [open-source backend](https://github.com/ACRA/acralyzer).

### Networking

 * [Retrofit](http://square.github.io/retrofit/) - A type-safe REST client for Android and Java which intelligently maps an API into a client interface using annotations. 
 * [Picasso](http://square.github.io/picasso/) - A powerful image downloading and caching library for Android.
 * [Ion](https://github.com/koush/ion) - Powerful asynchronous networking library. [Download](https://github.com/koush/ion#get-ion) as a jar here.
 * [Android Async HTTP](http://loopj.com/android-async-http/) - Asynchronous networking client for loading remote content such as JSON.
 * [Volley](http://developer.android.com/training/volley/index.html) - Google's HTTP library that makes networking for Android apps easier and most importantly, faster.
 * [Glide](https://github.com/bumptech/glide) - Picasso image loading alternative endorsed by Google 
 * [IceNet] (https://github.com/anton46/IceNet) -  Android networking wrapper consisting of a combination of Volley, OkHttp and Gson
 * [Android Universal Image Loader](https://github.com/nostra13/Android-Universal-Image-Loader) - Popular alternative for image loading that can replace Picasso or Glide.

### ListView

 * [EasyListViewAdapters](https://github.com/birajpatel/EasyListViewAdapters) - Building multi-row-type listview made much cleaner & easier.
 * [GridListViewAdapters](https://github.com/birajpatel/GridListViewAdapters) - Easily build unlimited Grid cards list like play-store. (ListView working as unlimited GridView)
 * [StickyListHeaders](https://github.com/emilsjolander/StickyListHeaders) - An android library for section headers that stick to the top of a ListView
 * [PinnedListView](https://github.com/beworker/pinned-section-listview) - Pinned Section with ListView
 * [ListViewAnimations](https://github.com/nhaarman/ListViewAnimations) - Easy way to animate ListView items
 * [EtsyStaggeredGrid](https://github.com/etsy/AndroidStaggeredGrid) - Staggered GridView
 * [Cardslib](https://github.com/gabrielemariotti/cardslib) - Card UI for Lists or Grids
 * [SwipeListView](https://github.com/47deg/android-swipelistview) - Swipeable List
 * [PullToRefresh-ListView](https://github.com/erikwt/PullToRefresh-ListView) - Easy to use pull-to-refresh functionality for ListViews. [Download](https://github.com/erikwt/PullToRefresh-ListView/archive/master.zip) and install as a [library project](http://imgur.com/a/N8baF).
 * [QuickReturn](https://github.com/lawloretienne/QuickReturn) - Reveal or hide a header or footer as the list is scrolled in a direction.

### Easy Navigation 

 * [PagerSlidingTabStrip](https://github.com/astuetz/PagerSlidingTabStrip) - An interactive indicator to navigate between the different pages of a ViewPager. 
 * [ViewPagerIndicator](https://github.com/JakeWharton/Android-ViewPagerIndicator) - Paging indicator widgets compatible with the ViewPager from the Android Support Library and ActionBarSherlock. 
 * [JazzyViewPager](https://github.com/jfeinstein10/JazzyViewPager) - Pager with more animations
 * [ParallaxPager](https://github.com/prolificinteractive/ParallaxPager) - ViewPager with Parallax scrolling effects
 * [SlidingMenu](https://github.com/jfeinstein10/SlidingMenu) - Library that allows developers to easily create applications with sliding menus like those made popular in the Google+, YouTube, and Facebook apps.
 * [Android Satellite Menu](https://github.com/siyamed/android-satellite-menu/) - Radial menu which is configurable reminiscent of the "Path" menu style.
 * [ArcMenu](https://github.com/daCapricorn/ArcMenu) - Alternate radial menu modeled after the "Path" menu style.
 * [AndroidSlidingUpPanel](https://github.com/umano/AndroidSlidingUpPanel) - Sliding Up Panel 
 * [DraggablePanel](https://github.com/pedrovgs/DraggablePanel) - Panels that can be dragged

### UI Components

 * [Crouton](https://github.com/keyboardsurfer/Crouton) - Context-sensitive, configurable alert notices much better than toasts. Download jar [from here](http://search.maven.org/#search%7Cga%7C1%7Cg%3A%22de.keyboardsurfer.android.widget%22). See [working sample code](https://github.com/codepath/android-crouton-sample)
 * [BetterPickers](https://github.com/derekbrameyer/android-betterpickers) - BetterPickers for easy input selection
 * [RoundedImageView](https://github.com/vinc3m1/RoundedImageView) - Easily round corners or create oval-shaped images with this popular library.
 * [Android StackBlur](https://github.com/kikoso/android-stackblur) - Dynamically blur images
 * [Android Bootstrap](https://github.com/Bearded-Hen/Android-Bootstrap) - Bootstrap UI widgets
 * [PhotoView](https://github.com/chrisbanes/PhotoView) - ImageView that supports touch gestures
 * [ShowcaseView](https://github.com/amlcurran/ShowcaseView) - Highlight the best bits of your app
 * [FadingActionBar](https://github.com/ManuelPeinado/FadingActionBar) - Cool actionbar fade effect
 * [AndroidViewAnimations](https://github.com/daimajia/AndroidViewAnimations) - Easily apply common animations
 * [ProgressWheel](https://github.com/Todd-Davies/ProgressWheel) - Better progress bar
 * [SmoothProgressBar](https://github.com/castorflex/SmoothProgressBar) - Horizontal indeterminate progress
 * [Rebound](http://facebook.github.io/rebound/) - Easy spring dynamics
 * [AndroidImageSlider](https://github.com/daimajia/AndroidImageSlider) - Animated image transitions
 * [Paralloid](https://github.com/chrisjenx/Paralloid) - Parallax sliding effect made easy
 * [FloatingActionButton](https://github.com/makovkastar/FloatingActionButton) - Material design floating buttons made easy
 * [Foursquare-CollectionPicker](https://github.com/anton46/Foursquare-CollectionPicker) - Item Picker which looks like Foursquare Tastes picker
 * [NexusDialog](https://github.com/dkharrat/NexusDialog) - Create form dialogs easily
 * [dialogplus](https://github.com/orhanobut/dialogplus) - Simple, easy dialogs
 * [Iconify](https://github.com/JoanZapata/android-iconify) - Easily embed icons into your app

### Drawing

 * [Leonids](https://github.com/plattysoft/Leonids) - Simple and easy particle effects ([See Tutorial](http://www.plattysoft.com/2014/05/30/leonids-particle-system-lib/))
 * [MPAndroidChart](https://github.com/PhilJay/MPAndroidChart) - A powerful Android chart view / graph view library
 * [AChartEngine](http://jaxenter.com/effort-free-graphs-on-android-with-achartengine-46199.html) - This is a charting software library for Android applications
 * [HoloGraphLibrary](https://github.com/Androguide/HoloGraphLibrary) - Newer graphing library
 * [EazeGraph](https://github.com/blackfizz/EazeGraph) - Another newer library with potential
 * [AndroidCharts](https://github.com/dacer/AndroidCharts) - Easy to use charts
 * [AndroidGraphView](http://android-graphview.org/) - library to create flexible and nice-looking diagrams.
 * [AndroidPlot](http://androidplot.com/docs/quickstart/) - plotting library for Android
 * [WilliamChart](https://github.com/diogobernardino/WilliamChart) - Flexible charting library with useful motion capabilities.
 * [HelloCharts](https://github.com/lecho/hellocharts-android) - Charts/graphs library for Android with support for scaling, scrolling and animations.

### Scanning

 * [ZXing](https://github.com/zxing/zxing) - Barcode or QR scanner
 * [barcodescanner](https://github.com/dm77/barcodescanner) - Newer alternative
 * [zxscanlib](https://github.com/LivotovLabs/zxscanlib) - ZXing alternative
 * [android-quick-response-code](https://code.google.com/p/android-quick-response-code/) - Another alternative

### Persistence

 * [ActiveAndroid](https://github.com/pardom/ActiveAndroid)
 * [greenDAO](https://github.com/greenrobot/greenDAO)
 * [SugarORM](http://satyan.github.io/sugar/)
 * [ORMLite](http://ormlite.com/sqlite_java_android_orm.shtml)
 * [Realm](https://github.com/realm/realm-java)
 * [NexusData](https://github.com/dkharrat/NexusData)
 * [Hawk](https://github.com/orhanobut/hawk) - Persistent secure key/value store

### Compatibility

 * [NineOldAndroids](http://nineoldandroids.com/) - Fully compatible animation library that works with all versions of Android. Widely used. [Download](https://github.com/JakeWharton/NineOldAndroids/zipball/master) and install as a [library project](http://imgur.com/a/N8baF).
 * [HoloEverywhere](https://github.com/Prototik/HoloEverywhere) - Backport Holo theme from Android 4.2 to 2.1+
 * [CropImage](https://github.com/biokys/cropimage) - Simple compatible cropping intent for images
 
## Resources

Check out the following resources for finding libraries:

 * [Android Elementals](https://github.com/cesards/AndroidElementals)
 * [Wasabeef Core Libraries](https://github.com/wasabeef/awesome-android-libraries)
 * [Wasabeef UI Libraries](https://github.com/wasabeef/awesome-android-ui)
 * [Android-Libs.com](http://android-libs.com)
 * <http://androidlibs.org/>
 * <http://appdevwiki.com/wiki/show/HomePage>
 * <http://androidweekly.net/toolbox>
 * <http://android-arsenal.com>

## References

 * <http://www.vogella.com/tutorials/AndroidUsefulLibraries/article.html>
 * <http://actionbarsherlock.com/>
 * <http://nineoldandroids.com/>
 * <https://github.com/roboguice/roboguice/wiki>
 * <https://github.com/excilys/androidannotations/wiki>
 * <https://github.com/erikwt/PullToRefresh-ListView>
 * https://github.com/jfeinstein10/SlidingMenu
 * <http://square.github.io/picasso/>