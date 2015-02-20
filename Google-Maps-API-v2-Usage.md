## Overview

This guide will instruct you on how use the Google Maps API v2 library to create a map within your application's activities and/or fragments.

### Setup Map If Necessary

Assuming you have added the GoogleMap v2 MapFragment to your layout, you should first run this method `onCreate()`. In my case, I am using the SupportMapFragment, either will work.

```java
protected void setUpMapIfNeeded() {
    // Do a null check to confirm that we have not already instantiated the map.
    if (mapFragment == null) {
        mapFragment = ((SupportMapFragment) getSupportFragmentManager().findFragmentById(R.id.map));
        // Check if we were successful in obtaining the map.
        if (mapFragment != null) {
           mapFragment.getMapAsync(new OnMapReadyCallback() {
                @Override
                public void onMapReady(GoogleMap map) {
                    loadMap(map);
                }
           });
        }
    }		   
}

// The Map is verified. It is now safe to manipulate the map.
protected void loadMap(GoogleMap googleMap) {
    if (map != null) {
       // ... use map here
    }
}
```

After you have run that it is safe for you to begin adding markers.

### Controlling the Camera

The map location center and zoom can be manipulated using the [moveCamera](http://developer.android.com/reference/com/google/android/gms/maps/GoogleMap.html#moveCamera\(com.google.android.gms.maps.CameraUpdate\)) or [animateCamera](http://developer.android.com/reference/com/google/android/gms/maps/GoogleMap.html#animateCamera\(com.google.android.gms.maps.CameraUpdate\)) which both take a [CameraUpdate](http://developer.android.com/reference/com/google/android/gms/maps/CameraUpdate.html) object. For example, to retarget the camera to a new latitude and longitude we can do:

```java
LatLng latLng = new LatLng(latitude, longitude);
CameraUpdate cameraUpdate = CameraUpdateFactory.newLatLng(latLng);
map.animateCamera(cameraUpdate);
```

Check the [CameraUpdateFactory](http://developer.android.com/reference/com/google/android/gms/maps/CameraUpdateFactory.html) for the available camera update types including `newLatLng`, `newLatLngZoom`, `zoomTo`, and more.

### Adding Markers to Map Fragment

We can add a marker to the map with the following code specifying the position (`LatLng`), icon (`BitmapDescriptor`), title, and snippet:

```java
// Use green marker icon
BitmapDescriptor defaultMarker =
    BitmapDescriptorFactory.defaultMarker(BitmapDescriptorFactory.HUE_GREEN);
// listingPosition is a LatLng point
Marker mapMarker = mapFragment.addMarker(new MarkerOptions()
    .position(listingPosition)					 								    
    .title("Some title here")
    .snippet("Some description here")
    .icon(defaultMarker));
```

### Show AlertDialog on LongClick

You can use the following code to bring up an `AlertDialog` for users to type a message on `MapLongClick` event. On completion, we can add a marker to the map which when pressed displays the message typed. 

#### Create Alert Dialog Template

First, we need to create a new xml file in `res/layout/message_item.xml` which will be displayed as the alert window:

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/layout_root"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="10dp" >

    <TextView
        android:id="@+id/tvTitle"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:paddingTop="20dp"
        android:text="Title:" />

    <EditText
        android:id="@+id/etTitle"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_alignLeft="@+id/etSnippet"
        android:layout_alignBottom="@+id/tvTitle"
        android:layout_toRightOf="@+id/tvTitle" >
    </EditText>

    <TextView
        android:id="@+id/tvSnippet"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_below="@+id/tvTitle"
        android:paddingTop="20dp"
        android:text="Snippet:" />

    <EditText
        android:id="@+id/etSnippet"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_alignBottom="@+id/tvSnippet"
        android:layout_below="@+id/tvTitle"
        android:layout_toRightOf="@+id/tvSnippet" >
    </EditText>

</RelativeLayout>
```

#### Implement OnMapLongClickListener Event 

If we want to setup a long click listener, we need to implement the `OnMapLongClickListener` in our Activity and setup the listener for the map:

```java
public class MapDemoActivity extends FragmentActivity implements
  GoogleApiClient.ConnectionCallbacks,
  GoogleApiClient.OnConnectionFailedListener,
  LocationListener,
  OnMapLongClickListener {

    ...

    protected void loadMap(GoogleMap googleMap) {
        map = googleMap;
        if (map != null) {
            // Attach long click listener to the map here
            map.setOnMapLongClickListener(this);
            // ...
        }
    }
    
    ...

    // Fires when a long press happens on the map
    @Override
    public void onMapLongClick(final LatLng point) {
      Toast.makeText(this, "Long Press", Toast.LENGTH_LONG).show();
      // Custom code here...
    }
}
```

Make sure to have your activity implement from the `OnMapLongClickListener` for this to work properly. Now, when we run this, we should see a toast when the user long clicks on the map. 

#### Define the Alert Dialog

Next, let's trigger an alert dialog which will be displayed to accept input from the user within `MapDemoActivity`:

```java
     // ... 

     @Override
     public void onMapLongClick(LatLng point) {
         Toast.makeText(getApplicationContext(), "Long Press", Toast.LENGTH_LONG).show();
         // Custom code here...
         // Display the alert dialog
         showAlertDialogForPoint(point);
     } 
     
     // Display the alert that adds the marker
     private void showAlertDialogForPoint(final LatLng point) {
        // inflate message_item.xml view
        View  messageView = LayoutInflater.from(MapDemoActivity.this).
          inflate(R.layout.message_item, null);
        // Create alert dialog builder
        AlertDialog.Builder alertDialogBuilder = new AlertDialog.Builder(this);
        // set message_item.xml to AlertDialog builder
        alertDialogBuilder.setView(messageView);

        // Create alert dialog
        final AlertDialog alertDialog = alertDialogBuilder.create();

        // Configure dialog button (OK)
        alertDialog.setButton(DialogInterface.BUTTON_POSITIVE, "OK", 
          new DialogInterface.OnClickListener() {
            @Override
            public void onClick(DialogInterface dialog, int which) {
                // Define color of marker icon
                BitmapDescriptor defaultMarker =
                   BitmapDescriptorFactory.defaultMarker(BitmapDescriptorFactory.HUE_GREEN);
                // Extract content from alert dialog
                String title = ((EditText) alertDialog.findViewById(R.id.etTitle)).
                    getText().toString();
                String snippet = ((EditText) alertDialog.findViewById(R.id.etSnippet)).
                    getText().toString();
                // Creates and adds marker to the map
                Marker marker = map.addMarker(new MarkerOptions()
                  .position(point)
                  .title(title)
                  .snippet(snippet)       
                  .icon(defaultMarker));
            }
        });

        // Configure dialog button (Cancel)
        alertDialog.setButton(DialogInterface.BUTTON_NEGATIVE, "Cancel", 
        new DialogInterface.OnClickListener() {
          public void onClick(DialogInterface dialog, int id) { dialog.cancel(); }
        });

        // Display the dialog
        alertDialog.show();
    }

    // ...
```

When we run the app now, the long click on the map should trigger a dialog which allows us to enter text which will be associated with a map marker.

### Falling Pin Animation

To implement falling pin animation, add marker to the desired position in map and then call this function with that marker.

```java
    private void dropPinEffect(final Marker marker) {
        // Handler allows us to repeat a code block after a specified delay
        final android.os.Handler handler = new android.os.Handler();
        final long start = SystemClock.uptimeMillis();
        final long duration = 1500;

        // Use the bounce interpolator
        final android.view.animation.Interpolator interpolator = 
            new BounceInterpolator();

        // Animate marker with a bounce updating its position every 15ms
        handler.post(new Runnable() {
            @Override
            public void run() {
                long elapsed = SystemClock.uptimeMillis() - start;
                // Calculate t for bounce based on elapsed time 
                float t = Math.max(
                        1 - interpolator.getInterpolation((float) elapsed
                                / duration), 0);
                // Set the anchor
                marker.setAnchor(0.5f, 1.0f + 14 * t);
           
                if (t > 0.0) {
                    // Post this event again 15ms from now.
                    handler.postDelayed(this, 15);
                } else { // done elapsing, show window
                    marker.showInfoWindow();
                }
            }
        });
    }
```

In `private void showAlertDialogForPoint` add the call to `dropPinEffect(marker);` at the end to animate the placement of the marker:

```java
    // Display the alert that adds the marker
    private void showAlertDialogForPoint(final LatLng point) {
        // ...

        alertDialog.setButton(DialogInterface.BUTTON_POSITIVE, "OK",
            new DialogInterface.OnClickListener() {
            @Override
            public void onClick(DialogInterface dialog, int which) {
                // Define color of marker icon
                BitmapDescriptor defaultMarker = BitmapDescriptorFactory
                    .defaultMarker(BitmapDescriptorFactory.HUE_GREEN);
                // Extract content from alert dialog
                String title = ((EditText) alertDialog.findViewById(R.id.etTitle))
                    .getText().toString();
                String snippet = ((EditText) alertDialog.findViewById(R.id.etSnippet))
                    .getText().toString();
                // Creates and adds marker to the map
                Marker marker = map.addMarker(new MarkerOptions().position(point)
                    .title(title).snippet(snippet).icon(defaultMarker));

                // Animate marker using drop effect
                // --> Call the dropPinEffect method here
                dropPinEffect(marker);
            }
          });

         // ...
    }
```

### Customize InfoWindow

If you want to use your own layout instead of the default options, you can do so by creating your own `InfoWindowAdapter` seen below. Create a layout file in `res/layout/custom_info_window.xml`:

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="#FFFFFF"
    android:orientation="vertical" >

    <TextView
        android:id="@+id/tv_info_window_title"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textColor="#000000"
        android:textStyle="bold"
        android:singleLine="true" />

         <TextView
            android:id="@+id/tv_info_window_description"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:textColor="#000000"
            android:text="Hours" />

</LinearLayout>
```

And here is the code you need to implement your own `InfoWindowAdpapter` class.


```java
class CustomWindowAdapter implements InfoWindowAdapter{
	LayoutInflater mInflater;
	
	public CustomWindowAdapter(LayoutInflater i){
		mInflater = i;
	}

	// This defines the contents within the info window based on the marker
	@Override
	public View getInfoContents(Marker marker) {
	    // Getting view from the layout file
	    View v = mInflater.inflate(R.layout.custom_info_window, null);
	    // Populate fields	
	    TextView title = (TextView) v.findViewById(R.id.tv_info_window_title);
	    title.setText(marker.getTitle());

	    TextView description = (TextView) v.findViewById(R.id.tv_info_window_description);
	    description.setText(marker.getSnippet());
	    // Return info window contents
	    return v;
	}
        
	// This changes the frame of the info window; returning null uses the default frame.
        // This is just the border and arrow surrounding the contents specified above
	@Override
	public View getInfoWindow(Marker marker) {
		return null;
	}
}
```

You would use this by setting your `InfoWindowAdapter` to this new class after you have initialized your map. In the case of our example running this after calling `setUpMapIfNeeded()` in `onCreate()`.

```java
map.setInfoWindowAdapter(new CustomWindowAdapter(getLayoutInflater()));
```

This will use the custom adapter to create the information window, allowing us to customize how the information is displayed.