## Overview

[Travis](https://travis-ci.com/) is a continuous integration service that enables you to run tests against your latest Android builds.  You can setup your Android projects to run both unit and integration tests, which can also include launching an emulator.  

### Signing Up

Login to [Travis CI](https://travis-ci.com/) and sign-in with your GitHub credentials.  You will need to grant access to any repositories that will be used.  If you are using a private repository and don't intend for the source code to be shared, you will need to signup for a paid subscription plan.

### Setup

You simply need to create a `.travis.yml` file in the root directory.  The simplest configuration to install the Build Tools and Android SDK 21.   You can launch the [[Gradle wrapper|Getting Started with Gradle]] to build and run emulator tests.

```yaml
language: android
android:
  components:
    - build-tools-22.0.1
    - android-21

script:
   - ./gradlew build connectedCheck
```

See the [docs](http://docs.travis-ci.com/user/languages/android/) here for more information.  By default, all SDK license agreements are accepted but you can also dictate which ones to accept.

#### Design Support Library

If you are using [Travis](https://travis-ci.org/) for automated Android builds, you will need to make sure to include the `extra-android-m2repository` in `travis.yml`:

```yaml
language: android
android:
  components:

     - extra-android-m2repository
```

Otherwise, you are likely to see the following error message:

```
06:27:04.304 [ERROR] [org.gradle.BuildExceptionReporter] Searched in the following locations:
06:27:04.304 [ERROR] [org.gradle.BuildExceptionReporter] file:/usr/local/android-sdk/extras/google/m2repository/com/android/support/support-v4/22.2.0/support-v4-22.2.0.pom 
06:27:04.305 [ERROR] [org.gradle.BuildExceptionReporter] file:/usr/local/android-sdk/extras/google/m2repository/com/android/support/support-v4/22.2.0/support-v4-22.2.0.jar
```

Currently, the [[Design Support Library]] must be downloaded from the [[SDK Manager|Installing Android SDK Tools]], which retrieves the package as a [local Maven repository](https://dl-ssl.google.com/android/repository/addon.xml).  It is currently not publicly available on a public Maven repository.