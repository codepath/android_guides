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

### Dagger Workflow

Dagger 2 is composed mainly 3 main parts:

#### 1. Modules

Modules are responsible for creating and configuring the objects in your "dependency graph" through a set of **`@Provides`** annotated functions. Examples modules might include:
  * An `ApplicationModule` that provides your base `Application` context.
  * A `NetworkModule` that has your http client, http cache, cookie store, image loader, and configures timeouts.
  * A `StorageModule` that has your database, sharedPreferences, and key value store.
  * An `ActivityModule` for a specific Activity that provides local data shared by its child `Fragment`s

More details on how to define a module below.

#### 2. Components

Components are used to group together and build Modules into an object we can use to **get dependencies from**, and **perform injection with**.

* Identify the dependent objects and its dependencies.
* Create a class with the `@Module` annotation, using the `@Provides` annotation for every method that returns a dependency.
* Request dependencies in your dependent objects using the @Inject annotation.
* Create an interface using the `@Component` annotation and add the classes with the `@Module` annotation created in the second step.
* Create an object of the `@Component` interface to instantiate the dependent object with its dependencies.

Let's take a look at how we can use these to manage our object dependencies.

**Stub! Needs Attention**

## References

* [Dagger Github Page](http://google.github.io/dagger/)
* [Sample project using Dagger 2](https://github.com/vinc3m1/nowdothis)
* [Vince Mi's Dagger 2 Slides](https://docs.google.com/presentation/d/1bkctcKjbLlpiI0Nj9v0QpCcNIiZBhVsJsJp1dgU5n98/)
* <http://code.tutsplus.com/tutorials/dependency-injection-with-dagger-2-on-android--cms-23345>
* [Jake Wharton's Dagger 2 Slides](https://speakerdeck.com/jakewharton/dependency-injection-with-dagger-2-devoxx-2014)