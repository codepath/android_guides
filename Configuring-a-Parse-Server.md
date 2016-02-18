### Overview

Parse provides a cloud-based backend service to build data-driven mobile apps quickly.  Facebook, which acquired the company more than 3 years ago, announced that the service would be shutting down on **January 28, 2017**.   An [open source version](https://github.com/ParsePlatform/parse-server) enables developers to continue using their apps was published, along with a [migration guide](https://parse.com/docs/server/guide#migrating).

While there are many
[alternate options to Parse](https://github.com/relatedcode/ParseAlternatives), most of them lack either the functionality, documentation, or sample code to enable quick prototyping.  For this reason, the open source Parse version is a good option to use with minimal deployment/configuration needed.

You can review this [Wiki](https://github.com/ParsePlatform/parse-server/wiki) to understand the current development progress of this app.  Push notifications, for instance, has recently been added but still has some outstanding issues to work on Android and iOS.

### Setting a new Parse Server

The steps described this guide walk through most of the process of setting an open source version with Parse.  There are obviously many other hosting options, but the one-click deploys made available with Heroku or Amazon AWS as discussed in [this guide](https://github.com/ParsePlatform/parse-server-example) are the simplest.   In both cases, you are likely to need a credit card attached to your account to activate.

#### Signing up with Heroku

Use Heroku if you have little or no experience with setting up web sites. Heroku allows you to manage changes to deploy easily by specifying a GitHub repository to use.  In addition, it comes with a UI data viewer from MongoLabs.  

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
      * If you intend to use Parse's Facebook authentication, set `FACEBOOK_APP_ID` to be the [FB application ID](https://developers.facebook.com/apps).
      
4. Deploy the Heroku app.  The app should be hosted at `https://<app name>.herokuapp.com`.

If you ever need to change these values later, you can go to (`https://dashboard.heroku.com/apps/<app name>/settings`). Check out [this guide](https://devcenter.heroku.com/articles/deploying-a-parse-server-to-heroku) for a more detailed set of steps for deploying Parse to Heroku.

#### Signing up with Amazon

**Note:** You need to setup only Heroku above OR Amazon below but not both in order to use Parse. 

Amazon AWS provides more advanced functionality, such as a load balancer, easy-to-upgrade instances, and [auto-scaling groups](https://aws.amazon.com/autoscaling/).  Use only if you are more comfortable with managing servers.  You do not get a UI data viewer out of the box.

1. You can create a new Parse instance by clicking on the button:

     <a title="Deploy to AWS" href="https://console.aws.amazon.com/elasticbeanstalk/home?region=us-west-2#/newApplication?applicationName=ParseServer&amp;solutionStackName=Node.js&amp;tierName=WebServer&amp;sourceBundleUrl=https://s3.amazonaws.com/elasticbeanstalk-samples-us-east-1/eb-parse-server-sample/parse-server-example.zip" target="_blank"><img src="https://camo.githubusercontent.com/f13c4f9254091b2f91ea161e9461491481d0e586/687474703a2f2f64302e6177737374617469632e636f6d2f70726f647563742d6d61726b6574696e672f456c61737469632532304265616e7374616c6b2f6465706c6f792d746f2d6177732e706e67" height="40" data-canonical-src="http://d0.awsstatic.com/product-marketing/Elastic%20Beanstalk/deploy-to-aws.png" style="max-width:100%;"></a>
2. Create an application name and click on `Review and Launch`.  Clicking on `Next` will take you the more advanced settings.
       <img src="http://imgur.com/yRrmmse.png"/>
3. The final step is to review your changes.  Click `Launch` when ready.
4. The machine configuration should begin to start.  Look for the `Software Configuration` panel to appear:

       <img src="http://imgur.com/YATjJlc.png"/>
5. Configure the environment variable settings.
       <img src="http://imgur.com/EdcNda3.png"/>
      * Set `APP_ID` for the app identifier.  If you do not set one, the default is set as `myAppId`.  You will need this info for the Client SDK setup.
      * Set `MASTER_KEY` to be the master key used to read all data.  
      * Leave `PARSE_MOUNT` to be `/parse`.  It does not need to be changed.
      * If you intend to use Facebook authentication, set `FACEBOOK_APP_ID` to be the [FB application ID](https://developers.facebook.com/apps).

6. Your machine should take a few minutes to spin up.  Once it's ready, you should be able to click on the URL provided:

       <img src="http://imgur.com/OjyvPdc.png"/>

Other tips:

* If you need to update the source code, you need to go the download the updated `.zip` file and upload to the main dashboard here:

       <img src="http://imgur.com/8XBkbnK.png"/>
        
* If you need to get back to the console after the instance has been created, you can click [here](https://console.aws.amazon.com/elasticbeanstalk/home?region=us-west-2).

* You mean need to get direct SSH access to this box to view the MongoDB data.  See [this article](http://stackoverflow.com/questions/4742478/ssh-to-elastic-beanstalk-instance) for more information.

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

If you are using Heroku, You can also verify whether the objects were created by clicking on the MongoDB instance in the Heroku panel:

<img src="http://imgur.com/bbj2e9N.png"/>

<img src="http://imgur.com/snPqYkz.png"/>

You can also setup [Robomongo](https://robomongo.org/) to connect to your remote mongo database hosted on Heroku to get a better data browser and dashboard for your app.

### Enabling Client SDK integration

Make sure you have the latest Parse-Android SDK in your `app/build.gradle` file.  It should be at least `1.13.0`:

```gradle
dependencies {
    compile 'com.parse:parse-android:1.13.0'
    compile 'com.parse:parseinterceptors:0.0.2' // for logging API calls to LogCat

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
public class ChatApplication extends Application {
    @Override
    public void onCreate() {
        super.onCreate();

        // set applicationId, and server server based on the values in the Heroku settings.
        // clientKey is not needed unless explicitly configured
        // any network interceptors must be added with the Configuration Builder given this syntax
        Parse.initialize(new Parse.Configuration.Builder(this)
                .applicationId("myAppId") // should correspond to APP_ID env variable
                .addNetworkInterceptor(new ParseLogInterceptor())
                .server("https://parse-testing-port.herokuapp.com/parse/").build());
    }
}
```

**Note**: make sure to use the extra trailing `/` when using the `.server()` call.  There appears to be a [bug](https://github.com/ParsePlatform/Parse-SDK-Android/issues/393) in the Android SDK that strips the URL without this trailing slash.

The `/parse/` path needs to match the `PARSE_MOUNT` environment variable, which is set to this value by default.

### Enabling Push Notifications

**Note**: Experimental Support for push notifications is now available with the open source Parse server as of v2.0.8, but there is work to enable it to work fully on Android devices (see this [issue](https://github.com/ParsePlatform/parse-server/issues/396)). See the [Wiki article](https://github.com/ParsePlatform/parse-server/wiki/Push) for more information.

#### GCM Setup

There are a few steps to make this process work.  **Note**: Push Notifications via [Google Cloud Messaging](Google-Cloud-Messaging) (GCM) will only work for devices and emulators that have Google Play installed.

1. Obtain a Sender ID and API Key.
      * Follow only step 1 of [this guide](http://guides.codepath.com/android/Google-Cloud-Messaging#step-1-register-with-google-developers-console) to obtain the Sender ID (equivalent to the Project Number) and API Key.  You do not need to follow the other steps because Parse provides much of code to handle GCM registration for you.

2. Set `GCM_ENABLE=1` in your environment variables to enable GCM support.  Set `GCM_SENDER_KEY` and `GCM_API_KEY` environment variables to correspond to the Sender ID and API Key in the previous step.  **Note**: this step only works if you have setup your Parse server to enable push support.  See the [CodePath fork](https://github.com/codepath/parse-server-example/blob/master/index.js#L15-L26) as an example about how to initialize the Parse server.

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
   heroku logs -app <app name>
   ```

   The logs should show the response from any types of network requests made to the site.  Check the `status` code.
   ```
   2016-02-07T08:28:14.292475+00:00 heroku[router]: at=info method=POST path="/parse/classes/Message" host=parse-testing-port.herokuapp.com request_id=804c2533-ac56-4107-ad05-962d287537e9 fwd="101.12.34.12" dyno=web.1 connect=1ms service=2ms status=404 bytes=179
   ```

* If you are seeing `Master key is invalid, you should only use master key to send push`, chances are you are trying to send Push notifications without enable client push.  On Parse.com you can simply enable a toggle switch but for hosted parse there is an outstanding [issue](https://github.com/ParsePlatform/parse-server/issues/396) that must be resolved to start supporting it.

* If you intend to use the Parse JavaScript SDK, you will also need to enable the `javaScriptKey` in your hosted `index.js` config.  Take a look at this [example](https://github.com/codepath/parse-server-example/blob/master/index.js#L26) to see where to add it.  Assuming you follow this example, you then should set your `JAVASCRIPT_KEY` environment variable.

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