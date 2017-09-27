## Overview

This is intended as an open-source version of [this blog post](https://medium.com/rocknnull/effective-java-for-android-cheatsheet-bf4e3433889a#.u1xjikhk7) and [netcyrax](https://medium.com/@netcyrax) is the **sole author of the original post** from which this guide is based.

Please add your own Java for Android best-practices, tips and tricks below!

### Builder Pattern

When you have an object that requires more than ~3 constructor parameters, use a builder to construct the object. It might be a little more verbose to write but it scales well and it’s very readable. If you are creating a value class, consider [AutoValue](https://medium.com/rocknnull/no-more-value-classes-boilerplate-the-power-of-autovalue-bbaf36cf8bbe#.cazel3w3g).

```java
class Movie {
    static class Builder {
        String title;
        Builder withTitle(String title) {
            this.title = title;
            return this;
        }
        Movie build() {
            return new Movie(title);
        }
    }

    private Movie(String title) {
    [...]    
    }
}
// Use like this:
Movie matrix = new Movie.Builder().withTitle("The Matrix").build();
```

### Static Factory Methods

Instead of using the new keyword and the constructor use a static factory method (and a private constructor). These factory methods are named, are not required to return a new instance of the object each time and can return a different subtype depending on what’s needed.

```java
class Movie {
    [...]
    public static Movie create(String title) {
        return new Movie(title);
    }
}
```

### Static Inner Classes

If you define an inner class that does not depend on the outer class, don’t forget to define it as static. Failing to do so will result in each instance of the inner class to have references to the outer class.

```java
class Movie {
    [...]
    static class MovieAward {
        [...]
    }
}
```

### Return Empty Collection

When having to return a list/collection with no result avoid null. Returning an empty collection leads to a simpler interface (no need to document and annotate the null-returning function) and avoids accidental NPE. Prefer to return the same empty collection rather than creating a new one.

```java
List<Movie> latestMovies() {
    if (db.query().isEmpty()) {
        return Collections.emptyList();
    }
    [...]
}
```

### Use StringBuilder

Having to concatenate a few Strings, + operator might do. Never use it for a lot of String concatenations; the performance is really bad. Prefer a StringBuilder instead.

```java
String latestMovieOneLiner(List<Movie> movies) {
    StringBuilder sb = new StringBuilder();
    for (Movie movie : movies) {
        sb.append(movie);
    }
    return sb.toString();
}
```

### Force No Instantiation

If you do **not want an object to be created using the new keyword**, enforce it using a **private constructor*. Especially useful for utility classes that contain only static functions.

```java
class MovieUtils {
    private MovieUtils() {}
    static String titleAndYear(Movie movie) {
        [...]
    }
}
```

### Avoid Mutability

Immutable is an object that stays the same for its entire lifetime. All the necessary data of the object are provided during its creation. There are various advantages to this approach like simplicity, thread-safety and shareability.

```java
class Movie {
    [...]
    Movie sequel() {
        return Movie.create(this.title + " 2");
    }
}
// Use like this:
Movie toyStory = Movie.create("Toy Story");
Movie toyStory2 = toyStory.sequel();
```

It might be difficult to make every class immutable. If that’s the case, make your class as immutable as possible (eg, fields private final and the class final). Object creation might be more expensive on mobile, therefore don’t over do it.

## References

 * <https://medium.com/rocknnull/effective-java-for-android-cheatsheet-bf4e3433889a#.u1xjikhk7>
 * <https://www.amazon.com/Effective-Java-2nd-Joshua-Bloch/dp/0321356683>