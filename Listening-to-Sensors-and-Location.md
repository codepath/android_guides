## Overview

With Android, there is a lot of access to hardware features using built-in Android libraries and the Google Play SDK. A few examples of things you might need to access:

 * Accessing hardware sensors like accelerometers, light sensors, etc.
 * Using the Locations API.
 * Using the Maps API.

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
private SensorManager mSensorManager;
private Sensor mLight;

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
	  mSensorManager.registerListener(mLightSensorListener, mLight, 
             SensorManager.SENSOR_DELAY_NORMAL);
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

### Sensors in the Background

In certain cases, an app wants to be listening to sensors in the background even while the app is not running. While this can be extremely draining to the battery if caution is not taken, this can be done with the use of a `Service`. 

See [this sensors in the background](http://code.tutsplus.com/tutorials/android-barometer-logger-acquiring-sensor-data--mobile-10558). To get sensor readings while the phone is asleep, we can implement a [partial wake lock as described here](http://nosemaj.org/android-persistent-sensors). Here's [working sample code](https://github.com/AndroidExamples/android-sensor-example) for this as well.

## Location Sensor

Location requires the use of the Google Play SDK. You must include `com.google.android.gms:play-services` in your `app/build.gradle` file.  You can use [this site](http://gradleplease.appspot.com/) to find the latest version to use, or you can refer to the [official SDK setup instructions](http://developer.android.com/google/play-services/setup.html).

In addition, if you are using Genymotion emulator, you must [install a Google Play Services APK file](https://github.com/codepath/android_guides/wiki/Genymotion-2.0-Emulators-with-Google-Play-support#setup-google-play-services) too.

The [Location API](http://www.vogella.com/articles/AndroidLocationAPI/article.html) is a higher-level API that wraps the underlying location sensor. You can accomplish tasks like:

 * Connect to the location sensor
 * Register for updates or accuracy changes
 * Get last location
 * Register for sensor connection events

```java
private GoogleApiClient mGoogleApiClient;

private long UPDATE_INTERVAL = 60000;  /* 60 secs */
private long FASTEST_INTERVAL = 5000; /* 5 secs */

public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    // Create the location client to start receiving updates
    mGoogleApiClient = new GoogleApiClient.Builder(this)
                          .addApi(LocationServices.API)
                          .addConnectionCallbacks(this)
                          .addOnConnectionFailedListener(this).build();
}

protected void onStart() {
    super.onStart();
    // Connect the client.
    mGoogleApiClient.connect();
}

protected void onStop() {
    // Disconnecting the client invalidates it.
    mGoogleApiClient.disconnect();
    super.onStop();
}

public void onConnected(Bundle dataBundle) {
    Location mCurrentLocation = LocationServices.FusedLocationApi.getLastLocation(mGoogleApiClient);
    Log.d("DEBUG", "current location: " + mCurrentLocation.toString());
    LatLng latLng = new LatLng(mCurrentLocation.getLatitude(), mCurrentLocation.getLongitude());
    startLocationUpdates();
}

@Override
public void onConnectionSuspended(int i) {
    if (i == CAUSE_SERVICE_DISCONNECTED) {
      Toast.makeText(this, "Disconnected. Please re-connect.", Toast.LENGTH_SHORT).show();
    } else if (i == CAUSE_NETWORK_LOST) {
      Toast.makeText(this, "Network lost. Please re-connect.", Toast.LENGTH_SHORT).show();
    }
}

 protected void startLocationUpdates() {
        mLocationRequest = new LocationRequest();
        mLocationRequest.setPriority(LocationRequest.PRIORITY_BALANCED_POWER_ACCURACY);
        mLocationRequest.setInterval(UPDATE_INTERVAL);
        mLocationRequest.setFastestInterval(FASTEST_INTERVAL);
        LocationServices.FusedLocationApi.requestLocationUpdates(mGoogleApiClient,
                mLocationRequest, this);
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

For using maps check out the [[Cliffnotes for Maps|Google Maps Fragment Guide]] or the [Android Maps Tutorial](http://www.vogella.com/articles/AndroidGoogleMaps/article.html).  See a [working source code example](https://github.com/codepath/android-google-maps-demo).

## References

* <http://developer.android.com/guide/topics/sensors/sensors_overview.html>
* <http://www.vogella.com/articles/AndroidLocationAPI/article.html>
* <http://www.vogella.com/articles/AndroidGoogleMaps/article.html>
* <https://developer.android.com/training/location/receive-location-updates.html>
* <https://www.youtube.com/watch?v=Rd_UrSB1MAY>