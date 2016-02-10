### Overview

Parse provides a cloud-based backend service to build data-driven mobile apps quickly.  Facebook, which acquired the company more than 3 years ago, announced that the service would be shutting down on **January 28, 2017**.   An [open source version](https://github.com/ParsePlatform/parse-server) enables developers to continue using their apps was published, along with a [migration guide](https://parse.com/docs/server/guide#migrating).

While there are many
[alternate options to Parse](https://github.com/relatedcode/ParseAlternatives), most of them lack either the functionality, documentation, or sample code to enable quick prototyping.  For this reason, the open source Parse version is a good option to use with minimal deployment/configuration needed.

You can review this [Wiki](https://github.com/ParsePlatform/parse-server/wiki) to understand the current development progress of this app.  Push notifications, for instance, are currently not implemented but appears to be supported soon.

### Setting a new Parse Server

The steps described [this guide](https://devcenter.heroku.com/articles/deploying-a-parse-server-to-heroku) walk through most of the process of setting an open source version with Parse.  There are obviously many other hosting options, but deploying with Heroku is probably the simplest.  The instructions basically entail the following steps:

1. [Register](https://id.heroku.com/signup/login) for a free Heroku account.
2. Establish a credit card through [Account Settings](https://dashboard.heroku.com/account) in order to add a free MongoDB sandbox instance.
3. Fork a copy of the [Parse server example](https://github.com/ParsePlatform/parse-server-example).
    * Modify `index.js` to add `clientKey` and push notifications support:

        ```javascript
        // For GCM Push support -- see https://github.com/ParsePlatform/parse-server/pull/311/files
        var pushConfig = { android: { senderId: process.env.GCM_SENDER_ID || '',
                                      apiKey: processs.env.GCM_API_KEY || ''}}

        var api = new ParseServer({
         databaseURI: databaseUri || 'mongodb://localhost:27017/dev',
         cloud: process.env.CLOUD_CODE_MAIN || __dirname + '/cloud/main.js',
         appId: process.env.APP_ID || 'myAppId',
         masterKey: process.env.MASTER_KEY || '', //Add your master key here. Keep it secret!
         clientKey: process.env.CLIENT_KEY || 'clientKey',
         push: pushConfig
        });
        ```

    * Push to this branch with this change.
4. Create a new Heroku app, pointing to this repository that was created.
5. Add the MongoDB instance as an Add-On.
6. Setup environment variables in the app settings (`https://dashboard.heroku.com/apps/<app name>/settings`):

    <img src="http://imgur.com/shCWGQX.png"/>
    * Set `MASTER_KEY` to be the master key used to read data.  Otherwise, the server will not load properly.
    * Set `APP_ID` for the app identifier.  If you do not set one, the default is set as `myAppId` (see [the source code](https://github.com/ParsePlatform/parse-server-example/blob/master/index.js#L14-L18)).
    * Verify the `MONGOLAB_URI` has been added.  It should be there if the MongoDB add-on was added.
    * Set `CLIENT_KEY` to be your client key.  You will use this info later for the Client SDK setup.
7. Deploy the Heroku app.  The app should be hosted at `https://<app name>.herokuapp.com`.

The important file to review for the Parse server example is [here](https://github.com/ParsePlatform/parse-server-example/blob/master/index.js).  You can see that it takes a few environment variables to run.

### Testing Deployment

After deployment, try to connect to the site.  You should see `I dream of being a web site.` if the site loaded correctly.   If you try to connect to the `/parse` endpoint, you should see `{error: "unauthorized"}`.  If both tests pass, the basic configuration is successful.

Next, make sure you can create Parse objects.  You do not need a client Key to write new data:

```bash
curl -X POST -H "X-Parse-Application-Id: myAppId" -H "Content-Type: application/json" -d '{"score":1337,"playerName":"Sean Plott","cheatMode":false}'   https://yourappname.herokuapp.com/parse/classes/GameScore
```

To read data back, you will need to specify the master key:

```bash
curl -X GET -H "X-Parse-Application-Id: myAppId" -H "X-Parse-Master-Key: abc"    https://yourappname.herokuapp.com/parse/classes/GameScore
```

You can also verify whether the objects were created by clicking on the MongoDB instance in the Heroku panel:

<img src="http://imgur.com/bbj2e9N.png"/>

<img src="http://imgur.com/snPqYkz.png"/>

### Enabling Client SDK integration

Make sure you have the latest Parse-Android SDK in your `app/build.gradle` file.  It should be at least `1.13.0`:

```gradle
dependencies {
    compile 'com.parse:parse-android:1.13.0'
    compile 'com.parse:parseinterceptors:0.0.2' // for logging API calls to LogCat

}
```

Modify your `Parse.initialize()` command to point to this Heroku server.  You must be on the latest Parse Android SDK to have these options.

```java
public class ChatApplication extends Application {
    @Override
    public void onCreate() {
        super.onCreate();

        // set applicationId, clientKey, and server based on the values in the Heroku settings.
        // any network interceptors must be added with the Configuration Builder given this syntax
        Parse.initialize(new Parse.Configuration.Builder(this)
                .applicationId("myAppId") // should correspond to APP_ID env variable
                .clientKey("clientKey")  // should correspond to CLIENT_KEY env variable
                .addNetworkInterceptor(new ParseLogInterceptor())
                .server("https://parse-testing-port.herokuapp.com/parse/").build());
    }
}
```

**Note**: make sure to use the extra trailing `/` when using the `.server()` call.  There appears to be a [bug](https://github.com/ParsePlatform/Parse-SDK-Android/issues/393) in the Android SDK that strips the URL without this trailing slash.

The `/parse/` path needs to match the `PARSE_MOUNT` environment variable, which is set to this value by default.

### Support Push Notifications

**Note**: Support for push notifications is almost available.  See this [pull request](https://github.com/ParsePlatform/parse-server/pull/311/files) for more details.

There are a few steps to make this process work.  **Note**: Push Notifications via [Google Cloud Messaging](Google-Cloud-Messaging) (GCM) will only work for devices and emulators that have Google Play installed.

1. Obtain a Sender ID and API Key.
      * Follow only step 1 of [this guide](http://guides.codepath.com/android/Google-Cloud-Messaging#step-1-register-with-google-developers-console) to obtain the Sender ID (equivalent to the Project Number) and API Key.  You do not need to follow the other steps because Parse provides much of code to handle GCM registration for you.

2. Set `GCM_SENDER_ID` and `GCM_API_KEY` in your Heroku environment variables.

3. Add a `meta-data` with the Sender ID in your AndroidManifest.xml.  Make sure the `id:` is used as the prefix (Android treats any metadata value that has a string of digits as an integer so Parse prefixes this value).  If you forget this step, Parse will register with its own Sender ID but you will see `SenderID mismatch` errors when trying to issue push notifications.

   ```xml
     <meta-data
              android:name="com.parse.push.gcm_sender_id"
              android:value="id:SENDER_ID_HERE"/>
   ```

4. Add the necessary permissions.  Make sure to change any instances of `com.codepath.parseportpush` to your application package name.  If you forget any of these permissions, it is likely the GCM registration will not succeed.

    ```xml

     <uses-permission android:name="android.permission.INTERNET" />
        <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />

        <uses-permission android:name="android.permission.WAKE_LOCK" />
        <uses-permission android:name="android.permission.VIBRATE" />
        <uses-permission android:name="com.google.android.c2dm.permission.RECEIVE" />

        <!--
          IMPORTANT: Change "com.codepath.parseportpush.permission.C2D_MESSAGE" in the lines below
          to match your app's package name + ".permission.C2D_MESSAGE".
        -->
        <permission android:protectionLevel="signature"
            android:name="com.codepath.parseportpush.permission.C2D_MESSAGE" />
        <uses-permission android:name="com.codepath.parseportpush.permission.C2D_MESSAGE" />

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
    ```

5. Test out push notifications using the [Parse Notifications API](https://parse.com/docs/rest/guide/#push-sending) with this Python script:

  ```python
  import json
  import requests

  headers = {'X-Parse-Application-Id': 'myAppId', "X-Parse-Master-Key": "abc", "Content-Type": "application/json"}

  data = {'where': {'deviceType': 'android'},
          "data": {
              "action": "com.example.UPDATE_STATUS",
              "alert": "Ricky Vaughn was injured during the game last night!",
              "name": "Vaughn",
              "newsItem": "Man bites dog"}
         }

  response = requests.post("http://yourappname.herokuapp.com/parse/push", headers=headers, data=json.dumps(data))
  ```

### Troubleshooting

* If you see `Application Error` or `An error occurred in the application and your page could not be served. Please try again in a few moments.`, double-check that you set a `MASTER_KEY` in the Heroku environment settings for that app.

  <img src="http://imgur.com/uMYwPmS.png">

* Download the Heroku Toolbelt app [here](https://toolbelt.heroku.com/) to help view system logs.

  <img src="http://imgur.com/Ch0mZOK.png"/>

  First, you must login with your Heroku login and password:

  ```bash
  heroku login
  ```

   You can then view the system logs by specifying the app name:
   ```bash
   heroku logs -app <app name>
   ```

   The logs should show the response from any types of network requests made to the site.  Check the `status` code.
   ```
   2016-02-07T08:28:14.292475+00:00 heroku[router]: at=info method=POST path="/parse/classes/Message" host=parse-testing-port.herokuapp.com request_id=804c2533-ac56-4107-ad05-962d287537e9 fwd="101.12.34.12" dyno=web.1 connect=1ms service=2ms status=404 bytes=179
   ```

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
