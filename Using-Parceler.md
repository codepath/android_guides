## Overview

Although creating Android [Parcelables](http://developer.android.com/reference/android/os/Parcelable.html) is usually at least 10x faster than using Serializable, creating [[Parcelable|Using-Parcelable]] objects requires creating a lot of boilerplate code in defining exactly the stream of data that should be serialized and deserialized as documented in [[this section|http://guides.codepath.com/android/Using-Parcelable#creating-a-parcelable-the-manual-way]].  

While there are IDE plugins to help facilitate the creating of these objects, another option is to leverage a third-party library called [Parceler](https://github.com/johncarl81/parceler) that will help automate this work.   Underneath the surface this library generates the necessary wrapper classes for you at compile time automatically, saving you the repetitive steps required for leveraging the performance benefits of Parcelables.

## Setup

Inside the `app/build.gradle` file add the following dependencies:

```gradle
dependencies {
    compile 'org.parceler:parceler-api:1.1.6'
    annotationProcessor 'org.parceler:parceler:1.1.6'
}
```

Make sure to [[upgrade|Getting-Started-with-Gradle#upgrading-gradle]] to the latest Gradle version to use the `annotationProcessor` syntax. 

## Usage

### Converting a model from Serializable objects

Suppose we have an User object that implements the `Serializable` interface:

```java
public class User implements Serializable {
    private String firstName;
    private String lastName;

    public User(String firstName, String lastName) {
       this.firstName = firstName;
       this.lastName = lastName;
    }
}
```

There are several requirements to convert this object to one that can be used by this library:

1. Remove the `Serializable interface` back to its original form.
2. Annotate the class with the `@Parcel` decorator.  
3. Use only public fields (private fields cannot be detected during annotation) that need to be serialized.
4. Create a public constructor with no arguments for the annotation library.

```java
@Parcel
public class User {
    // fields must be package private
    String firstName;
    String lastName;

    // empty constructor needed by the Parceler library 
    public User() {
    }

    public User(String firstName, String lastName) {
       this.firstName = firstName;
       this.lastName = lastName;
    }
}
```

### Wrapping Up a Parcel

Next, simply wrap your objects with `Parcels.wrap()`:

```java
User user = new User("John", "Doe");
Intent intent = new Intent(this, MyActivity.class);
intent.putExtra("user", Parcels.wrap(user));
startActivity(intent);
```

### Unwrapping a Parcel

On the receiving side, we need to unwrap the object:

```java
User user = (User) Parcels.unwrap(getIntent().getParcelableExtra("user"));
```

The Parceler library works by using the `@Parcel` annotation to generate the wrapper classes for you.  It works with many of the most standard Java types, including the ones defined [here](https://github.com/johncarl81/parceler#parcel-attribute-types).

### How it works

You can also look at your `app/build/generated/source/apt` directory to see how it generates these wrapper classes.  Parceler essentially handles the steps described in [[this section|Using-Parcelable#creating-a-parcelable-the-manual-way]].

<img src="https://imgur.com/6cR07Ae.png"/> 

## Using with ORM libraries

Some ORM libraries require extending the Java object with fields that Parceler is unable to serialize or deserialize.  In these cases, you should limit what fields should be analyzed in the inheritance using the `@Parcel(analyze={})` decorator:

```java
@Parcel(analyze={User.class})   // add Parceler annotation here
public class User extends BaseModel {
}
```

In this case only parameters from User class will be serialized avoiding any fields from BaseModel.

* [[DBFlow|DBFlow-Guide#using-with-the-parceler-library]]
* [Realm.IO](https://github.com/johncarl81/parceler/issues/57)

## Troubleshooting

* Getting `java.lang.ClassCastException: SomeObject$$Parcelable cannot be cast to SomeObject` when extracting a Parcel from a `Bundle`?
  * Be sure to call `Parcels.unwrap` when extracting the parcel from the `Bundle`:

    ```java
    User user = (User) Parcels.unwrap(someIntent.getParcelableExtra("user"));
    ```

* Getting a null exception when accessing a member instance stored within a `Parceler` object? 
  * Be sure that **all custom java objects** stored as fields within a Parceler object are **themselves also Parcels**. 
  * Make sure that every parceled field is properly converted into a `Parceler` object.
  * Breakpoint at the point right before the object is being wrapped and verify the member field is correct.
  * Breakpoint again at the point right after the object is extracted and unwrapped and verify member field is correct.

## References

* <http://parceler.org/>
