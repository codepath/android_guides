## Overview

This guide covers dependency injection using [Dagger 2](http://google.github.io/dagger/).

### What is Dependency Injection?

Dependency injection is a way to separate concerns between **configuration** and **usage**

![Configuration vs Usage](http://imgur.com/WsKXyM9.jpg)

For the Usage side (e.g. Activities, Fragments, Views) you only need to worry about:

1. Getting the object you need (simply annotating with `@Inject`)
2. Using it.

On the configuration side you get:
* Less duplicated code
* Easy singleton management
* Easier mocking for testing and variants
* Easy configuration of complex dependencies

### Dagger Vocabulary

Dagger 2 exposes a number of special annotations:

* `@Module` for the classes whose methods provide dependencies
* `@Provides` for the specific methods within `@Module` classes
* `@Component` is a collection of modules that can perform injection
* `@Inject` to request a dependency (a constructor argument, or a field)
* `@Singleton` and custom scopes to define singletons within specific scopes. (We recommend using custom scopes like `@PerApp` or `@PerActivity` for clarity of how long a "singleton" lives for)

Let's take a look at the Dagger workflow within an app.

### Dagger Workflow Overview

Dagger 2 is composed mainly 3 main parts:

1. [Modules](#1-modules)
2. [Components](#2-components)
3. [Injection](#3-injection)

#### 1. Modules

Modules are responsible for creating and configuring the objects in your "dependency graph" through a set of **`@Provides`** annotated functions. Examples modules might include:
  * An `ApplicationModule` that provides your base `Application` context.
  * A `NetworkModule` that has your http client, http cache, cookie store, image loader, and configures timeouts.
  * A `StorageModule` that has your database, sharedPreferences, and key value store.
  * An `ActivityModule` for a specific Activity that provides local data shared by its child `Fragment`s

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

#### 2. Components

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

Notice within the annotation, we specify the Modules that @Provide the component and any other full components this component might depend on.

`inject(Class thing)` methods are used to explicitly define classes that have fields annotated with `@Inject` to be injected.

`Class thing()` methods are used to expose dependencies either to direct consumers or to dependent components to consume.


#### 3. Injection

Dependencies are used or "injected" through the functions on a `Component`.

There are 3 ways to get a dependency from a component:
1. Directly by calling `component.thing()`
2. Annotating a field on an object with `@Inject`, then calling `component.inject(object)`. Oftentimes the object is `this`.
3. Using constructor injection by annotating your constructor with @Inject, and letting Dagger create your class for you.



### Defining custom scopes

Defining custom scope(s) is highly recommended as it 

## References

* [Dagger Github Page](http://google.github.io/dagger/)
* [Sample project using Dagger 2](https://github.com/vinc3m1/nowdothis)
* [Vince Mi's Codepath Meetup Dagger 2 Slides](https://docs.google.com/presentation/d/1bkctcKjbLlpiI0Nj9v0QpCcNIiZBhVsJsJp1dgU5n98/)
* <http://code.tutsplus.com/tutorials/dependency-injection-with-dagger-2-on-android--cms-23345>
* [Jake Wharton's Devoxx Dagger 2 Slides](https://speakerdeck.com/jakewharton/dependency-injection-with-dagger-2-devoxx-2014)