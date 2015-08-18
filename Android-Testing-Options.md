## Overview

Automated Testing is an important topic that helps us ensure quality when building Android apps. There are many different testing tools and frameworks we can use while developing Android apps. This guide will take a look at some of the more popular approaches available.

### Unit Testing

Unit testing is about testing a particular component (i.e activity) in isolation of other components.

 * [[Robolectric|Unit-Testing-with-Robolectric]] - Popular Android unit test framework that allows faster test execution by running tests on the JVM (no device or emulator needed).
 * [JUnit](http://junit.org/) - Popular Java unit test framework. Most of the Android test frameworks are built on top of JUnit.

### Instrumentation Testing
 * [Espresso](https://code.google.com/p/android-test-kit/wiki/Espresso) - Extensible Android UI Test Framework provided by Google that handles test synchronization very well.
 * [UIAutomator](https://developer.android.com/training/testing/ui-testing/uiautomator-testing.html) - Android UI Test Framework provided by Google for testing across multiple apps at the same time.
 * [Google Android Testing](http://developer.android.com/tools/testing/testing_android.html) - This is the testing framework included as part of the platform (even Google doesn't seem to use this).
 * [[Robotium|UI-Testing-with-Robotium]] - Third party Android UI Test Framework ([comparison with Espresso](http://stackoverflow.com/a/20487527/5154829)).
 * [Selendroid](http://selendroid.io/) - Selenium for Android.

### Validation Frameworks
 * [Assertj-Android](http://square.github.io/assertj-android/) - An extension of [AssertJ](http://joel-costigliola.github.io/assertj/) (fluent assertions for Java) extended for Android (formerly known as FEST Android).
 * [MoreAsserts](http://developer.android.com/reference/android/test/MoreAsserts.html) - Extension of JUnit assertions to add assertions for sets, lists, etc.
 * [ViewAsserts](http://developer.android.com/reference/android/test/ViewAsserts.html) - Platform provided helper methods to validate view layout.

### Mocking Frameworks

 * [Mockito](http://mockito.org/) - Popular mocking framework for Java unit testing. 
 * [EasyMock](http://easymock.org/) - Mocking framework that also has record / replay support. 

### Other

 * [MonkeyRunner](http://developer.android.com/tools/help/monkeyrunner_concepts.html) - API Toolkit that allows controlling the Android device from Python code.
 * [Monkey](http://developer.android.com/tools/help/monkey.html) - Program that runs on the device and does random clicks, typing text, etc.
 * [Cloud Test Lab](https://developers.google.com/cloud-test-lab/?hl=en) - Large bank of devices that you can submit your tests too and have them run across a large range of different devices.
 * [Spoon](http://square.github.io/spoon/) - Run your tests across multiple devices at the same time and see aggregated reports.
