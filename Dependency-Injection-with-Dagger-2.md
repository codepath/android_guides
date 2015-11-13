## Overview

Many Android apps have some type of inherent implicit dependency chain.  For instance, a Twitter API client for instance may be built using a networking library such as [Retrofit](http://square.github.io/retrofit/). To use this library, you might also need to add parsing libraries such as [[Gson|Leveraging-the-Gson-Library]].  In addition, classes that implement authentication or caching may require accessing [[shared preferences|Storing-and-Accessing-SharedPreferences]] or other common storage, creating another set of inter-related dependencies.

Dagger 2 analyzes these dependencies for you and generates code to help wire them together.  While there have been other Java dependency injection frameworks in the past, many of them suffered limitations in relying on XML and required validating dependency issues at run-time, or incurred performance penalties during startup. [Dagger 2](http://google.github.io/dagger/) relies on purely using Java [annotation processors](https://www.youtube.com/watch?v=dOcs-NKK-RA) and compile-time checks to analyze and verify dependencies and is considered to be one of the most efficient dependency injection framework built to date.

### Advantages

Here is a list of other advantages for using Dagger 2:

 * **Simplifies access to singletons**. Just as the [ButterKnife](http://jakewharton.github.io/butterknife/) library makes it easier to define references to Views and event handlers, Dagger 2 provides a simply way to obtain references to shared instances.  For instance,  once we declare in Dagger our singleton instances such as  `MyTwitterApiClient` or `SharedPreferences`, we can declare fields with a simple `@Inject` annotation:

```java
public class MainActivity extends Activity {
   @Inject MyTwitterApiClient mTwitterApiClient;
   @Inject SharedPreferences sharedPreferences;

   public void onCreate(Bundle savedInstance) {
       // assign singleton instances to fields
       InjectorClass.inject(this);
   } 
```

 * **Easy configuration of complex dependencies**. There is an implicit order in which your modules are often created.   Dagger 2 walks through the dependency graph and generates code that is both easy to understand and trace, while also saving you from writing the large amount of of boilerplate code you would normally need to write by hand to obtain references and pass them to other objects as dependencies.  It also helps simplify refactoring, since you can focus on what modules to build rather then focusing on the order in which they need to be created.

 * **Easier unit and integration testing**  Because the dependency graph is created for us, we can easily swap out modules that make network responses that mock out this behavior.

### Setup

Add these three lines to your `app/build.gradle` file:

```gradle
dependencies {
    compile 'com.google.dagger:dagger:2.0'
    provided 'com.google.dagger:dagger-compiler:2.0'
    provided 'org.glassfish:javax.annotation:10.0-b28'
}
```

Note that the `provided` keyword refers to dependencies that are only needed at compilation.  The Dagger compiler generates code that are used to create the dependency graph of the classes defined in your source code.  These classes are added to the IDE class path during compilation.  

### Using Dagger 2

The simplest example is to show how to centralize all your singleton creation with Dagger 2.  Suppose you weren't using any type of dependency injection framework and wrote code in your Twitter client similar to the following:

```java
OkHttpClient client = new OkHttpClient();
 
// Instantiate Gson
Gson gson = new GsonBuilder().create();
GsonConverterFactory converterFactory = GsonConverterFactory.create(Gson);

// Build Retrofit
Retrofit retrofit = new Retrofit.Builder()
                                .baseUrl("https://api.github.com")
                                .addConverterFactory(converterFactory)
                                .client(client)  // custom client
                                .build();
```

#### Declare your singletons

You need to define what modules should be included as part of the dependency chain.  For instance, if we wish to make a single `Retrofit` instance tied to the application lifecycle and available to all our activities and fragments, we first need to make Dagger aware that a `Retrofit` instance can be provided.   We create a class called `NetModule.java` and annotate it with `@Module` to signal to Dagger to search within the available methods for possible instance providers:

```java
@Module
public class NetModule { // Dagger will look in here 

}
```

Note that the methods that will actually expose available return types must also be annotated with `@Provides` decorator.  The `Singleton` annotation also signals to the Dagger compiler that the instance should be created only once in the application.  In the following example, we are specifying the `Gson` return type that can be used as part of the dependency list.  

```java
@Module
public class NetModule {

   @Provides  // Dagger will only look for methods annotated with @Provides
   @Singleton
   Gson provideGson() {  
       GsonBuilder gsonBuilder = new GsonBuilder();
       gsonBuilder.setFieldNamingPolicy(FieldNamingPolicy.LOWER_CASE_WITH_UNDERSCORES);
       return gsonBuilder.create();
   }
```

Note that the method name `provideGson()` itself does not matter and can be named anything.  The return type annotated with a `@Provides` decorator is used to associate this instantiation with any other modules of the same time.  The `@Singleton` annotation is used to declare to Dagger to be only initialized only once.  

We also declare a provider for an `OkHttpClient` instance using the same approach:

```java
@Provides
@Singleton
OkHttpClient provideOkHttpClient() {
    return new OkHttpClient();
}
```

A `Retrofit` instance depends both on a `Gson` and `OkHttpClient` instance, so we can define another method within the same class that takes these two types.  The `@Provides` annotation and these two parameters in the method will cause Dagger to recognize that there is a dependency on `Gson` and `OkHttpClient` to build a `Retrofit` instance:

```java

@Provides
@Singleton
Retrofit provideRetrofit(Gson gson, OkHttpClient okHttpClient) {
   Retrofit retrofit = new Retrofit.Builder()
                .addConverterFactory(GsonConverterFactory.create(gson))
                .baseUrl(baseUrl)
                .client(okHttpClient)
                .build();
        return retrofit;
    }
```

#### Define injection targets

Dagger provides a way for the fields in your activities, fragments, or services to be assigned references to these singletons by delegating this work to an injector class that will handle this work.  The fields need to be annotated with `@Inject` and call an `inject()` method with this class.  In this way, Dagger 2 will search its dependency graph to try to find matching return types to use.  For instance, in the example below, it will attempt to find a provider that returns `MyTwitterApiClient` and a `SharedPreference` type:

```java
public class MainActivity extends Activity {
   @Inject MyTwitterApiClient mTwitterApiClient;
   @Inject SharedPreferences sharedPreferences;

  public void onCreate(Bundle savedInstance) {
       // assign singleton instances to fields
       InjectorClass.inject(this);
   } 
```

The injector class in Dagger 2 must be defined with a `@Component` decorator.  The activities, services, or fragments that will can be added should be declared in this class with individual `inject()` methods: 

```java
@Singleton
@Component(modules={NetModule.class})
public interface AppComponent {
   void inject(MyActivity activity);
   void inject(MyFragment fragment);
   void inject(MyService service);
}
```

**Note** that base classes are not be sufficient as injection targets.  Dagger 2 relies on strongly typed classes, so you must specify explicitly which ones should be defined.   (There are [suggestions](https://blog.gouline.net/2015/05/04/dagger-2-even-sharper-less-square/) to workaround the issue, but the code to do so may be more complicated to trace than simply defining them.)

#### Code generation

An important aspect of Dagger 2 is that generates code for classes annotated with the `@Component` interface.  You can use a a class prefixed with `Dagger_` (i.e. `Dagger_TwitterApiComponent.java`) that will be responsible for instantiating an instance of our dependency graph and using it to perform the injection work for fields annotated with `@Inject`.  

We should do all this work within an `Application` class since these instances should be declared only once throughout the entire lifespan of the application:

```java
public class MyApp extends Application {

    private AppComponent mAppComponent;

    @Override
    public void onCreate() {
        super.onCreate();
        
        // specify the full namespace of the component
        // Dagger_xxxx (where xxxx = component name)
        mAppComponent = com.codepath.dagger.components.DaggerAppComponent.builder()
                .netModule(new NetModule());
    }

    public AppComponent getAppComponent() {
       return mAppComponent;
    }
```

We also modify the app name `label` to correspond to `MyApp` to ensure that this application is called:

```xml
  <application
        android:allowBackup="true"
        android:label="MyApp">
```

Within our activity, we simply need to get access to these component and call `inject()`.  

```java
public class MyActivity extends Activity {
  @Inject MyTwitterApiClient mTwitterApiClient;
  @Inject SharedPreferences sharedPreferences;

  public void onCreate(Bundle savedInstance) {
        // assign singleton instances to fields
        getApplication().getAppComponent()).inject(this);
    } 
```

### Custom Scopes

The example above showed that we used `@Singleton`.  There is also another way to define how long the singleton can live (i.e. per activity, per user).

Here is an example module:

```java
@Module
public class SampleAppModule {

  static final String PREFS_DEFAULT = "myapp";

  final SampleApp app;

  public AppModule(SampleApp app) {
    this.app = app;
  }

  @Provides @PerApp SampleApp provideSampleApp() {
    return app;
  }

  @Provides @PerApp Application provideApplication(SampleApp app) {
    return app;
  }

  @Provides @PerApp SharedPreferences provideSharedPrefs(Application app) {
    return app.getSharedPreferences(PREFS_DEFAULT, Context.MODE_PRIVATE);
  }

  @Provides @PerApp Gson provideGson() {
    return new Gson();
  }
}
```

Notice the `@Module` annotation on the class and the `@Provides` annotations on the functions. Functions in modules annotated with @Provides are called when the dependency is injected or used by another dependency. If the function is annotated with a **scope**, in this case `@PerApp`, it is only created once for that scope.

### Components

Components are used to group together and build Modules into an object we can use to **get dependencies from**, and/or **inject fields with**. Basically, it's the glue between the Module and the injection points, first by configuring what modules and other components it depends on, then by listing the explicit classes it can be used to inject.

A Component must:

* Define the modules it is composed of as an argument in the `@Component` annotation
* Define any dependencies on other `Component`s it has.
* Define functions to inject explicit types with dependencies. TODO: Link to details
* Expose any internal dependencies to be accessed externally or by other Components.

Example Component:
```java
@PerApp
@Component(
    modules = {
        SampleAppModule.class,
        DataModule.class,
        NetworkModule.class
    }
    dependencies = {
        OtherComponent.class
    }
)
public interface SampleAppComponent {
  public void inject(SomeActivity someActivity);

  public void inject(SomeFragment someFragment);

  public void inject(SomeOtherFragment someOtherFragment);

  Application application();

  Picasso picasso();

  Gson gson();

  OkHttpClient okHttpClient();
}

```

### Other ways for injection

Dependencies are used or "injected" through the functions on a `Component`.

There are 3 ways to get a dependency from a component:

1. Directly by calling `component.thing()`.
2. Annotating a field on an object with `@Inject`, then calling `component.inject(object)`. Often the object is `this`.
3. Using constructor injection by annotating your constructor with `@Inject`, and letting Dagger create your class for you.

## References

* [Dagger 2 Github Page](http://google.github.io/dagger/)
* [Sample project using Dagger 2](https://github.com/vinc3m1/nowdothis)
* [Vince Mi's Codepath Meetup Dagger 2 Slides](https://docs.google.com/presentation/d/1bkctcKjbLlpiI0Nj9v0QpCcNIiZBhVsJsJp1dgU5n98/)
* <http://code.tutsplus.com/tutorials/dependency-injection-with-dagger-2-on-android--cms-23345>
* [Jake Wharton's Devoxx Dagger 2 Slides](https://speakerdeck.com/jakewharton/dependency-injection-with-dagger-2-devoxx-2014)
* [Dagger 2 Google Developers Talk](https://www.youtube.com/watch?v=oK_XtfXPkqw)
* [Dagger 1 to Dagger 2](http://frogermcs.github.io/dagger-1-to-2-migration/)
* [Testing Dagger 2 on Android](http://fernandocejas.com/2015/04/11/tasting-dagger-2-on-android/)
* [Dagger 2 Testing with Mockito](http://blog.sqisland.com/2015/04/dagger-2-espresso-2-mockito.html#sthash.IMzjLiVu.dpuf)
* [Snorkeling with Dagger 2](https://github.com/konmik/konmik.github.io/wiki/Snorkeling-with-Dagger-2) 