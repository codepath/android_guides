## Overview 

Many Android apps rely on instantiating objects that often require other dependencies.  For instance, a Twitter API client may be built using a networking library such as [[Retrofit|Consuming-APIs-with-Retrofit]]. To use this library, you might also need to add parsing libraries such as [[Gson|Leveraging-the-Gson-Library]].  In addition, classes that implement authentication or caching may require accessing [[shared preferences|Storing-and-Accessing-SharedPreferences]] or other common storage, requiring instantiating them first and creating an inherent dependency chain.

If you're not familiar with Dependency Injection, watch [this](https://www.youtube.com/watch?v=IKD2-MAkXyQ) quick video.

Dagger 2 analyzes these dependencies for you and generates code to help wire them together.  While there are other Java dependency injection frameworks, many of them suffered limitations in relying on XML, required validating dependency issues at run-time, or incurred performance penalties during startup. [Dagger 2](http://google.github.io/dagger/) relies purely on using Java [annotation processors](https://www.youtube.com/watch?v=dOcs-NKK-RA) and compile-time checks to analyze and verify dependencies.  It is considered to be one of the most efficient dependency injection frameworks built to date.

### Advantages

Here is a list of other advantages for using Dagger 2:

 * **Simplifies access to shared instances**. Just as the [[ButterKnife|Reducing-View-Boilerplate-with-Butterknife]] library makes it easier to define references to Views, event handlers, and resources, Dagger 2 provides a simple way to obtain references to shared instances.  For instance,  once we declare in Dagger our singleton instances such as  `MyTwitterApiClient` or `SharedPreferences`, we can declare fields with a simple `@Inject` annotation:

```java
public class MainActivity extends Activity {
   @Inject MyTwitterApiClient mTwitterApiClient;
   @Inject SharedPreferences sharedPreferences;

   public void onCreate(Bundle savedInstance) {
       // assign singleton instances to fields
       InjectorClass.inject(this);
   } 
```

 * **Easy configuration of complex dependencies**. There is an implicit order in which your objects are often created.   Dagger 2 walks through the dependency graph and [[generates code|Dependency-Injection-with-Dagger-2#code-generation]] that is both easy to understand and trace, while also saving you from writing the large amount of boilerplate code you would normally need to write by hand to obtain references and pass them to other objects as dependencies.  It also helps simplify refactoring, since you can focus on what modules to build rather than focusing on the order in which they need to be created.

 * **Easier unit and integration testing**  Because the dependency graph is created for us, we can easily swap out modules that make network responses and mock out this behavior.

 * **Scoped instances**  Not only can you easily manage instances that can last the entire application lifecycle, you can also leverage Dagger 2 to define instances with shorter lifetimes (i.e. bound to a user session, activity lifecycle, etc.).

### Setup

Android Studio by default will not allow you to navigate to generated Dagger 2 code as legitimate classes because they are not normally added to the source path.  Adding the `annotationProcessor` plugin will add these files into the IDE classpath and enable you to have more visibility.

Make sure to [[upgrade|Getting-Started-with-Gradle#upgrading-gradle]] to the latest Gradle version to use the `annotationProcessor` syntax: 

```gradle
dependencies {
  implementation 'com.google.dagger:dagger-android:2.20'
  implementation 'com.google.dagger:dagger-android-support:2.20' // if you use the support libraries
  annotationProcessor 'com.google.dagger:dagger-android-processor:2.20'
  annotationProcessor 'com.google.dagger:dagger-compiler:2.20'
}
```

If you are using Kotlin, then you should use the following setup:

```gradle
...
apply plugin: 'kotlin-kapt'

dependencies {
  implementation 'com.google.dagger:dagger-android:2.20'
  implementation 'com.google.dagger:dagger-android-support:2.20' // if you use the support libraries
  kapt 'com.google.dagger:dagger-android-processor:2.20'
  kapt 'com.google.dagger:dagger-compiler:2.20'
}
```

Note that the `compileOnly` keyword refers to dependencies that are only needed at compilation.  Since Dagger 2 generates code that is used by your app at runtime, the implementation configuration should be used for the dagger libraries. The Dagger compiler generates code that is used to create the dependency graph of the classes defined in your source code.  These classes are added to the IDE class path during compilation. The `annotationProcessor` keyword, which is understood by the Android Gradle plugin, does not add these classes to the class path, they are used only for annotation processing, which prevents accidentally referencing them.

### Creating Singletons
![Dagger Injections Overview](https://raw.githubusercontent.com/codepath/android_guides/master/images/dagger_general.png)

The simplest example is to show how to centralize all your singleton creation with Dagger 2.  Suppose you weren't using any type of dependency injection framework and wrote code in your Twitter client similar to the following:

```java
OkHttpClient client = new OkHttpClient();

// Enable caching for OkHttp
int cacheSize = 10 * 1024 * 1024; // 10 MiB
Cache cache = new Cache(getApplication().getCacheDir(), cacheSize);
client.setCache(cache);

// Used for caching authentication tokens
SharedPreferences sharedPreferences = PreferenceManager.getDefaultSharedPreferences(this);

// Instantiate Gson
Gson gson = new GsonBuilder().create();
GsonConverterFactory converterFactory = GsonConverterFactory.create(gson);

// Build Retrofit
Retrofit retrofit = new Retrofit.Builder()
                                .baseUrl("https://api.github.com")
                                .addConverterFactory(converterFactory)
                                .client(client)  // custom client
                                .build();
```

#### Declare your singletons 

You need to define what objects should be included as part of the dependency chain by creating a Dagger 2 **module**.  For instance, if we wish to make a single `Retrofit` instance tied to the application lifecycle and available to all our activities and fragments, we first need to make Dagger aware that a `Retrofit` instance can be provided.   

Because we wish to setup caching, we need an Application context.  Our first Dagger module, `AppModule.java`, will be used to provide this reference.  We will define a method annotated with `@Provides` that informs Dagger that this method is the constructor for the `Application` return type (i.e., it is the method in charge of providing the instance of the Application class):

```java
@Module
public class AppModule {

    Application mApplication;

    public AppModule(Application application) {
        mApplication = application;
    }

    @Provides
    @Singleton
    Application providesApplication() {
        return mApplication;
    }
}
```

We create a class called `NetModule.java` and annotate it with `@Module` to signal to Dagger to search within the available methods for possible instance providers.  

The methods that will actually expose available return types should also be annotated with the `@Provides` annotation.  The `@Singleton` annotation also signals to the Dagger compiler that the instance should be created only once in the application.  In the following example, we are specifying `SharedPreferences`, `Gson`, `Cache`, `OkHttpClient`, and `Retrofit` as the return types that can be used as part of the dependency list.  

```java
@Module
public class NetModule {

    String mBaseUrl;
    
    // Constructor needs one parameter to instantiate.  
    public NetModule(String baseUrl) {
        this.mBaseUrl = baseUrl;
    }

    // Dagger will only look for methods annotated with @Provides
    @Provides
    @Singleton
    // Application reference must come from AppModule.class
    SharedPreferences providesSharedPreferences(Application application) {
        return PreferenceManager.getDefaultSharedPreferences(application);
    }

    @Provides
    @Singleton
    Cache provideOkHttpCache(Application application) { 
        int cacheSize = 10 * 1024 * 1024; // 10 MiB
        Cache cache = new Cache(application.getCacheDir(), cacheSize);
        return cache;
    }

   @Provides 
   @Singleton
   Gson provideGson() {  
       GsonBuilder gsonBuilder = new GsonBuilder();
       gsonBuilder.setFieldNamingPolicy(FieldNamingPolicy.LOWER_CASE_WITH_UNDERSCORES);
       return gsonBuilder.create();
   }

   @Provides
   @Singleton
   OkHttpClient provideOkHttpClient(Cache cache) {
      OkHttpClient.Builder client = new OkHttpClient.Builder();
      client.cache(cache);
      return client.build();
   }

   @Provides
   @Singleton
   Retrofit provideRetrofit(Gson gson, OkHttpClient okHttpClient) {
      Retrofit retrofit = new Retrofit.Builder()
                .addConverterFactory(GsonConverterFactory.create(gson))
                .baseUrl(mBaseUrl)
                .client(okHttpClient)
                .build();
        return retrofit;
    }
}
```

Note that the method names (i.e. `provideGson()`, `provideRetrofit()`, etc) do not matter and can be named anything.  The return type annotated with a `@Provides` annotation is used to associate this instantiation with any other modules of the same type.  The `@Singleton` annotation is used to declare to Dagger that the provided object is to be only initialized only once during the entire lifecycle of the Component which uses that Module.  

A `Retrofit` instance depends both on a `Gson` and `OkHttpClient` instance, so we can define another method within the same class that takes these two types.  The `@Provides` annotation and these two parameters in the method will cause Dagger to recognize that there is a dependency on `Gson` and `OkHttpClient` to build a `Retrofit` instance.

#### Define injection targets

Dagger provides a way for the fields in your activities, fragments, or services to be assigned references simply by annotating the fields with an `@Inject` annotation and calling an `inject()` method.  Calling `inject()` will cause Dagger 2 to locate the singletons in the dependency graph to try to find a matching return type.  If it finds one, it assigns the references to the respective fields. For instance, in the example below, it will attempt to find a provider that returns `MyTwitterApiClient` and a `SharedPreferences` type:

```java
public class MainActivity extends Activity {
   @Inject MyTwitterApiClient mTwitterApiClient;
   @Inject SharedPreferences sharedPreferences;

  public void onCreate(Bundle savedInstance) {
       // assign singleton instances to fields
       InjectorClass.inject(this);
   } 
```

The injector class used in Dagger 2 is called a **component**.  It assigns references in our activities, services, or fragments to have access to singletons we earlier defined.  We will need to annotate this class with a `@Component` annotation. Note that the activities, services, or fragments that are allowed to request the dependencies declared by the modules (by means of the `@Inject` annotation) should be declared in this class with individual `inject()` methods: 


```java
@Singleton
@Component(modules={AppModule.class, NetModule.class})
public interface AppComponent {
   void inject(MainActivity activity);
   // void inject(MyFragment fragment);
   // void inject(MyService service);
}
```

**Note** that base classes are not sufficient as injection targets.  Dagger 2 relies on strongly typed classes, so you must specify explicitly which ones should be defined.   (There are [suggestions](https://blog.gouline.net/2015/05/04/dagger-2-even-sharper-less-square/) to workaround the issue, but the code to do so may be more complicated to trace than simply defining them.)

#### Code generation

An important aspect of Dagger 2 is that the library generates code for classes annotated with the `@Component` interface.  You can use a class prefixed with `Dagger` (i.e. `DaggerTwitterApiComponent.java`) that will be responsible for instantiating an instance of our dependency graph and using it to perform the injection work for fields annotated with `@Inject`.  See the [[setup guide|Dependency-Injection-with-Dagger-2#setup]].
### Instantiating the component

We should do all this work within a specialization of the `Application` class since these instances should be declared only once throughout the entire lifespan 
@Documented
@Retention(value=RetentionPolicy.RUNTIME)
public @interface MyActivityScope
{
}
```are.net/nakhimovich/advanced-dagger-talk-from-360anDev)