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
public final static int CAPTURE_IMAGE_ACTIVITY_REQUEST_CODE = 1034;
public String photoFileName = "photo.jpg";

public void onLaunchCamera(View view) {
    // create Intent to take a picture and return control to the calling application
    Intent intent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
    intent.putExtra(MediaStore.EXTRA_OUTPUT, getPhotoFileUri(photoFileName)); // set the image file name
    // start the image capture Intent
    startActivityForResult(intent, CAPTURE_IMAGE_ACTIVITY_REQUEST_CODE);
}

@Override
public void onActivityResult(int requestCode, int resultCode, Intent data) {
    if (requestCode == CAPTURE_IMAGE_ACTIVITY_REQUEST_CODE) {
       Uri takenPhotoUri = getPhotoFileUri(photoFileName);
       // by this point we have the camera photo on disk
       // Bitmap takenImage = BitmapFactory.decodeFile(filePath);
    }
}

// Returns the Uri for a photo stored on disk given the fileName
public void getPhotoFileUri(fileName) {
  File mediaStorageDir = new File(
        Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_PICTURES), "MyCustomApp");

    // Create the storage directory if it does not exist
    if (!mediaStorageDir.exists() && !mediaStorageDir.mkdirs()){
        Log.d("MyCustomApp", "failed to create directory");
    }
    // Specify the file target for the photo
    fileUri = Uri.fromFile(new File(mediaStorageDir.getPath() + File.separator + fileName));
}
```

Check out the official [Photo Basics](http://developer.android.com/training/camera/photobasics.html) guide for more details.

### Accessing Stored Media

Similar to the camera, the media picker implementation depends on the level of customization required:

 * The easy way - launch the Gallery with an intent, and get the media URI in onActivityResult.
 * The hard way - fetch thumbnail and full-size URIs from the MediaStore ContentProvider.

Easy way is to use an intent to launch the gallery:

```java
// PICK_PHOTO_CODE is a constant integer
public final static int PICK_PHOTO_CODE = 1046;

public void onPickPhoto(View view) {
    Intent intent = new Intent(Intent.ACTION_PICK,
        MediaStore.Images.Media.EXTERNAL_CONTENT_URI);
    startActivityForResult(intent, PICK_PHOTO_CODE);
}

@Override
public void onActivityResult(int requestCode, int resultCode, Intent data) {
   if (requestCode == PICK_PHOTO_CODE) {
      photoUri = getFileUri(data.getData());
      // Do something with the photo based on Uri
      Bitmap bitmap = MediaStore.Images.Media.getBitmap(this.getContentResolver(), imageUri);
   }
}

// Used to retrieve the actual filesystem Uri based on the media store result
private String getFileUri(Uri mediaStoreUri) {
		String[] filePathColumn = { MediaStore.Images.Media.DATA };
    Cursor cursor = getActivity().getContentResolver().query(mediaStoreUri,
            filePathColumn, null, null, null);
    cursor.moveToFirst();
    int columnIndex = cursor.getColumnIndex(filePathColumn[0]);
    String fileUri = cursor.getString(columnIndex);
    cursor.close();
    return fileUri;
}
```

### Playing Videos

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