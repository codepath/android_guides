## Usage

The [Fused Location API](http://developer.android.com/intl/es/training/location/retrieve-current.html) is a higher-level Google Play Services API that wraps the underlying location sensors like GPS. You can accomplish tasks like:

 * Register for location connection events
 * Connect to the location sensor
 * Register for updates or accuracy changes
 * Get last location

### Installation

Fused location requires the use of the Google Play SDK. You must include the library in your  `app/build.gradle` file: 

```gradle
dependencies {
    compile 'com.google.android.gms:play-services-location:8.1.0'
}
```

You can use [this site](http://gradleplease.appspot.com/) to find the latest version to use, or you can refer to the [official SDK setup instructions](http://developer.android.com/google/play-services/setup.html).

In addition, if you are using Genymotion emulator, you must [[install a Google Play Services APK file|Genymotion-2.0-Emulators-with-Google-Play-support#setup-google-play-services]] too.

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

* <http://developer.android.com/training/location/retrieve-current.html>
* <https://developer.android.com/training/location/receive-location-updates.html>
* <http://www.vogella.com/articles/AndroidLocationAPI/article.html>
* <http://www.vogella.com/articles/AndroidGoogleMaps/article.html>