Let's take a look at the most common implicit intents such as making a phone call, launching a web address, sending an email, etc.

## Phone Call

Permissions:

```xml
<uses-permission android:name="android.permission.CALL_PHONE" />
```

Intent:

```java
Intent callIntent = new Intent(Intent.ACTION_CALL);
callIntent.setData(Uri.parse("tel:0377778888"));
if (callIntent.resolveActivity(getPackageManager()) != null) {
    startActivity(callIntent);
}
```

> **Caution** It's possible that a user won't have any apps that handle the implicit intent you send to *startActivity()*. If that happens, the call will fail and your app will crash. To verify that an activity will receive the intent, call *resolveActivity()* on your *Intent* object. If the result is non-null, then there is at least one app that can handle the intent and it's safe to call *startActivity()*. If the result is null, you should not use the intent and, if possible, you should disable the feature that issue the intent.

## Send Email (to Phone Email Client)

Compose an email in the phone email client:

```java
Intent intent = new Intent(Intent.ACTION_SEND);
intent.setType("plain/text");
intent.putExtra(Intent.EXTRA_EMAIL, new String[] { "some@email.address" });
intent.putExtra(Intent.EXTRA_SUBJECT, "subject");
intent.putExtra(Intent.EXTRA_TEXT, "mail body");
if (intent.resolveActivity(getPackageManager()) != null) {
    startActivity(Intent.createChooser(intent, ""));
}
```

## Send Email (to Gmail)

Gmail does not examine the extra Intent fields, so in order to use this intent, you need to use the `Intent.ACTION_SENDTO` and pass a `mailto:` URI with the subject and body URL encoded.

```java
String uriText =
    "mailto:youremail@gmail.com" + 
    "?subject=" + Uri.encode("some subject text here") + 
    "&body=" + Uri.encode("some text here");

Uri uri = Uri.parse(uriText);

Intent sendIntent = new Intent(Intent.ACTION_SENDTO);
sendIntent.setData(uri);
if (sendIntent.resolveActivity(getPackageManager()) != null) {
   startActivity(Intent.createChooser(sendIntent, "Send email")); 
}
```

## Launch Website

Launch a website in the phone browser:

```java
Intent browserIntent = new Intent(Intent.ACTION_VIEW, Uri.parse("http://www.google.com"));
if (browserIntent.resolveActivity(getPackageManager()) != null) {
    startActivity(browserIntent);
}
```

You can also launch a Chrome tab if the app.  Take a look at [[this guide|Chrome-Custom-Tabs#setup]] for how to launch this implicit intent.

## Open Google Play Store

Open app page on Google Play:

```java
Intent intent = new Intent(Intent.ACTION_VIEW, 
  Uri.parse("market://details?id=" + context.getPackageName()));
if (intent.resolveActivity(getPackageManager()) != null) {
    startActivity(intent);
}

```

## Compose SMS

```java
Uri smsUri = Uri.parse("tel:" + to);
Intent intent = new Intent(Intent.ACTION_VIEW, smsUri);
intent.putExtra("address", to);
intent.putExtra("sms_body", message);
intent.setType("vnd.android-dir/mms-sms");//here setType will set the previous data null.
if (intent.resolveActivity(getPackageManager()) != null) {
    startActivity(intent);
}
```

## Google Maps
 
Show location in maps application:

```java
 Intent intent = new Intent();
intent.setAction(Intent.ACTION_VIEW);
String data = String.format("geo:%s,%s", latitude, longitude);
if (zoomLevel != null) {
    data = String.format("%s?z=%s", data, zoomLevel);
}
intent.setData(Uri.parse(data));
if (intent.resolveActivity(getPackageManager()) != null) {
    startActivity(intent);
}

```

## Capture Photo

```java
Uri uri = Uri.fromFile(new File(file));
Intent intent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
intent.putExtra(MediaStore.EXTRA_OUTPUT, uri);
if (intent.resolveActivity(getPackageManager()) != null) {
    startActivity(intent);
}

```

## Sharing Content

Images or binary data:

```java
Intent sharingIntent = new Intent(Intent.ACTION_SEND);
sharingIntent.setType("image/jpg");
Uri uri = Uri.fromFile(new File(getFilesDir(), "foo.jpg"));
sharingIntent.putExtra(Intent.EXTRA_STREAM, uri.toString());
if (sharingIntent.resolveActivity(getPackageManager()) != null) {
    startActivity(Intent.createChooser(sharingIntent, "Share image using"));
}
```

or HTML:

```java
Intent sharingIntent = new Intent(Intent.ACTION_SEND);
sharingIntent.setType("text/html");
sharingIntent.putExtra(android.content.Intent.EXTRA_TEXT, Html.fromHtml("<p>This is the text shared.</p>"));
if (sharingIntent.resolveActivity(getPackageManager()) != null) {
    startActivity(Intent.createChooser(sharingIntent,"Share using"));
}
```