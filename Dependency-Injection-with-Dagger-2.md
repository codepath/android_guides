## Overview

Many Android apps rely on instantiating objects that often require other dependencies.  For instance, a Twitter API client for instance may be built using a networking library such as [[Retrofit|Consuming-APIs-with-Retrofit]]. To use this library, you might also need to add parsing libraries such as [[Gson|Leveraging-the-Gson-Library]].  In addition, classes that implement authentication or caching may require accessing [[shared preferences|Storing-and-Accessing-SharedPreferences]] or other common storage, requiring instantiating them first and creating an inherent dependency chain.

Dagger 2 analyzes these dependencies for you and generates code to help wire them together.  While there have been other Java dependency injection frameworks in the past, many of them suffered limitations in relying on XML, required validating dependency issues at run-time, or incurred performance penalties during startup. [Dagger 2](http://google.github.io/dagger/) relies on purely using Java [annotation processors](https://www.youtube.com/watch?v=dOcs-NKK-RA) and compile-time checks to analyze and verify dependencies.  It is considered to be one of the most efficient dependency injection framework built to date.

### Advantages

Here is a list of other advantages for using Dagger 2:

 * **Simplifies access to shared instances**. Just as the [ButterKnife](http://jakewharton.github.io/butterknife/) library makes it easier to define references to Views and event handlers, Dagger 2 provides a simply way to obtain references to shared instances.  For instance,  once we declare in Dagger our singleton instances such as  `MyTwitterApiClient` or `SharedPreferences`, we can declare fields with a simple `@Inject` annotation:

```java
public class MainActivity extends Activity {
   @Inject MyTwitterApiClient mTwitterApiClient;
   @Inject SharedPreferences sharedPreferences;

   public void onCreate(Bundle savedInstance) {
       // assign singleton instances to fields
       InjectorClass.inject(this);
   } 
```

 * **Easy configuration of complex dependencies**. There is an implicit order in which your objects are often created.   Dagger 2 walks through the dependency graph and [[generates code|Dependency-Injection-with-Dagger-2#code-generation]] that is both easy to understand and trace, while also saving you from writing the large amount of of boilerplate code you would normally need to write by hand to obtain references and pass them to other objects as dependencies.  It also helps simplify refactoring, since you can focus on what modules to build rather then focusing on the order in which they need to be created.

 * **Easier unit and integration testing**  Because the dependency graph is created for us, we can easily swap out modules that make network responses and mock out this behavior.

 * **Scoped instances**  Not only can you easily manage instances that can last the entire application lifecycle, you can also leverage Dagger 2 to define instances with shorter lifetimes (i.e. bound to a user session, activity lifecycle, etc.)

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

Android Studio by default will not recognize a lot of generated Dagger 2 code as legitimate classes, but adding the `android-apt` plugin will add these files into the IDE class path and enable you to have more visibility:

Add this line to your root `build.gradle`:

```gradle
 dependencies {
     // other classpath definitions here
     classpath 'com.neenbedankt.gradle.plugins:android-apt:1.8'
 }
```

Then make sure to apply the plugin in your `app/build.gradle`:

```gradle
// add after applying plugin: 'com.android.application'  
apply plugin: 'com.neenbedankt.android-apt'
```

### Creating Singletons

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

You need to define what objects should be included as part of the dependency chain by creating a Dagger 2 **module**.  For instance, if we wish to make a single `Retrofit` instance tied to the application lifecycle and available to all our activities and fragments, we first need to make Dagger aware that a `Retrofit` instance can be provided.   We create a class called `NetModule.java` and annotate it with `@Module` to signal to Dagger to search within the available methods for possible instance providers.  

The methods that will actually expose available return types should also be annotated with `@Provides` decorator.  The `Singleton` annotation also signals to the Dagger compiler that the instance should be created only once in the application.  In the following example, we are specifying the `Gson` return type that can be used as part of the dependency list.  

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

   @Provides
   @Singleton
   OkHttpClient provideOkHttpClient() {
      return new OkHttpClient();
   }

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
}
```

Note that the method name `provideGson()` itself does not matter and can be named anything.  The return type annotated with a `@Provides` decorator is used to associate this instantiation with any other modules of the same time.  The `@Singleton` annotation is used to declare to Dagger to be only initialized only once.  

A `Retrofit` instance depends both on a `Gson` and `OkHttpClient` instance, so we can define another method within the same class that takes these two types.  The `@Provides` annotation and these two parameters in the method will cause Dagger to recognize that there is a dependency on `Gson` and `OkHttpClient` to build a `Retrofit` instance.

#### Define injection targets

Dagger provides a way for the fields in your activities, fragments, or services to be assigned references simply by annotating the fields with an `@Inject` annotation and calling an `inject()` method.  Calling `inject()` will cause Dagger 2 to locate the singletons in the dependency graph to try to find a matching return type.  If it finds one, it assigns the references to the respective fields. For instance, in the example below, it will attempt to find a provider that returns `MyTwitterApiClient` and a `SharedPreference` type:

```java
public class MainActivity extends Activity {
   @Inject MyTwitterApiClient mTwitterApiClient;
   @Inject SharedPreferences sharedPreferences;

  public void onCreate(Bundle savedInstance) {
       // assign singleton instances to fields
       InjectorClass.inject(this);
   } 
```

The injector class used in Dagger 2 is called a **component**.  It assigns references in our activities, services, or fragments to have access to singletons we earlier defined.  We will need to annotate this class with a `@Component` declaration. Note that the activities, services, or fragments that will can be added should be declared in this class with individual `inject()` methods: 


```java
@Singleton
@Component(modules={NetModule.class})
public interface AppComponent {
   void inject(MainActivity activity);
   // void inject(MyFragment fragment);
   // void inject(MyService service);
}
```

**Note** that base classes are not be sufficient as injection targets.  Dagger 2 relies on strongly typed classes, so you must specify explicitly which ones should be defined.   (There are [suggestions](https://blog.gouline.net/2015/05/04/dagger-2-even-sharper-less-square/) to workaround the issue, but the code to do so may be more complicated to trace than simply defining them.)

#### Code generation

An important aspect of Dagger 2 is that the library generates code for classes annotated with the `@Component` interface.  You can use a class prefixed with `Dagger_` (i.e. `Dagger_TwitterApiComponent.java`) that will be responsible for instantiating an instance of our dependency graph and using it to perform the injection work for fields annotated with `@Inject`.  See the [[setup guide|Dependency-Injection-with-Dagger-2#setup]] and make sure you've included the `android-apt` plugin.

### Instantiating the component

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

Because we are overriding the default `Application` class, we also modify the app name `label` to correspond to `MyApp`.  This way your application will use this application class to handle the initial instantiation.

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

The example above showed that we used singletons that lasted the entire lifecycle of the application.  If we wish to create instances that are tied to the lifespan of an activity or fragment, Dagger 2 also enables the ability to create scoped instances.  Dagger 2 itself does not know about an activity or fragment so the responsibility lies on you to be consistent about keeping references.

The `@Singleton` annotation is provided as part of the Java annotation library.  We need to define our own `PerActivity` interface:

```java
import java.lang.annotation.Retention;
import javax.inject.Scope;

import static java.lang.annotation.RetentionPolicy.RUNTIME;

/**
 * A scoping annotation to permit objects whose lifetime should
 * conform to the life of the activity to be memoized in the
 * correct component.
 */
@Scope
@Retention(RUNTIME)
public @interface PerActivity {
}
```
See this [example code](https://github.com/google/dagger/tree/master/examples/android-activity-graphs) and Stack Overflow [discussion](http://stackoverflow.com/questions/28411352/what-determines-the-lifecycle-of-a-component-object-graph-in-dagger-2?rq=1).

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
* [Jake Wharton's Devoxx Dagger 2 Talk](https://www.parleys.com/tutorial/5471cdd1e4b065ebcfa1d557/)
* [Dagger 2 Google Developers Talk](https://www.youtube.com/watch?v=oK_XtfXPkqw)
* [Dagger 1 to Dagger 2](http://frogermcs.github.io/dagger-1-to-2-migration/)
* [Testing Dagger 2 on Android](http://fernandocejas.com/2015/04/11/tasting-dagger-2-on-android/)
* [Dagger 2 Testing with Mockito](http://blog.sqisland.com/2015/04/dagger-2-espresso-2-mockito.html#sthash.IMzjLiVu.dpuf)
* [Snorkeling with Dagger 2](https://github.com/konmik/konmik.github.io/wiki/Snorkeling-with-Dagger-2) 
* [Dependency Injection in Java](https://www.objc.io/issues/11-android/dependency-injection-in-java/)
* [Component Dependency vs. Submodules in Dagger 2](http://jellybeanssir.blogspot.de/2015/05/component-dependency-vs-submodules-in.html)