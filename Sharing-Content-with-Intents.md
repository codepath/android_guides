## Overview

Intents allow us to communicate data between Android apps and implicit intents can also accept actions. One of those actions is the `ACTION_SEND` command which indicates we want to send data across apps. To send data, all you need to do is specify the data and its type, and the system will identify compatible receiving activities and display them to the user.

Sending and receiving data between applications with intents is most commonly used for social sharing of content. Intents allow users to share information quickly and easily, using their favorite applications.

### Sharing Local Content

You can send content by invoking an implicit intent with `ACTION_SEND`. 

#### Sending HTML

```java
Intent sharingIntent = new Intent(Intent.ACTION_SEND);
sharingIntent.setType("text/html");
sharingIntent.putExtra(android.content.Intent.EXTRA_TEXT, Html.fromHtml("<p>This is the text shared.</p>"));
startActivity(Intent.createChooser(sharingIntent, "Share using"));
```

#### Sending Images

To send images or binary data:

```java
final Intent shareIntent = new Intent(Intent.ACTION_SEND);
shareIntent.setType("image/jpg");
final File photoFile = new File(getFilesDir(), "foo.jpg");
shareIntent.putExtra(Intent.EXTRA_STREAM, Uri.fromFile(photoFile));
startActivity(Intent.createChooser(shareIntent, "Share image using"));
```

#### Sending Links

Sending URL links should simply use `text/plain` type:

```java
Intent shareIntent = new Intent(Intent.ACTION_SEND);
shareIntent.setType("text/plain"); 
shareIntent.putExtra(Intent.EXTRA_TEXT, "http://codepath.com");
startActivity(Intent.createChooser(shareIntent, "Share link using"));
```

#### Sharing Multiple Types

In certain cases, we might want to send an image along with text. This can be done with:

```java
String text = "Look at my awesome picture";
Uri pictureUri = Uri.parse("file://my_picture");
Intent shareIntent = new Intent();
shareIntent.setAction(Intent.ACTION_SEND);
shareIntent.putExtra(Intent.EXTRA_TEXT, text);
shareIntent.putExtra(Intent.EXTRA_STREAM, imageUri);
shareIntent.setType("image/*");
shareIntent.addFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION);
startActivity(Intent.createChooser(shareIntent, "Share images..."));
```

Sharing multiple images can be done with:

```java
Intent shareIntent = new Intent(Intent.ACTION_SEND_MULTIPLE);
shareIntent.putParcelableArrayListExtra(Intent.EXTRA_STREAM, imageUris);
shareIntent.setType("image/*");
```

