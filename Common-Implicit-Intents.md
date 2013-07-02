### Phone Call

Permissions:

```xml
<uses-permission android:name="android.permission.CALL_PHONE" />
```

Intent:

```java
Intent callIntent = new Intent(Intent.ACTION_CALL);
callIntent.setData(Uri.parse("tel:0377778888"));
startActivity(callIntent);
```

### Send Email

Compose an email in the phone email client:

```java
Intent intent = new Intent(Intent.ACTION_SEND);
intent.setType("plain/text");
intent.putExtra(Intent.EXTRA_EMAIL, new String[] { "some@email.address" });
intent.putExtra(Intent.EXTRA_SUBJECT, "subject");
intent.putExtra(Intent.EXTRA_TEXT, "mail body");
startActivity(Intent.createChooser(intent, ""));
```

### Launch Website

Launch a website in the phone browser:

```java
Intent browserIntent = new Intent(Intent.ACTION_VIEW, Uri.parse("http://www.google.com"));
startActivity(browserIntent);
```

### Open Google Play Store

Open app page on Google Play:

```java
Intent i = new Intent(Intent.ACTION_VIEW, 
  Uri.parse("market://details?id=" + context.getPackageName()));
startActivity(i);
```

### Compose SMS

```java
Uri smsUri = Uri.parse("tel:" + to);
Intent intent = new Intent(Intent.ACTION_VIEW, smsUri);
intent.putExtra("address", to);
intent.putExtra("sms_body", message);
intent.setType("vnd.android-dir/mms-sms");
startActivity(intent);
```

### Google Maps:

Show location in maps application:

```java
 Intent intent = new Intent();
intent.setAction(Intent.ACTION_VIEW);
String data = String.format("geo:%s,%s", latitude, longitude);
if (zoomLevel != null) {
    data = String.format("%s?z=%s", data, zoomLevel);
}
intent.setData(Uri.parse(data));
startActivity(intent);
```