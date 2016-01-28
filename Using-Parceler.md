### Overview

Although creating Android [Parcelables](http://developer.android.com/reference/android/os/Parcelable.html) is usually at least 10x faster than using Serializable, creating [[Parcelable|Using-Parcelable]] objects requires creating a lot of boilerplate code in defining exactly the stream of data that should be serialized and deserialized as documented in [[this section|http://guides.codepath.com/android/Using-Parcelable#creating-a-parcelable-the-manual-way]].  While there are IDE plugins to help facilitate the creating of these objects, another option is to leverage a third-party library called [Parceler](https://parceler.org) that will help automate this work.   Underneath the surface this library generates the necessary wrapper classes for you at compile time automatically, saving you the repetitive steps required for leveraging the performance benefits of Parcelables.

### Setup

To setup, we need to add the [android-apt](https://bitbucket.org/hvisser/android-apt) plugin to our classpath in our root `build.gradle` file.  This plugin enables the Parceler library to be used for annotation processing but not added to the final build.

```gradle
dependencies {
  classpath 'com.neenbedankt.gradle.plugins:android-apt:1.8'
}
```

Inside the `app/build.gradle` file, we should apply the plugin before the Parceler dependencies are added.  This way, the `apt` keyword can be used, which is primarily used for annotation processing and keeps the libraries from being added to the classpath.

```gradle
apply plugin: 'com.neenbedankt.android-apt'

dependencies {
  compile 'org.parceler:parceler-api:1.0.4'
  apt 'org.parceler:parceler:1.0.4'
}
```

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
3. Use only public fields (private fields cannot be detected during annotation)
4. Create a public constructor with no arguments for the annotation library too.

```java
@Parcel
public class User implements Serializable {
    // fields must be public
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

Next, simply wrap your objects with `Parcel.wrap()`:

```java
User user = new User("John", "Doe");
Intent intent = new Intent(this, MyActivity.class);
intent.putExtra("user", Parcels.wrap(user));
startActivity(intent);
```

On the receiving side, we simply need to unwrap the object:

```java
User user = (User) Parcels.unwrap(getIntent().getParcelableExtra("user"));
```

The Parceler library works by using the `@Parcel` annotation to generate the wrapper classes for you.  It works with many of the most standard Java types, including the ones defined [here](https://github.com/johncarl81/parceler#parcel-attribute-types).

### How it works

You can also look at your `app/build/generated/source/apt` directory to see how it generates these wrapper classes.  Parceler essentially handles the steps described in [[this section|Using-Parcelable#creating-a-parcelable-the-manual-way]].

<img src="http://imgur.com/6cR07Ae.png"/> 

### References

* <http://parceler.org/>
