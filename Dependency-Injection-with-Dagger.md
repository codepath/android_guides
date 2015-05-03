## Overview

This guide covers dependency injection using [Dagger 2](http://google.github.io/dagger/). But first, **what exactly is dependency injection?**

When you have an object that needs or depends on another object to do its work, you have a dependency. Dependency injection makes dependencies easier to manage throughout your app by supplying dependent objects as they are needed.

In short, Dagger 2 uses dependency injection to make **constructing objects and their use within an app significantly easier and less messy**.

### Dagger Vocabulary

Dagger 2 exposes a number of special annotations:

* `@Module` for the classes whose methods provide dependencies
* `@Provides` for the specific methods within `@Module` classes
* `@Inject` to request a dependency (a constructor, a field, or a method)
* `@Component` is a bridge interface between modules and injection

Let's take a look at the Dagger workflow within an app.

### Dagger Workflow

To implement Dagger 2 correctly, you have to follow these steps:

* Identify the dependent objects and its dependencies.
* Create a class with the `@Module` annotation, using the `@Provides` annotation for every method that returns a dependency.
* Request dependencies in your dependent objects using the @Inject annotation.
* Create an interface using the `@Component` annotation and add the classes with the `@Module` annotation created in the second step.
* Create an object of the `@Component` interface to instantiate the dependent object with its dependencies.

Let's take a look at how we can use these to manage our object dependencies.

**Stub! Needs Attention**

## References

* <http://google.github.io/dagger/>
* <https://github.com/vinc3m1/nowdothis>
* [Dagger 2 Slides](https://docs.google.com/presentation/d/1bkctcKjbLlpiI0Nj9v0QpCcNIiZBhVsJsJp1dgU5n98/edit#slide=id.p)
* <http://code.tutsplus.com/tutorials/dependency-injection-with-dagger-2-on-android--cms-23345>