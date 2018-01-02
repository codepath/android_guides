## Overview

In Android, the common "pull to refresh" UX concept is not built in to a ListView. However, many Android applications would like to make use of this concept for their feeds. This is useful for all sorts of feeds such as a Twitter timeline. This effect can be achieved using the `SwipeRefreshLayout` from the [support library](http://developer.android.com/tools/support-library/index.html), which was recently introduced and back-ported to all versions down to Android API level 4.

<img src="https://i.imgur.com/6BecRWd.gif" height="400" />&nbsp;
<img src="https://i.imgur.com/PtY0Ju1.gif" height="400" />

## Using SwipeRefreshLayout

[SwipeRefreshLayout](https://developer.android.com/reference/android/support/v4/widget/SwipeRefreshLayout.html) is a ViewGroup that can hold only one scrollable view as a child. This can be either a `ScrollView` or an `AdapterView` such as a `ListView` or a `RecyclerView`. 

**Note:** This layout only exists within more recent versions of support-v4 as [explained in this post](http://stackoverflow.com/a/23325011/313399). Edit your `app/build.gradle` file to include a support library later than version 19:

```gradle
apply plugin: 'com.android.application'

//...

dependencies {
    // ...
    compile 'com.android.support:support-v4:26.0.2'
}
```

Make sure your support library is up to date by adding to your root `gradle.file`:

```gradle
allprojects {
    repositories {
        // add below
        maven { url "https://maven.google.com" }
    }
}
```

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
        // Only ever call `setContentView` once right at the top
        setContentView(R.layout.activity_main);
        // Lookup the swipe container view
        swipeContainer = (SwipeRefreshLayout) findViewById(R.id.swipeContainer);
        // Setup refresh listener which triggers new data loading
        swipeContainer.setOnRefreshListener(new SwipeRefreshLayout.OnRefreshListener() {
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
        // Send the network request to fetch the updated data
        // `client` here is an instance of Android Async HTTP
        // getHomeTimeline is an example endpoint.
        client.getHomeTimeline(new JsonHttpResponseHandler() {
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

**Having issues?** Be sure to check out the [[troubleshooting tips|Implementing-Pull-to-Refresh-Guide#troubleshooting]] if you are running into issues with swipe to refresh.

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

// Add a list of items -- change to type used
public void addAll(List<Tweet> list) { 
    items.addAll(list); 
    notifyDataSetChanged(); 
}
```

### Step 3: Setup SwipeRefreshLayout

Next, we need to configure the `SwipeRefreshLayout` during view initialization in the activity. The activity that instantiates `SwipeRefreshLayout` should add an `OnRefreshListener` to be notified whenever the swipe to refresh gesture is completed. 

The `SwipeRefreshLayout` will notify the listener each and every time the gesture is completed again; the listener is responsible for correctly determining when to actually initiate a refresh of its content. 

You can do this the same way you can configure the `SwipeRefreshLayout` for a `ListView` as shown in [Setup SwipeRefreshLayout](#step-2-setup-swiperefreshlayout) section.

## Troubleshooting

If you aren't able to get the swipe to refresh working, check the following tips:

* **Did you accidentally call `setContentView` twice?** Ensure that inside your activity, you've only called [`setContentView`](http://developer.android.com/reference/android/app/Activity.html#setContentView\(android.view.View\)) once as the 2nd line of your `onCreate` method. 

* **Did you invoke `setRefreshing(false)` after data finished loading?** With the swipe to refresh control, you are responsible for notifying the system once the new data has been loaded into the list. You must make sure to invoke [setRefreshing](http://developer.android.com/reference/android/support/v4/widget/SwipeRefreshLayout.html#setRefreshing\(boolean\)) only **after** the data has come back and not before. This means if you are loading data from the network, calling this within the `onSuccess` method.

* **Did you clear out the old items before updating the list?** Make sure that in order for the new items to be displayed that you **clear the list of any old items if needed**. In other words, if you are replacing items in the list with new versions, be sure to remove the old versions from the adapter first with `adapter.clear();`

* **Are you using CoordinatorLayout?** If you are [[using a CoordinatorLayout|Handling-Scrolls-with-CoordinatorLayout]] to manage scrolling, be sure to move the `app:layout_behavior="@string/appbar_scrolling_view_behavior"` property to the `SwipeRefreshLayout` rather than the child `RecyclerView` or `ListView`.

## References

* <http://antonioleiva.com/swiperefreshlayout/>
