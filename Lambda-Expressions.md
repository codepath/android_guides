### Overview

Java 8 [lambda expressions](https://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html) help eliminate boilerplate code that makes the syntax verbose and less clear.  For instance, consider the standard basic click listener:

```java
myButton.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        Log.d("debug", "Button clicked");
    }
});
```

Lambda expressions can greatly simplify this code, especially in this case where the event listener has only one method that needs to be implemented:

```java
myButton.setOnClickListener(v -> Log.d("debug", "Button clicked"); );
```

If you have more than one line to execute, then you should surround the block of code with braces:

```java
myButton.setOnClickListener(v -> { 
  Log.d("debug", "Button clicked"); 
  Toast.makeText(MyActivity.this, "here", Toast.LENGTH_LONG).show();
});
```

### Method References

You may also notice in some cases lambda expressions also contain double colons `::`.  These refer to a new syntax in Java 8 known as [method references](http://baddotrobot.com/blog/2014/02/18/method-references-in-java8/).  You can reference a class or instance and pass along the method that will handle the event:

```java
public void onCreate() { 
   myButton.setOnClickListener(this::logError);
}

public void logError(View v) {
  Log.d("debug", "Button clicked"); 
}
```

### RxJava

Lambda expressions are especially helpful in [[RxJava|RxJava]] as well.  Take a look at the code below for creating an Observable and subscribing an Observer to it.

Creating and subscribing to an observable without lambdas:

```java
Observable.just("1", "2", "3")
          .subscribe(new Subscriber<String>() {
                         @Override
                         public void onCompleted() {
                            Log.d("debug", "complete");
                         }

                         @Override
                         public void onError(Throwable throwable) {
                            Log.d("debug", throwable.getMessage());
                         }

                         @Override
                         public void onNext(String s) {
                            Log.d("debug", s);
                         }
                     });
```

Consider the same code with lambda expressions:

```java
Observable.just("1", "2", "3")
          .subscribe(
          value -> Log.d("debug", value),
          throwable -> Log.d("debug", throwable.getMessage()),
          () -> Log.d("debug", "complete"));
```

Lambda expressions rely on type inference to fill in the blanks. Notice that the right-hand side of the arrow does not require a return statement if you do not surround the block with `{` and `}`.  Also notice that a function with zero or multiple arguments need parenthesis enclosing them.

You can look to the left of Android Studio to see how it is inferring which type to use:

<img src="https://imgur.com/n1RrHpT.png">

## Setup

Make sure you have JDK 8 installed or higher.  Click [here](http://www.oracle.com/technetwork/java/javase/downloads/index.html) in case you need to download it.  If you are using a continuous integration service, you should also make sure to be specifying a JDK 8 environment.

To use Java 8 lambda expressions in your Android code, you can either use the [Gradle Retrolambda plugin](https://github.com/evant/gradle-retrolambda) developed by Evan Tatarka. While [Android Jack toolchain](https://source.android.com/source/jack.html) is also an option it is currently deprecated as per this [announcement](https://android-developers.googleblog.com/2017/03/future-of-java-8-language-feature.html).

### Retrolambda Setup

* Open the root `build.gradle` file and add the following dependency:

```gradle
buildscript {
  dependencies {
    classpath 'me.tatarka:gradle-retrolambda:3.5.0'
  }
}
```

* Modify the app module `build.gradle` file to apply the **me.tatarka.retrolambda** plugin:
```gradle
apply plugin: 'com.android.application'

apply plugin: 'me.tatarka.retrolambda' // make sure to apply last!
```

Make sure to specify the plug-in last, especially if the `android-apt` plug-in used according to this [article](https://medium.com/android-news/retrolambda-on-android-191cc8151f85#.c5vbxdwst).

* Add a new `compileOptions` block, then `sourceCompatibility` and `targetCompatibility` Java version should be set as 1.8

```groovy 
android { 
  compileOptions {
      sourceCompatibility JavaVersion.VERSION_1_8
      targetCompatibility JavaVersion.VERSION_1_8
  }
}
```

* If you have multiple Java versions installed, set the Java 8 JDK path in `retrolambda` block:

```groovy 
retrolambda {
    jdk '/path/to/java-8/jdk'
}
```

#### Limitations

* Using the Android lint detector will trigger a `java.lang.UnsupportedOperationException: Unknown ASTNode child: LambdaExpression`.  To get around this [issue](https://github.com/evant/gradle-retrolambda/issues/96), you need to add this third-party package that replaces the parsing engine for Java to support lamda expressions:

```gradle
buildscript {
  dependencies {
     classpath 'me.tatarka.retrolambda.projectlombok:lombok.ast:0.2.3.a2'
  }
}
```

* If you intend to use Retrolambda with ProGuard, make sure to add this line to your configuration.

```
-dontwarn java.lang.invoke.*
```

### Jack Toolchain Setup (Experimental)

Android provided a way to use some Java 8 language features including `lambda expressions` in your Android project by enabling the **Jack toolchain**.  You should not use this approach currently since [it is deprecated](https://android-developers.googleblog.com/2017/03/future-of-java-8-language-feature.html).  To do this, edit your module level `build.gradle` file as follows:

```groovy 
android {
  ...
  defaultConfig {
    ...
    jackOptions {
      enabled true
    }
  }
  compileOptions {
    sourceCompatibility JavaVersion.VERSION_1_8
    targetCompatibility JavaVersion.VERSION_1_8
  }
}
```

Sync your gradle file, if you encounter any build error, you may need to download the latest `Android SDK Build-tools` from the `SDK Manager`.

**Known issues with using the Jack toolchain**

* Android Studio Instant Run does not currently work with Jack and will be disabled while using the new toolchain. 

* Because Jack does not generate intermediate class files when compiling an app, tools that depend on these files for example, lint detectors, do not currently work with Jack. 

* Annotation support for libraries such as [[Dagger 2|Dependency-Injection-with-Dagger-2]] to show you in your Android project may not work with Jack.  The most recent experimental releases are starting to add [this support](http://stackoverflow.com/questions/31789967/new-jack-toolchain-crashes-when-using-android-apt-plugin).

## Converting Lambda Expressions with Android Studio

If you wish to convert your code to lambda expressions, move your cursor to the slightly greyed out section of your anonymous class and look on the left-hand side of the Android Studio editor for the light bulb:

<img src="https://imgur.com/KDzMS8l.png"/>

Once you see the `Replace with lambda` appear, you can also apply `Fix all` click on the left-hand side to convert all the possible candidates automatically as [outlined here](http://stackoverflow.com/a/36746855).

## Troubleshooting

* Getting `An exception has occurred in the compiler (1.8.0_05)` or `com.sun.tools.javac.code.Symbol$CompletionFailure` or `java.lang.invoke.MethodType not found`? 
  * Try swapping the order of `apply plugin: 'retrolambda'` and `apply plugin: "android"`. 
  * Check the path to the JDK in Android Studio settings to ensure correctness.
  * Check the path to the JDK in `build.gradle` to ensure correctness and consistency. 

## Attribution

This guide was originally drafted by [Adegeye Mayowa](https://github.com/mayojava).

## References

* <https://www.captechconsulting.com/blogs/getting-started-with-rxjava-and-android>
