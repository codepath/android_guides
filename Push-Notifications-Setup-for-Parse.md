## Overview

The instructions below apply to the open source version of Parse, not hosted Parse.   There are two parts to setting up Parse for push notifications.  First, you need to make sure your Parse server is setup properly.  Second, you need to add support with changes to your Android application.

### Parse Server Setup

Make sure to take a look at [[Configuring a Parse Server]] to understand how to setup your own self-hosted Parse server.  If a server has already been setup and configured properly to support Firebase Cloud Messaging (FCM), you can skip this section.

#### Obtain FCM API Key

First, you will need to obtain a Firebase API Key.  You can follow only step 1 of [this guide](https://github.com/codepath/android_guides/wiki/Google-Cloud-Messaging/b7ab0d3329898f147b2fe7a32c731f9ce251893c#step-1-register-with-google-developers-console) to obtain the Sender ID (equivalent to the Project Number) and API Key.  You do not need to follow the other steps because Parse provides much of code to handle FCM registration for you.  Remember the FCM API key provided.

#### Update Parse Server Config

Modify your `index.js` to add support for FCM.  See [this example](https://github.com/codepath/parse-server-example/blob/master/index.js#L15-L18):

```javascript
if (process.env.FCM_API_KEY) {
   pushConfig['android'] = { 
   apiKey: process.env.FCM_API_KEY || ''};
}
```

This repo has some additional environment variables configurations added that help facilitate sending push    notifications (i.e. see `FCM_API_KEY` in [index.js](https://github.com/codepath/parse-server-example/blob/master/index.js#L37-L40)).  You need to pass this configuration to the instantiation of the Parse server:

```javascript
var api = new ParseServer({
          // other variables here
              push: pushConfig,
          });
```

#### Setup environment variables

If you are using Heroku to configure the server, make sure to set the following environment variables:

   * Set `FCM_API_KEY` environment variables to correspond to the API Key in the previous step.  

      <img src="https://imgur.com/KgU2S2y.png"/>

   * Confirm the `SERVER_URL` environment variable is set to the URL and Parse mount location (i.e. `http://yourappname.herokuapp.com/parse`).  

   * Verify that `cloud/main.js` is the default value of `CLOUD_CODE_MAIN` environment variable.  

#### Add Parse Cloud Code

Modify [cloud/main.js](https://github.com/codepath/parse-server-example/blob/master/cloud/main.js) yourself to add custom code to send Push notifications.  See the [Parse Cloud code](http://parseplatform.github.io/docs/cloudcode/guide/#cloud-code-cloud-functions) documentation and [Parse JavaScript guide](https://parseplatform.github.io/docs/js/guide/#queries) to see what types of queries can be performed. Also take a look at the [Parse Push wiki](https://github.com/ParsePlatform/parse-server/wiki/Push#api) about the types of push data that can be sent.  Finally, see [these examples](https://github.com/ParsePlatform/parse-server/issues/401#issuecomment-183767065) for other ways of sending too.  

#### Redeploy Code

Next, redeploy the code.  If you are using Heroku, you need to connect your own forked repository and redeploy.  

<img src="https://i.imgur.com/OmxXc6s.png"/>

### Parse Client Setup

Before getting started, make sure you have Google Play installed on the emulator or device, since push notifications via [Firebase Cloud Messaging](Google-Cloud-Messaging) will only work for devices and emulators that have Google Play installed.

Also verify that you have added the Parse Client SDK as shown in [[this guide|Building-Data-driven-Apps-with-Parse#setup]].

### Firebase setup

Make sure to create a Firebase app at https://console.firebase.com.  Click on the option to add Firebase to your Android app:

<img src="https://imgur.com/pSDOxFt.png" height=350 />

Enter an Android package name (i.e. `com.codepath.pushtest`) used and download the `google-services.json` file. Add it to your app/ dir (see these [docs](https://developers.google.com/android/guides/google-services-plugin#adding_the_json_file) more information.)

<img src="https://imgur.com/T4hU5J7.png" width=650 />

Verify the applicationId in your `AndroidManifest.xml` matches the package name defined for your Firebase app.

#### Create a Firebase Instance ID receiver

You will also need to add a Firebase Instance ID service to listen for token updates.  This service will automatically send the updated device token to the Parse server:

```xml
<application>
  <service
    android:name="com.parse.fcm.ParseFirebaseInstanceIdService"
    android:exported="true">
    <intent-filter>
        <action android:name="com.google.firebase.INSTANCE_ID_EVENT" />
    </intent-filter>
  </service>
</application>
```

#### Create a service listening for push notifications

Listen for Firebase push notification messages and decode them in the format that the Parse Server sends:

```xml
<application>
  <service
    android:name=".MyFirebaseMessagingService">
    <intent-filter>
        <action android:name="com.google.firebase.MESSAGING_EVENT"/>
    </intent-filter>
  </service>
</application>
```

#### Listen for Parse Push events

Add the ParsePushBroadcast Receiver to handle opening up the appropriate activity for notifications.

```xml
<application>
  <receiver
    android:name="com.parse.ParsePushBroadcastReceiver"
    android:exported="false">
    <intent-filter>
        <action android:name="com.parse.push.intent.RECEIVE" />
        <action android:name="com.parse.push.intent.DELETE" />
        <action android:name="com.parse.push.intent.OPEN" />
    </intent-filter>
  </receiver>
</application>
```

You can also create your own custom broadcast receiver too.  If you use Parse's default [ParsePushBroadcastReceiver](https://github.com/ParsePlatform/Parse-SDK-Android/blob/master/Parse/src/main/java/com/parse/ParsePushBroadcastReceiver.java#L155-L160), using either `alert` or `title` as a key/value pair will trigger a notification message. See [this section](http://docs.parseplatform.org/android/guide/#receiving-pushes) of the Parse documentation.

### Receiving Pushes on Android

If you wish to listen to any pushes sent to a specific channel, you can subscribe by doing the following:

```java
public static final String CHANNEL_NAME = "abcd";

@Override
public void onCreate() {
  ParsePush.subscribeInBackground(CHANNEL_NAME);
}
```

After following the steps outlined above, be sure to check out the following resources for more information:

 * [ParsePlatform Push Docs](http://docs.parseplatform.org/android/guide/#push-notifications)
 * [ParsePlatform Push QuickStart](https://github.com/ParsePlatform/parse-server/wiki/Push#quick-start)
 * [CodePath Parse Push Example](https://github.com/codepath/ParsePushNotificationExample) 

Refer to these resources for more information on Parse Server push. 

### Custom Receivers

There are other options for creating custom receivers as well:

* You can also create your own custom receiver as shown in [this example](https://github.com/codepath/ParsePushNotificationExample/blob/master/app/src/main/java/com/test/MyCustomReceiver.java).

* You can programmatically register for broadcast notifications by creating an intent filter and registering the receiver: 

```java
public class MapDemoActivity extends AppCompatActivity {

    MarkerUpdatesReceiver markerUpdatesReceiver;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
     markerUpdatesReceiver = new MarkerUpdatesReceiver(this);
     // implicit broadcast, Android O still allows doing so inside activity, fragment, or service
     IntentFilter intentFilter = new IntentFilter("com.parse.push.intent.RECEIVE");  
     registerReceiver(markerUpdatesReceiver, intentFilter);
    }

    @Override
    protected void onDestroy() {
      super.onDestroy();
      // avoid memory leaks
      if (markerUpdatesReceiver != null) {
        unregisterReceiver(markerUpdatesReceiver);
      }
    }
    
}
```

NOTE: Parse's Android SDK requires at least one receiver to be declared on startup in your `AndroidManifest.xml` file in order for push messages to be received.

### Testing Push from the Client Side

Let's setup a quick way to validate push is setup correctly.  First, we need to have this echo reply Parse Cloud function defined.  

```javascript
// Defined in cloud/main.js on Parse server side
Parse.Cloud.define('pingReply', function(request, response) {
  var params = request.params;
  var customData = params.customData;

  if (!customData) {
    response.error("Missing customData!")
  }

  var sender = JSON.parse(customData).sender;
  var query = new Parse.Query(Parse.Installation);
  query.equalTo("installationId", sender);

  Parse.Push.send({
  where: query,
  // Parse.Push requires a dictionary, not a string.
  data: {"alert": "The Giants scored!"},
  }, { success: function() {
     console.log("#### PUSH OK");
  }, error: function(error) {
     console.log("#### PUSH ERROR" + error.message);
  }, useMasterKey: true});

  response.success('success');
});
```

On the client side, we need to execute this Parse Cloud function:

```java
JSONObject payload = new JSONObject();

try {
  payload.put("sender", ParseInstallation.getCurrentInstallation().getInstallationId());
} catch (JSONException e) {
  e.printStackTrace();
}

HashMap<String, String> data = new HashMap<>();
data.put("customData", payload.toString());

ParseCloud.callFunctionInBackground("pingReply", data);
```


### Security Model

The security model chosen for the open source version of Parse is to require any type of push functionality to be implemented as Parse Cloud code executed only on the server side.  Unlike Parse's hosted service, **you cannot implement this type of code** on the actual client:

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

Instead, you need to write your own server-side Parse code and have the client invoke it. 

### Troubleshooting 

#### Client Issues

* Make sure to enable logging by using `Parse.setLogLevel(Parse.LOG_LEVEL_DEBUG);` to track down permission issues.  

* Double-check your `<uses-permissions>` is outside the `<application>` tag.
* Verify that you added ParseFirebaseInstanceIdService and ParseFirebaseMessagingService to your AndroidManifest.xml file.
* If you are using Facebook's [[Stetho library|Debugging-with-Stetho]] with your Android client, you can see the LogCat statements and verify that FCM tokens (labeled as GCM push tokens) are being registered by API calls to the `/parse/classes/_Installation` endpoint:

```
: Url : http://192.168.3.116:1337/parse/classes/_Installation
```

You should be able to see the `deviceToken`, `installationId`, and `appName` registered:

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

You might also see an abbreviated version similar to the following if the FCM token is registered later:

```
11-06 06:32:20.349 25417-25599/com.example.mapdemo I/ParseLogInterceptor: Body : {
  "pushType": "gcm",
  "deviceToken": "YOUR-TOKEN_HERE",
  "objectId": "Q1Y0GKgktO"
}
11-06 06:32:20.349 25417-25599/com.example.mapdemo I/ParseLogInterceptor: --------------
```

* If FCM is fully setup, your app if properly configured should register itself with your Parse server.  Check your `_Installation` table to verify that the entries were being saved. Clear your app cache or uninstall the app if an entry in the  `_Installation` table hasn't been added.

* Verify that you have all the permissions for FCM setup in your `AndroidManifest.xml` file and that you have the correct receivers configured.

#### Server Issues

* Make sure you are on latest open source Parse version: [![npm version](https://img.shields.io/npm/v/parse-server.svg?style=flat)](https://www.npmjs.com/package/parse-server)  You will want to verify what version is set in your `package.json` file (i.e. https://github.com/ParsePlatform/parse-server-example/blob/master/package.json#L15).  Make sure to update this file and redeploy.

* Add the `VERBOSE=1` to the Heroku dashboard.  Also add `DEBUG=apn,node-gcm` to show specific FCM logs:

     <img src="https://imgur.com/K1MU5gL.png"/>

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

If you see `Can not find sender for push type android`, it means you forgot to set the environment variables `FCM_API_KEY`.


