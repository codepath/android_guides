## Overview

Intents allow us to communicate data between Android apps and implicit intents accept actions. One of those actions is the `ACTION_SEND` command which indicates we want to send data across apps. To send data, all you need to do is specify the data and its type, and the system will identify compatible receiving activities and display them to the user.

Sending and receiving data between applications with intents is most commonly used for social sharing of content. Intents allow users to share information quickly and easily, using their favorite applications.

### Sharing Local Content

You can send content by invoking an implicit intent with `ACTION_SEND`. 

#### Sending HTML

```java
Intent sharingIntent = new Intent(Intent.ACTION_SEND);
sharingIntent.setType("text/html");
sharingIntent.putExtra(android.content.Intent.EXTRA_TEXT, Html.fromHtml("<p>This is the text shared.</p>"));
startActivity(Intent.createChooser(sharingIntent,"Share using"));
```

#### Sending Images

To send images or binary data:

```java
Intent shareIntent = new Intent(Intent.ACTION_SEND);
shareIntent.setType("image/jpg");
Uri uri = Uri.fromFile(new File(getFilesDir(), "foo.jpg"));
shareIntent.putExtra(Intent.EXTRA_STREAM, uri.toString());
startActivity(Intent.createChooser(sharingIntent, "Share image using"));
```

### Sharing Remote Images

You may want to send an image that were loaded from a remote URL. Assuming you are using a third party library like `SmartImageView`, here is how you might share an image that came from the network and was loaded into an ImageView. First, we need to load the bitmap into the view from the URL:

```java
// Get access to SmartImageView object and the loaded Bitmap 
SmartImageView ivImage = (SmartImageView) findViewById(R.id.ivResult);
ivImage.setImageUrl(imageUrl);
```

and then later assuming after the image has completed loading, this is how you can trigger a share:

```java
// Can be triggered by a view event such as a button press
public void onShareItem(View v) {
    // Get access to bitmap image from view
    SmartImageView ivImage = (SmartImageView) findViewById(R.id.ivResult);
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
        File file =  new File(Environment.getExternalStoragePublicDirectory(  
	        Environment.DIRECTORY_DOWNLOADS), "share_image_" + System.currentTimeMillis() + ".png");
        file.getParentFile().mkdirs();
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

Make sure to add the appropriate permissions to your `AndroidManifest.xml`:

```xml
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
```

and setup the "SD Card" within the emulator device settings:

<img src="http://i.imgur.com/nvA2ZKz.png" width="300" />

### ShareActionProvider

This is how you can easily use an ActionBar share icon to activate a ShareIntent. We need to add an ActionBar menu item in `res/menu/` in the XML specifying the `ShareActionProvider` class.

**Note:** This is **an alternative to using a sharing intent** as described in the previous section. You either can use a sharing intent **or** the provider as described below. Also, `ShareActionProvider` is only available in API 14 or above unless the supportv7 library is included.

**Note:** The following assumes the use of a third-party image library `SmartImageView` with a release that supports callbacks such as [this smart-image-view jar](https://www.dropbox.com/s/k2ljqalmzlqymkh/android-smart-image-view-3-27-14.jar), note that the following code snippet won't work with certain older versions of `SmartImageView`.

```xml
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <item
            android:id="@+id/menu_item_share"
            android:showAsAction="ifRoom"
            android:title="Share"
            android:actionProviderClass=
                "android.widget.ShareActionProvider" />
    ...
</menu>
```

Get access to share provider menu item in Java so you can attach the share intent later:

```java
private ShareActionProvider miShareAction;

@Override
public boolean onCreateOptionsMenu(Menu menu) {
    // Inflate menu resource file.
    getMenuInflater().inflate(R.menu.second_activity, menu);
    // Locate MenuItem with ShareActionProvider
    MenuItem item = menu.findItem(R.id.menu_item_share);
    // Fetch reference to the share action provider
    miShareAction = (ShareActionProvider) item.getActionProvider();
    // Return true to display menu
    return true;
}
```

Construct and attach the share intent for the provider by calling this method when the image has loaded:

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    // ...
    // Get access to SmartImageView object and then load Bitmap 
    SmartImageView ivImage = (SmartImageView) findViewById(R.id.ivResult);
    ivImage.setImageUrl(result.getFullUrl(), new OnCompleteListener() {
        @Override
        public void onComplete() {
            // Setup share intent now that image has loaded
            setupShareIntent();
        }
    });
}

// Gets the image URI and setup the associated share intent to hook into the provider
public void setupShareIntent() {
    // Fetch Bitmap Uri locally
    SmartImageView ivImage = (SmartImageView) findViewById(R.id.ivResult);
    Uri bmpUri = getLocalBitmapUri(ivImage); // see previous remote images section
    // Create share intent as described above
    Intent shareIntent = new Intent();
    shareIntent.setAction(Intent.ACTION_SEND);
    shareIntent.setAction(Intent.ACTION_SEND);
    shareIntent.putExtra(Intent.EXTRA_STREAM, bmpUri);
    shareIntent.setType("image/*");
    // Attach share event to the menu item provider
    miShareAction.setShareIntent(shareIntent);
}
```

Make sure to add the appropriate permissions to your `AndroidManifest.xml`:

```xml
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
```

Check out the [official guide for easy sharing](http://developer.android.com/training/sharing/shareaction.html) for more information.

## References

* <http://developer.android.com/training/sharing/send.html>
* <http://developer.android.com/training/sharing/shareaction.html>
* <http://developer.android.com/reference/android/widget/ShareActionProvider.html>