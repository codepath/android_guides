## Overview

"Magic constants" or type annotations are regular old Java constants decorated in a way that allows tools to recognize them as special values. Their syntax is not different from any other Java constant. Instead, an annotation with an easily readable name which lists all related constants has to be created. 

This annotation can then decorate a return value or a method parameter, giving hints to tools you use about the values that are accepted or can be expected. Their main purpose is to enforce type safety for method parameters and/or method return values while writing the source code.

This is typically used to **replace enums**. Google discourages the use of enums on Android app development as stated on their developer guidelines: "Enums often require more than twice as much memory as static constants. You should strictly avoid using enums on Android."

## IntDef

`IntDef` is a way of replacing an integer enum where there's a parameter that should only accept explicit int values. For example, suppose we want to record the type of a feed item as shown below:

```java
public class ItemTypeDescriptor {
  public static final int TYPE_MUSIC = 0;
  public static final int TYPE_PHOTO = 1;
  public static final int TYPE_TEXT = 2;

  public final int itemType;

  public ItemTypeDescriptor(int itemType) {
    this.itemType = itemType;
  }
}
```

Right now there is no validations to ensure that the type passed into the constructor is valid. We can use `IntDef` to ensure that the value is one of the expected values by adding annotations:

```java
public class ItemTypeDescriptor {
  // ... type definitions
  // Describes when the annotation will be discarded
  @Retention(RetentionPolicy.SOURCE)
  // Enumerate valid values for this interface
  @IntDef({TYPE_MUSIC, TYPE_PHOTO, TYPE_TEXT})
  // Create an interface for validating int types
  public @interface ItemTypeDef {}
  // Declare the constants
  public static final int TYPE_MUSIC = 0;
  public static final int TYPE_PHOTO = 1;
  public static final int TYPE_TEXT = 2;

  // Mark the argument as restricted to these enumerated types
  public ItemTypeDescriptor(@ItemTypeDef int itemType) {
    this.itemType = itemType;
  }
}
```

The same constructor call from the last example will now show an error in Android Studio because now it knows what values to expect. Refer to the [official Android annotations guide](https://developer.android.com/studio/write/annotations.html#enum-annotations) for more details.

## StringDef

`StringDef` is similarly a way of replacing an string enum where there's a parameter that should only accept explicit string values. For example, suppose we want to record the value for a setting as shown below:

```java
public class FilterColorDescriptor {
  public static final String FILTER_BLUE = "blue";
  public static final String FILTER_RED = "red";
  public static final String FILTER_GRAY = "gray";

  public final String filterColor;

  public FilterColorDescriptor(String filterColor) {
    this.filterColor = filterColor;
  }
}
```

Right now there is no validations to ensure that the type passed into the constructor is valid. We can use `StringDef` to ensure that the value is one of the expected values by adding annotations:

```java
public class FilterColorDescriptor {
  // ... type definitions
  // Describes when the annotation will be discarded
  @Retention(RetentionPolicy.SOURCE)
  // Enumerate valid values for this interface
  @StringDef({FILTER_BLUE, FILTER_RED, FILTER_GRAY})
  // Create an interface for validating String types
  public @interface FilterColorDef {}
  // Declare the constants
  public static final String FILTER_BLUE = "blue";
  public static final String FILTER_RED = "red";
  public static final String FILTER_GRAY = "gray";

  // Mark the argument as restricted to these enumerated types
  public FilterColorDescriptor(@FilterColorDef String filterColor) {
    this.filterColor = filterColor;
  }
}
```

The same constructor call from the last example will now show an error in Android Studio because now it knows what values to expect. Refer to the [official Android annotations guide](https://developer.android.com/studio/write/annotations.html#enum-annotations) for more details.

## References

* <https://developer.android.com/studio/write/annotations.html#enum-annotations>
* <https://infinum.co/the-capsized-eight/articles/magic-constants-in-android-development>
* <http://blog.shamanland.com/2016/02/int-string-enum.html>