See this [stackoverflow post](http://stackoverflow.com/a/27501666) for more details. 

**Note:** Facebook does not properly recognize multiple shared elements. See [this facebook specific bug for more details](https://developers.facebook.com/x/bugs/332619626816423/) and share using their SDK. 

#### Share in Facebook

Facebook doesn't work well with normal sharing intents when sharing multiple content elements as [discussed in this bug](https://developers.facebook.com/x/bugs/332619626816423/). To share posts with facebook, we need to:

1. Create a [new Facebook app here](https://developers.facebook.com/apps/) (follow the instructions)
2. Add the Facebook SDK to your Android project
3. Share using this code snippet:

```java
   public void setupFacebookShareIntent() {
        ShareDialog shareDialog;
        FacebookSdk.sdkInitialize(getApplicationContext());
        shareDialog = new ShareDialog(this);

        ShareLinkContent linkContent = new ShareLinkContent.Builder()
                .setContentTitle("Title")
                .setContentDescription(
                        "\"Body Of Test Post\"")
                .setContentUrl(Uri.parse("http://someurl.com/here"))
                .build();
        
        shareDialog.show(linkContent);
    }
```

### Sharing Remote Images

You may want to send an image that were loaded from a remote URL. Assuming you are using a third party library like `Picasso`, here is how you might share an image that came from the network and was loaded into an ImageView.  There are two ways to accomplish this.   The first way, shown below, takes the bitmap from the view and loads it into a file.  

```java
// Get access to ImageView 
ImageView ivImage = (ImageView) findViewById(R.id.ivResult);
// Fire async request to load image
Picasso.with(context).load(imageUrl).into(ivImage);
```

and then later assuming after the image has completed loading, this is how you can trigger a share:

```java
// Can be triggered by a view event such as a button press
public void onShareItem(View v) {
    // Get access to bitmap image from view
    ImageView ivImage = (ImageView) findViewById(R.id.ivResult);
    // Get access to the URI for the bitmap
    Uri bmpUri = getLocalBitmapUri(ivImage);
    if (bmpUri != null) {
        // Construct a ShareIntent with link to image
        Intent shareIntent = new Intent();
        shareIntent.setAction(Intent.ACTION_SEND);
        shareIntent.putExtra(Intent.EXTRA_STREAM, bmpUri);
        shareIntent.setType("image/*");
        // Launch sharing dialog for image
        startActivity(Intent.createChooser(shareIntent, "Share Image"));	
    } else {
        // ...sharing failed, handle error
    }
}

// Returns the URI path to the Bitmap displayed in specified ImageView
public Uri getLocalBitmapUri(ImageView imageView) {
    // Extract Bitmap from ImageView drawable
    Drawable drawable = imageView.getDrawable();
    Bitmap bmp = null;
    if (drawable instanceof BitmapDrawable){
       bmp = ((BitmapDrawable) imageView.getDrawable()).getBitmap();
    } else {
       return null;
    }
    // Store image to default external storage directory
    Uri bmpUri = null;
    try {
        // Use methods on Context to access package-specific directories on external storage.
        // This way, you don't need to request external read/write permission.
        // See https://youtu.be/5xVh-7ywKpE?t=25m25s
        File file =  new File(getExternalFilesDir(Environment.DIRECTORY_PICTURES), "share_image_" + System.currentTimeMillis() + ".png");
        FileOutputStream out = new FileOutputStream(file);
        bmp.compress(Bitmap.CompressFormat.PNG, 90, out);
        out.close();
        bmpUri = Uri.fromFile(file);
    } catch (IOException e) {
        e.printStackTrace();
    }
    return bmpUri;
}
```

Make sure to setup the "SD Card" within the emulator device settings:

<img src="https://i.imgur.com/nvA2ZKz.png" width="300" />

### Sharing files with API 24 or higher

If you are using Android API 24 or higher, private File URI resources (file:///) cannot be shared.  You must wrap the File object as a content provider (content://) using the [FileProvider](https://developer.android.com/reference/android/support/v4/content/FileProvider.html) class.

First, you must declare this FileProvider in your `AndroidManifest.xml` file within the `<application>` tag:
  
```xml
<provider
  android:name="android.support.v4.content.FileProvider"
  android:authorities="com.codepath.fileprovider"
  android:exported="false"
  android:grantUriPermissions="true">
    <meta-data
            android:name="android.support.FILE_PROVIDER_PATHS"
            android:resource="@xml/fileprovider" />
</provider>
```

Next, create a resource directory called `xml` and create a `fileprovider.xml`.  This declaration will give access to the application's external path directory.

```xml
<?xml version="1.0" encoding="utf-8"?>
<paths>
    <external-files-path
        name="images"
        path="Pictures" />
</paths>
```

Finally, you will convert the File object into a content provider using the FileProvider class:

```java
// getExternalFilesDir() + "/Pictures" should match the declaration in fileprovider.xml paths
File file = new File(getExternalFilesDir(Environment.DIRECTORY_PICTURES), "share_image_" + System.currentTimeMillis() + ".png");

// wrap File object into a content provider
bmpUri = FileProvider.getUriForFile(MyActivity.this, "com.codepath.fileprovider", file);
```

### Sharing Remote Images (without explicit file IO)

The second way to share an Image does not require you to write the image into a file.  This code can safely be executed on the UI thread. The approach was suggested on this webpage http://www.nurne.com/2012/07/android-how-to-attach-image-file-from.html. 

```java
ImageView siv = (ImageView) findViewById(R.id.ivResult);
Drawable mDrawable = siv.getDrawable();
Bitmap mBitmap = ((BitmapDrawable)mDrawable).getBitmap();

String path = Images.Media.insertImage(getContentResolver(), 
    mBitmap, "Image Description", null);

Uri uri = Uri.parse(path);
return uri;
```

You get the `Drawable` from the `ImageView`.  You get the `Bitmap` from the `Drawable`.  Put that bitmap into the Media image store.  That gives you a path which can be used instead of a file path or URL.  Note the original webpage had an additional problem with immutable bitmaps, solved by drawing the bitmap into a canvas (never shown on screen).  See linked page above for details.

**Note:** There is a [common bug on emulators](https://code.google.com/p/android/issues/detail?id=75447) that will cause `MediaStore.Images.Media.insertImage` to fail with `E/MediaStore﹕ Failed to insert image` unless the media directory is first initialized as described in the link.

### ShareActionProvider

<img src="https://imgur.com/B8hEYut.png"/>

This is how you can easily use an ActionBar share icon to activate a ShareIntent. The below focuses on the **support ShareActionProvider** for use with `AppCompatActivity`.

**Note:** This is **an alternative to using a sharing intent** as described in the previous section. You either can use a sharing intent **or** the provider as described below. 

First, we need to add an ActionBar menu item in `res/menu/` in the XML specifying the `ShareActionProvider` class. 

```xml
<menu xmlns:android="http://schemas.android.com/apk/res/android"
      xmlns:app="http://schemas.android.com/apk/res-auto">

    <item
        android:id="@+id/menu_item_share"
        app:showAsAction="ifRoom"
        android:title="Share"
        app:actionProviderClass="android.support.v7.widget.ShareActionProvider" />
    ...
</menu>
```

Next, get access to share provider menu item in the Activity so we can attach the share intent later:

```java
private ShareActionProvider miShareAction;

@Override
public boolean onCreateOptionsMenu(Menu menu) {
    // Inflate menu resource file.
    getMenuInflater().inflate(R.menu.second_activity, menu);
    // Locate MenuItem with ShareActionProvider
    MenuItem item = menu.findItem(R.id.menu_item_share);
    // Fetch reference to the share action provider
    miShareAction = (ShareActionProvider) MenuItemCompat.getActionProvider(item);
    // Return true to display menu
    return true;
}
```

**Note**: `ShareActionProvider` does not respond to `onOptionsItemSelected()` events, so you set the share action provider as soon as it is possible. 

#### Attach Share Intent for Remote Image

Now, once you've setup the ShareActionProvider menu item, construct and attach the share intent for the provider but only **after image has been loaded** as shown below using the `Callback` for `Picasso`.

```java

private Intent shareIntent;

@Override
protected void onCreate(Bundle savedInstanceState) {
    // ...
    // Get access to ImageView
    ImageView ivImage = (ImageView) findViewById(R.id.ivResult);
    // Load image async from remote URL, setup share when completed
    Picasso.with(this).load(result.getFullUrl()).into(ivImage, new Callback() {
        @Override
        public void onSuccess() {
            // Setup share intent now that image has loaded
            prepareShareIntent();
            attachShareIntentAction();
        }
        
        @Override
        public void onError() { 
            // ...
        }
   });
}

// Gets the image URI and setup the associated share intent to hook into the provider
public void prepareShareIntent() {
    // Fetch Bitmap Uri locally
    ImageView ivImage = (ImageView) findViewById(R.id.ivResult);
    Uri bmpUri = getLocalBitmapUri(ivImage); // see previous remote images section
    // Construct share intent as described above based on bitmap
    shareIntent = new Intent();
    shareIntent.setAction(Intent.ACTION_SEND);
    shareIntent.putExtra(Intent.EXTRA_STREAM, bmpUri);
    shareIntent.setType("image/*");   
}

// Attaches the share intent to the share menu item provider
public void attachShareIntentAction() {
    if (miShareAction != null && shareIntent != null)
        miShareAction.setShareIntent(shareIntent);
}

@Override
public boolean onCreateOptionsMenu(Menu menu) {
    // Inflate menu resource file.
    getMenuInflater().inflate(R.menu.second_activity, menu);
    // Locate MenuItem with ShareActionProvider
    MenuItem item = menu.findItem(R.id.menu_item_share);
    // Fetch reference to the share action provider
    miShareAction = (ShareActionProvider) MenuItemCompat.getActionProvider(item);
    attachShareIntentAction(); // call here in case this method fires second
    // Return true to display menu
    return true;
}
```

#### Attach Share for a WebView URL

We can use a similar approach if we wish to create a share action for the current URL that is being loaded in a [[WebView|Working-with-the-WebView]]:

```java
@Override
public boolean onCreateOptionsMenu(Menu menu) {
  MenuInflater inflater = getMenuInflater();
  inflater.inflate(R.menu.menu_article, menu);

  MenuItem item = menu.findItem(R.id.menu_item_share);
  ShareActionProvider miShare = (ShareActionProvider) MenuItemCompat.getActionProvider(item);
  Intent shareIntent = new Intent(Intent.ACTION_SEND);
  shareIntent.setType("text/plain"); 

  // get reference to WebView
  WebView wvArticle = (WebView) findViewById(R.id.wvArticle);
  // pass in the URL currently being used by the WebView
  shareIntent.putExtra(Intent.EXTRA_TEXT, wvArticle.getUrl());

  miShare.setShareIntent(shareIntent);
  return super.onCreateOptionsMenu(menu);
}
```

Check out the [official guide for easy sharing](http://developer.android.com/training/sharing/shareaction.html) for more information.

## References

* <http://developer.android.com/training/sharing/send.html>
* <http://developer.android.com/training/sharing/shareaction.html>
* <http://developer.android.com/reference/android/widget/ShareActionProvider.html>
* <https://medium.com/@benexus/dealing-with-permissions-when-sharing-files-android-m-cee9ecc287bf>