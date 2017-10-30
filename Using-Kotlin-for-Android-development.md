## What is Kotlin?

[Kotlin](https://kotlinlang.org) is a language by [JetBrains](https://www.jetbrains.com), the company behind [IntelliJ IDEA](https://www.jetbrains.com/idea), which Android Studio is based on, and other developer tools. Kotlin is purposely built for large scale software projects to improve upon Java with a focus on readability, correctness, and developer productivity.

The language was created in response to limitations in Java which were hindering development of JetBrains' software products and after an evaluation of all other JVM languages proved unsuitable. Since the goal of Kotlin was for use in improving their products, it focuses very strongly on interop with Java code and the Java standard library.

## Why Kotlin?

- 100% interoperable with Java - Kotlin and Java can co-exist in one project. You can continue to use existing libraries in Java.
- Concise - Drastically reduce the amount of boilerplate code you need to write.
- Safe - Avoid entire classes of errors such as null pointer exceptions.
- It's functional - Kotlin uses many concepts from functional programming, such as lambda expressions.


## Syntax Crash Course

### Variables

Defining local variables

Assign-once (read-only) local variable:

```kotlin
val a: Int = 1
val b = 1   // `Int` type is inferred
val c: Int  // Type required when no initializer is provided
c = 1       // definite assignment
```
```java
int a = 1;
int b = 1;
int c;
c = 1;
```

Mutable variable:

```kotlin
var x = 5 // `Int` type is inferred
x += 1
```

### Functions

Function having two Int parameters with Int return type:

```kotlin
fun sum(a: Int, b: Int) :Int {
	return a + b
}
```

Function with an expression body and inferred return type:
```kotlin
fun sum(a: Int, b: Int) = a + b
```
Function returning no meaningful value:
```kotlin
fun printSum(a: Int, b: Int): Unit {
  print(a + b)
}
```

Unit return type can be omitted:

```kotlin
fun printSum(a: Int, b: Int) {
  print(a + b)
}
```

### Using collections

Iterating over a collection:

```kotlin
for (name in names)
  println(name)
```

Checking if a collection contains an object using in operator:

```kotlin
if (text in names) // names.contains(text) is called
  print("Yes")
```

Using lambda expressions to filter and map collections:

```kotlin
names
    .filter { it.startsWith("A") }
    .sortedBy { it }
    .map { it.toUpperCase() }
    .forEach { print(it) }
```

### Null Safety

```kotlin
val x: String? = "Hi"
x.length // Won't compile
val y: String = null // Won't compile
```
Dealing with null

```kotlin
// using the safe call operator ?.
x?.length // This returns x.length if x is not null, and null otherwise

// Elvis Operator ?:
val len = x?.length ?: -1 // This will return -1 if x is null
```


## Configure your development environment

To be able to write and compile Kotlin code in your Android application you need to do the following:

1. Install Android Studio
  First thing you need is to have Android Studio installed.
2. Install Kotlin plugin
  Under `Preferences (OSX)` or `Settings (Windows/Linux)` -> `Plugins` -> `Browse Repositories` type `Kotlin` to find the Kotlin plugin. Click `Install` and follow the instructions.
3. Configure Gradle
  The Kotlin plugin includes a tool which does the Gradle configuration for us.
  - Click on `Tools` -> `Kotlin` -> `Configure Kotlin in Project`
<br/><img src="https://i.imgur.com/efeuBdX.png" width="500" />

  - Select `Android with Gradle`
<br/><img src="https://i.imgur.com/cfJOIQH.png" width="500" />

  - Choose`All Modules` -> Select the `Kotlin compiler and the runtime version` you want from the dropdown and click `OK`.
<br/><img src="https://i.imgur.com/ITVWnp0.png" width="500" />


Your `build.gradle` file will look like this:

```gradle
apply plugin: 'com.android.application'
apply plugin: 'kotlin-android'

android {
    compileSdkVersion 23
    buildToolsVersion "23.0.3"

    defaultConfig {
        applicationId "com.example.hellokotlin"
        minSdkVersion 10
        targetSdkVersion 23
        versionCode 1
        versionName "1.0"
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
    sourceSets {
        main.java.srcDirs += 'src/main/kotlin'
    }
}

dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    testCompile 'junit:junit:4.12'
    compile 'com.android.support:appcompat-v7:23.4.0'
    compile "org.jetbrains.kotlin:kotlin-stdlib:$kotlin_version"
}
repositories {
    mavenCentral()
}
```

## Writing your first Kotlin Code

You can start by converting your existing Java File to Kotlin file. Open your Java File -> Click on `Code` menu item -> select `Convert Java File to Kotlin File`. Your converted file would look like this:

```kotlin
class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
    }

}
```

Starting from Kotlin version `1.0.2`, action to create new activity in Kotlin has been added. To create new Android Kotlin activity, Go to `File` -> `New`->`Kotlin Activity`.

## Attribution

This guide was originally put together by [Kirk Saviour (@savekirk)](https://github.com/savekirk) as referenced [on this thread](https://github.com/codepath/android_guides/issues/169#issuecomment-222051554).

## References

- <https://medium.com/@calren24/kotlin-in-action-chapter-2-kotlin-basics-430a905ef4d8>
- <https://medium.com/@calren24/kotlin-in-action-chapter-1-what-and-why-9d2899560755>
- http://antonioleiva.com/kotlin-for-android-introduction/
- http://antonioleiva.com/kotlin-android-create-project/
- https://kotlinlang.org
- https://kotlinlang.org/docs/reference/
- https://www.youtube.com/watch?v=A2LukgT2mKc
- https://docs.google.com/document/d/1ReS3ep-hjxWA8kZi0YqDbEhCqTt29hG8P44aA9W0DM8/preview?hl=ru&forcehl=1
- <https://blog.simon-wirtz.de/kotlin-features-miss-java/>