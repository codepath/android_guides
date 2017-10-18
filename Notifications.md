## Overview

A notification is a message you can display to the user outside of your application's normal UI. Notifications appear in the phone's notification area and then can be expanded to see more information.

This is typically used to keep the user informed about events that are coming in that they should be aware of such as new email messages, new chat messages or an upcoming calendar event.

Notifications can include actions that let the user take a relevant action from within the notification drawer. At the minimum, all notifications consist of a base layout including:

 * Icon of the app or relevant user image 
 * Title and message
 * Timestamp

Starting with API level 26 (Oreo), notification channels were introduced to give more control over the notifications for users. Notification channels can be enabled/disabled by the user in the Settings app or upon long clicking the notification. Notifications are sorted based on the importance level of the channels. The developers targeting API level 26 and above need to set a channel to a notification at the minimum.  

For a more detailed look at the design of notifications check out the [Notifications Design](http://developer.android.com/design/patterns/notifications.html) guide.

## Usage

Creating notifications is fairly straightforward. First you construct the notification using the [NotificationCompat.Builder](http://developer.android.com/reference/android/support/v4/app/NotificationCompat.Builder.html) and then you add the notification to the [NotificationManager](http://developer.android.com/reference/android/app/NotificationManager.html).

Beginning with API Level 26, **a notification must belong to a channel** as shown below. 

### Creating a Notification Channel

For **API level 26 and above**, we need to create a [notification channel](https://developer.android.com/reference/android/app/NotificationChannel.html), provide a unique string id, name, and [importance](https://developer.android.com/reference/android/app/NotificationChannel.html#setImportance(int)) level. Setting the description helps the user to identify the channel. Create the notification channel in the app's [application](http://guides.codepath.com/android/Understanding-the-Android-Application-Class) class. Creating a notification channel again with same id updates the name and description with the new values.

```java
// Configure the channel
int importance = NotificationManager.IMPORTANCE_DEFAULT;
NotificationChannel channel = new NotificationChannel("myChannelId", "My Channel", importance);
channel.setDescription("Reminders");
// Register the channel with the notifications manager
NotificationManager mNotificationManager =
                (NotificationManager) context.getSystemService(Context.NOTIFICATION_SERVICE);
mNotificationManager.createNotificationChannel(channel);
```

An app can create multiple channels with different ids. Also, one can delete existing channels, however, be aware that they are visible in the settings app. 

### Creating a Notification

Let's build a basic notification using the [NotificationCompat.Builder](http://developer.android.com/reference/android/support/v4/app/NotificationCompat.Builder.html). Typically this will contain at least an icon, a title, body. 

For **API level 26 and above**, we need to set a notification channel using the updated [NotificationCompat.Builder](https://developer.android.com/reference/android/support/v4/app/NotificationCompat.Builder.html#NotificationCompat.Builder).

```java
NotificationCompat.Builder mBuilder =
        // Builder class for devices targeting API 26+ requires a channel ID
        new NotificationCompat.Builder(this, "myChannelId")
        .setSmallIcon(R.drawable.notification_icon)
        .setContentTitle("My notification")
        .setContentText("Hello World!");
```

For **API level 25 and below**, we create the notification without any channel:

```java
NotificationCompat.Builder mBuilder =
        // this Builder class is deprecated
        new NotificationCompat.Builder(this)
        .setSmallIcon(R.drawable.notification_icon)
        .setContentTitle("My notification")
        .setContentText("Hello World!");
```

Then we just have to append the notification using the [NotificationManager](http://developer.android.com/reference/android/app/NotificationManager.html):

```java
NotificationManager mNotificationManager =
    (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
// mId allows you to update the notification later on.
mNotificationManager.notify(mId, mBuilder.build());
```

A generic method for creating a simple notification might be:

```java
//  createNotification(56, R.drawable.ic_launcher, "New Message", 
//      "There is a new message from Bob!");
private void createNotification(int nId, int iconRes, String title, String body) {
	NotificationCompat.Builder mBuilder = new NotificationCompat.Builder(
			this).setSmallIcon(iconRes)
			.setContentTitle(title)
			.setContentText(body);

	NotificationManager mNotificationManager = 
            (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
	// mId allows you to update the notification later on.
	mNotificationManager.notify(nId, mBuilder.build());
}
```

### Adding an Action

Notifications can also have actions attached that the user can perform by clicking. Although having an action attached to the notification is optional, you should almost always have at least one action attached.

In order for the notification service to launch an intent in the original application without necessarily starting it up, the notification service will need to use the same permissions required by this application.  A [pending intent](http://stackoverflow.com/questions/2808796/what-is-an-android-pendingintent) is created, which wraps an intent with a token that grants these permissions to the notifications service.  For more information about pending intents, see the [official Android documentation](http://developer.android.com/reference/android/app/PendingIntent.html).

```java
// First let's define the intent to trigger when notification is selected
// Start out by creating a normal intent (in this case to open an activity)
Intent intent = new Intent(this, SomeActivity.class);
// Next, let's turn this into a PendingIntent using
//   public static PendingIntent getActivity(Context context, int requestCode, 
//       Intent intent, int flags)
int requestID = (int) System.currentTimeMillis(); //unique requestID to differentiate between various notification with same NotifId
int flags = PendingIntent.FLAG_CANCEL_CURRENT; // cancel old intent and create new one
PendingIntent pIntent = PendingIntent.getActivity(this, requestID, intent, flags);
// Now we can attach the pendingIntent to a new notification using setContentIntent
Notification noti = new NotificationCompat.Builder(this)
        .setSmallIcon(R.drawable.notification_icon)
        .setContentTitle("My notification")
        .setContentText("Hello World!")
        .setContentIntent(pIntent)
        .setAutoCancel(true) // Hides the notification after its been selected
        .build();
// Get the notification manager system service
NotificationManager mNotificationManager = 
    (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
// mId allows you to update the notification later on.
mNotificationManager.notify(0, noti);
```

You can also add up to three additional action buttons as well:

```java
// Setup your pending intents first and 
// then call addAction(int icon, CharSequence title, PendingIntent intent)
Notification noti = new NotificationCompat.Builder(this).
    .....
    .setContentIntent(pIntent)
    .addAction(R.drawable.icon, "Share", sIntent)
    .addAction(R.drawable.icon, "Ignore", iIntent).build();
```

### Adding Direct Reply Support

Android N (API 24) now supports the ability to respond directly to a notification.    The UI contains action icons below the notification.  In the example below, there are 3 different actions to take:

<img src="https://1.bp.blogspot.com/-l-WaE4Rkd6Q/V1hWYDm2slI/AAAAAAAADIQ/0aV2uhEUGNcsRUTpGUsSk2MVPQnwuLywQCLcB/s400/image01.png"/>

First, let's assume we are setting up the notification without this functionality:
```java
// Use standard notification manager builder class
NotificationCompat.Builder builder = new NotificationCompat.Builder(this)
                .setSmallIcon(R.drawable.ic_launcher)
                .setContentTitle("Image Download Complete!");
```

To leverage the direct reply, you need to setup an activity or service to receive it.  In the example below, we will setup an [[intent service|Starting-Background-Services#creating-an-intentservice]] called `DirectReplyIntent`.  Create this intent service and make sure to declare this service in your `AndroidManifest.xml` file:

```java
public class DirectReplyIntent extends IntentService {

    public static String KEY_TEXT_REPLY = "key_text_reply";
    public static String KEY_NOTIFY_ID = "key_notify_id";

    public DirectReplyIntent() {
        super("DirectReplyIntent");
    }

    @Override
    protected void onHandleIntent(Intent intent) {
      // handle notification here
    }
```

Make sure to include this new intent service in your `AndroidManifest.xml`:

```xml
<application>
  <service android:name=".services.DirectReplyIntent"/>
</application>
```

Next, we will create a pending intent to launch this service assuming that API version 24 or higher is used:

```java
if (android.os.Build.VERSION.SDK_INT >= 24) {

  // Setup a DirectReply IntentService
  Intent directReplyIntent = new Intent(this, DirectReplyIntent.class);

  // pass the notification ID -- it should be a UUID
  directReplyIntent.putExtra(KEY_NOTIFY_ID, 82);

  // only handle one pending intent at a time
  int flags = FLAG_CANCEL_CURRENT;

  PendingIntent directReplyPendingIntent = PendingIntent.getService(this, 0, directReplyIntent, flags);
```

We need to set the placeholder text and the key name that will be used to store the text by creating a `RemoteInput` class:

```java
  // Set the placeholder text when the user clicks on the Reply button
  // RemoteInput originally came from Android Wear
  public static String KEY_TEXT_REPLY = "key_text_reply";

  RemoteInput remoteInput = new RemoteInput.Builder(KEY_TEXT_REPLY)
                    .setLabel("Type Message").build();
```

We need to use this `RemoteInput` class and create a **notification action**.  This notification action will then be passed to the **notification builder** class.

```java
    // Generate the notification action
    NotificationCompat.Action action = new NotificationCompat.Action.Builder(
                    R.drawable.ic_launcher, "Reply", directReplyPendingIntent)
                    .addRemoteInput(remoteInput).build();
    builder.addAction(action);
  } // end of minimum API version 24 check
```

Finally, we should create the notification.  Note that we need to reference the notification ID so that it can be used to acknowledge it in the reply.  Earlier, we passed this notification ID as part of the pending intent.

```java
// Create the notification outside the minimum API 24 version check
Notification noti = builder.build();
NotificationManager mNotificationManager = (NotificationManager) getSystemService(
                Context.NOTIFICATION_SERVICE);

// NOTIF_ID allows you to update the notification later on.
mNotificationManager.notify(NOTIF_ID, noti);
```

Finally, we need to setup the `DirectReplyIntent` service to receive this message:

```java
public class DirectReplyIntent extends IntentService {

  // Previous declarations here

  // ...continued 
  @Override
  protected void onHandleIntent(Intent intent) {
    CharSequence directReply = getMessageText(intent);
    if (directReply != null) {
       Notification repliedNotification =
         new NotificationCompat.Builder(DirectReplyIntent.this)
                            .setSmallIcon(R.drawable.ic_launcher)
                            .setContentText("Received: " + directReply)
                            .build();

      NotificationManager notificationManager = (NotificationManager) getSystemService(
        Context.NOTIFICATION_SERVICE);

      int notifyId = intent.getIntExtra(KEY_NOTIFY_ID, -1);
      notificationManager.notify(notifyId, repliedNotification);
    }  
  }

  @TargetApi(Build.VERSION_CODES.KITKAT_WATCH)
  private CharSequence getMessageText(Intent intent) {
    // Decode the reply text
    Bundle remoteInput = RemoteInput.getResultsFromIntent(intent);
    if (remoteInput != null) {
      return remoteInput.getCharSequence(KEY_TEXT_REPLY);
    }
    return null;
  }
}
```

The final result:

<img src="https://imgur.com/c3an6Cv.png">

The approach can also be used for receiving text in an activity.  The only difference is that you need to use 
`PendingIntent.getActivity()` instead of `PendingIntent.getService()`, and receiving data needs to use
`getIntent()` in the activity instead of the `onHandleIntent()`.

### Adding an Expanded View

Android 4.1 supports expandable notifications. You can also setup the notification to have an expanded view when pressed which gives additional details. See more about that on the [official guide](http://developer.android.com/guide/topics/ui/notifiers/notifications.html#ApplyStyle);

There are three styles to be used with the big view: big picture style, big text style, Inbox style. The following code demonstrates the usage of the BigTextStyle() which allows to use up to 256 dp.

```java
Notification noti = new NotificationCompat.Builder(this).
.....
.setStyle(new NotificationCompat.BigTextStyle().bigText(longText)) 
```

### Canceling Notifications

The user can dismiss all notification or if you set your notification to auto-cancel it is also removed once the user selects it.
You can also call the cancel() for a specific notification ID on the NotificationManager.

```java
// Store some notification id as `nId`
NotificationManager mNotificationManager = 
  (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
mNotificationManager.cancel(nId);
```

See  the [official notifications guide](http://developer.android.com/guide/topics/ui/notifiers/notifications.html) for more details.

### Adding a Large Icon

If the notification should display a larger higher-quality icon on the left, we can use the `setLargeIcon` method to set a large (64x64) image as the icon. A common usage of this is to show a picture of the person's face that is associated with the notification:

<img src="https://i.imgur.com/iuPmvzA.png" />

This can be done with the following code:

```java
// .setLargeIcon expects a bitmap
Bitmap largeIcon = BitmapFactory.decodeResource(getResources(), 
    R.drawable.my_large_icon);
// Pass in the bitmap as the large icon
NotificationCompat.Builder mBuilder = new NotificationCompat.Builder(this)
    ...
    .setLargeIcon(largeIcon);
```

Note that if the large icon is set, then the small icon supplied will be displayed minified on the right-hand side instead.

## References

 * <http://developer.android.com/guide/topics/ui/notifiers/notifications.html>
 * <http://developer.android.com/design/patterns/notifications.html>
 * <http://www.vogella.com/articles/AndroidNotifications/article.html>
 * <http://www.javacodegeeks.com/2013/08/android-notification-with-sound-and-icon-tutorial.html>
 * <http://developer.android.com/reference/android/app/Notification.html>
 * <http://developer.android.com/reference/android/support/v4/app/NotificationCompat.Builder.html>
 * <http://android-developers.blogspot.com/2016/06/notifications-in-android-n.html>
 * <https://code.tutsplus.com/tutorials/android-o-how-to-use-notification-channels--cms-28616>
