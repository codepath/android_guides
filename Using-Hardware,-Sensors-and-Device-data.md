## Overview

With Android, there is a lot of access to hardware features using built-in Android libraries and the Google Play SDK. A few examples of things you might need to access:

 * Taking a photo/video with the camera.
 * Accessing and playing photo/video/audio.
 * Accessing hardware sensors like accelerometers, light sensors, etc.
 * Using the Locations API.
 * Using the Maps API.

### Using the Camera

The [camera](http://developer.android.com/guide/topics/media/camera.html) implementation depends on the level of customization required:

 * The easy way - launch the camera with an intent, designating a file path, and handle the onActivityResult.
 * The hard way - use the Camera API to embed the camera preview within your app, adding your own custom controls.

Easy way works in most cases, using the intent to [launch the camera](http://developer.android.com/guide/topics/media/camera.html):

```java
public void getPhotoFileUri(fileName) {
  File mediaStorageDir = new File(
        Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_PICTURES),
        "MyCameraApp");

    // Create the storage directory if it does not exist
    if (!mediaStorageDir.exists() && !mediaStorageDir.mkdirs()){
        Log.d("MyCameraApp", "failed to create directory");
    }
    // Specify the file target for the photo
    fileUri = Uri.fromFile(new File(mediaStorageDir.getPath() + File.separator +
		        fileName));
}

public void onLaunchCamera(View view) {
    // create Intent to take a picture and return control to the calling application
    Intent intent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
    intent.putExtra(MediaStore.EXTRA_OUTPUT, getPhotoFileUri("photo.jpg")); // set the image file name
    // start the image capture Intent
    startActivityForResult(intent, CAPTURE_IMAGE_ACTIVITY_REQUEST_CODE);
}
```

### Accessing Stored Media

Similar to the camera, the media picker implementation depends on the level of customization required:

 * The easy way - launch the Gallery with an intent, and get the media URI in onActivityResult.
 * The hard way - fetch thumbnail and full-size URIs from the MediaStore ContentProvider.

Easy way is to use an intent to launch the gallery:

```java
// PICK_PHOTO_CODE is a constant integer
public void onPickPhoto(View view) {
    Intent intent = new Intent(Intent.ACTION_PICK,
        MediaStore.Images.Media.EXTERNAL_CONTENT_URI);
    startActivityForResult(intent, PICK_PHOTO_CODE);
}
```

### Playing Videos

[VideoView](http://developer.android.com/reference/android/widget/VideoView.html) is the default wrapper for MediaPlayer for playing videos:

```java
public void playUrl(String url) {
    // videoview defined in the layout XML
    videoView.setVideoPath(url);
    videoView.setMediaController(new MediaController(this));       
    videoView.requestFocus();   
    videoView.start();
}
```

### Accessing Sensors

Different devices have a [variety of sensors](http://developer.android.com/guide/topics/sensors/sensors_overview.html) that can be accessed via the Sensor framework. Possible tasks include:

 * List available sensors
 * Determine sensor capabilities (range, resolution, etc)
 * Acquire raw sensor data
 * Register sensor event listeners

You can register for sensor events:

```java
public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.main);

    mSensorManager = 
        (SensorManager)getSystemService(Context.SENSOR_SERVICE);
    mLight = mSensorManager.getDefaultSensor(Sensor.TYPE_LIGHT);
    mSensorManager.registerListener(this, mLight,    
        SensorManager.SENSOR_DELAY_NORMAL);
}

public void onSensorChanged(SensorEvent event) {
    ...
}
```

### Location

Location requires the use of the [Google Play SDK](http://developer.android.com/google/play-services/setup.html). The [Location API](http://www.vogella.com/articles/AndroidLocationAPI/article.html) is a higher-level API that wraps the underlying location sensor. You can accomplish tasks like:

 * Connect to the location sensor
 * Register for updates or accuracy changes
 * Get last location
 * Register for sensor connection events

```java
public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
		
    mLocationClient = new LocationClient(this, this, this);
    mLocationClient.connect();
}

public void onConnected(Bundle arg0) {
    Location mCurrentLocation = mLocationClient.getLastLocation();
    Log.d("DEBUG", "current location: " + mCurrentLocation.toString());
}
```

and register for location updates:

```java
public void onLocationChanged(Location location) {
    // Report to the UI that the location was updated
    String msg = "Updated Location: " +
        Double.toString(location.getLatitude()) + "," +
        Double.toString(location.getLongitude());
    Toast.makeText(this, msg, Toast.LENGTH_SHORT).show();
}
```

For using maps check out the [Android Maps Tutorial](http://www.vogella.com/articles/AndroidGoogleMaps/article.html).

## References
 
 * <http://developer.android.com/guide/topics/media/camera.html>
 * <http://developer.android.com/reference/android/hardware/Camera.html> 
 * <http://developer.android.com/reference/android/widget/VideoView.html>
 * <http://developer.android.com/guide/topics/sensors/sensors_overview.html>
 * <http://www.vogella.com/articles/AndroidLocationAPI/article.html>
 * <http://www.vogella.com/articles/AndroidGoogleMaps/article.html>