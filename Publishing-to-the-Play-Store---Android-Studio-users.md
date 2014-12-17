##Necessary steps
1. Remove Log method calls
2. Digitally sign and export the app
3. Sign up for a Publisher Account on Google Play
4. Pay a one-time fee of $25
5. Fill out a form to add a new app to your account
6. Publish the app!

### Disable Log calls
To avoid deleting all the log calls manually, Android Studio provides tools to do so easily. 
By configuring [Proguard](http://developer.android.com/tools/help/proguard.html), statements to strip calls to any of the Log methods can be added.

Firstly, Proguard needs to be enabled in the module's `build.gradle` file. Also, change the standard default proguard settings file to `proguard-android-optimize.txt`, which is better suited for optimizing the apk file.


```bash
buildTypes {
        release {
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }
```

Next, add the following lines to the `proguard-rules.pro` file:
```java
-assumenosideeffects class android.util.Log {
    public static boolean isLoggable(java.lang.String, int);
    public static int v(...);
    public static int i(...);
    public static int w(...);
    public static int d(...);
    public static int e(...);
}
```
This option for Proguard is only applicable when optimizing and will remove all the listed methods.

**Note:** It is possible to publish an app without this step, but it's highly recommended to do it for performance reasons.

**Note:** When logging sensitive information, it is needed to use another method to remove the strings that are actually logged in the log methods.

### Generate release ready apk
Digitally signing and exporting is easier than it sounds, using the wizard built into Android Studio.
