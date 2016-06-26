### Overview

Java 8 [lambda expressions](https://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html) help eliminate boilerplate code that makes the syntax verbose and less clear.  For instance, even basic click listeners syntax can be simplified:

```java
myButton.setOnClickListener(new View.OnClickListener() {
                    @Override
                    public void onClick(View v) {
                        Log.d("debug", "Button clicked");
                    }
                });

```

With event listeners that only have one method that need to be implemented, lambda expressions can be used to simplify the syntax:

```java
myButton.setOnClickListener(v -> {
            Log.d("debug", "Button clicked");
});
```

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

Lambda expressions rely on type inference to fill in the blanks.   You can look to the left of Android Studio to see how it is inferring which type to use:

<img src="http://imgur.com/n1RrHpT.png">

## Setup

Make sure you have JDK 8 installed or higher.  Click [here](http://www.oracle.com/technetwork/java/javase/downloads/index.html) in case you need to download it.  If you are using a continuous integration service, you should also make sure to be specifying a JDK 8 environment.

To use Java 8 lambda expressions in your Android code, you can either use the [Gradle Retrolambda plugin](https://github.com/evant/gradle-retrolambda) developed by Evan Tatarka or use the new [Android Jack toolchain](https://source.android.com/source/jack.html).  

### Retrolambda Setup

* Open the root `build.gradle` file and add the following dependency:

```gradle
buildscript {
  dependencies {
    classpath 'me.tatarka:gradle-retrolambda:3.2.5'
  }
}
```

* Modify the app module `build.gradle` file to apply the **me.tatarka.retrolambda** plugin:
```gradle
apply plugin: 'com.android.application'
apply plugin: 'me.tatarka.retrolambda'
```

* Add a new `compileOptions` block, then `sourceCompatibility` and `targetCompatibility` Java version should be set as 1.8

```groovy 
compileOptions {
    sourceCompatibility JavaVersion.VERSION_1_8
    targetCompatibility JavaVersion.VERSION_1_8
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

### Using Jack Toolchain

Android provided a way to use some Java 8 language features including `lambda expressions` in your Android project by enabling the **Jack toolchain**. To do this, edit your module level `build.gradle` file as follows:

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

* Tools like `android-apt` which is required for using `Dagger 2` in your Android project do not currently work with Jack.

## Attribution

This guide was originally drafted by [Adegeye Mayowa](https://github.com/mayojava)

## References

* <https://www.captechconsulting.com/blogs/getting-started-with-rxjava-and-android>