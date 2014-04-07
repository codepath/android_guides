## Overview

With Android, there is a lot of access to hardware features using built-in Android libraries and the Google Play SDK. A few examples of things you might need to access:

 * Taking a photo/video with the camera.
 * Accessing and playing video and audio.
 * Accessing hardware sensors like accelerometers, light sensors, etc.
 * Using the Locations API.
 * Using the Maps API.

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

Easy way works in most cases, using the intent to [launch the camera](http://developer.android.com/guide/topics/media/camera.html):

```java
public final String APP_TAG = "MyCustomApp";
public final static int CAPTURE_IMAGE_ACTIVITY_REQUEST_CODE = 1034;
public String photoFileName = "photo.jpg";

public void onLaunchCamera(View view) {
    // create Intent to take a picture and return control to the calling application
    Intent intent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
    intent.putExtra(MediaStore.EXTRA_OUTPUT, getPhotoFileUri(photoFileName)); // set the image file name
    // Start the image capture intent to take photo
    startActivityForResult(intent, CAPTURE_IMAGE_ACTIVITY_REQUEST_CODE);
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
```

Check out the official [Photo Basics](http://developer.android.com/training/camera/photobasics.html) guide for more details.

## Accessing Stored Media

Similar to the camera, the media picker implementation depends on the level of customization required:

 * The easy way - launch the Gallery with an intent, and get the media URI in onActivityResult.
 * The hard way - fetch thumbnail and full-size URIs from the MediaStore ContentProvider.

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

Easy way is to use an intent to launch the gallery:

```java
// PICK_PHOTO_CODE is a constant integer
public final static int PICK_PHOTO_CODE = 1046;

// Trigger gallery selection for a photo
public void onPickPhoto(View view) {
    // Create intent for picking a photo from the gallery
    Intent intent = new Intent(Intent.ACTION_PICK,
        MediaStore.Images.Media.EXTERNAL_CONTENT_URI);
    // Bring up gallery to select a photo
    startActivityForResult(intent, PICK_PHOTO_CODE);
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

Note that there is a try-catch block required around the `MediaStore.Images.Media.getBitmap` line which was removed from above for brevity.

## Accessing Sensors

Different devices have a [variety of sensors](http://developer.android.com/guide/topics/sensors/sensors_overview.html) that can be accessed via the Sensor framework. Possible tasks related to sensors include:

 * List available sensors
 * Determine sensor capabilities (range, resolution, etc)
 * Acquire raw sensor data
 * Register sensor event listeners

### Sensor Types

Common sensors that devices have available are for temperature, light, pressure, acceleration, motion, and orientation. Here's a list of guides:

 * [Motion Sensors](http://developer.android.com/guide/topics/sensors/sensors_motion.html)
 * [Position Sensors](http://developer.android.com/guide/topics/sensors/sensors_position.html)
 * [Environment Sensors](http://developer.android.com/guide/topics/sensors/sensors_environment.html)

See the [full list of sensors](http://developer.android.com/guide/topics/sensors/sensors_overview.html#sensors-intro) for more details.

### Listening to Sensors

You can register for sensor events:

```java
private SensorEventListener mLightSensorListener = new SensorEventListener() {
	@Override
	public void onSensorChanged(SensorEvent event) {
		Log.d("MY_APP", event.toString());
	}

	@Override
	public void onAccuracyChanged(Sensor sensor, int accuracy) {
		Log.d("MY_APP", sensor.toString() + " - " + accuracy);
	}
};

@Override
protected void onCreate(Bundle savedInstanceState) {
	super.onCreate(savedInstanceState);
	setContentView(R.layout.activity_sensor);
	// Get sensor manager
	mSensorManager = (SensorManager) getSystemService(Context.SENSOR_SERVICE);
	// Get the default sensor of specified type
	mLight = mSensorManager.getDefaultSensor(Sensor.TYPE_LIGHT);
}

@Override
protected void onResume() {
	super.onResume();
	if (mLight != null) {
	  mSensorManager.registerListener(mLightSensorListener, mLight, SensorManager.SENSOR_DELAY_NORMAL);
	}
}

@Override
protected void onPause() {
  super.onPause();
  if (mLight != null) {
      mSensorManager.unregisterListener(mLightSensorListener);
  }
}
```

It's also important to note that this example uses the `onResume()` and `onPause()` callback methods to register and unregister the sensor event listener. As a best practice you should always disable sensors you don't need, especially when your activity is paused. Failing to do so can drain the battery in just a few hours because some sensors have substantial power requirements and can use up battery power quickly.

### Require Sensors

If you are publishing your application on Google Play you can use the <uses-feature> element in your manifest file to filter your application from devices that do not have the appropriate sensor configuration for your application. 

```xml
<uses-feature android:name="android.hardware.sensor.accelerometer" android:required="true" />
```

If you add this element and descriptor to your application's manifest, users will see your application on Google Play only if their device has an accelerometer. You should set the descriptor to `android:required="true"` only if your application relies entirely on a specific sensor.

### Location Sensor

Location requires the use of the [Google Play SDK](http://developer.android.com/google/play-services/setup.html). The [Location API](http://www.vogella.com/articles/AndroidLocationAPI/article.html) is a higher-level API that wraps the underlying location sensor. You can accomplish tasks like:

 * Connect to the location sensor
 * Register for updates or accuracy changes
 * Get last location
 * Register for sensor connection events

```java
LocationClient mLocationClient;

public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    // Connect the location client to start receiving updates
    mLocationClient = new LocationClient(this, this, this);
    mLocationClient.connect();
}

public void onConnected(Bundle arg0) {
    Location mCurrentLocation = mLocationClient.getLastLocation();
    Log.d("DEBUG", "current location: " + mCurrentLocation.toString());
}
```

and register for location updates with `onLocationChanged`:

```java
public void onLocationChanged(Location location) {
    // Report to the UI that the location was updated
    String msg = "Updated Location: " +
        Double.toString(location.getLatitude()) + "," +
        Double.toString(location.getLongitude());
    Toast.makeText(this, msg, Toast.LENGTH_SHORT).show();
}
```

For using maps check out the [[Cliffnotes for Maps|Google Maps Fragment Guide]] or the [Android Maps Tutorial](http://www.vogella.com/articles/AndroidGoogleMaps/article.html).

## Playing Videos

[VideoView](http://developer.android.com/reference/android/widget/VideoView.html) is the default wrapper for MediaPlayer for playing videos:

```java
public void playUrl(String url) {
    VideoView videoView = (VideoView) findViewById(R.id.video_view);
    videoView.setVideoPath(url);
    videoView.setMediaController(new MediaController(this));       
    videoView.requestFocus();   
    videoView.start();
}
```

See our [[Video and Audio Playback and Recording]] cliffnotes for a more detailed look.

## References
 
 * <http://developer.android.com/guide/topics/media/camera.html>
 * <http://developer.android.com/reference/android/hardware/Camera.html> 
 * <http://developer.android.com/reference/android/widget/VideoView.html>
 * <http://developer.android.com/guide/topics/sensors/sensors_overview.html>
 * <http://www.vogella.com/articles/AndroidLocationAPI/article.html>
 * <http://www.vogella.com/articles/AndroidGoogleMaps/article.html>