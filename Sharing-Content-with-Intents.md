## Overview

Intents allow us to communicate data between Android apps and implicit intents accept actions. One of those actions is the `ACTION_SEND` command which indicates we want to send data across apps. To send data, all you need to do is specify the data and its type, and the system will identify compatible receiving activities and display them to the user.

Sending and receiving data between applications with intents is most commonly used for social sharing of content. Intents allow users to share information quickly and easily, using their favorite applications.

### Sending Content

You can send content by invoking an implicit intent. 

#### Sending Images

To send images or binary data:

```java
Intent shareIntent = new Intent(Intent.ACTION_SEND);
shareIntent.setType("image/jpg");
Uri uri = Uri.fromFile(new File(getFilesDir(), "foo.jpg"));
shareIntent.putExtra(Intent.EXTRA_STREAM, uri.toString());
startActivity(Intent.createChooser(sharingIntent, "Share image using"));
```

You may want to send an image that comes from a remote URL. Assuming you are using a third party library like SmartImageView, here is how you might share an image that came from the network and was loaded in an ImageView. First, we load the bitmap into the view:

```java
// Get access to SmartImageView object and the loaded Bitmap 
SmartImageView ivImage = (SmartImageView) findViewById(R.id.ivResult);
ivImage.setImageUrl(imageUrl);
```

and then later assuming it has been loaded, this is how you can trigger a share:

```java
public void onShareItem(View v) {
    // Get access to bitmap image from view
    SmartImageView ivImage = (SmartImageView) findViewById(R.id.ivResult);
    Bitmap bitmap = ((BitmapDrawable) ivImage.getDrawable()).getBitmap();
    // Write image to default external storage directory   
    Uri bmpUri = null;
    try {
        File file =  new File(Environment.getExternalStoragePublicDirectory(  
	   	   Environment.DIRECTORY_DOWNLOADS), "share_image.png");  
	FileOutputStream out = new FileOutputStream(file);
	bitmap.compress(Bitmap.CompressFormat.PNG, 90, out);
	out.close();
	bmpUri = Uri.fromFile(file);
    } catch (IOException e) {
	e.printStackTrace();
    }
	
    if (bmpUri != null) {
        // Construct a ShareIntent with link to image
	Intent shareIntent = new Intent();
	shareIntent.setAction(Intent.ACTION_SEND);
	shareIntent.putExtra(Intent.EXTRA_STREAM, bmpUri);
	shareIntent.setType("image/*");
	// Launch sharing dialog for image
	startActivity(Intent.createChooser(shareIntent, "Share Content"));	
    } else {
	// ...sharing failed, handle error
    }
}
```

#### Sending HTML

```java
Intent sharingIntent = new Intent(Intent.ACTION_SEND);
sharingIntent.setType("text/html");
sharingIntent.putExtra(android.content.Intent.EXTRA_TEXT, Html.fromHtml("<p>This is the text shared.</p>"));
startActivity(Intent.createChooser(sharingIntent,"Share using"));
```

### ShareActionProvider

This is how you can easily use an ActionBar share icon to activate a ShareIntent. Add an ActionBar icon in `menu.xml` specifying the `ShareActionProvider` class:

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

Get access to menu item in Java so you can attach the share intent:

```java
@Override
public boolean onCreateOptionsMenu(Menu menu) {
    // Inflate menu resource file.
    getMenuInflater().inflate(R.menu.second_activity, menu);
    // Locate MenuItem with ShareActionProvider
    MenuItem item = menu.findItem(R.id.menu_item_share);
    // Fetch and store ShareActionProvider
    miShareAction = (ShareActionProvider) item.getActionProvider();
    // Return true to display menu
    return true;
}
```

Attach the share intent for the provider:

```java
// Fetch Bitmap Uri locally
Uri bmpUri = getLocalBitmapUri(); // see previous section
// Create share intent as described above
Intent shareIntent = new Intent();
shareIntent.setAction(Intent.ACTION_SEND);
shareIntent.setAction(Intent.ACTION_SEND);
shareIntent.putExtra(Intent.EXTRA_STREAM, bmpUri);
shareIntent.setType("image/*");
// Attach share event to the menu item provider
miShareAction.setShareIntent(shareIntent);
```

Check out the [official guide for easy sharing](http://developer.android.com/training/sharing/shareaction.html) for more information.

## References

* <http://developer.android.com/training/sharing/send.html>
* <http://developer.android.com/training/sharing/shareaction.html>
* <http://developer.android.com/reference/android/widget/ShareActionProvider.html>