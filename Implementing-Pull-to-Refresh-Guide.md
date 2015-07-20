## Overview

In Android, the common "pull to refresh" UX concept is not built in to a ListView. However, many Android applications would like to make use of this concept for their feeds. This is useful for all sorts of feeds such as a Twitter timeline. This effect can be achieved using either the `SwipeRefreshLayout` from the [support library](http://developer.android.com/tools/support-library/index.html), which was recently introduced and back-ported to all versions down to Android API level 4.

## Using SwipeRefreshLayout

[SwipeRefreshLayout](https://developer.android.com/reference/android/support/v4/widget/SwipeRefreshLayout.html) is a ViewGroup that can hold only one scrollable view as a child. This can be either a `ScrollView` or an `AdapterView` such as a `ListView` or a `RecyclerView`. 

**Note:** This layout only exists within more recent versions of support-v4 as [explained in this post](http://stackoverflow.com/a/23325011/313399). Edit your `app/build.gradle` file to include a support library later than version 19:

```gradle
apply plugin: 'com.android.application'

//...

dependencies {
    // ...
    compile 'com.android.support:support-v4:22.2.1'
}
```

You must download and use a [recent jar](https://dl-ssl.google.com/android/repository/support_r20.zip) of the support library for this to work or install the support library via the Android Studio SDK Manager:

1. Open the SDK Manager from Android Studio with Tools -> Android -> SDK Manager
2. The support library is under "Extras"

Once you have a recent version support library installed, we can continue.

## ListView with SwipeRefreshLayout

### Step 1: Wrap ListView

We can use this by first wrapping the scrollable view with a `SwipeRefreshLayout` in the XML layout:

```xml
<android.support.v4.widget.SwipeRefreshLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/swipeContainer"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

  <ListView
      android:id="@+id/lvItems"
      android:layout_width="match_parent"
      android:layout_height="wrap_content"
      android:layout_alignParentLeft="true"
      android:layout_alignParentTop="true" >
  </ListView>

</android.support.v4.widget.SwipeRefreshLayout>
```

### Step 2: Setup SwipeRefreshLayout

Next, we need to configure the `SwipeRefreshLayout` during view initialization in the activity:

```java
public class TimelineActivity extends Activity {
    private SwipeRefreshLayout swipeContainer;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        swipeContainer = (SwipeRefreshLayout) findViewById(R.id.swipeContainer);
        // Setup refresh listener which triggers new data loading
        swipeContainer.setOnRefreshListener(new OnRefreshListener() {
            @Override
            public void onRefresh() {
                // Your code to refresh the list here.
                // Make sure you call swipeContainer.setRefreshing(false)
                // once the network request has completed successfully.
                fetchTimelineAsync(0);
            } 
        });
        // Configure the refreshing colors
        swipeContainer.setColorSchemeResources(android.R.color.holo_blue_bright, 
                android.R.color.holo_green_light, 
                android.R.color.holo_orange_light, 
                android.R.color.holo_red_light);
    }

    public void fetchTimelineAsync(int page) {
        client.getHomeTimeline(0, new JsonHttpResponseHandler() {
            public void onSuccess(JSONArray json) {
                // Remember to CLEAR OUT old items before appending in the new ones
                adapter.clear();
                // ...the data has come back, add new items to your adapter...
                adapter.addAll(...);
                // Now we call setRefreshing(false) to signal refresh has finished
                swipeContainer.setRefreshing(false);
            }

            public void onFailure(Throwable e) {
                Log.d("DEBUG", "Fetch timeline error: " + e.toString());
            }
        });
    }
}
```

**Note** that upon successful reload, we must also signal that the refresh has completed by calling `setRefreshing(false)`. Also note that you should **clear out old items** before appending the new ones during a refresh.

## RecyclerView with SwipeRefreshLayout

### Step 1: Wrap RecyclerView

Just like the previous section, wrap the scrollable view, in this case a `RecyclerView` with a `SwipeRefreshLayout` in the XML layout:

```xml
<android.support.v4.widget.SwipeRefreshLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/swipeContainer"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

  <android.support.v7.widget.RecyclerView
      android:id="@+id/rvItems"
      android:layout_width="match_parent"
      android:layout_height="wrap_content"
      android:layout_alignParentLeft="true"
      android:layout_alignParentTop="true" />

</android.support.v4.widget.SwipeRefreshLayout>
```

### Step 2: Update RecyclerView.Adapter

Make sure to have helper methods in your `RecyclerView` adapter to clear items from the underlying dataset or add items to it.

```java
/* Within the RecyclerView.Adapter class */

// Clean all elements of the recycler
public void clear() { 
    items.clear(); 
    notifyDataSetChanged(); 
}

// Add a list of items
public void addAll(List<list> list) { 
    items.addAll(list); 
    notifyDataSetChanged(); 
}
```

### Step 3: Setup SwipeRefreshLayout

Next, we need to configure the `SwipeRefreshLayout` during view initialization in the activity. The activity that instantiates `SwipeRefreshLayout` should add an `OnRefreshListener` to be notified whenever the swipe to refresh gesture is completed. 

The `SwipeRefreshLayout` will notify the listener each and every time the gesture is completed again; the listener is responsible for correctly determining when to actually initiate a refresh of its content. 

You can do this the same way you can configure the `SwipeRefreshLayout` for a `ListView` as shown in [Setup SwipeRefreshLayout](#step-2-setup-swiperefreshlayout) section.

## References

* <http://antonioleiva.com/swiperefreshlayout/>