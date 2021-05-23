## Overview

In Android, the common "pull to refresh" UX concept is not built in to a ListView/RecyclerView. However, many Android applications would like to make use of this concept for their feeds. This is useful for all sorts of feeds such as a Twitter timeline. This effect can be achieved using the `SwipeRefreshLayout` [class](https://developer.android.com/reference/androidx/swiperefreshlayout/widget/SwipeRefreshLayout)

<img src="https://i.imgur.com/PtY0Ju1.gif" height="400" />
<img src="https://materialdoc.com/images/custom_swipe.gif" height="400" />&nbsp;

## Using SwipeRefreshLayout

[SwipeRefreshLayout](https://developer.android.com/reference/androidx/swiperefreshlayout/widget/SwipeRefreshLayout.html) is a ViewGroup that can hold only one scrollable view as a child. This can be either a `ScrollView` or an `AdapterView` such as a `ListView` or a `RecyclerView`. 

Edit your `app/build.gradle` file to include a  library:

```gradle
apply plugin: 'com.android.application'

//...

dependencies {
    // ...
    implementation 'androidx.swiperefreshlayout:swiperefreshlayout:1.0.0'
}
```

Make sure your libraries is up to date by adding to your root `gradle.file`:

```gradle
allprojects {
    repositories {
        // requires Gradle v4.1+
        google()
    }
}
```

## RecyclerView with SwipeRefreshLayout

### Step 1: Wrap RecyclerView

Just like the previous section, wrap the scrollable view, in this case a `RecyclerView` with a `SwipeRefreshLayout` in the XML layout:

```xml
<androidx.swiperefreshlayout.widget.SwipeRefreshLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/swipeContainer"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

  <androidx.recyclerview.widget.RecyclerView
      android:id="@+id/rvItems"
      android:layout_width="match_parent"
      android:layout_height="wrap_content"
      android:layout_alignParentLeft="true"
      android:layout_alignParentTop="true" />

</androidx.swiperefreshlayout.widget.SwipeRefreshLayout>
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
```kotlin
    fun clear() {
        tweets.clear()
        notifyDataSetChanged()
    }

    fun addAll(tweetList: List<Tweet>) {
        tweets.addAll(tweetList)
        notifyDataSetChanged()
    }
```

### Step 3: Setup SwipeRefreshLayout

Next, we need to configure the `SwipeRefreshLayout` during view initialization in the activity. The activity that instantiates `SwipeRefreshLayout` should add an `OnRefreshListener` to be notified whenever the swipe to refresh gesture is completed. 

The `SwipeRefreshLayout` will notify the listener each and every time the gesture is completed again; the listener is responsible for correctly determining when to actually initiate a refresh of its content. 

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

```kotlin
class TimelineActivity: Activity {
    lateinit var swipeContainer: SwipeRefreshLayout

    override fun void onCreate(savedInstanceState: Bundle) {
        super.onCreate(savedInstanceState)
        // Only ever call `setContentView` once right at the top
        setContentView(R.layout.activity_main)
        // Lookup the swipe container view
        swipeContainer = findViewById(R.id.swipeContainer)
        // Setup refresh listener which triggers new data loading

        swipeContainer.setOnRefreshListener {
            // Your code to refresh the list here.
            // Make sure you call swipeContainer.setRefreshing(false)
            // once the network request has completed successfully.
            fetchTimelineAsync(0)
        }

        // Configure the refreshing colors
        swipeContainer.setColorSchemeResources(android.R.color.holo_blue_bright, 
                android.R.color.holo_green_light, 
                android.R.color.holo_orange_light, 
                android.R.color.holo_red_light);
    }

    fun fetchTimelineAsync(page: Integer) {
        // Send the network request to fetch the updated data
        // `client` here is an instance of Android Async HTTP
        // getHomeTimeline is an example endpoint.
        client.getHomeTimeline(object: JsonHttpResponseHandler() {
            override fun onSuccess(statusCode: Int, headers: okhttp3.Headers, json: JSON) {
                // Remember to CLEAR OUT old items before appending in the new ones
                adapter.clear()
                // ...the data has come back, add new items to your adapter...
                adapter.addAll(...)
                // Now we call setRefreshing(false) to signal refresh has finished
                swipeContainer.setRefreshing(false)
            }

            override fun onFailure(
                statusCode: Int,
                headers: okhttp3.Headers,
                response: String,
                throwable: Throwable
            ) {
                Log.d("DEBUG", "Fetch timeline error", throwable)
            }
        })
    }
}
```

**Note** that upon successful reload, we must also signal that the refresh has completed by calling `setRefreshing(false)`. Also note that you should **clear out old items** before appending the new ones during a refresh.

### Using with Paging Library

If you are using SwipeRefreshLayout with Android's new [[Paging Library|Paging Library Guide]], the data sources used to provide data to the RecyclerView need to be invalidated.  Review [[this guide|https://guides.codepath.com/android/Paging-Library-Guide#using-with-swiperefreshlayout]] for more information.

## SwipeRefreshLayout with ListView (deprecated)

**Note**: `ListView` is an old UI component that is no longer used in modern Android applications. Only refer this guide if you intend to update some old code that still relies on ListView.

### Step 1: Setting SwipeRefreshLayout

Set SwipeRefreshLayout at the Layout you want the SwipeRefresh functionality

**activity_main.xml**
```xml
<androidx.swiperefreshlayout.widget.SwipeRefreshLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/swipe_container"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

</androidx.swiperefreshlayout.widget.SwipeRefreshLayout>
```
### Step 2: Set your ListView inside Layout

**activity_main.xml**
```xml
<androidx.swiperefreshlayout.widget.SwipeRefreshLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/swipe_container"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <ListView
        android:id="@+id/listView"
        android:layout_width="match_parent"
        android:layout_height="wrap_content">

    </ListView>

</androidx.swiperefreshlayout.widget.SwipeRefreshLayout>
```
_You could use a ScrollView instead a ListView_

### Step 3: Now in your main_activity set your code

In the activity who points to activity_main.xml, which is main_activity(in this example), this code should be enough

**main_activity.java**
```java
public class MainActivity extends AppCompatActivity {
    SwipeRefreshLayout swipeLayout;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        // Getting SwipeContainerLayout
        swipeLayout = findViewById(R.id.swipe_container);
        // Adding Listener
        swipeLayout.setOnRefreshListener(new OnRefreshListener() {
            @Override
            public void onRefresh() {
                // Your code here
                Toast.makeText(getApplicationContext(), "Works!", Toast.LENGTH_LONG).show();
                // To keep animation for 4 seconds
                new Handler().postDelayed(new Runnable() {
                    @Override public void run() {
                        // Stop animation (This will be after 3 seconds)
                        swipeLayout.setRefreshing(false);
                    }
                }, 4000); // Delay in millis
            }
        });

        // Scheme colors for animation
        swipeLayout.setColorSchemeColors(
                getResources().getColor(android.R.color.holo_blue_bright),
                getResources().getColor(android.R.color.holo_green_light),
                getResources().getColor(android.R.color.holo_orange_light),
                getResources().getColor(android.R.color.holo_red_light)
        );
    }
}
```

Now just run your application!

You could check this example on [GitHub](https://github.com/Luttos/Android/tree/master/SwipeToRefresh).

## Troubleshooting

If you aren't able to get the swipe to refresh working, check the following tips:

* **Did you accidentally call `setContentView` twice?** Ensure that inside your activity, you've only called [`setContentView`](http://developer.android.com/reference/android/app/Activity.html#setContentView\(android.view.View\)) once as the 2nd line of your `onCreate` method. 

* **Did you invoke `setRefreshing(false)` after data finished loading?** With the swipe to refresh control, you are responsible for notifying the system once the new data has been loaded into the list. You must make sure to invoke [setRefreshing](https://developer.android.com/reference/androidx/swiperefreshlayout/widget/SwipeRefreshLayout.html#setRefreshing\(boolean\)) only **after** the data has come back and not before. This means if you are loading data from the network, calling this within the `onSuccess` method.

* **Did you clear out the old items before updating the list?** Make sure that in order for the new items to be displayed that you **clear the list of any old items if needed**. In other words, if you are replacing items in the list with new versions, be sure to remove the old versions from the adapter first with `adapter.clear();`

* **Are you using CoordinatorLayout?** If you are [[using a CoordinatorLayout|Handling-Scrolls-with-CoordinatorLayout]] to manage scrolling, be sure to move the `app:layout_behavior="@string/appbar_scrolling_view_behavior"` property to the `SwipeRefreshLayout` rather than the child `RecyclerView` or `ListView`.

## References

* <https://developer.android.com/reference/androidx/swiperefreshlayout/widget/SwipeRefreshLayout>
* <http://antonioleiva.com/swiperefreshlayout/>