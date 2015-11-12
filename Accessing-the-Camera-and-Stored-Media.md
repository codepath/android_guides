## Overview

This guide covers how to work with the camera and how to access media stored on the phone.

## Using the Camera

The [camera](http://developer.android.com/guide/topics/media/camera.html) implementation depends on the level of customization required:

 * The easy way - launch the camera with an intent, designating a file path, and handle the onActivityResult.
 * The hard way - use the Camera API to embed the camera preview within your app, adding your own custom controls.

Make sure to enable access to the external storage first before using the camera:

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
  ...>
    <!- ... -->

    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />

    <!- ... -->
</manifest>
```

**Note:** The permissions model has changed starting in Marshmallow. If your `targetSdkVersion` >= `23` and you are running on a Marshmallow (or later) device / emulator, you'll need to follow this guide on [[implementing runtime permissions|Understanding-App-Permissions#runtime-permissions]] in order to get these permissions.

### Using Capture Intent

Easy way works in most cases, using the intent to [launch the camera](http://developer.android.com/guide/topics/media/camera.html):

```java
public final String APP_TAG = "MyCustomApp";
public final static int CAPTURE_IMAGE_ACTIVITY_REQUEST_CODE = 1034;
public String photoFileName = "photo.jpg";

public void onLaunchCamera(View view) {
    // create Intent to take a picture and return control to the calling application
    Intent intent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
    intent.putExtra(MediaStore.EXTRA_OUTPUT, getPhotoFileUri(photoFileName)); // set the image file name
    
    // If you call startActivityForResult() using an intent that no app can handle, your app will crash.
    // So as long as the result is not null, it's safe to use the intent.
    if (intent.resolveActivity(getPackageManager()) != null) {
        // Start the image capture intent to take photo
        startActivityForResult(intent, CAPTURE_IMAGE_ACTIVITY_REQUEST_CODE);
    }
}

@Override
public void onActivityResult(int requestCode, int resultCode, Intent data) {
    if (requestCode == CAPTURE_IMAGE_ACTIVITY_REQUEST_CODE) {
       if (resultCode == RESULT_OK) {
         Uri takenPhotoUri = getPhotoFileUri(photoFileName);
         // by this point we have the camera photo on disk
         Bitmap takenImage = BitmapFactory.decodeFile(takenPhotoUri.getPath());
         // Load the taken image into a preview
         ImageView ivPreview = (ImageView) findViewById(R.id.ivPreview);
         ivPreview.setImageBitmap(takenImage);   
       } else { // Result was a failure
    	   Toast.makeText(this, "Picture wasn't taken!", Toast.LENGTH_SHORT).show();
       }
    }
}

// Returns the Uri for a photo stored on disk given the fileName
public Uri getPhotoFileUri(String fileName) {
    // Only continue if the SD Card is mounted
    if (isExternalStorageAvailable()) {
        // Get safe storage directory for photos
        File mediaStorageDir = new File(
            Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_PICTURES), APP_TAG);

        // Create the storage directory if it does not exist
        if (!mediaStorageDir.exists() && !mediaStorageDir.mkdirs()){
            Log.d(APP_TAG, "failed to create directory");
        }

        // Return the file target for the photo based on filename
        return Uri.fromFile(new File(mediaStorageDir.getPath() + File.separator + fileName));
    }
    return null;
}

private boolean isExternalStorageAvailable() {
			String state = Environment.getExternalStorageState();
			if (state.equals(Environment.MEDIA_MOUNTED)) {
				return true;
			}
			return false;
		}
