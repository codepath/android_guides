## What is Kotlin?

[Kotlin](https://kotlinlang.org) is a language by [JetBrains](https://www.jetbrains.com), the company behind [IntelliJ IDEA](https://www.jetbrains.com/idea), which Android Studio is based on, and other developer tools. Kotlin is purposely built for large scale software projects to improve upon Java with a focus on readability, correctness, and developer productivity.

The language was created in response to limitations in Java which were hindering development of JetBrains' software products and after an evaluation of all other JVM languages proved unsuitable. Since the goal of Kotlin was for use in improving their products, it focuses very strongly on interop with Java code and the Java standard library.

## Why Kotlin?

- 100% interoperable with Java - Kotlin and Java can co-exist in one project. You can continue to use existing libraries in Java.
- Concise - Drastically reduce the amount of boilerplate code you need to write.
- Safe - Avoid entire classes of errors such as null pointer exceptions.
- It's functional - Kotlin uses many concepts from functional programming, such as lambda expressions.

Take a look at [this cheatsheet and quick reference](https://koenig-media.raywenderlich.com/uploads/2018/08/RW_Kotlin_Cheatsheet_1_0.pdf).

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

Recent version of Android Studio now provide Kotlin as the default language when creating new projects.  Just make sure to leave the default programming language as Kotlin!

<img src="https://imgur.com/Wc9sxmY.png">

Your `build.gradle` file will look like this example:

```gradle
apply plugin: 'com.android.application'
apply plugin: 'kotlin-android'

android {
    compileSdkVersion 29
    buildToolsVersion "29.0.3"

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
    implementation 'com.android.support:appcompat-v7:23.4.0'
    implementation "org.jetbrains.kotlin:kotlin-stdlib-jdk7:$kotlin_version"
}
repositories {
    jcenter()
}
```

Note the implementation reference of `org.jetbrains.kotlin:kotlin-stdlib-jdk7`.  For more understanding about the differences between kotlin-stdlib, kotlin-sdklib-jdk7, and kotlin-sdklib-jdk8, see this [link](https://medium.com/@mbonnin/the-different-kotlin-stdlibs-explained-83d7c6bf293) for more information.

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
- [Kotlin Idioms](http://kotlinlang.org/docs/reference/idioms.html)
