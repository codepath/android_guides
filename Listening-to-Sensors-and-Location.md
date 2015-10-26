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

Fused location requires the use of the Google Play SDK. You must include `com.google.android.gms:play-services` in your `app/build.gradle` file.  You can use [this site](http://gradleplease.appspot.com/) to find the latest version to use, or you can refer to the [official SDK setup instructions](http://developer.android.com/google/play-services/setup.html).

In addition, if you are using Genymotion emulator, you must [[install a Google Play Services APK file|Genymotion-2.0-Emulators-with-Google-Play-support#setup-google-play-services]] too.

The [Fused Location API](http://developer.android.com/intl/es/training/location/retrieve-current.html) is a higher-level Google Play Services API that wraps the underlying location sensors like GPS. You can accomplish tasks like:

 * Register for location connection events
 * Connect to the location sensor
 * Register for updates or accuracy changes
 * Get last location

### Adding Permissions 

Add the following permissions to the `AndroidManifest.xml`:

```xml
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
```

This provides the ability to access location information at coarse and fine granularities. You can remove `ACCESS_FINE_LOCATION` this will still work with less precision. 

### Connecting to the LocationServices API

Inside an Activity, put the following to connect and start receiving location updates:

```java
private GoogleApiClient mGoogleApiClient;
private LocationRequest mLocationRequest;

private long UPDATE_INTERVAL = 10 * 1000;  /* 10 secs */
private long FASTEST_INTERVAL = 2000; /* 2 sec */

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
    LocationServices.FusedLocationApi.removeLocationUpdates(mGoogleApiClient, this);
    mGoogleApiClient.disconnect();
    super.onStop();
}

public void onConnected(Bundle dataBundle) {
    // Get last known recent location. 
    Location mCurrentLocation = LocationServices.FusedLocationApi.getLastLocation(mGoogleApiClient);
    // Note that this can be NULL if last location isn't already known.
    if (mCurrentLocation != null) {
      // Print current location if not null
      Log.d("DEBUG", "current location: " + mCurrentLocation.toString());
      LatLng latLng = new LatLng(mCurrentLocation.getLatitude(), mCurrentLocation.getLongitude());
    }
    // Begin polling for new location updates.
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

// Trigger new location updates at interval
protected void startLocationUpdates() {
    // Create the location request
    mLocationRequest = LocationRequest.create()
        .setPriority(LocationRequest.PRIORITY_HIGH_ACCURACY)
        .setInterval(UPDATE_INTERVAL)
        .setFastestInterval(FASTEST_INTERVAL);
    // Request location updates
    LocationServices.FusedLocationApi.requestLocationUpdates(mGoogleApiClient,
        mLocationRequest, this);
}
```

and then register for location updates with `onLocationChanged`:

```java
public void onLocationChanged(Location location) {
    // New location has now been determined
    String msg = "Updated Location: " +
        Double.toString(location.getLatitude()) + "," +
        Double.toString(location.getLongitude());
    Toast.makeText(this, msg, Toast.LENGTH_SHORT).show();
    // You can now create a LatLng Object for use with maps
    LatLng latLng = new LatLng(location.getLatitude(), location.getLongitude());
}
```

For more information on the Fused Location API, refer to [this Google guide](http://developer.android.com/intl/es/training/location/retrieve-current.html).

For using maps check out the [[Cliffnotes for Maps|Google Maps Fragment Guide]] or the [Android Maps Tutorial](http://www.vogella.com/articles/AndroidGoogleMaps/article.html).  See a [working source code example](https://github.com/codepath/android-google-maps-demo). 

### Troubleshooting Location Updates

Location updates should always be done using the `GoogleApiClient` leveraging the `LocationServices.API` as shown above. Do not use the older [Location APIs](https://developer.android.com/intl/es/guide/topics/location/index.html) which are much less reliable. Even when using the correct `FusedLocationApi`, there are a lot of things that can go wrong. Consider the following potential issues:

 * **Did you add the necessary permissions?** Make sure your app has `INTERNET` and `ACCESS_COARSE_LOCATION` permissions to ensure that location can be accessed as illustrated in the guide above.
 * **Are you getting `null` when calling `LocationServices.FusedLocationApi.getLastLocation`**? This is normal since this method only returns if there is already a location recently retrieved by another application. If this returns null, this means you need start receiving location updates with `LocationServices.FusedLocationApi.requestLocationUpdates` before receiving the location as shown above.
 * **Are you trying to get location on the genymotion emulator?** Ensure you've enabled GPS and configured a lat/lng properly. Try restarting the emulator if needed and re-enabling GPS or **trying a device** (or the official emulator) instead to rule out genymotion specific issues.

You can also review the following troubleshooting resources:

 * [FusedLocationApi.getLastLocation null](http://stackoverflow.com/questions/29796436/why-is-fusedlocationapi-getlastlocation-null)
 * [Beginner's Guide to Location](http://blog.teamtreehouse.com/beginners-guide-location-android)
 * [Sample Treehouse Source Code](https://github.com/treehouse/android-location-example-refactored/tree/master/app/src/main/java/teamtreehouse/com/iamhere)

With this, you should have location updates working as expected.

## References

* <http://developer.android.com/guide/topics/sensors/sensors_overview.html>
* <http://www.vogella.com/articles/AndroidLocationAPI/article.html>
* <http://www.vogella.com/articles/AndroidGoogleMaps/article.html>
* <https://developer.android.com/training/location/receive-location-updates.html>
* <https://www.youtube.com/watch?v=Rd_UrSB1MAY>
* <http://developer.android.com/intl/es/training/location/retrieve-current.html>