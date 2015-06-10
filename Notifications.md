## Overview

A notification is a message you can display to the user outside of your application's normal UI. Notifications appear in the phone's notification area and then can be expanded to see more information.

This is typically used to keep the user informed about events that are coming in that they should be aware of such as new email messages, new chat messages or an upcoming calendar event.

Notifications can include actions that let the user take a relevant action from within the notification drawer. At the minimum, all notifications consist of a base layout including:

 * Icon of the app or relevant user image 
 * Title and message
 * Timestamp

For a more detailed look at the design of notifications check out the [Notifications Design](http://developer.android.com/design/patterns/notifications.html) guide.

## Usage

Creating notifications is fairly straightforward. First you construct the notification using the [NotificationCompat.Builder](http://developer.android.com/reference/android/support/v4/app/NotificationCompat.Builder.html) and then you add the notification to the [NotificationManager](http://developer.android.com/reference/android/app/NotificationManager.html).

### Creating a Notification

Let's build a basic notification using the [NotificationCompat.Builder](http://developer.android.com/reference/android/support/v4/app/NotificationCompat.Builder.html). Typically this will contain at least an icon, a title, body:

```java
NotificationCompat.Builder mBuilder =
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

The action inside the notification should be a [PendingIntent](http://developer.android.com/reference/android/app/PendingIntent.html) which will fire when the user presses on the notification item.

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
// Now we can attach this to the notification using setContentIntent
Notification noti =
        new NotificationCompat.Builder(this)
        .setSmallIcon(R.drawable.notification_icon)
        .setContentTitle("My notification")
        .setContentText("Hello World!")
        .setContentIntent(pIntent).build();
        
// Hide the notification after its selected
noti.setAutoCancel(true);

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

<img src="http://i.imgur.com/iuPmvzA.png" />

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