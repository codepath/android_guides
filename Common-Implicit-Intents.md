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
startActivity(callIntent);
```

## Send Email (to Phone Email Client)

Compose an email in the phone email client:

```java
Intent intent = new Intent(Intent.ACTION_SEND);
intent.setType("plain/text");
intent.putExtra(Intent.EXTRA_EMAIL, new String[] { "some@email.address" });
intent.putExtra(Intent.EXTRA_SUBJECT, "subject");
intent.putExtra(Intent.EXTRA_TEXT, "mail body");
startActivity(Intent.createChooser(intent, ""));
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
startActivity(Intent.createChooser(sendIntent, "Send email")); 
```

## Launch Website

Launch a website in the phone browser:

```java
Intent browserIntent = new Intent(Intent.ACTION_VIEW, Uri.parse("http://www.google.com"));
startActivity(browserIntent);
```

## Open Google Play Store

Open app page on Google Play:

```java
Intent intent = new Intent(Intent.ACTION_VIEW, 
  Uri.parse("market://details?id=" + context.getPackageName()));
startActivity(intent);
```

## Compose SMS

```java
Uri smsUri = Uri.parse("tel:" + to);
Intent intent = new Intent(Intent.ACTION_VIEW, smsUri);
intent.putExtra("address", to);
intent.putExtra("sms_body", message);
intent.setType("vnd.android-dir/mms-sms");
startActivity(intent);
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
startActivity(intent);
```

## Capture Photo

```java
Uri uri = Uri.fromFile(new File(file));
Intent intent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
intent.putExtra(MediaStore.EXTRA_OUTPUT, uri);
startActivity(intent);
```

## Sharing Content

Images or binary data:

```java
Intent sharingIntent = new Intent(Intent.ACTION_SEND);
sharingIntent.setType("image/jpg");
Uri uri = Uri.fromFile(new File(getFilesDir(), "foo.jpg"));
sharingIntent.putExtra(Intent.EXTRA_STREAM, uri.toString());
startActivity(Intent.createChooser(sharingIntent, "Share image using"));
```

or HTML:

```java
Intent sharingIntent = new Intent(Intent.ACTION_SEND);
sharingIntent.setType("text/html");
sharingIntent.putExtra(android.content.Intent.EXTRA_TEXT, Html.fromHtml("<p>This is the text shared.</p>"));
startActivity(Intent.createChooser(sharingIntent,"Share using"));
```