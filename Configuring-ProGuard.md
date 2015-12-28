ProGuard is a tool to help minify, obfuscate, and optimize your code.  It is not only especially useful for reducing the overall size of your Android application as well as removing unused classes and methods that contribute towards the intrinsic [64k method limit](http://developer.android.com/tools/building/multidex.html#avoid) of Android applications.  This last aspect means that ProGuard is often recommended to be used both in development and production especially for larger applications.  

ProGuard is normally only enabled for release builds and setup in your `app/build.gradle` config.  You can enable these settings on both your release and debug builds by enabling the `minifyEnabled` option:

```gradle
android {
   buildTypes {
      release {
          minifyEnabled true
          proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
      }
      dev {
          minifyEnabled true
          proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
      }
}
```

The Android SDK comes with ProGuard included as well as default settings file, which are specified as [proguard-android.txt](https://android.googlesource.com/platform/sdk/+/master/files/proguard-android.txt).  It is important to include this file since it explicitly includes configuration settings such as explicitly stating that all View getter and setter methods should not be removed.   The `proguard-rules.pro` file is the file that you will use to configure.

### Development

When using in development/debug testing, you may wish to turn on a few settings that may add to compile time as well as make it harder to troubleshoot.  For instance, to ensure that no code optimizations or obfuscation is done, the following options should be declared in your ProGuard config:

```java
# Workaround for ProGuard not recognizing dontobfuscate
# https://speakerdeck.com/chalup/proguard
-dontobfuscate
-dontoptimize
-optimizations !code/allocation/variable
```

### Third-Party Libraries

One limitation in ProGuard is that dynamically generated classes such as those used in Butterknife, Otto, or Dagger 2 need to be explicitly declared.  In many cases, compilation will not continue until these issues are resolved.  In other cases, if you do not specify it, ProGuard will often strip these methods from being used and your app will often crash at run-time.   

See [this link](https://github.com/krschultz/android-proguard-snippets/tree/master/libraries) for the standard set of library configs.  It's important to check the documentation of each third-party library to see the most updated settings.  You can often try to experiment too by incrementally adding the necessary lines to best understand the impact of each line configuration.

#### ButterKnife

```
-keep class butterknife.** { *; }
-dontwarn butterknife.internal.**
-keep class **$$ViewBinder { *; }

-keepclasseswithmembernames class * {
    @butterknife.* <fields>;
}

-keepclasseswithmembernames class * {
    @butterknife.* <methods>;
}
```

#### Retrofit

```
-dontwarn retrofit.**
-keep class retrofit.** { *; }
-keepattributes Signature
-keepattributes Exceptions
```

#### OkHttp

```
-keepattributes Signature
-keepattributes *Annotation*
-keep class com.squareup.okhttp.** { *; }
-keep interface com.squareup.okhttp.** { *; }
-dontwarn com.squareup.okhttp.**

# Okio
-keep class sun.misc.Unsafe { *; }
-dontwarn java.nio.file.*
-dontwarn org.codehaus.mojo.animal_sniffer.IgnoreJRERequirement
```

#### Gson

```
-keep class sun.misc.Unsafe { *; }
-keep class com.google.gson.stream.** { *; }
```

#### Parcels

```
keep class * implements android.os.Parcelable {
  public static final android.os.Parcelable$Creator *;
}

-keep class org.parceler.Parceler$$Parcels
```

### References

* <https://github.com/krschultz/android-proguard-snippets/tree/master/libraries/>
* <http://developer.android.com/tools/help/proguard.html/>