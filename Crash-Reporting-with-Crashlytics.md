# Overview

Crashlytics is a part of Fabric.io and gives mobile app developers insight into their apps’ performance, so you can pinpoint and fix issues quickly and easily. It is **free to use for everybody.**

# How to start, step by step guide

1. Register on [Fabric.io](https://fabric.io)
2. Download Android Studio plugin **Fabric for Android**
3. After restart Android Studio, open your project and click Fabric button ![](https://i.imgur.com/UobXlWS.png) to login into your account
4. Next click on **+New app** and select your organization which was given during registration
5. Select Crashlytics from available kits ![](https://i.imgur.com/K5g8HpK.png)
6. On next step select **install** button to run integration wizard ![](https://i.imgur.com/7hGWrj9.png)
7. Wizard provide all required code for you. You can either copy it yourself or let wizard to do it for you 
![](https://i.imgur.com/XBGP5XD.png)
8. After finish you have to synchronize gradle and run your project to verify configuration 
![](https://i.imgur.com/f1UAOSs.png)
9. That's it, you successfully configured Crashlytics. Each of uncaught exceptions will be send to Fabric
10. To test crash reporting let's put this code into your onCreate method
```java
    throw new RuntimeException("This is a crash");
```
11. After few second you can see crash in Fabric dashboard
![](https://i.imgur.com/MQbyJy1.png)


# How to manually configure Crashlytics 
> build.gradle
 ```javascript
buildscript {
  repositories {
    maven { url 'https://maven.fabric.io/public' }
  }

  dependencies {
  // We recommend changing it to the latest version from our changelog: 
  // https://docs.fabric.io/android/changelog.html#fabric-gradle-plugin
    classpath 'io.fabric.tools:gradle:1.+'
  }
}
apply plugin: 'com.android.application'
// Put Fabric plugin after Android plugin
apply plugin: 'io.fabric'

repositories {
  maven { url 'https://maven.fabric.io/public' }
}
dependencies {
  ...
  compile('com.crashlytics.sdk.android:crashlytics:2.6.8@aar') {
    transitive = true;
    }
}

```
> AndroidManifest.xml
```xml=
<application
...
  <meta-data
      android:name="io.fabric.ApiKey"
      android:value="YOUR_API_KEY" />
</application>
<uses-permission android:name="android.permission.INTERNET" />
```

> MainActivity or CustomApplication class
```java
import com.crashlytics.android.Crashlytics;
import io.fabric.sdk.android.Fabric;

...

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        Fabric.with(this, new Crashlytics());
```


# Enhance Crash Reports
Crashlytics provides 4 logging mechanisms right out of the box: logging, custom keys, user information, and caught exceptions

## Caught Exceptions
___
For any individual app session, only the most recent 8 logged exceptions are stored.
___
>All logged exceptions will appear as “non-fatal” issues in the Fabric dashboard. 
>Crashlytics processes exceptions on a dedicated background thread, so the performance impact to your app is minimal. To reduce your users’ network traffic, Crashlytics batches logged exceptions together and sends them the next time the app launches.

```java
try {
  myMethodThatThrows();
} catch (Exception e) {
  Crashlytics.logException(e);
  // handle your exception here!
}
```

## Custom Logging
Logged messages are associated with your crash data and are visible in the Crashlytics dashboard if you look at the specific crash itself.

```java
Crashlytics.log("Higgs-Boson detected! Bailing out...");
```

## Custom Keys
___
Crashlytics supports a maximum of 64 key/value pairs. Once you reach this threshold, additional values are not saved. Each key/value pair can be up to 1 KB in size.
___
Crashlytics allows you to associate arbitrary key/value pairs with your crash reports, which are viewable right from the Crashlytics dashboard. Setting keys are as easy as calling: Crashlytics.setString(key, value) or one of the related methods. Options are:
```java
void setBool(String key, boolean value);
void setDouble(String key, double value);
void setFloat(String key, float value);
void setInt(String key, int value);
```
## User Information
You can use Crashlytics.setUserIdentifier to provide an ID number, token, or hashed value that uniquely identifies the end-user of your application without disclosing or transmitting any of their personal information. This value is displayed right in the Fabric dashboard.

```java
void Crashlytics.setUserIdentifier(String identifier);
void Crashlytics.setUserName(String name);
void Crashlytics.setUserEmail(String email);
```

# Disable Crashlytics for Debug Builds
If you don’t need Crashlytics crash reporting or beta distribution for debug builds, you can safely speed up your debug-builds by disabling the plugin entirely with these two steps:

First, add this to your app’s build.gradle:
```javascript
android {
    buildTypes {
        debug {
          // Disable fabric build ID generation for debug builds
          ext.enableCrashlytics = false
          ...
```
Next, disable the Crashlytics kit at runtime. Otherwise, the Crashlytics kit will throw the following error:


> com.crashlytics.android.core.CrashlyticsMissingDependencyException:
This app relies on Crashlytics. Please sign up for access at https://fabric.io/sign_up`

You can disable the kit at runtime for debug builds only with the following code:

```java
// Set up Crashlytics, disabled for debug builds
Crashlytics crashlyticsKit = new Crashlytics.Builder()
    .core(new CrashlyticsCore.Builder().disabled(BuildConfig.DEBUG).build())
    .build();

// Initialize Fabric with the debug-disabled crashlytics.
Fabric.with(this, crashlyticsKit);
```

## Attribution

This guide was originally authored by [Mateusz Utkała](https://github.com/DonMat) for the CodePath guides.

## References

* <https://fabric.io/kits/android/crashlytics/install>