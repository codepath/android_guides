### Overview

Although creating Android Parcelables is usually at least 10x faster than using Serializable, creating [[Parcelable|Using-Parcelable]] objects requires creating a lot of boilerplate code in defining exactly the stream of data that should be serialized and deserialized.  While there are IDE plugins to help facilitate the creating of these objects, another option is to leverage a third-party library called [Parceler](https://parceler.org) that will help automate this work.   Underneath the surface this library generates the necessary wrapper classes for you at compile time automatically, saving you the repetitive steps required for leveraging the performance benefits of Parcelables.

### Setup

To setup, we need to add the [android-apt](https://bitbucket.org/hvisser/android-apt) plugin to our classpath in our root `build.gradle` file:

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
    String firstName;
    String lastName;

    public User(String firstName, String lastName) {
       this.firstName = firstName;
       this.lastName = lastName;
    }
}
```

Simply remove this interface back to its original form and annotate the class with the `@Parcel` decorator.  You also need to create a public constructor with no arguments for the annotation lbirary to 

```java
@Parcel
public class User implements Serializable {
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

### Serializing for a collection of Java objects

Suppose you had another Java object that required a list of these User objects and wished to also convert this class containing these collection of items into a `Parcelable` object:

```java
public class Repository {
  List<User> participants;
}
```

While Parceler supports generating serialized items of standard Java types, it does not know how to serialize/deserialize this list.  For this reason, you may need to create a custom serializer and deserializer for this purpose.  We need to first add the `@Parcel` annotation but should also specify an explicit class that will handle the work of managing this list using the `ParcelPropertyConverter`:

```java
@Parcel
public class Repository {
    @ParcelPropertyConverter(UserListParcelConverter.class)
    List<User> participants;
}
```

We then need to implement this `ParcelConverter` class:

```java

public class UserListParcelConverter implements ParcelConverter<List<User>> {

    @Override
    public void toParcel(List<User> input, Parcel parcel) {
        if (input == null) {
            parcel.writeInt(-1);
        } else {
            parcel.writeInt(input.size());
            for (User item : input) {
                parcel.writeParcelable(Parcels.wrap(item), 0);
            }
        }
    }

    @Override
    public List<User> fromParcel(Parcel parcel) {
        int size = parcel.readInt();
        if (size < 0) {
            return null;
        }
        ArrayList<User> items = new ArrayList<>();
        for (int i = 0; i < size; ++i) {
            items.add((User) Parcels.unwrap(parcel.readParcelable(User.class.getClassLoader())));
        }
        return items;
    }
}
```

### References

* <http://parceler.org/>
