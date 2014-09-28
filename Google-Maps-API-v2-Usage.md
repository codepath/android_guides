## Overview

This guide will instruct you on how use the Google Maps API v2 library to create a map within your application's activities and/or fragments.

### Setup Map If Necessary

Assuming you have added the GoogleMapv2 mapfragment to your layout, you should first run this method `onCreate()`. In my case, I am using the SupportMapFragment, either will work.

```java
private void setUpMapIfNeeded() {
    // Do a null check to confirm that we have not already instantiated the map.
    if (mapFragment == null) {
        mapFragment = ((SupportMapFragment) getFragmentManager().findFragmentById(R.id.map)).getMap();
        // Check if we were successful in obtaining the map.
        if (mapFragment != null) {
           // The Map is verified. It is now safe to manipulate the map.
        }
    }		   
}
```

After you have run that it is safe for you to begin adding markers.

### Adding Markers to Map Fragment

```java
Marker mapMarker = mapFragment.addMarker(new MarkerOptions()
    .position(listingPosition)					 								    
    .title(listing.name)
    .snippet("Hours: " + listing.hoursOpen + " - "						 						  
        + listing.hoursClose)
    .icon(BitmapDescriptorFactory.fromResource(getMarkerType(listing.marker))));
```

### Customize InfoWindow

If you want to use your own layout instead of the default options, you can do so by creating your own InfoWindowAdapter seen below.

Here is the layout file `custom_info_window.xml`

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

And here is the code you need to implement your own InfoWindowAdpapter class.


```java
class CustomWindowAdapter implements InfoWindowAdapter{
	LayoutInflater mInflater;
	
	public CustomWindowAdapter(LayoutInflater i){
		mInflater = i;
	}

	@Override
	public View getInfoContents(Marker marker) {
	    // Getting view from the layout file
	    View v = mInflater.inflate(R.layout.custom_info_window, null);
	    	   
	    TextView title = (TextView) v.findViewById(R.id.tv_info_window_title);
	    title.setText(marker.getTitle());

	    TextView description = (TextView) v.findViewById(R.id.tv_info_window_description);
	    description.setText(marker.getSnippet());

	    return v;
	}

	@Override
	public View getInfoWindow(Marker marker) {
		// TODO Auto-generated method stub
		return null;
	}
}
```


You would use this by setting your InfoWindowAdapter to this new class after you have initialized your map. In the case of my example I am running this after I have run `setUpMapIfNeeded()` in my `onCreate()`.

```java
mapFragment.setInfoWindowAdapter(new CustomWindowAdapter(getActivity().getLayoutInflater(), mapRatingHash));
```

### Show AlertDialog on LongClick

You can use the following code to bring up an `AlertDialog` for users to type a message on MapLongClick event. On completion, it adds a marker to the maps which when pressed displays the message in an info window.

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
        android:text="Title : " />

    <EditText
        android:id="@+id/etTitle"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_alignBottom="@+id/tvTitle"
        android:layout_toRightOf="@+id/tvTitle" >
    </EditText>

    <TextView
        android:id="@+id/tvSnippet"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_below="@+id/tvTitle"
        android:paddingTop="20dp"
        android:text="Snippet : " />

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

```java

public class MapDemoActivity extends FragmentActivity implements
		GooglePlayServicesClient.ConnectionCallbacks,
		GooglePlayServicesClient.OnConnectionFailedListener,
		OnMapLongClickListener {

        ...

        @Override
	protected void onCreate(Bundle savedInstanceState) {
		
                ...


		map.setOnMapLongClickListener(this);
	}

        ...

        @Override
	public void onMapLongClick(final LatLng point) {
		Toast.makeText(getApplicationContext(), "long press", Toast.LENGTH_LONG)
				.show();

		// get message_item.xml view
		View MessageView = LayoutInflater.from(MapDemoActivity.this).inflate(R.layout.message_item, null);

		AlertDialog.Builder alertDialogBuilder = new AlertDialog.Builder(MapDemoActivity.this);
		
		// set message_item.xml to alertdialog builder
		alertDialogBuilder.setView(MessageView);
		final EditText etTitle = (EditText) MessageView.findViewById(R.id.etTitle);
		final EditText etSnippet = (EditText) MessageView.findViewById(R.id.etSnippet);
		
		// set dialog message
		alertDialogBuilder
				.setCancelable(false)
				.setPositiveButton("OK", new DialogInterface.OnClickListener() {
					public void onClick(DialogInterface dialog, int id) {
						// Create and add marker
						Marker marker = map.addMarker(new MarkerOptions()
								.position(point)
								.title(etTitle.getText().toString())
								.snippet(etSnippet.getText().toString())							
								.icon(BitmapDescriptorFactory.defaultMarker(BitmapDescriptorFactory.HUE_RED)));
					}
				})
				.setNegativeButton("Cancel",
						new DialogInterface.OnClickListener() {
							public void onClick(DialogInterface dialog, int id) {
								dialog.cancel();
							}
						});

		// create alert dialog
		AlertDialog alertDialog = alertDialogBuilder.create();

		// show it
		alertDialog.show();

	}
}
```

### Falling Pin Animation

To implement falling pin animation, add marker to the desired position in map and then call this function with that marker.

```java
private void dropPinEffect(final Marker marker) {
        final Handler handler = new Handler();
        final long start = SystemClock.uptimeMillis();
        final long duration = 1500;

        final Interpolator interpolator = new BounceInterpolator();

        handler.post(new Runnable() {
            @Override
            public void run() {
                long elapsed = SystemClock.uptimeMillis() - start;
                float t = Math.max(
                        1 - interpolator.getInterpolation((float) elapsed
                                / duration), 0);
                marker.setAnchor(0.5f, 1.0f + 14 * t);

                if (t > 0.0) {
                    // Post again 15ms later.
                    handler.postDelayed(this, 15);
                } else {
                    marker.showInfoWindow();

                }
            }
        });
    }
```

```java
Marker marker = map.addMarker(new MarkerOptions()
		.position(point)
		.title(etMessage.getText().toString())								
                .icon(BitmapDescriptorFactory.defaultMarker(BitmapDescriptorFactory.HUE_RED)));
						
dropPinEffect(marker);		
```
