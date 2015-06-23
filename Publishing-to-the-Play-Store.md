## Necessary steps

1. Remove Log method calls
2. Digitally sign and export the app
3. Sign up for a Publisher Account on Google Play
4. Upload APK and enter application details


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

To make use of it, click Generate Signed APK under Build menu.

![](http://i.imgur.com/mf91VDf.png)

Select the module that has the app code, if prompted to do so.

Create a new key store if first time. Else, skip. 

The key store holds the private digital key needed to sign the application. Store it in a common location, for further use. 

Provide an Alias for the key in the key store and enter its validity in number of years.

Fill in at least one field of the certificate's organizational details.

![](http://i.imgur.com/rH4kjv1.png)

Pick desired key from key store or create a new one.

![](http://i.imgur.com/h2wGaKW.png)

Finally, pick the folder in which the apk file will be saved. Make sure the Build Type is set to release.

![](http://i.imgur.com/uWvSJ9x.png)

**Note:** Same key store is needed to sign updates for the same app.
**Note:** For different apps a new keystore could be generated every time, but the recommended approach is to generate one key store per developer/organization and then sign all apps with the same key store.
**Note:** Remember your password and the keystore location.

### Become a Publisher

[Here](http://developer.android.com/distribute/googleplay/start.html) you can find a useful checklist of all the steps required to sign up for a publisher account.

At the same page you can also find instructions on **Setting Up a Google Wallet Merchant Account**, if there's the case of selling apps, providing in app purchases or subscriptions. 
 
1. Go to [Google Play Developer Console](https://play.google.com/apps/publish/) which enables developers to easily publish and distribute their applications.
2. Enter basic information about your developer identity.
3. Read and accept the Developer Distribution Agreement for your country or region.
4. Pay the $25 USD registration fee.
5. Confirm verification by e-mail.

### Developer console

This is the place to manage the (un)published apps. There is the possibility to get everything ready in a draft status before actually publishing the application.

[This](http://developer.android.com/distribute/tools/launch-checklist.html) is another very important checklist to help understand the publishing process and determine which steps were left out.

### Automating Play Store publishing

See [[Automating Publishing to the Play Store]] for an advanced tutorial about streamlining the process for pushing your updates.

## References

* <http://developer.android.com/tools/publishing/publishing_overview.html>
* <http://developer.android.com/distribute/googleplay/start.html>
* <http://developer.android.com/distribute/tools/launch-checklist.html>