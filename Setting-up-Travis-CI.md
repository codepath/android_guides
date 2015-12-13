## Overview

[Travis](https://travis-ci.com/) is a continuous integration service that enables you to run tests against your latest Android builds.  You can setup your projects to run both unit and integration tests, which can also include launching an emulator.  

### Signing Up

Login to [Travis CI](https://travis-ci.com/) and sign-in with your GitHub credentials.  You will need to grant access to any repositories that will be used.  If you are using a private repository and don't intend for the source code to be shared, you will need to signup for a paid subscription plan.

### Setup

You simply need to create a `.travis.yml` file in the root directory.  The simplest configuration to install the Build Tools and Android SDK 23.   You can launch the [[Gradle wrapper|Getting Started with Gradle]] to build and run emulator tests.

```yaml
language: android
android:
  components:
    - platform-tools
    - build-tools-23.0.1
    - android-23

script:
   - ./gradlew build connectedCheck
```

See the [docs](http://docs.travis-ci.com/user/languages/android/) here for more information.  By default, all SDK license agreements are accepted but you can also dictate which ones to accept.

#### Design Support Library

If you are intending to use the new [[Design Support Library]], you will need to make sure to include the `extra-android-m2repository` component in `travis.yml`:

```yaml
language: android
android:
  components:

     - extra-android-m2repository
```

Otherwise, you are likely to see the following error message:

```
Searched in the following locations:
  file:/usr/local/android-sdk/extras/google/m2repository/com/android/support/support-v4/22.2.0/support-v4-22.2.0.pom 
  file:/usr/local/android-sdk/extras/google/m2repository/com/android/support/support-v4/22.2.0/support-v4-22.2.0.jar
```

The reason is that the [[Design Support Library]] must be downloaded from the [[SDK Manager|https://developer.android.com/tools/help/sdk-manager.html]], which looks in the [repository manifest](https://dl-ssl.google.com/android/repository/addon.xml) for the `Local Maven repository for Support Libraries` package.  The support library is not yet available on a public Maven repository for licensing reasons.

#### Google Play Services

If you intend to use Google Play Services with Travis, you will also want to add the `extra-google-m2repository` for similar reasons:

```yaml
language: android
android:
  components:
     - extra-google-m2repository
```

### Troubleshooting

If you see an error code 137, chances are that the Travis build has ran out of memory trying to load all your dependencies.

```bash
com.android.ide.common.process.ProcessException: org.gradle.process.internal.ExecException: Process 'command '/usr/lib/jvm/java-7-oracle/bin/java'' finished with non-zero exit value 137
```

If you are using [Google Play Services](https://developers.google.com/android/guides/setup), try to be more selective about the modules you import.  In prior versions before 6.5, you had to include all packages which often caused the 65K method limit to be reached.  If you only wish to use the Google Cloud Messaging package and not Google Fitness or Google Wear, you do not need to import 
`com.google.android.gms:play-services:8.3.0`.  Instead, you can simply specify the libraries explicitly:

```gradle
compile 'com.google.android.gms:play-services-gcm:8.3.0'
```
