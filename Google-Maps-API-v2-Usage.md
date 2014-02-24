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