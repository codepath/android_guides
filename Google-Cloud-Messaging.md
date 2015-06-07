## Overview

Google Cloud Messaging for Android (GCM) is a service that allows you to send data from your server to your users' Android-powered device, and also to receive messages from devices on the same connection. 

A GCM implementation includes a Google-provided connection server, a 3rd-party app server that interacts with the connection server, and a GCM-enabled client app running on an Android device:

![GCM Arch](http://i.imgur.com/9XzwPqc.png)

In order to use GCM, we need to go through the following steps:

 1. Register with Google Console and enable GCM
    - Obtain Sender ID (Project Number)
    - Obtain Server API Key
 2. Integrate GCM into our Android app
    - Register in app for a GCM Registration ID
    - Transmit the Registration ID (Device ID) to our web server
    - Register a GCM Receiver to handle incoming messages
 3. Develop HTTP Server with GCM endpoints
    - Endpoint for registering a user with a device ID
    - Endpoint for sending a push notification to a specified set of device IDs

## Step 1: Register with Google Developers Console

 - Go to the [Google Developers Console](https://cloud.google.com/console)
 - If this is the first time you visit Google Developer Console, you need to create a Project
 - Find the **Project Number** at the top of the page and record this for later use
 - Under "APIs & Auth" >> API, enable "Google Cloud Messaging for Android"
 - Under "APIs & Auth" >> Credentials, "Create New Key" => "Server Key", record key for later

Before continuing make sure:

 - `Google Cloud Messaging for Android` is enabled in APIs
 - Stored your "Project Number" as listed in developer console in APIs
 - Stored your "Server Key" as listed in developer console in Credentials

Now let's setup our Android client.
 
## Step 2: Setup Android Client

### Download Google Play Services

First, let's download and setup the Google Play Services SDK. Open **Eclipse ⇒ Windows ⇒ Android SDK** Manager and check whether you have already downloaded "Google Play services" or not under Extras section. If not, select "Google Play services" and install the package.

![Play Services](http://i.imgur.com/3ZJsWEw.png)

### Import Google Play Services

**Android Studio Users:** 

Add the following to your Gradle file

```
dependencies {
  compile "com.google.android.gms:play-services:6.5.87"
  compile 'com.android.support:support-v4:22.2.0'
}
```

**Eclipse Users:** 

After downloading the play services we need to import the project to Eclipse which will be used as a library for our maps project.

1. In Eclipse goto **File ⇒ Import ⇒ Android ⇒ Existing Android Code Into Workspace**

2. Click on Browse and select "Google Play Services" project from your android sdk folder. You can locate the play services library project from
`<your-android-sdk-path>\extras\google\google_play_services\libproject\google-play-services_lib`

3. Be sure to check "Copy projects into workspace" option as shown in the below image.

![Google Play Load](http://i.imgur.com/EHefxss.png)

### Add GCM Permissions

```xml
<manifest package="com.example.gcm" ...>
  <uses-permission android:name="android.permission.INTERNET" />
  <uses-permission android:name="android.permission.GET_ACCOUNTS" />
  <uses-permission android:name="android.permission.WAKE_LOCK" />
  <uses-permission android:name="com.google.android.c2dm.permission.RECEIVE" />
  <permission android:name="com.example.gcm.permission.C2D_MESSAGE"
      android:protectionLevel="signature" />
  <uses-permission android:name="com.example.gcm.permission.C2D_MESSAGE" />
  
  <application 
     ...
  />
</manifest>
```

### Copy over `GCMClientManager.java`

Copy source of [GCMClientManager.java](https://gist.github.com/nesquena/fede844799d103586f75) to your Android project. Use in your first Activity with the following:

```java
public class FirstActivity extends Activity {
    private GCMClientManager pushClientManager;
       String PROJECT_NUMBER = "<YOUR PROJECT NUMBER HERE>";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        pushClientManager = new GCMClientManager(this, PROJECT_NUMBER);
        pushClientManager.registerIfNeeded(new RegistrationCompletedHandler() {
           @Override
           public void onSuccess(String registrationId, boolean isNewRegistration) {
    	       Toast.makeText(MainActivity.this, registrationId, 
    	          Toast.LENGTH_SHORT).show();
    	       // SEND async device registration to your back-end server 
    	       // linking user with device registration id
    	       // POST https://my-back-end.com/devices/register?user_id=123&device_id=abc
           }

           @Override
           public void onFailure(String ex) {
             super.onFailure(ex);
             // If there is an error registering, don't just keep trying to register.
             // Require the user to click a button again, or perform
             // exponential back-off when retrying.
           }
        });
    }
}
```
	 

## Create Broadcast Receiver and Message Handler

This receiver's `onReceive` method will be called by the GCM service because of a `receiver` registration in the `AndroidManifest.xml`. This will simply invoke the `GcmMessageHandler` to manage the push:

```java
import android.app.Activity;
import android.content.ComponentName;
import android.content.Context;
import android.content.Intent;
import android.support.v4.content.WakefulBroadcastReceiver;

public class GcmBroadcastReceiver extends WakefulBroadcastReceiver {
    // Receives the broadcast directly from GCM service
    @Override
    public void onReceive(Context context, Intent intent) {
        // Explicitly specify that GcmMessageHandler will handle the intent.
        ComponentName comp = new ComponentName(context.getPackageName(),
                GcmMessageHandler.class.getName());
        // Start the service, keeping the device awake while it is executing.
        startWakefulService(context, (intent.setComponent(comp)));
        // Return successful
        setResultCode(Activity.RESULT_OK);
    }
}
```

and then let's define the corresponding `GCMMessageHandler.java` which handles the push message in the `onHandleIntent` method:

```java
import android.app.IntentService;
import android.app.NotificationManager;
import android.content.Context;
import android.content.Intent;
import android.os.Bundle;
import android.support.v4.app.NotificationCompat;
import android.util.Log;
import com.google.android.gms.gcm.GoogleCloudMessaging;

public class GcmMessageHandler extends IntentService {
        public static final int MESSAGE_NOTIFICATION_ID = 435345;

	public GcmMessageHandler() {
		super("GcmMessageHandler");
	}

	@Override
	protected void onHandleIntent(Intent intent) {
		// Retrieve data extras from push notification
		Bundle extras = intent.getExtras();
		GoogleCloudMessaging gcm = GoogleCloudMessaging.getInstance(this);
		// The getMessageType() intent parameter must be the intent you received
		// in your BroadcastReceiver.
		String messageType = gcm.getMessageType(intent);
		// Keys in the data are shown as extras
		String title = extras.getString("title");
		String body = extras.getString("body");
		// Create notification or otherwise manage incoming push
		createNotification(title, body);
		// Log receiving message
		Log.i("GCM", "Received : (" + messageType + ")  " + extras.getString("title"));
		// Notify receiver the intent is completed
		GcmBroadcastReceiver.completeWakefulIntent(intent);
	}

	// Creates notification based on title and body received
	private void createNotification(String title, String body) {
		Context context = getBaseContext();
		NotificationCompat.Builder mBuilder = new NotificationCompat.Builder(context)
		    .setSmallIcon(R.drawable.ic_launcher).setContentTitle(title)
		    .setContentText(body);
		NotificationManager mNotificationManager = (NotificationManager) context
				.getSystemService(Context.NOTIFICATION_SERVICE);
		mNotificationManager.notify(MESSAGE_NOTIFICATION_ID, mBuilder.build());
	}

}
```

Note that above the push message is handled by creating a notification. There are a few actions that are commonly taken when a push is received:

 * [[Create a notification|Notifications#creating-a-notification]]
 * [[Update an Activity|Starting-Background-Services#communicating-from-a-service-to-an-application]]
 * [Launch new activity](https://github.com/codepath/ParsePushNotificationExample/blob/master/src/com/test/MyCustomReceiver.java#L69)

You can review examples of these [outlined in this more elaborate code sample](https://github.com/codepath/ParsePushNotificationExample/blob/master/src/com/test/MyCustomReceiver.java). 

In certain cases when receiving a push, you want to update an activity if the activity is on the screen. Otherwise, you want to raise a notification. The solutions to this are [outlined in this post](http://stackoverflow.com/a/18311830/313399) with a [code sample here](http://stackoverflow.com/a/15949723/313399).

### Register with Manifest

Finally we need to register the receiver class with GCM in the `AndroidManifest.xml` tagging the type of request (category) of the push:

```xml
 <application
     ...>
         <!-- ... -->
         
         <activity
             ...>
         </activity>
         
         <receiver
             android:name=".GcmBroadcastReceiver"
             android:permission="com.google.android.c2dm.permission.SEND" >
             <intent-filter>
                 <action android:name="com.google.android.c2dm.intent.RECEIVE" />
                 <category android:name="com.codepath.android.gcm" />
             </intent-filter>
         </receiver>
         
         <service android:name=".GcmMessageHandler" />
         
         <meta-data android:name="com.google.android.gms.version"
            android:value="@integer/google_play_services_version" />
     </application>
 
</manifest>
```

Note we also register the `GcmMessageHandler` as a service and setup the play services meta-data required for push messaging to work. Now we just need a web server to keep track of the different device IDs and manage the pushing of messages to different devices.

## Step 3: Setup Web Server

### Overview

Your HTTP web server needs two features:

- 1. Endpoint for mapping a `user_id` to an android `device_id`
- 2. Logic for sending specified `data` messages to sets of android `device_ids`

### 1. Implement Registration Endpoint

We need an endpoint for **registering a user id to a device id**:

  - Probably requires a database table for user to device mappings
  - Create a device_id linked to a registered user for later access
  - Sample endpoint: `POST /register?user_id=123&device_id=abc`

### 2. Implement Sending Logic

We need code for **sending data to specified device ids**:

  - To send GCM pushes, send a POST request to GCM send endpoint with data and registered ids
    - `POST https://android.googleapis.com/gcm/send`
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
        "registration_ids": ["abc", "def"] 
      }
      ```
  
This sending code can be exposed as an endpoint or utilized on the server-side to notify users when new items are created or available.

### Sample Ruby Server Implementation

A simple Sinatra-based ruby application that supports these endpoints might look like:

```ruby
# Device table with three fields (registration id, user id and platform)
class Device
    include Mongoid::Document
    
    field :reg_id, :type => String
    field :user_id, :type => String
    field :os, :type => String, :default => 'android'    
end

# Registration endpoint mapping device_id to user_id
# POST /register?reg_id=abc&user_id=123
post '/register' do
  if Device.where(reg_id: params[:reg_id]).count == 0
    device = Device.new(:reg_id => params[:reg_id], :user => params[:user_id], :os => 'android')
    device.save!
  end
end

# Sending logic
# send_gcm_message(["abc", "cdf"])
def send_gcm_message(title, body, reg_ids)
  # Find devices with the corresponding reg_ids
  devices = Device.where(:reg_id => reg_ids)
  # Construct JSON payload
  post_args = {
    :registration_ids => registration_ids,
    :data => {
      :title  => title,
      :body => body,
      :anything => "foobar"
    }
  }
  # Send the request with JSON args and headers
  RestClient.post 'https://android.googleapis.com/gcm/send', post_args.to_json, 
    :Authorization => 'key=' + settings.AUTHORIZE_KEY, :content_type => :json, :accept => :json
end
```

Now we can register users with device ids and send messages to them when necessary.

## Easy Local Testing with Ruby

We can easily send messages to GCM registered devices using the  ruby [GCM gem](https://github.com/spacialdb/gcm). First we must have [ruby installed](https://www.ruby-lang.org/en/installation/) (default on OSX) and then we can install GCM with:

```
gem install gcm
```

and send test pushes by running `irb` in the command-line:

```ruby
require 'gcm'
gcm = GCM.new("YOUR-SERVER-KEY-HERE")
registration_ids= ["YOUR-DEVICE-ID", "ANOTHER-DEVICE-ID"]
options = { :data => { :title =>"foobar", :body => "this is a longer message" } }
response = gcm.send(registration_ids, options)
```

This will send messages to the devices specified.

## References

- <http://hmkcode.com/android-google-cloud-messaging-tutorial/>
- <https://developer.android.com/google/gcm/client.html>