```

Check out the official [Photo Basics](http://developer.android.com/training/camera/photobasics.html) guide for more details.

### Loading the Bitmap

In certain cases, when loading a bitmap with `BitmapFactory.decodeFile(file)` decoding the Bitmap in memory may actually cause a crash with a `OutOfMemoryError: Failed to allocate` error. Check out the  [Loading Bitmaps Efficiently](http://developer.android.com/training/displaying-bitmaps/load-bitmap.html) guide and this [stackoverflow post](http://stackoverflow.com/questions/29624487/java-lang-outofmemoryerror-failed-to-allocate-a-31961100-byte-allocation-with-1) for an overview of the solutions.

### Resizing the Picture

Photos taken with the Camera intent are often quite large. After taking a photo, you may want to consider [[resizing the Bitmap|Working-with-the-ImageView#scaling-a-bitmap]] to a more manageable size before displaying in an `ImageView`.

### Rotating the Picture

When using the Camera intent to capture a photo, the picture is always taken in the orientation the camera is built into the device. To get your image rotated correctly you'll have to read the orientation information that is stored into the picture (EXIF meta data) and perform the following transformation:

```java
public Bitmap rotateBitmapOrientation(String photoFilePath) {
    // Create and configure BitmapFactory
    BitmapFactory.Options bounds = new BitmapFactory.Options();
    bounds.inJustDecodeBounds = true;
    BitmapFactory.decodeFile(photoFilePath, bounds);
    BitmapFactory.Options opts = new BitmapFactory.Options();
    Bitmap bm = BitmapFactory.decodeFile(photoFilePath, opts);
    // Read EXIF Data
    ExifInterface exif = null;
    try {
      exif = new ExifInterface(photoFilePath);
     } catch (IOException e) {
      e.printStackTrace();
     }
    String orientString = exif.getAttribute(ExifInterface.TAG_ORIENTATION);
    int orientation = orientString != null ? Integer.parseInt(orientString) : ExifInterface.ORIENTATION_NORMAL;
    int rotationAngle = 0;
    if (orientation == ExifInterface.ORIENTATION_ROTATE_90) rotationAngle = 90;
    if (orientation == ExifInterface.ORIENTATION_ROTATE_180) rotationAngle = 180;
    if (orientation == ExifInterface.ORIENTATION_ROTATE_270) rotationAngle = 270;
    // Rotate Bitmap
    Matrix matrix = new Matrix();
    matrix.setRotate(rotationAngle, (float) bm.getWidth() / 2, (float) bm.getHeight() / 2);
    Bitmap rotatedBitmap = Bitmap.createBitmap(bm, 0, 0, bounds.outWidth, bounds.outHeight, matrix, true);
    // Return result
    return rotatedBitmap;
}
```

See [this guide](http://stackoverflow.com/a/12933632/313399) for the source for this answer. Be aware that on certain devices even the EXIF data isn't set properly, in which case you should [checkout this workaround](http://stackoverflow.com/a/8864367/313399) for a fix.

### Building a Custom Camera

Instead of using the capture intent to capture photos "the easy way", a custom camera can be used within your app directly leveraging the [Camera2 API](https://developer.android.com/reference/android/hardware/camera2/package-summary.html). This custom camera is much more complicated to implement but [sample code can be found here](https://github.com/googlesamples/android-Camera2Basic). However, there are a number of third-party libraries available to make custom camera easier:

 * [CWAC-Cam2](https://github.com/commonsguy/cwac-cam2)
 * [EasyCamera](https://github.com/Glamdring/EasyCamera)
 * [SquareCamera](https://github.com/boxme/SquareCamera)
 * [Many other libraries](http://android-arsenal.com/tag/141)

Leveraging `Camera2` or the libraries above, apps can develop a camera that functions in anyway required including custom overlays for depositing checks, taking pictures with a particular form factor, or scanning custom barcodes.

## Accessing Stored Media

Similar to the camera, the media picker implementation depends on the level of customization required:

 * The easy way - launch the Gallery with an intent, and get the media URI in onActivityResult.
 * The hard way - fetch thumbnail and full-size URIs from the MediaStore ContentProvider.

Make sure to enable access to the external storage first before using the camera (**Note:** The permissions model has changed starting in Marshmallow. If your `targetSdkVersion` >= `23` and you are running on a Marshmallow (or later) device / emulator, you'll need to follow this guide on implementing [[runtime permissions|Understanding-App-Permissions#runtime-permissions]] in order to get these permissions):

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
  ...>
    <!- ... -->

    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />

    <!- ... -->
</manifest>
```

Easy way is to use an intent to launch the gallery:

```java
// PICK_PHOTO_CODE is a constant integer
public final static int PICK_PHOTO_CODE = 1046;

// Trigger gallery selection for a photo
public void onPickPhoto(View view) {
    // Create intent for picking a photo from the gallery
    Intent intent = new Intent(Intent.ACTION_PICK,
        MediaStore.Images.Media.EXTERNAL_CONTENT_URI);
    
    // If you call startActivityForResult() using an intent that no app can handle, your app will crash.
    // So as long as the result is not null, it's safe to use the intent.
    if (intent.resolveActivity(getPackageManager()) != null) {
       // Bring up gallery to select a photo
       startActivityForResult(intent, PICK_PHOTO_CODE);
    }
}

@Override
public void onActivityResult(int requestCode, int resultCode, Intent data) {
    if (data != null) {
        Uri photoUri = data.getData();
        // Do something with the photo based on Uri
        Bitmap selectedImage = MediaStore.Images.Media.getBitmap(this.getContentResolver(), photoUri);
        // Load the selected image into a preview
        ImageView ivPreview = (ImageView) findViewById(R.id.ivPreview);
        ivPreview.setImageBitmap(selectedImage);   
    }
}
```

Note that there is a try-catch block required around the `MediaStore.Images.Media.getBitmap` line which was removed from above for brevity. Check out [this stackoverflow post](http://stackoverflow.com/questions/5309190/android-pick-images-from-gallery/5309217#5309217) for an alternate approach using mimetypes to restrict content user can select.

## References

 * <http://developer.android.com/guide/topics/media/camera.html>
 * <http://developer.android.com/reference/android/hardware/Camera.html> 