### Overview

Parse provides a cloud-based backend service to build data-driven mobile apps quickly.  Facebook, which acquired the company more than 3 years ago, announced that the service would be shutting down on **January 28, 2017**.   An [open source version](https://github.com/ParsePlatform/parse-server) enables developers to continue using their apps was published, along with a [migration guide](https://parse.com/docs/server/guide#migrating).

While there are many
[alternate options to Parse](https://github.com/relatedcode/ParseAlternatives), most of them lack either the functionality, documentation, or sample code to enable quick prototyping.  For this reason, the open source Parse version is a good option to use with minimal deployment/configuration needed.

#### Differences with Open Source Parse

You can review this [Wiki](https://github.com/ParsePlatform/parse-server/wiki) to understand the current development progress of this app.  There are a few notable differences in the open source version:

* **Authentication**: By default, only an application ID is needed to authenticate with open source Parse.  The [base configuration](https://github.com/ParsePlatform/parse-server-example/blob/master/index.js#L13-L18) that comes with the one-click deploy options does not require authenticating with any other types of keys.   Therefore, specifying client keys on Android or iOS is not needed.

* **Push notifications**: Because of the implicit [security issues](https://github.com/ParsePlatform/parse-server/issues/396#issuecomment-183792657) with allowing push notifications to be sent through Android or iOS directly to other devices, this feature is disabled.  Normally in Parse.com you can toggle an option to override this security restriction.  For open source Parse, you must implement pre-defined code written in JavaScript that can be called by the clients to execute, otherwise known as [Parse Cloud]( http://blog.parse.com/announcements/pushing-from-the-javascript-sdk-and-cloud-code/).

* **Single app aware**: The current version only supports single app instances.  There is ongoing work to make this version multi-app aware.  However, if you intend to run many different apps with different datastores, you currently would need to instantiate separate instances.

* **File upload limitations**: The backend for open source is backed by MongoDB, and the default storage layer relies on Mongo's GridFS layer.  The current limit is set for 20 MB but if you depend on storing large files, you should really configure the server to use Amazon's Simple Storage Service (S3).

Many of the options need to be configured by tweaking your own configuration.  You may wish to fork the [code](https://github.com/ParsePlatform/parse-server-example/) that helps instantiate a Parse server and change them based on your own needs.  The guide below includes instructions on [[how to add push notifications|Configuring-a-Parse-Server#enabling push notifications]] and [[storing files with Parse|Configuring-a-Parse-Server#storing files with parse]].

### Setting a new Parse Server

The steps described this guide walk through most of the process of setting an open source version with Parse.  There are obviously many other hosting options, but the one-click deploys made available with Heroku as discussed in [this guide](https://github.com/ParsePlatform/parse-server-example) are the simplest.   In both cases, you are likely to need a credit card attached to your account to activate.

#### Signing up with Heroku

Use Heroku if you have little or no experience with setting up web sites. Heroku allows you to manage changes to deploy easily by specifying a GitHub repository to use.  In addition, it comes with a UI data viewer from mLab.  

1. Sign Up / Sign In at [Heroku](https://www.heroku.com)

2. Click on the button below to create a Parse App

      <a href="https://dashboard.heroku.com/new?button-url=https%3A%2F%2Fgithub.com%2FParsePlatform%2Fparse-server-example&template=https%3A%2F%2Fgithub.com%2FParsePlatform%2Fparse-server-example"><img src="https://camo.githubusercontent.com/c0824806f5221ebb7d25e559568582dd39dd1170/68747470733a2f2f7777772e6865726f6b7563646e2e636f6d2f6465706c6f792f627574746f6e2e706e67" alt="Deploy" data-canonical-src="https://www.herokucdn.com/deploy/button.png" style="max-width:100%;"></a>

2. Make sure to enter an App Name.  Scroll to the bottom of the page.

      <img src="http://imgur.com/0JcJrn5.png">

3. Make sure to change the config values.
      <img src="http://imgur.com/vQV0X0S.png"/>
      * Leave `PARSE_MOUNT` to be `/parse`.  It does not need to be changed.
      * Set `APP_ID` for the app identifier.  If you do not set one, the default is set as `myAppId`.  You will need this info for the Client SDK setup.
      * Set `MASTER_KEY` to be the master key used to read/write all data.  **Note**: in hosted Parse, client keys are not used by default.
      * Set `SERVER_URL` to be `http://yourappname.herokuapp.com/parse`.  Assuming you have left `PARSE_MOUNT` to be /parse, this will enable the use of Parse Cloud to work correctly.
      * If you intend to use Parse's Facebook authentication, set `FACEBOOK_APP_ID` to be the [FB application ID](https://developers.facebook.com/apps).
      * If you intend to setup push notifications, there are additional environment variables such as `GCM_SENDER_KEY` and `GCM_API_KEY` that will need to be configured.  See [[this section|Configuring-a-Parse-Server#enabling-push-notifications]] for the required steps.

4. Deploy the Heroku app.  The app should be hosted at `https://<app name>.herokuapp.com`.

If you ever need to change these values later, you can go to (`https://dashboard.heroku.com/apps/<app name>/settings`). Check out [this guide](https://devcenter.heroku.com/articles/deploying-a-parse-server-to-heroku) for a more detailed set of steps for deploying Parse to Heroku.

Now, we can [[test our deployment|Configuring-a-Parse-Server#testing-deployment]] to verify that the Parse deployment is working as expected!

### Testing Deployment

After deployment, try to connect to the site.  You should see `I dream of being a web site.` if the site loaded correctly.   If you try to connect to the `/parse` endpoint, you should see `{error: "unauthorized"}`.  If both tests pass, the basic configuration is successful.

Next, make sure you can create Parse objects.  You do not need a client Key to write new data:

```bash
curl -X POST -H "X-Parse-Application-Id: myAppId" \
-H "Content-Type: application/json" \
-d '{"score":1337,"playerName":"Sean Plott","cheatMode":false}' \
https://yourappname.herokuapp.com/parse/classes/GameScore
```

Be sure to **replace the values** for `myAppId` and the server URL. If you see `Cannot POST` error then be sure both the `X-Parse-Application-Id` and the URL are correct for your application. To read data back, you will need to specify your master key as well:

```bash
curl -X GET -H "X-Parse-Application-Id: myAppId" -H "X-Parse-Master-Key: abc"  \
https://yourappname.herokuapp.com/parse/classes/GameScore
```

Be sure to **replace the values** for `myAppId` and the server URL. If these commands work as expected, then your Parse instance is now setup and ready to be used!

### Browsing Parse Data

There are several options that allow you to view the data.  First, you can use the `mLab` viewer to examine the store data.  Second, you can setup the open source verson of the Parse Dashboard, which gives you a similar UI used in hosted Parse.  Finally, you can use Robomongo.

#### mLab

The hosted Parse instance deployed uses [mLab](https://mlab.com/) (previously called MongoLab) to store all of your data. mLab is a hosted version of [MongoDB](https://www.mongodb.org/) which is a document-store which uses JSON to store your data.

If you are using Heroku, you can verify whether the objects were created by clicking on the MongoDB instance in the Heroku panel:

<img src="http://imgur.com/bbj2e9N.png"/>

<img src="http://imgur.com/snPqYkz.png"/>

#### Parse Dashboard

You can also install Parse's open source dashboard locally. Download [NodeJS v4.3](https://nodejs.org/en/download/) or higher.  Make sure you have at least Parse server v2.1.3 or higher (later versions include a `/parse/serverInfo` that is needed).

```bash
npm install -g parse-dashboard
parse-dashboard --appId myAppId --masterKey myMasterKey --serverURL "https://yourapp.herokuapp.com/parse"
```

Connect to your dashboard at `http://localhost:4040/apps`. Assuming you have specified the correct application ID, master Key, and server URL, as well as installed a Parse open source version v2.1.3 or higher, you should see the app appear correctly:

<img src="http://imgur.com/Z0Rz5Xs.png"/>

#### Robomongo

You can also setup [Robomongo](https://robomongo.org/download) to connect to your remote mongo database hosted on Heroku to get a better data browser and dashboard for your app.

<a href="https://robomongo.org/download"><img src="http://i.imgur.com/9Qtt6Xs.png" width="450" /></a>

To access mLab databases using Robomongo, be sure to go the MongoDB instance in the Heroku panel as shown above. Look for the following URL: `mongodb://<dbuser>:<dbpassword>@ds017212.mlab.com:11218/heroku_2flx41aa`. Use that to identify the login credentials:

```
address: ds017212.mlab.com
port: 11218
db: heroku_2flx41aa
user: dbuser
password: dbpassword
```

Using that cross-platform app to easily access and modify the data for your Parse MongoDB data.

### Enabling Client SDK integration

Make sure you have the latest Parse-Android SDK in your `app/build.gradle` file.  It should be at least `1.13.1`:

```gradle
dependencies {
    compile 'com.parse:parse-android:1.13.1'
    compile 'com.parse:parseinterceptors:0.0.2' // for logging API calls to LogCat
    compile 'com.parse.bolts:bolts-android:1.+'
}
```

If you are migrating from a previous Parse configuration, make sure to delete the `CLIENT_KEY` reference in your `AndroidManifest.xml` file.  You can in fact delete both lines since we will be using a custom config builder in the next step.

```xml
 <!--- these lines should be deleted in lieu of our custom Parse builder -->
        <meta-data
            android:name="com.parse.APPLICATION_ID"
            android:value="myAppId" />
        <meta-data
            android:name="com.parse.CLIENT_KEY"
            android:value="clientKey" />
 <!--- end of delete --->
```

Modify your `Parse.initialize()` command to point to this newly created server.  You must be on the latest Parse Android SDK to have these options.

```java
public class ParseApplication extends Application {
    @Override
    public void onCreate() {
        super.onCreate();

        // set applicationId, and server server based on the values in the Heroku settings.
        // clientKey is not needed unless explicitly configured
        // any network interceptors must be added with the Configuration Builder given this syntax
        Parse.initialize(new Parse.Configuration.Builder(this)
                .applicationId("myAppId") // should correspond to APP_ID env variable
                .clientKey(null)  // set explicitly unless clientKey is explicitly configured on Parse server
                .addNetworkInterceptor(new ParseLogInterceptor())
                .server("https://parse-testing-port.herokuapp.com/parse/").build());
    }
}
```

The `/parse/` path needs to match the `PARSE_MOUNT` environment variable, which is set to this value by default.

### Troubleshooting Parse Server

* If you see `Application Error` or `An error occurred in the application and your page could not be served. Please try again in a few moments.`, double-check that you set a `MASTER_KEY` in the environment settings for that app.

  <img src="http://imgur.com/uMYwPmS.png">

* If you are using Heroku, download the Heroku Toolbelt app [here](https://toolbelt.heroku.com/) to help view system logs.

  <img src="http://imgur.com/Ch0mZOK.png"/>

  First, you must login with your Heroku login and password:

  ```bash
  heroku login
  ```

   You can then view the system logs by specifying the app name:
   ```bash
   heroku logs --app <app name>
   ```

   The logs should show the response from any types of network requests made to the site.  Check the `status` code.
   ```
   2016-02-07T08:28:14.292475+00:00 heroku[router]: at=info method=POST path="/parse/classes/Message" host=parse-testing-port.herokuapp.com request_id=804c2533-ac56-4107-ad05-962d287537e9 fwd="101.12.34.12" dyno=web.1 connect=1ms service=2ms status=404 bytes=179
   ```

* If you are seeing `Master key is invalid, you should only use master key to send push`, chances are you are trying to send Push notifications without enable client push.  On Parse.com you can simply enable a toggle switch but for hosted parse there is an outstanding [issue](https://github.com/ParsePlatform/parse-server/issues/396) that must be resolved to start supporting it.

* You can also use Facebook's [Stetho](http://facebook.github.io/stetho/) interceptor to watch network logs with Chrome:

    ```gradle
    dependencies {
      compile 'com.facebook.stetho:stetho:1.3.0'
    }
    ```

  And add this network interceptor as well:

  ```java
     Stetho.initializeWithDefaults(this); // init Stetho before Parse

     Parse.initialize(new Parse.Configuration.Builder(this)
       .addNetworkInterceptor(new ParseStethoInterceptor())
  ```

* If you wish to troubleshoot your Parse JavaScript code is to run the Parse server locally (see [instructions](https://github.com/ParsePlatform/parse-server-example#for-local-development)).  You should also install node-inspector for Node.js, which allows you to use Chrome or Safari to step through the code yourself:

   ```bash
   npm install node-inspector
   node --debug index.js
   node_modules/.bin/node-inspector
   ```

   Open up http://127.0.0.1:8080/?port=5858 locally.   You can use the Chrome debugging tools to set breakpoints in the JavaScript code.

   Point your Android client to this server::

   ```java
   Parse.initialize(new Parse.Configuration.Builder(this)
        .applicationId("myAppId")
        .server("http://192.168.3.116:1337/parse/")  // lookup your IP address of your machine
   ```

### Enabling Push Notifications

**Note**: Support for push notifications is now available with the open source Parse server.   However, unlike Parse's own service, **you cannot implement this type of code** on the actual client:

```java
// Note: This does NOT work with Parse Server at this time
ParsePush push = new ParsePush();
push.setChannel("mychannel");
push.setMessage("this is my message");
push.sendInBackground();
```

You will likely see this error in the API response:

```json
{
    "code": 115,
    "error": "Master key is invalid, you should only use master key to send push"
}
```

Instead, you need to write your own server-side Parse code and have the client invoke it.  Fortunately, the process is fairly straightforward:

1. Fork this [repo](https://github.com/codepath/parse-server-example).  This repo is similar to the package that you used for your one-click deploy.  This repo has some additional environment variables configurations added that help facilitate sending push notifications (i.e. see `GCM_SENDER_ID`, and `GCM_API_KEY` in [index.js](https://github.com/codepath/parse-server-example/blob/master/index.js#L37-L40))
2. Make sure to confirm the `SERVER_URL` environment variable is set to the URL and Parse mount location (i.e. `http://yourappname.herokuapp.com/parse`).
3. Verify that `cloud/main.js` is the default value of `CLOUD_CODE_MAIN` environment variable.  
4. Modify [cloud/main.js](https://github.com/codepath/parse-server-example/blob/master/cloud/main.js) yourself to add custom code to send Push notifications.  See [these examples](https://github.com/ParsePlatform/parse-server/issues/401#issuecomment-183767065) for other ways of sending too.  
      * If you use Parse's default [ParsePushBroadcastReceiver](https://github.com/ParsePlatform/Parse-SDK-Android/blob/master/Parse/src/main/java/com/parse/ParsePushBroadcastReceiver.java#L155-L160), using either `alert` or `title` as a key/value pair will trigger a notification message. See [this section](https://parse.com/docs/android/guide#push-notifications-receiving-pushes) of the Parse documentation.
      * You can also create your own custom receiver as shown in [this example](https://github.com/codepath/ParsePushNotificationExample/blob/master/app/src/main/java/com/test/MyCustomReceiver.java).
5. Redeploy the code.  If you are using Heroku, you need to connect your own forked repository and redeploy.  

     <img src="http://i.imgur.com/OmxXc6s.png"/>

6. Enable Google Cloud Messaging for your Android client (see [[instructions|Configuring-a-Parse-Server#gcm-setup]] below).

7. Assuming the function is named `pushChannelTest`, modify your Android code to invoke this function by using the `callFunctionInBackground()` call.  Any parameters should be passed as a `HashMap`:

     ```java

     HashMap<String, String> test = new HashMap<>();
     test.put("channel", "testing");

     ParseCloud.callFunctionInBackground("pushChannelTest", test);
     ```
8. Make sure to register the GCM token to the server:

     ```java
     Parse.initialize(...);

     // Need to register GCM token
     ParseInstallation.getCurrentInstallation().saveInBackground();
     ```

#### Troubleshooting Push Notifications

* If you are using Facebook's Stetho library with your Android client, you can see the LogCat statements and verify that GCM tokens are being registered by API calls to the `/parse/classes/_Installation` endpoint:

```
: Url : http://192.168.3.116:1337/parse/classes/_Installation
```

You should be able to se the `deviceToken`, `installationId`, and `appName` registered:

```
03-02 03:17:27.859 9362-9596/com.test I/ParseLogInterceptor:
Body : {
   "pushType": "gcm",
   "localeIdentifier": "en-US",
   "deviceToken": XXX,
   "appVersion": "1.0",
   "deviceType": "android",
   "appIdentifier": "com.test",
   "installationId": "XXXX",
   "parseVersion": "1.13.1",
   "appName": "PushNotificationDemo",
   "timeZone": "America\/New_York"
}
```

* Make sure you are on latest open source Parse version: [![npm version](https://img.shields.io/npm/v/parse-server.svg?style=flat)](https://www.npmjs.com/package/parse-server)  You will want to verify what version is set in your `package.json` file (i.e. https://github.com/ParsePlatform/parse-server-example/blob/master/package.json#L15).  Make sure to update this file and redeploy.

* If GCM is fully setup, your app if properly configured should register itself with your Parse server.  Check your `_Installation` table to verify that the entries were being saved. Clear your app cache or uninstall the app if an entry in the  `_Installation` table hasn't been added.

* Inside your `AndroidManifest.xml` definition, make sure your `gcm_sender_id` is prefixed with `id:` (i.e. `id:123456`).  Parse needs to begin with an `id:` to work correctly.

* You can use this curl command with your application key and master key to send a push to all Android devices:

```bash
 curl -X POST -H "X-Parse-Application-Id: myAppId" -H "X-Parse-Master-Key: myMasterKey" \
-H "Content-Type: application/json" \
-d '{"where": {"deviceType": "android"}, "data": {"action": "com.example.UPDATE_STATUS", "newsItem": "Man bites dog", "name": "Vaughn", "alert": "Ricky Vaughn was injured during the game last night!"}}' \
https://parse-testing-port.herokuapp.com/parse/push/
```

* Use `heroku logs --app <app_name>` to see what is happening on the server side:

```bash
2016-03-03T09:26:50.032609+00:00 app[web.1]: #### PUSH OK
```

If you see `Can not find sender for push type android`, it means you forgot to set the environment variables `GCM_SENDER_ID` and `GCM_API_KEY`.

* Make sure you have **not** included `com.google.android.gms:play-services-gcm:8.4.0` in your Gradle configuration.  Parse's Android SDK library already includes code to deal with the GCM registration.

* Verify that you have all the permissions for GCM setup in your `AndroidManifest.xml` file and that you have the correct receivers configured.

#### GCM Setup

**Note**: The Parse Android SDK currently does not support Firebase Cloud Messaging, which is the new name for Google Cloud Messaging (see [discussion issue](https://github.com/ParsePlatform/Parse-SDK-Android/pull/452)).  You should follow the steps below to ensure that the older version of GCM is used.  In particular, the permissions in the manifest file that you need to set are checked by the Parse Android SDK, so you should keep the old permissions as shown below.

1. Make sure you have Google Play installed on the emulator or device, since push notifications via [Google Cloud Messaging](Google-Cloud-Messaging) (GCM) will only work for devices and emulators that have Google Play installed.  

2. Obtain a Sender ID and API Key.
      * Follow only step 1 of [this guide](https://github.com/codepath/android_guides/wiki/Google-Cloud-Messaging/b7ab0d3329898f147b2fe7a32c731f9ce251893c#step-1-register-with-google-developers-console) to obtain the Sender ID (equivalent to the Project Number) and API Key.  You do not need to follow the other steps because Parse provides much of code to handle GCM registration for you.

3. Set `GCM_SENDER_KEY` and `GCM_API_KEY` environment variables to correspond to the Sender ID and API Key in the previous step.  **Note**: this step only works if you have setup your Parse server to enable push support.  See the [CodePath fork](https://github.com/codepath/parse-server-example/blob/master/index.js#L15-L26) as an example about how to initialize the Parse server.

      <img src="http://imgur.com/KgU2S2y.png"/>

4. Add a `meta-data` with the Sender ID in your AndroidManifest.xml.  Make sure the `id:` is used as the prefix (Android treats any metadata value that has a string of digits as an integer so Parse prefixes this value).  If you forget this step, Parse will register with its own Sender ID but you will see `SenderID mismatch` errors when trying to issue push notifications.

     ```xml
        <meta-data
              android:name="com.parse.push.gcm_sender_id"
              android:value="id:SENDER_ID_HERE"/>
     ```

5. Add the necessary permissions:
    * android.permission.INTERNET
    * android.permission.ACCESS_NETWORK_STATE
    * android.permission.WAKE_LOCK
    * android.permission.GET_ACCOUNTS
    * android.permission.VIBRATE (optional)
    * com.google.android.c2dm.permission.RECEIVE
    * context.getPackageName() + .permission.C2D_MESSAGE (i.e. `com.yourpackagename.permission.C2D_MESSAGE`)

    ```xml
        <uses-permission android:name="android.permission.INTERNET" />
        <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
        <uses-permission android:name="android.permission.WAKE_LOCK" />
        <uses-permission android:name="android.permission.GET_ACCOUNTS" />
        <uses-permission android:name="android.permission.VIBRATE" />
        <uses-permission android:name="com.google.android.c2dm.permission.RECEIVE" />
        <!--
          IMPORTANT: Change "com.codepath.parseportpush.permission.C2D_MESSAGE" in the lines below
          to match your app's package name + ".permission.C2D_MESSAGE".
        -->
        <permission android:protectionLevel="signature"
            android:name="com.codepath.parseportpush.permission.C2D_MESSAGE" />
        <uses-permission android:name="com.codepath.parseportpush.permission.C2D_MESSAGE" />
    ```

6. Declare a service, Parse-specific broadcast receiver, and a GCM receiver within the `application` tag of the `AndroidManifest.xml` file:
      * Add a `PushService` service.
      * The Parse receiver should can handle `ACTION_PUSH_RECEIVE`, `ACTION_PUSH_OPEN`, and `ACTION_PUSH_DELETE` events.
      * The GCM broadcast receiver should handle `com.google.android.c2dm.intent.RECEIVE` and `com.google.android.c2dm.intent.REGISTRATION` events.   It also should include a `category` tag that directs GCM registration responses only for your application package name.
      ```xml
      <application>
          <service android:name="com.parse.PushService" />
          <receiver android:name="com.parse.ParsePushBroadcastReceiver"
                    android:exported="false">
                <intent-filter>
                      <action android:name="com.parse.push.intent.RECEIVE" />
                      <action android:name="com.parse.push.intent.DELETE" />
                      <action android:name="com.parse.push.intent.OPEN" />
                </intent-filter>
           </receiver>
            <receiver android:name="com.parse.GcmBroadcastReceiver"
                  android:permission="com.google.android.c2dm.permission.SEND">
                  <intent-filter>
                      <action android:name="com.google.android.c2dm.intent.RECEIVE" />
                      <action android:name="com.google.android.c2dm.intent.REGISTRATION" />

                      <!--
                        IMPORTANT: Change "com.codepath.parseportpush" to match your app's package name.
                      -->
                      <category android:name="com.codepath.parseportpush" />
                  </intent-filter>
            </receiver>
      </application>
      ```

### Storing Files with Parse

You can continue to save large files to your Parse backend using `ParseFile` without any major changes:

```java
byte[] data = "Working at Parse is great!".getBytes();
ParseFile file = new ParseFile("resume.txt", data);
file.saveInBackground();
```

By default, the open source version of Parse relies on the MongoDB [GridStore](https://mongodb.github.io/node-mongodb-native/api-generated/gridstore.html) adapter to store large files.  There is an alternate option to leverage Amazon's Simple Storage Service (S3) but still should be considered experimental.

#### Using Amazon S3

If you wish to store the files in an Amazon S3 bucket, you will need to make sure to setup your Parse server to use the S3 adapter instead of the default GridStore adapter.  See [this Wiki](https://github.com/ParsePlatform/parse-server/wiki/Configuring-File-Adapters#configuring-s3adapter) for how to configure your setup.  The steps basically include:

1. Modify the Parse server to use the S3 adapter.  See [these changes](https://github.com/codepath/parse-server-example/blob/master/index.js#L20-L29) as an example.
2. Create an Amazon S3 bucket.
3. Create an Amazon user with access to this S3 bucket.  
4. Generate an authorized key/secret pair to write to this bucket.
5. Set the environment variables:
      * Set `S3_ENABLE` to be 1.
      * Set `AWS_BUCKET_NAME` to be the AWS bucket name.
      * Set `AWS_ACCESS_KEY` and ` AWS_SECRET_ACCESS_KEY` to be the user that has access to read/write to this S3 bucket.

When testing, try to write a file and use the Amazon S3 console to see if the files were created in the right place.
