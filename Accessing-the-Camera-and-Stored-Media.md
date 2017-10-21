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

**Note:** The permissions model has changed starting in Marshmallow. If your `targetSdkVersion` >= `23` and you are running on a Marshmallow (or later) device, you may need to [[enable runtime permissions|Managing-Runtime-Permissions-with-PermissionsDispatcher]]. You should also read more about the [[runtime permissions changes|Understanding-App-Permissions#runtime-permissions]].

If you are using `targetSdkVersion` >= `24`, you also must configure a `FileProvider` as show in this [[section|Sharing-Content-with-Intents#sharing-files-with-api-24-or-higher]].  The example below uses `com.codepath.fileprovider` and should match the `authorities` XML tag specified.

### Using Capture Intent

Easy way works in most cases, using the intent to [launch the camera](http://developer.android.com/guide/topics/media/camera.html):

```java
public final String APP_TAG = "MyCustomApp";
public final static int CAPTURE_IMAGE_ACTIVITY_REQUEST_CODE = 1034;
public String photoFileName = "photo.jpg";
File photoFile;

public void onLaunchCamera(View view) {
    // create Intent to take a picture and return control to the calling application
    Intent intent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
    // Create a File reference to access to future access
    photoFile = getPhotoFileUri(photoFileName);  

    // wrap File object into a content provider
    // required for API >= 24
    // See https://guides.codepath.com/android/Sharing-Content-with-Intents#sharing-files-with-api-24-or-higher
    Uri fileProvider = FileProvider.getUriForFile(MyActivity.this, "com.codepath.fileprovider", photoFile);
    intent.putExtra(MediaStore.EXTRA_OUTPUT, fileProvider); 
    
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
         // by this point we have the camera photo on disk
         Bitmap takenImage = BitmapFactory.decodeFile(photoFile);
         // RESIZE BITMAP, see section below
         // Load the taken image into a preview
         ImageView ivPreview = (ImageView) findViewById(R.id.ivPreview);
         ivPreview.setImageBitmap(takenImage);   
       } else { // Result was a failure
    	   Toast.makeText(this, "Picture wasn't taken!", Toast.LENGTH_SHORT).show();
       }
    }
}

// Returns the File for a photo stored on disk given the fileName
public File getPhotoFileUri(String fileName) {
    // Only continue if the SD Card is mounted
    if (isExternalStorageAvailable()) {
        // Get safe storage directory for photos
        // Use `getExternalFilesDir` on Context to access package-specific directories.
        // This way, we don't need to request external read/write runtime permissions.
        File mediaStorageDir = new File(
            getExternalFilesDir(Environment.DIRECTORY_PICTURES), APP_TAG);

        // Create the storage directory if it does not exist
        if (!mediaStorageDir.exists() && !mediaStorageDir.mkdirs()){
            Log.d(APP_TAG, "failed to create directory");
        }

        // Return the file target for the photo based on filename
        File file = new File(mediaStorageDir.getPath() + File.separator + fileName);

        return file;
    }
    return null;
}

// Returns true if external storage for photos is available
private boolean isExternalStorageAvailable() {
    String state = Environment.getExternalStorageState();
    return state.equals(Environment.MEDIA_MOUNTED);
}
```

Check out the official [Photo Basics](http://developer.android.com/training/camera/photobasics.html) guide for more details.

### Loading the Bitmap

In certain cases, when loading a bitmap with `BitmapFactory.decodeFile(file)` decoding the Bitmap in memory may actually cause a crash with a `OutOfMemoryError: Failed to allocate` error. Check out the  [Loading Bitmaps Efficiently](http://developer.android.com/training/displaying-bitmaps/load-bitmap.html) guide and this [stackoverflow post](http://stackoverflow.com/questions/29624487/java-lang-outofmemoryerror-failed-to-allocate-a-31961100-byte-allocation-with-1) for an overview of the solutions.

### Resizing the Picture

Photos taken with the Camera intent are often quite large and take a very long time to load from disk. After taking a photo, you should strongly consider [[resizing the Bitmap|Working-with-the-ImageView#scaling-a-bitmap]] to a more manageable size and then **storing that smaller bitmap to disk**. We can then use that resized bitmap before displaying in an `ImageView`.

Resizing a large bitmap and writing to disk can be done with:

```java
// See code above
Uri takenPhotoUri = getPhotoFileUri(photoFileName);
// by this point we have the camera photo on disk
Bitmap rawTakenImage = BitmapFactory.decodeFile(takenPhotoUri.getPath());
// See BitmapScaler.java: https://gist.github.com/nesquena/3885707fd3773c09f1bb
Bitmap resizedBitmap = BitmapScaler.scaleToFitWidth(rawTakenImage, SOME_WIDTH);
```

Then we can write that smaller bitmap back to disk with:

```java
// Configure byte output stream
ByteArrayOutputStream bytes = new ByteArrayOutputStream();
// Compress the image further
resizedBitmap.compress(Bitmap.CompressFormat.JPEG, 40, bytes);
// Create a new file for the resized bitmap (`getPhotoFileUri` defined above)
Uri resizedUri = getPhotoFileUri(photoFileName + "_resized");
File resizedFile = new File(resizedUri.getPath());
resizedFile.createNewFile();
FileOutputStream fos = new FileOutputStream(resizedFile);
// Write the bytes of the bitmap to file
fos.write(bytes.toByteArray());
fos.close();
```

Now, we can store the path to that resized image and load that from disk instead for much faster load times.

### Rotating the Picture

When using the Camera intent to capture a photo, the picture is always taken in the orientation the camera is built into the device. To get your image rotated correctly you'll have to read the orientation information that is stored into the picture (EXIF meta data) and perform the following transformation using the [ExifInterface Support Library](https://android-developers.googleblog.com/2016/12/introducing-the-exifinterface-support-library.html):

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

See [this guide](http://stackoverflow.com/a/12933632/313399) for the source for this answer. Be aware that on certain devices even the EXIF data isn't set properly, in which case you should [checkout this workaround](http://stackoverflow.com/a/8864367/313399) for a fix. You can [read more here about the ExifInterface Support Library](https://android-developers.googleblog.com/2016/12/introducing-the-exifinterface-support-library.html).

### Applying Filters to Images

For applying filters to your captured images, check out the following libraries:

 * [CameraFilter](https://github.com/nekocode/CameraFilter) - Realtime camera filters. Process frames by OpenGL shaders.
 * [photofilter](https://github.com/mukeshsolanki/photofilter) - Apply filters to images after they are captured.

### Building a Custom Camera

Instead of using the capture intent to capture photos "the easy way", a custom camera can be used within your app directly leveraging the [Camera2 API](https://developer.android.com/reference/android/hardware/camera2/package-summary.html). This custom camera is much more complicated to implement but [sample code can be found here](https://github.com/googlesamples/android-Camera2Basic) and this [CameraView](https://github.com/google/cameraview?files=1) from Google aims to help Android developers easily integrate Camera features. There is also a  [Google video overview of Camera2](https://www.youtube.com/watch?v=Xtp3tH27OFs) which explains how `Camera2` works.  

There are a number of `Camera2` tutorials you can review:

 * [Android Lollipop's New Camera](http://willowtreeapps.com/blog/camera2-and-you-leveraging-android-lollipops-new-camera/)
 * [Android Camera Tutorial](http://inducesmile.com/android/android-camera-api-tutorial/).'
 * [Android Camera2 Tutorial](http://coderzpassion.com/android-working-camera2-api/)
 * [Android Camera2 Video Tutorial](https://www.youtube.com/watch?v=dfeencf-TpM)
 * [Camera2 Explained](http://pierrchen.blogspot.com/2015/01/android-camera2-api-explained.html)

There are also a number of third-party libraries available to make custom cameras easier:

 * [MaterialCamera](https://github.com/afollestad/material-camera)
 * [CWAC-Cam2](https://github.com/commonsguy/cwac-cam2)
 * [EasyCamera](https://github.com/Glamdring/EasyCamera)
 * [SquareCamera](https://github.com/boxme/SquareCamera)
 * [Many other libraries](http://android-arsenal.com/tag/141)

Leveraging `Camera2` or the libraries above, apps can develop a camera that functions in anyway required including custom overlays for depositing checks, taking pictures with a particular form factor, or scanning custom barcodes.

## Accessing Stored Media

Similar to the camera, the media picker implementation depends on the level of customization required:

 * The easy way - launch the Gallery with an intent, and get the media URI in onActivityResult.
 * The hard way - fetch thumbnail and full-size URIs from the MediaStore ContentProvider.

Make sure to enable access to the external storage first before using the camera (**Note:** The permissions model has changed starting in Marshmallow. If your `targetSdkVersion` >= `23` and you are running on a Marshmallow (or later) device, you may need to [[enable runtime permissions|Managing-Runtime-Permissions-with-PermissionsDispatcher]]. You should also read more about the [[runtime permissions changes|Understanding-App-Permissions#runtime-permissions]]):

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

### Selecting Multiple Images from Gallery

First, from the above example, we can add the `Intent.EXTRA_ALLOW_MULTIPLE` flag to the intent:

```java
Intent intent = new Intent(Intent.ACTION_PICK);
intent.setType("image/*");
intent.putExtra(Intent.EXTRA_ALLOW_MULTIPLE, true);
startActivityForResult(Intent.createChooser(intent, "Select Picture"), PICK_PHOTO_CODE);
```

and then inside of `onActivityResult`, we can access all the photos selected with:

```java
@Override
public void onActivityResult(int requestCode, int resultCode, Intent data) {
    if (data.getClipData() != null) {
        ClipData mClipData = data.getClipData();
         mArrayUri = new ArrayList<Uri>();
         mBitmapsSelected = new ArrayList<Bitmap>();
         for (int i = 0; i < mClipData.getItemCount(); i++) {
             ClipData.Item item = mClipData.getItemAt(i);
             Uri uri = item.getUri();
             mArrayUri.add(uri);
             // !! You may need to resize the image if it's too large
             Bitmap bitmap = MediaStore.Images.Media.getBitmap(this.getContentResolver(), photoUri);
             mBitmapsSelected.add(bitmap);
         }
    }
}
```

**Note:** that you may need to [load the selected bitmaps efficiently](https://developer.android.com/training/displaying-bitmaps/load-bitmap.html) or [[resize them|Working-with-the-ImageView#scaling-a-bitmap]] if they are large images to avoid encountering `OutOfMemoryError` exceptions. 

### Custom Gallery Selector

Alternatively, we can use a custom gallery selector that is implemented inside of our application to take full control over the gallery picking user experience. Check out [this custom gallery source code gist](https://gist.github.com/nesquena/55f3d8c87759f10a5343e3bb478f8858) or [older libraries wrapping this up](https://github.com/luminousman/MultipleImagePick) for reference. You can also take a look at [older tutorials on custom galleries](http://geekonjava.blogspot.com/2015/10/easy-multiple-image-pick-android.html). 

### File Pickers

For allowing users to pick files from their system or online services, check out these helpful filepicker libraries:

 * [Android-FilePicker](https://github.com/DroidNinja/Android-FilePicker)
 * [MaterialFilePicker](https://github.com/nbsp-team/MaterialFilePicker)

These allow users to pick files beyond just images and are easy to drop into any app.

## References

 * <http://developer.android.com/guide/topics/media/camera.html>
 * <http://developer.android.com/reference/android/hardware/Camera.html>
