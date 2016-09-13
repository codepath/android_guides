## Overview

Automated Testing is an important topic that helps us ensure quality when building Android apps. There are many different testing tools and frameworks we can use while developing Android apps. This guide will take a look at some of the more popular approaches available. For someone first starting with testing, we recommend looking at [[Robolectric|Unit-Testing-with-Robolectric]] for unit testing, [Espresso](https://google.github.io/android-testing-support-library/docs/espresso/index.html) for UI testing, [Assertj-Android](http://square.github.io/assertj-android/) for better validation support, and [Mockito](http://mockito.org/) for mocking.

Helpful external testing resources include:

 * [Google Android Testing CodeLab](https://codelabs.developers.google.com/codelabs/android-testing/index.html)
 * [Google Testing Sample Code](https://github.com/googlesamples/android-testing)
 * [Espresso Samples](https://github.com/chiuki/espresso-samples)
 * [Chiu-ki Talk on Advanced Espresso](https://realm.io/news/chiu-ki-chan-advanced-android-espresso-testing/)

Review the sections below for further resources and guides.

### Unit Testing

Unit testing is about testing a particular component (i.e. an activity or model object) in isolation of other components. In Android, unit tests do not require a device or emulator. These tests are typically placed in the `app/src/test/java` folder.

 * [[Robolectric|Unit-Testing-with-Robolectric]] - Popular Android unit test framework that allows faster test execution by running tests on the JVM (no device or emulator needed).
 * [JUnit](http://junit.org/) - Popular Java unit test framework. Most of the Android test frameworks are built on top of JUnit.

### Instrumentation Testing

Android Instrumentation is a set of "hooks" into the Android system that allow you to control the lifecycle of Android components (i.e. drive the activity lifecycle yourself instead of having these driven by the system). These tests require an actual device or emulator to run and are typically placed in the `app/src/androidTest/java` folder.
 
 * [[Espresso|UI-Testing-with-Espresso]] - Extensible Android UI Test Framework provided by Google that handles test synchronization very well.
 * [UIAutomator](https://developer.android.com/training/testing/ui-testing/uiautomator-testing.html) - Android UI Test Framework provided by Google for testing across multiple apps at the same time.
 * [Google Android Testing](http://developer.android.com/tools/testing/testing_android.html) - This is the testing framework included as part of the platform.
 * [[Robotium|UI-Testing-with-Robotium]] - Third party Android UI Test Framework ([comparison with Espresso](http://stackoverflow.com/a/20487527/5154829))
 * [Selendroid](http://selendroid.io/) - Selenium for Android

### Richer Validation Support

Simple [JUnit assertions](http://junit.org/apidocs/org/junit/Assert.html) leave a lot to be desired when working on Android. The following libraries and classes help fill this gap. 

 * [Assertj-Android](http://square.github.io/assertj-android/) - An extension of [AssertJ](http://joel-costigliola.github.io/assertj/) (fluent assertions for Java) extended for Android (formerly known as FEST Android).
 * [MoreAsserts](http://developer.android.com/reference/android/test/MoreAsserts.html) - Extension of JUnit assertions to add assertions for sets, lists, etc.
 * [ViewAsserts](http://developer.android.com/reference/android/test/ViewAsserts.html) - Helper methods to validate view layout

### Mocking Frameworks

Mocking out dependencies is a very powerful capability when writing tests. These frameworks provide rich mocking support on Android.

 * [Mockito](http://mockito.org/) - Popular mocking framework for Java
 * [EasyMock](http://easymock.org/) - Mocking framework that has record / replay support 

### Tools

A collection of useful tools.

 * [Cloud Test Lab](https://developers.google.com/cloud-test-lab/?hl=en) - Large bank of devices where you can submit your tests and have them run across different devices.
 * [MonkeyRunner](http://developer.android.com/tools/help/monkeyrunner_concepts.html) - API Toolkit that allows controlling the Android device from Python code.
 * [Monkey](http://developer.android.com/tools/help/monkey.html) - Program that runs on the device and performs random clicks, text input, etc.
 * [Spoon](http://square.github.io/spoon/) - Run your tests across multiple devices at the same time and see aggregated reports / screenshots.
