## Overview

Often your app will have secret credentials or API keys that you need to have in your app to function but you'd rather not have easily extracted from your app.  

If you are using dynamically generated secrets, the most effective way to store this information is to use the [Android Keystore API](https://developer.android.com/training/articles/keystore.html).  You should not store them in [[shared preferences|Storing-and-Accessing-SharedPreferences]] without encrypting this data first because they can be extracted when performing a backup of your data.  

The alternative is to disable backups by setting `android:allowBackup` in your `AndroidManifest.xml` file:

```xml
<application ...
    android:allowBackup="true">
</application>
```

You can also specify which files to avoid backups too by reviewing this [doc](https://developer.android.com/guide/topics/data/autobackup.html#Files).

### Storing Fixed Keys

For storing fixed API keys, the following common strategies exist for storing secrets in your source code:

 * [[Hidden in BuildConfigs|Storing-Secret-Keys-in-Android#hidden-in-buildconfigs]]
 * [[Embedded in resource file|Storing-Secret-Keys-in-Android#secrets-in-resource-files]]
 * [[Obfuscating with Proguard|Configuring-ProGuard]]
 * [Disguised or Encrypted Strings](https://developer.android.com/training/articles/keystore.html)
 * Hidden in Native Libraries with NDK
 * Hidden as constants in source code

The simplest approach for storing secrets in to keep them as resource files that are simply not checked into source control.   The approaches below two ways of accomplishing the same goal.

### Hidden in BuildConfigs

First, create a file `apikey.properties` in your root directory with the values for different secret keys:

```
CONSUMER_KEY=XXXXXXXXXXX
CONSUMER_SECRET=XXXXXXX
```

To avoid these keys showing up in your repository, make sure to exclude the file from being checked in by adding to your `.gitignore` file:

```
apikey.properties
```

Next, add this section to read from this file in your `app/build.gradle` file.  You'll also create compile-time options that will be generated from this file by using the `buildConfigField` definition:

```gradle

def apikeyPropertiesFile = rootProject.file("apikey.properties")
def apikeyProperties = new Properties()
apikeyProperties.load(new FileInputStream(apikeyPropertiesFile))
 
android {

  defaultConfig {
     
     // should correspond to key/value pairs inside the file   
    buildConfigField("String", "CONSUMER_KEY", apikeyProperties['CONSUMER_KEY'])
    buildConfigField("String", "CONSUMER_SECRET", apikeyProperties['CONSUMER_SECRET'])
  }
}
```

You can now access these two fields anywhere within your source code with the `BuildConfig` object provided by Gradle:

```java
BuildConfig.CONSUMER_KEY
BuildConfig.CONSUMER_SECRET
```

Now you have access to as many secret values as you need within your app, but will avoid checking in the actual values into your git repository. To read more about this approach, check out [this article](https://medium.com/@geocohn/keeping-your-android-projects-secrets-secret-393b8855765d)

### Secrets in Resource Files

Start by creating a resource file for your secrets called `res/values/secrets.xml` with a string pair per secret value:

```xml
<!-- Inside of `res/values/secrets.xml` -->
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string name="parse_application_id">xxxxxx</string>
    <string name="google_maps_api_key">zzzzzz</string>
</resources>
```

Once these keys are in the file, Android will automatically merge it into your resources, where you can access them exactly as you would your normal strings. You can access the secret values in your Java code with:

```java
// inside of an Activity, `getString` is called directly
String secretValue = getString(R.string.parse_application_id);
// inside of another class (requires a context object to exist)
String secretValue = context.getString(R.string.parse_application_id);
```

If you need your keys in another XML file such as in `AndroidManifest.xml`, you can just use the XML notation for accessing string resources:

```xml
@string/google_maps_api_key
```

Since your secrets are now in an individual file, they're simple to ignore in your source control system (for example, in Git, you would add this to the '.gitignore' file in your repository) by entering this on the command line within your project git repository:

```
echo "**/*/res/values/secrets.xml" > .gitignore
```

**Verification:** To make sure this worked, check the `.gitignore` file within your git repository, and make sure that this line referencing `secrets.xml` exists. Now, go to commit files to Git and make sure that **you do not see the `secrets.xml` file in the staging area**. You do not want to commit this file to Git.

This process is not bulletproof. As resources, they are somewhat more vulnerable to decompilation of your application package, and so they are discoverable if somebody really wants to know them. This solution does, however, prevent your secrets just sitting in plaintext in source control waiting for someone to use, and also has the advantage of being simple to use, leveraging Android's resource management system, and requiring no extra libraries.


However, **none of these strategies will ensure the protection of your keys** and [your secrets aren't safe](https://rammic.github.io/2015/07/28/hiding-secrets-in-android-apps/). The best way to protect secrets is to never reveal them in the code in the first place. Compartmentalizing sensitive information and operations on your own backend server/service should always be your first choice. 

If you do have to consider a hiding scheme, you should do so with the realization that you can only make the reverse engineering process harder and may add significant complication to the development, testing, and maintenance of your app in doing so. Check out [this excellent StackOverflow post](https://stackoverflow.com/a/14572051/313399) for a detailed breakdown of the obfuscation options available.

The simplest and most straightforward approach is outlined below which is to **store your secrets within a resource file**. Keep in mind that most of the other more complex approaches above are at best only marginally more secure.

### Using the Android Keystore API

To understand the Android Keystore API, you must first understand that encrypting secrets requires both public key and symmetric cryptography.  In public key cryptography, data can be encrypted with one key and decrypted with the other key.  In symmetric cryptography, the same key is used to encrypt and decrypt the data.   The Keystore API uses both types of cryptography in order to safeguard secrets.

A public/private key RSA pair is generated, which is stored in the Android device's keystore and protected usually by the device PIN.  An AES-based symmetric key is also generated, which is used to encrypt and decrypt the secrets.  The app needs to store this AES symmetric key to later decode, so it is encrypted by the RSA public key first before persisted.  When the app runs, it gives this encrypted AES key to the Keystore API, which will decode the data using its private RSA key.  In this way, data cannot be decoded without the use of the device keystore.

Read this [Medium blog](https://medium.com/@ericfu/securely-storing-secrets-in-an-android-application-501f030ae5a3) for more information about how to use the Keystore API.   Do not use the [Qlassified Android](https://github.com/Q42/Qlassified-Android) library because it introduces an additional 20K methods to your Android program.  You can use the [Android Vault](https://github.com/BottleRocketStudios/Android-Vault/tree/master/AndroidVault/vault/src/androidTest/java/com/bottlerocketstudios/vault) library, which will also help facilitate the rotation of RSA keys, which usually have an expiration date of 1-5 years.

See also the [official Google sample](https://github.com/googlesamples/android-BasicAndroidKeyStore) for using the Android Keystore.

## Resources

* <https://joshmcarthur.com/2014/02/16/keeping-secrets-in-an-android-application.html>
* <https://rammic.github.io/2015/07/28/hiding-secrets-in-android-apps/>
* <https://medium.com/@ericfu/securely-storing-secrets-in-an-android-application-501f030ae5a3/>