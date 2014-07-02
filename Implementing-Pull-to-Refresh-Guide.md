## Overview

In Android, the common "pull to refresh" UX concept is not built in to a ListView. However, many Android applications would like to make use of this concept for their feeds. This is useful for all sorts of feeds such as a Twitter timeline.

Thankfully, there is an excellent third-party [pull-to-refresh](https://github.com/erikwt/PullToRefresh-ListView) library that can be used to make this interaction extremely easy to implement. 

**Note:** If using Gradle and Android Studio, you may want to check out this alternative library called [chrisbanes/ActionBar-PullToRefresh](https://github.com/chrisbanes/ActionBar-PullToRefresh). [Quick start](https://github.com/chrisbanes/ActionBar-PullToRefresh/wiki/QuickStart-Stock) can be followed for easy setup.

### Step 1: Installation

Download the zip of [pull-to-refresh](https://github.com/erikwt/PullToRefresh-ListView/archive/master.zip) and then [import the library folder](http://imgur.com/a/N8baF) into Eclipse with **File => Import => Existing Android Code Into Workspace**. Call the project "PullToRefresh" and then start the import. This folder is now a "library project" and can then be included as a library in your Android apps. Make sure to mark this project as a library and then include the library in your application Check the [Detailed Instructions Here](http://imgur.com/a/N8baF) for a step-by-step of this process.

### Step 2: Replace ListView

Next, for each ListView that we want to add pull-to-refresh functionality to, we need to replace the `ListView` in the XML with the new third-party `eu.erikw.PullToRefreshListView` which has all the same features of a ListView. For example:

```xml
<ListView
    android:id="@+id/pull_to_refresh_listview"
    android:layout_height="fill_parent"
    android:layout_width="fill_parent" />
```

would be replaced with:

```xml
<eu.erikw.PullToRefreshListView
    android:id="@+id/pull_to_refresh_listview"
    android:layout_height="fill_parent"
    android:layout_width="fill_parent" />
```

All other properties in the XML can stay the same.

### Step 3: Setup OnRefreshListener

Now, we need to specify a `OnRefreshListener` for each `PullToRefreshListView` which describes the process for refreshing the contents. This is usually an asynchronous network request for new data from an API. For example:

```java
public class TimelineActivity extends Activity {
    PullToRefreshListView lvTweets;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_timeline);
        lvTweets = (PullToRefreshListView) findViewById(R.id.lvHomeTimeline);
        // Set a listener to be invoked when the list should be refreshed.
        lvTweets.setOnRefreshListener(new OnRefreshListener() {
            @Override
            public void onRefresh() {
                // Your code to refresh the list contents
                // Make sure you call listView.onRefreshComplete()
                // once the loading is done. This can be done from here or any
                // place such as when the network request has completed successfully.
                fetchTimelineAsync(0);
            }
        });
    }

    public void fetchTimelineAsync(int page) {
        client.getHomeTimeline(0, new JsonHttpResponseHandler() {
            public void onSuccess(JSONArray json) {
                // ...the data has come back, finish populating listview...
                // Now we call onRefreshComplete to signify refresh has finished
                lvTweets.onRefreshComplete();
            }

            public void onFailure(Throwable e) {
                Log.d("DEBUG", "Fetch timeline error: " + e.toString());
            }
        });
    }
}
```

Make sure that when the network request is complete that you call `lvTweets.onRefreshComplete();` in order to complete the refresh as shown above.

### (Optional) Step 4: Customize Refresh Style

The library has rich support for [customizing the refresh graphics](https://github.com/erikwt/PullToRefresh-ListView#style). To do so, override the style attributes to your liking, like the following example:

``` xml
<style name="ptr_text">
        
    <!-- Change the text style and color -->
    <item name="android:textStyle">bold|italic</item>
    <item name="android:textColor">#cccccc</item>
</style>
```

The various styles you can override are:

* ptr_headerContainer
* ptr_header
* ptr_arrow
* ptr_spinner
* ptr_text

Check out the [default styles file](https://github.com/erikwt/PullToRefresh-ListView/blob/master/libraryproject/res/values/ptr_default_style.xml) for a style reference. You can also see the [official library readme](https://github.com/erikwt/PullToRefresh-ListView#style) for more details.

### Troubleshooting

#### Error inflating class eu.erikw.PullToRefreshListView

Be sure to run a `Project => Clean` on both the library project and the application if you receive this exception. 

#### Refresh never seems to complete

Make sure that when the data is finished loading that you call `lvTweets.onRefreshComplete();` in order to complete the refresh.

#### Can't import into Gradle

If using Gradle and Android Studio, you may want to check out this alternative library called [chrisbanes/ActionBar-PullToRefresh](https://github.com/chrisbanes/ActionBar-PullToRefresh). [Quick start](https://github.com/chrisbanes/ActionBar-PullToRefresh/wiki/QuickStart-Stock) can be followed for easy setup.

## References

* <https://github.com/erikwt/PullToRefresh-ListView> - Third-party library for pull to refresh