## Usage

The [Fused Location API](http://developer.android.com/intl/es/training/location/retrieve-current.html) is a higher-level Google Play Services API that wraps the underlying location sensors like GPS. You can accomplish tasks like:

 * Register for location connection events
 * Connect to the location sensor
 * Register for updates or accuracy changes
 * Get last location

For even simpler location updates, check out this [smart-location-lib](https://github.com/mrmans0n/smart-location-lib) locations API wrapper.

### Installation

Fused location requires the use of the Google Play SDK. You must include the library in your  `app/build.gradle` file: 

```gradle
dependencies {
    compile 'com.google.android.gms:play-services-location:11.0.1'
}
```

To find the latest version to use refer to the [official SDK setup instructions](http://developer.android.com/google/play-services/setup.html).

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

**Note:** The permissions model has changed starting in Marshmallow. If your `targetSdkVersion` >= `23` and you are running on a Marshmallow (or later) device, you may need to [[enable runtime permissions|Managing-Runtime-Permissions-with-PermissionsDispatcher]]. You can also read more about the [[runtime permissions changes here|Understanding-App-Permissions#runtime-permissions]].

### Connecting to the LocationServices API

Inside an Activity, put the following to connect and start receiving location updates:

```java
private LocationRequest mLocationRequest;

private long UPDATE_INTERVAL = 10 * 1000;  /* 10 secs */
private long FASTEST_INTERVAL = 2000; /* 2 sec */

public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    startLocationUpdates(); 
}

// Trigger new location updates at interval
protected void startLocationUpdates() {

    // Create the location request to start receiving updates
    mLocationRequest = new LocationRequest();
    mLocationRequest.setPriority(LocationRequest.PRIORITY_HIGH_ACCURACY);
    mLocationRequest.setInterval(UPDATE_INTERVAL);
    mLocationRequest.setFastestInterval(FASTEST_INTERVAL);

    // Create LocationSettingsRequest object using location request
    LocationSettingsRequest.Builder builder = new LocationSettingsRequest.Builder();
    builder.addLocationRequest(mLocationRequest);
    LocationSettingsRequest locationSettingsRequest = builder.build();

    // Check whether location settings are satisfied
    // https://developers.google.com/android/reference/com/google/android/gms/location/SettingsClient
    SettingsClient settingsClient = LocationServices.getSettingsClient(this);
    settingsClient.checkLocationSettings(locationSettingsRequest);

    // new Google API SDK v11 uses getFusedLocationProviderClient(this)
    getFusedLocationProviderClient(this).requestLocationUpdates(mLocationRequest, new LocationCallback() {
      @Override
      public void onLocationResult(LocationResult locationResult) {
         // do work here
         onLocationChanged(locationResult.getLastLocation();
      }
    },
    Looper.myLooper());
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

If you want the last known location, you can implement this function:

```java
public void getLastLocation() {
    // Get last known recent location using new Google Play Services SDK (v11+)
    FusedLocationProviderClient locationClient = getFusedLocationProviderClient(this);

    locationClient.getLastLocation()
                .addOnSuccessListener(new OnSuccessListener<Location>() {
                    @Override
                    public void onSuccess(Location location) {
                        // GPS location can be null if GPS is switched off
                        if (location != null) {
                            onLocationChanged(location);
                        }
                    }
                })
                .addOnFailureListener(new OnFailureListener() {
                    @Override
                    public void onFailure(@NonNull Exception e) {
                        Log.d("MapDemoActivity", "Error trying to get last GPS location");
                        e.printStackTrace();
                    }
                });
}
```

To allow Google Maps to show the icon that will take you to the GPS location, you will need to first request 
`ACCESS_FINE_LOCATION` permission and call `setMyLocationEnabled`:

```java
@Override
public void onMapReady(GoogleMap googleMap) {

   if(checkPermissions()) {
     googleMap.setMyLocationEnabled(true);
   }
}

private boolean checkPermissions() {
  if (ContextCompat.checkSelfPermission(this,
     Manifest.permission.ACCESS_FINE_LOCATION) == PackageManager.PERMISSION_GRANTED) {
     return true;
  } else {
     requestPermissions();
     return false;
  }
}

private void requestPermissions() {
        ActivityCompat.requestPermissions(this,
                new String[]{Manifest.permission.ACCESS_FINE_LOCATION},
                REQUEST_FINE_LOCATION);
}
```
 
For more information on the Fused Location API, refer to [this Google guide](http://developer.android.com/intl/es/training/location/retrieve-current.html).

For using maps check out the [[Cliffnotes for Maps|Google Maps Fragment Guide]] or the [Android Maps Tutorial](http://www.vogella.com/articles/AndroidGoogleMaps/article.html).  See a [working source code example](https://github.com/codepath/android-google-maps-demo). 

### Troubleshooting Location Updates

Location updates should always be done using the `FusedLocationProviderClient` leveraging the `LocationServices.API` as shown above. Do not use the older [Location APIs](https://developer.android.com/intl/es/guide/topics/location/index.html) which are much less reliable. Even when using the correct `FusedLocationApi`, there are a lot of things that can go wrong. Consider the following potential issues:

 * **Did you add the necessary permissions?** Make sure your app has `INTERNET` and `ACCESS_COARSE_LOCATION` permissions to ensure that location can be accessed as illustrated in the guide above.
 * **Are you trying to get location on the genymotion emulator?** Ensure you've enabled GPS and configured a lat/lng properly. Try restarting the emulator if needed and re-enabling GPS or **trying a device** (or the official emulator) instead to rule out Genymotion specific issues.
 * **Are you failed to add Google Play Services Location dependencies when using Firebase?** Firebase dependencies version is usually not up to date with recent Google Play Services, thus showing a conflict error when Gradle trying to get dependencies. To solve this, simply match Google Play Services Location version with your Firebase version.

You can also review the following troubleshooting resources:

 * [Beginner's Guide to Location](http://blog.teamtreehouse.com/beginners-guide-location-android)
 * [Sample Treehouse Source Code](https://github.com/treehouse/android-location-example-refactored/tree/master/app/src/main/java/teamtreehouse/com/iamhere)

With this, you should have location updates working as expected.

## References

* <http://developer.android.com/training/location/retrieve-current.html>
* <https://developer.android.com/training/location/receive-location-updates.html>
* <http://www.vogella.com/articles/AndroidLocationAPI/article.html>
* <http://www.vogella.com/articles/AndroidGoogleMaps/article.html>
* <https://github.com/googlesamples/android-play-location/blob/master/BasicLocationSample/app/src/main/java/com/google/android/gms/location/sample/basiclocationsample/MainActivity.java>
* <https://github.com/googlesamples/android-play-location/blob/master/LocationUpdates/app/src/main/java/com/google/android/gms/location/sample/locationupdates/MainActivity.java>