## Overview

**Google Cloud Messaging** (GCM) for Android is a service that allows you to send data from your server to your users' Android-powered device and also to receive messages from devices on the same connection.  It is now known as **Firebase Cloud Messaging** (FCM) after a new site launched during the Google's I/O 2016 conference that unifies analytics and messaging into one platform.  Review this [slide deck](https://docs.google.com/presentation/d/16h4KOCbP2zmyqNqhfB4ulE9JtO-jwBT1j3hMgcX-his/edit?usp=sharing) for more context.

![FCM Arch](https://i.imgur.com/9XzwPqc.png)

### How it works

Much of the heavy lifting in supporting push notifications on Android is facilitated by Google-powered [connection servers](https://developers.google.com/cloud-messaging/server.html).  These Google servers provide an API for messages to be sent from your server and relay these messages to any Android/iOS devices authorized to receive them.

An Android device with Google Play Services will already have FCM client support available.  For push notifications to be received, an app must first obtain a token by registering with a Google server:

<img src="https://imgur.com/5UPxP3n.png" height=300/>

This token then must be passed along to your server so that it can be used to send subsequent push notifications:

<img src="https://imgur.com/ItRPQ7N.png" height=300/>

Push notifications can be received assuming your app has registered to listen for FCM-based messages:

<img src="https://imgur.com/adiFo8w.png" height=300/>

In other words, in order to implement FCM, your app will need both a Google server and your own server.  When your app gets a token from Google, it needs to forward this token to your server.  This token should be persisted by the server so that it can be used to make API calls to the Google server.  With this approach, your server and the Android device do not need to create a persistent connection and the responsibility of queuing and relaying messages is all handled by Google's servers.

### Setup

In order to use FCM, we need to go through the following steps:

 1. Signup with Firebase console.
    - Associate an existing app or create a new one.
    - Provide the app name and SHA-1 signature of debug key used to sign your app.
    - Download a `google-services.json` configuration file.
 2. Integrate FCM into our Android app
    - Register for an [instance ID](https://plus.google.com/+AndroidDevelopers/posts/DMshVTyzqcL) and generate a token
    - Transmit the token to our web server
    - Register a FCM Receiver to handle incoming messages
    - Register an InstanceID Listener Service to handle updated tokens
 3. Develop HTTP Server with FCM endpoints
    - Endpoint for registering a user with a registration token
    - Endpoint for sending a push notification to a specified set of registration tokens

## Step 1: Register with Firebase Developers Console

In order to use FCM, you need to login to [https://console.firebase.google.com/](https://console.firebase.google.com/).  You should be given a choice on the right-hand side as to whether to create a new project or import an existing Google app into Firebase.

<img src="https://imgur.com/9uZggGW.png"/>

### Creating a new project

Click on `Create New Project`.  You should see the window pop up:

<img src="https://imgur.com/b3puXlh.png"/>

### Using an existing project

-  If you have existing app that used to be using FCM, click on `Import Google Project`.   Make sure to click on `Add Firebase` when you're done.

After you've created or imported an existing project into Firebase, you need to add the package name and add the SHA-1 signing certificate (see [link](https://developers.google.com/android/guides/client-auth) to obtain):

![](https://i.imgur.com/CmmqFJ7.png)

Click on `Add Firebase to your Android app`:

<img src="https://imgur.com/qTWfk3r.png"/>

Click on `Add App` and then copy the `google-services.json` file that will get downloaded into your`app/` dir.  This file includes the  project information and API keys needed to connect to Firebase.  Make sure to preserve the filename, since it will be used in the next step.

Finally, add the Google services to your root `build.gradle` file's classpath:

```gradle
buildscript {
  dependencies {
    // Add this line
    classpath 'com.google.gms:google-services:3.0.0'
  }
}
```

Add to your existing `app/build.gradle` at the end of the file:

```gradle
// Add to the bottom of the file
apply plugin: 'com.google.gms.google-services'
```


## Step 2: Setup Android Client

**Note**: If you need to upgrade your existing GCM libraries to FCM, see [these instructions](https://developers.google.com/cloud-messaging/android/android-migrate-fcm#update_your_instanceidlistenerservice). The instructions below have been updated to reflect the depedencies on the FCM package.

### Download Google Play Services

First, let's download and setup the Google Play Services SDK. Open `Tools`->`Android`->`SDK Manager` and check whether or not you have already downloaded `Google Play Services` under the Extras section.  Make sure to update to the latest version to ensure the Firebase package is available:

![](https://i.imgur.com/VYM0m59.png)

### Add Google Repository

Also open `Tools`->`Android`->`SDK Manager` and click on the `SDK Tools` tab.  Make sure that under `Support Repository` you have installed the `Google Repository`.  If you forget this step, you are not likely to be able to include the Firebase messaging library in the next step.

<img src="https://imgur.com/QBvyEAH.png"/>

### Update to SDK Tools

Also make sure to upgrade to SDK Tools 25.2.2.  You may have Firebase authentication issues with lower SDK Tools version.

### Import Firebase Messaging Library

Add the following to your Gradle file:

```gradle
dependencies {
   compile 'com.google.firebase:firebase-messaging:9.4.0'
}
```

### Implement a Registration Intent Service

You will want to implement an [[Intent Service|Starting-Background-Services#creating-an-intentservice]], which will execute as a background thread instead of being tied to the lifecycle of an Activity.   In this way, you can ensure that push notifications can be received by your app if a user navigates away from the activity while this registration process is occuring.

First, you will need to create a `RegistrationIntentService` class and make sure it is declared in your `AndroidManifest.xml` file and within the `application` tag:

```java
<service android:name=".RegistrationIntentService" android:exported="false"/>
```

Inside this class, you will need to request an [instance ID](https://developers.google.com/instance-id/reference/) from Google that will be a way to uniquely identify the device and app.  Assuming this request is successful, a token that can be used to send notifications to the app should be generated too.

```java
// abbreviated tag name
public class RegistrationIntentService extends IntentService {

    // abbreviated tag name
    private static final String TAG = "RegIntentService";

    public RegistrationIntentService() {
        super(TAG);
    }

    @Override
    protected void onHandleIntent(Intent intent) {
        // Make a call to Instance API
        FirebaseInstanceId instanceID = FirebaseInstanceId.getInstance();
        String senderId = getResources().getString(R.string.gcm_defaultSenderId);
        try {
            // request token that will be used by the server to send push notifications
            String token = instanceID.getToken();
            Log.d(TAG, "FCM Registration Token: " + token);

            // pass along this data
            sendRegistrationToServer(token);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    private void sendRegistrationToServer(String token) {
        // Add custom implementation, as needed.
    }
}
```

You will want to record whether the token was sent to the server and may wish to store the token in your [[Shared Preferences|Storing-and-Accessing-SharedPreferences]]:

```java
    public static final String SENT_TOKEN_TO_SERVER = "sentTokenToServer";
    public static final String FCM_TOKEN = "FCMToken";

    @Override
    protected void onHandleIntent(Intent intent) {
        SharedPreferences sharedPreferences = PreferenceManager.getDefaultSharedPreferences(this);

        // Fetch token here
        try {
          // save token
          sharedPreferences.edit().putString(FCM_TOKEN, token).apply();
          // pass along this data
          sendRegistrationToServer(token);
        } catch (IOException e) {
            Log.d(TAG, "Failed to complete token refresh", e);
            // If an exception happens while fetching the new token or updating our registration data
            // on a third-party server, this ensures that we'll attempt the update at a later time.
            sharedPreferences.edit().putBoolean(SENT_TOKEN_TO_SERVER, false).apply();
        }
    }

    private void sendRegistrationToServer(String token) {
        // send network request

        // if registration sent was successful, store a boolean that indicates whether the generated token has been sent to server
        SharedPreferences sharedPreferences = PreferenceManager.getDefaultSharedPreferences(this);
        sharedPreferences.edit().putBoolean(SENT_TOKEN_TO_SERVER, true).apply();
     }
```

You will want to make sure to dispatch this registration intent service when starting up your main activity.  There is a Google Play Services check that may require your activity to be launched in order to display a dialog error message, which is why it is being initiated in the activity instead of application.

```java

public class MyActivity extends AppCompatActivity {

  /**
     * Check the device to make sure it has the Google Play Services APK. If
     * it doesn't, display a dialog that allows users to download the APK from
     * the Google Play Store or enable it in the device's system settings.
     */
    private boolean checkPlayServices() {
        GoogleApiAvailability apiAvailability = GoogleApiAvailability.getInstance();
        int resultCode = apiAvailability.isGooglePlayServicesAvailable(this);
        if (resultCode != ConnectionResult.SUCCESS) {
            if (apiAvailability.isUserResolvableError(resultCode)) {
                apiAvailability.getErrorDialog(this, resultCode, PLAY_SERVICES_RESOLUTION_REQUEST)
                        .show();
            } else {
                Log.i(TAG, "This device is not supported.");
                finish();
            }
            return false;
        }
        return true;
    }

    @Override
    protected void onCreate() {
        if(checkPlayServices()) {
          Intent intent = new Intent(this, RegistrationIntentService.class);
          startService(intent);
        }
    }
}
```

### Create a InstanceID ListenerService

According to this Google official [documentation](https://developers.google.com/instance-id/guides/android-implementation), the instance ID server issues callbacks periodically (i.e. 6 months) to request apps to refresh their tokens.  To support this possibility, we need to extend from `InstanceIDListenerService` to handle token refresh changes.  We should create a file called `MyInstanceIDListenerService.java` that will override this base method and launch an intent service for `RegistrationIntentService` to fetch the token:

```java
public class MyInstanceIDListenerService extends FirebaseInstanceIdService {

    @Override
    public void onTokenRefresh() {
        // Fetch updated Instance ID token and notify of changes
        Intent intent = new Intent(this, RegistrationIntentService.class);
        startService(intent);
    }
}
```

You also need to add the service to your `AndroidManifest.xml` file within the `application` tag:

```xml
<service
  android:name="com.example.MyInstanceIDListenerService"
  android:exported="false">
  <intent-filter>
     <action android:name="com.google.firebase.INSTANCE_ID_EVENT"/>
  </intent-filter>
</service>
```

### Listening for push notifications

Let's define `FCMMessageHandler.java` that extends from `FirebaseMessagingService` that will process the message received:

```java
import com.google.firebase.messaging.FirebaseMessagingService;
import com.google.firebase.messaging.RemoteMessage;

import android.app.NotificationManager;
import android.content.Context;
import android.os.Bundle;
import android.support.v4.app.NotificationCompat;

public class FCMMessageHandler extends FirebaseMessagingService {
    public static final int MESSAGE_NOTIFICATION_ID = 435345;

    @Override
    public void onMessageReceived(RemoteMessage remoteMessage) {
        Map<String, String> data = remoteMessage.getData();
        String from = remoteMessage.getFrom();

        Notification notification = remoteMessage.getNotification();
        createNotification(notification);
    }

    // Creates notification based on title and body received
    private void createNotification(Notification notification) {
        Context context = getBaseContext();
        NotificationCompat.Builder mBuilder = new NotificationCompat.Builder(context)
                .setSmallIcon(R.mipmap.ic_launcher).setContentTitle(notification.getTitle())
                .setContentText(notification.getBody());
        NotificationManager mNotificationManager = (NotificationManager) context
                .getSystemService(Context.NOTIFICATION_SERVICE);
        mNotificationManager.notify(MESSAGE_NOTIFICATION_ID, mBuilder.build());
    }

}
```

We need to register the receiver class with FCM in the `AndroidManifest.xml` tagging the type of request (category) of the push:

```xml
 <application
     ...>
         <!-- ... -->

         <activity
             ...>
         </activity>

         <service
            android:name=".FCMMessageHandler"
            android:exported="false" >
            <intent-filter>
                <action android:name="com.google.firebase.MESSAGING_EVENT" />
            </intent-filter>
         </service>

     </application>

</manifest>
```

Note that above the push message is handled by creating a notification. There are a few actions that are commonly taken when a push is received:

 * [[Create a notification|Notifications#creating-a-notification]]
 * [[Update an Activity|Starting-Background-Services#communicating-from-a-service-to-an-application]]
 * [Launch new activity](https://github.com/codepath/ParsePushNotificationExample/blob/master/app/src/main/java/com/test/MyCustomReceiver.java#L69)

You can review examples of these [outlined in this more elaborate code sample](https://github.com/codepath/ParsePushNotificationExample/blob/master/app/src/main/java/com/test/MyCustomReceiver.java).

In certain cases when receiving a push, you want to update an activity if the activity is on the screen. Otherwise, you want to raise a notification. The solutions to this are [outlined in this post](http://stackoverflow.com/a/18311830/313399) with a [code sample here](http://stackoverflow.com/a/15949723/313399).

### Testing

You can use the `Grow` -> `Notifications` to send test messages to verify that your app is correctly receiving messages.  Select the `User segment` and choose the `app` to receive the notifications.  If you want to pass custom data, you will need to select `Advanced options` to send custom data.  

![](https://i.imgur.com/0BvxwSo.png)

## Step 3: Setup Web Server

Now we just need a web server to keep track of the different device tokens and manage the pushing of messages to different devices.

### Overview

Your HTTP web server needs two features:

- 1. Endpoint for mapping a `user_id` to an android `reg_token`
- 2. Logic for sending specified `data` messages to sets of android `reg_tokens`

### 1. Implement Registration Endpoint

We need an endpoint for **registering a user id to a device token**:

  - Probably requires a database table for user to device mappings
  - Create a reg_token linked to a registered user for later access
  - Sample endpoint: `POST /register?user_id=123&reg_token=abc`

### 2. Implement Sending Logic

We need code for **sending data to specified registration tokens**:

  - To send GCM pushes, send a POST request to GCM send endpoint with data and registered ids
    - `POST https://gcm-http.googleapis.com/gcm/send`
  - Request requires two headers:
    - `Authorization: key=<YOUR SERVER API KEY>`
    - `Content-Type: application/json`
  - HTTP POST body can be in JSON. Below is an example of the body of the request to GCM.

      ```json
      {
        "data": {
          "title": "Test Title",
          "body": "Test Body"
        },
        "registration_ids": ["token1", "token2"]
      }
      ```

This sending code can be exposed as an endpoint or utilized on the server-side to notify users when new items are created or available.  Note that the `registration_ids` parameter is synonymous with device tokens.  if you only need to send a message to one device, you can use `to` instead of `registration_ids`:

   ```json
      {
        "data": {
          "title": "Test Title",
          "body": "Test Body"
        },
        "to": "token1"
      }
   ```

### Sample Ruby Server Implementation

A simple Sinatra-based ruby application that supports these endpoints is included below.  The key point is that there is a `/register` endpoint, which is needed to record the registration token for a particular example.  In a production environment, these POST requests should be sent by an authenticated user, so the token can be associated with that individual.  In this example, the `user_id` must be specified.

#### Setup

First, let's install a few packages that will be used for this sample application.

```bash
gem install sinatra
gem install rest-client
gem install sequel
```

#### Sample web server

```ruby
require 'sinatra'
require 'rest-client'
require 'sequel'

# Create a SQLite3 database

DB = Sequel.connect('sqlite://gcm-test.db')

# Create a Device table if it doesn't exist
DB.create_table? :Device do
  primary_key :reg_id
  String :user_id
  String :reg_token
  String :os, :default => 'android'
end

Device = DB[:Device]  # create the dataset

# Registration endpoint mapping reg_token to user_id
# POST /register?reg_token=abc&user_id=123
post '/register' do
  if Device.filter(:reg_token => params[:reg_token]).count == 0
    device = Device.insert(:reg_token => params[:reg_token], :user_id => params[:user_id], :os => 'android')
  end
end

# Ennpoint for sending a message to a user
# POST /send?user_id=123&title=hello&body=message
post '/send' do
  # Find devices with the corresponding reg_tokens
  reg_tokens = Device.filter(:user_id => params[:user_id]).map(:reg_token).to_a
  if reg_tokens.count != 0
    send_gcm_message(params[:title], params[:body], reg_tokens)
  end
end

# Sending logic
# send_gcm_message(["abc", "cdf"])
def send_gcm_message(title, body, reg_tokens)
  # Construct JSON payload
  post_args = {
    # :to field can also be used if there is only 1 reg token to send
    :registration_ids => reg_tokens,
    :data => {
      :title  => title,
      :body => body,
      :anything => "foobar"
    }
  }

  # Send the request with JSON args and headers
  RestClient.post 'https://gcm-http.googleapis.com/gcm/send', post_args.to_json,
    :Authorization => 'key=' + AUTHORIZE_KEY, :content_type => :json, :accept => :json
end
```

#### Testing

We can startup this Sinatra server and register the token granted to our Android client with it:

```bash
curl http://localhost:4567/register -d "reg_token=<REG TOKEN>&user_id=123"
```

If we wish to send a message, we would simply type:

```bash
curl http://localhost:4567/send -d "user_id=123&title=hello&body=message"
```

## Easy Local Testing with Curl

To verify that API calls to Google servers are working currently, We can quickly test by sending commands with curl:

```bash
curl -s "https://android.googleapis.com/gcm/send" -H "Authorization: key=[API_KEY_HERE]" -H "Content-Type: application/json" -d '{"to": "[REG_TOKEN_HERE]", "data": {"score": 123}}'
```

## Easy Local Testing with Ruby

We can also easily send messages to FCM registered devices using the  ruby [FCM gem](https://github.com/spacialdb/GCM). First we must have [ruby installed](https://www.ruby-lang.org/en/installation/) (default on OSX) and then we can install FCM with:

```
gem install gcm
```

and send test pushes by running `irb` in the command-line:

```ruby
require 'gcm'
gcm = GCM.new("YOUR-SERVER-KEY-HERE")
reg_tokens = ["YOUR-DEVICE-TOKEN", "ANOTHER-DEVICE-TOKEN"]
options = { :data => { :title =>"foobar", :body => "this is a longer message" } }
response = gcm.send(reg_tokens, options)
```

This will send messages to the devices specified.

## Subscribing clients to Topic-Based Messages

FCM also supports opt-in topic-based subscriptions for clients, which does not require passing along the device token to your server.  On the client side, we can also subscribe to this topic simply with two lines.  The `getInstance()` call automatically makes a call to grab an Instance ID, so if we do not wish to pass along our device token, our `RegistrationIntentService` class can look like the following:

```java
@Override
protected void onHandleIntent(Intent intent) {
  FirebaseMessaging.getInstance().subscribeToTopic("/topics/" + "dogs");
}
```

Inside your custom `FCMListenerService`, you will also need to check for these topic-based messages by checking the `from` String:

```java
@Override
public void onMessageReceived(RemoteMessage remoteMessage) {
   String message = remoteMessage.getNotification().getBody();
   String from = remoteMessage.getFrom();
   if (from.startsWith("/topics/dogs")) {
      Log.d(TAG, "got here");
   } else {
      // normal downstream message.
   }
```

Sending to subscribers simply involves changing the `to:` to be prefaced with `/topics/[topic_name]`:

```bash
curl -s "https://gcm-http.googleapis.com/GCM/send" -H "Authorization: key=[AUTHORIZATION_KEY]" -H "Content-Type: application/json" -d '{"to": "/topics/hello", "data": {"score": 123}}'

```

## Rate Limits

FCM is free to use.  In Jan. 2015, Google announced new rate limits in this [blog post](https://plus.google.com/+AndroidDevelopers/posts/Kc2whqR66zt) for FCM.  There is a per minute / per device limit that can be sent.  More technical details are included in this Stack Overflow [posting](http://stackoverflow.com/questions/26790810/rate-limit-exceeded-error-when-using-google-cloud-messaging-api/26790811#26790811).

## Quickstart sample

Take a look at Google's [quickstart sample](https://github.com/firebase/quickstart-android/tree/master/messaging) to test out FCM.

## Deploying to Heroku

If you wanted to actually deploy this sinatra app to production, you can do so for free with [heroku](https://heroku.com). Full source code and details for doing so can be [found on this repo](https://github.com/mjpablo23/FirebaseCloudMessagingHerokuServer). 

## References

- <https://developers.google.com/cloud-messaging/gcm>
- <https://www.youtube.com/watch?v=YoaP6hcDctM> (Google I/O 2012 talk)
- <https://www.youtube.com/watch?v=y76rjidm8cU> (Google I/O 2013 talk)
- <https://www.youtube.com/watch?v=gJatfdattno> (Google I/O 2015 talk)
- <http://android-developers.blogspot.com/2015/05/a-closer-look-at-google-play-services-75.html>
