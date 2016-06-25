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
Observable.just("Hello RxAndroid")
	.subscribe(new Action1<String>() {
		@Override
		public void call(String s) {
			Log.d("Emitted", s);
		}
	});
```

Consider the same code with lambda expressions:

```java
Observable.just("Hello RxAndroid")
	.subscribe(s -> Log.d("Emitted", s));
```

## Setup

Make sure you have JDK 8 installed or higher.  Click [here](http://www.oracle.com/technetwork/java/javase/downloads/index.html) in case you need to download it.  If you are using a continuous integration service, you should also make sure to be specifying a JDK 8 environment.

To use Java 8 lambda expressions in your Android code, you can either use the `Gradle Retrolambda` plugin developed by Evan Tatarka or use the new [Android Jack toolchain](https://source.android.com/source/jack.html).  

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

Android studio instant run does not currenty work with Jack and will be disabled while using the new toolchain. Because Jack does not generate intermediate class files when compiling an app, tools that depend on these files for example, Lint detectors, do not currently work with Jack. Also tools like `android-apt` which is required for using `Dagger 2` in your Android project does not currently work with Jack.

## Attribution

This guide was originally drafted by [Adegeye Mayowa](https://github.com/mayojava)