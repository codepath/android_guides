### Paging Library Guide

A common feature is to load data automatically as a user scrolls through the items (i.e. infinite scroll).  Previously there were only [[third party options|Endless-Scrolling-with-AdapterViews-and-RecyclerView]] that could accomplish this goal.  Google's new Paging Library now provides this support.  

### Setup

Make sure to add the Google Maven repository in your root `build.gradle`:
```gradle
allprojects {
  repositories {
    google()
    jcenter()
  }
}
```

Next, add the Paging library to your `app/build.gradle`:

```gradle
dependencies {
     implementation "android.arch.paging:runtime:1.0.0"
}
```

### Modifying your RecyclerView

The Paging Library can only be used with [[RecyclerView|Using-the-RecyclerView]].  If you are using ListView to display your lists, you should first migrate to using the [[ViewHolder pattern|Using-an-ArrayAdapter-with-ListView#improving-performance-with-the-viewholder-pattern]] before transitioning to RecyclerView.  Moving to RecyclerView may also require changing how [[click handling|Using-the-RecyclerView#attaching-click-handlers-to-items]] is performed.

#### Modifying to ListAdapter

Once your lists are using RecyclerView, the next step is to change your adapters to inherit from the `ListAdapter` class instead of the `RecyclerView.Adapter` class.  See [this guide](https://github.com/codepath/android_guides/wiki/Using-the-RecyclerView#using-with-listadapter) for more information.  

#### Moving to PagedListAdapter

Once the ListAdapter is made, we will modify the `ListAdapter` to be a `PagedListAdapter`.  The `PagedListAdapter` enables for data to be loaded in chunks and does not require all the data to be loaded in memory.

Change from:

```java
public class TweetAdapter extends RecyclerView.Adapter<PostAdapter.ViewHolder> {
```

To:

```java
public class TweetAdapter extends ListAdapter<Post, PostAdapter.ViewHolder> {
```

In addition, the adapter should no longer retain a copy of its current list.  The `getItem()` method should be used instead.

```kotlin
override fun onBindViewHolder(holder: PostAdapter.ViewHolder, position: Int) {

    // getItem() should be used with ListAdapter
    val tweet = getItem(position)

    // null placeholders if the PagedList is configured to use them
    // only works for data sets that have total count provided (i.e. PositionalDataSource)
    if (tweet == null)
    {
        return;
    }
    // handle remaining work here
    // ..
}
```

#### Add a data source

The next step is to define our data sources.  If you are already making existing network or database calls somewhere else, you most likely will need to move this code into a data source.   You will also need to first pick what kind of [data source](https://developer.android.com/reference/android/arch/paging/DataSource) to use:

  * **ItemKeyedDataSource** - provides some type of key to use to fetch the next data size and unknown list size (example: Twitter API)
  * **PageKeyedDataSource** - provides a next/before link and has unknown list size (example: Facebook Graph API)
  * **PositionalDataSource** - Useful for sources that provide a fixed size list with known PositionalDataSource (example: Room, Parse)

##### Using ItemKeyedDataSource

The first step is to define the key that will be used to determine the next page of data.  In the Twitter API case, for instance, the Twitter Post ID can be used as a way to query for posts older than that Tweet ID.  The last post ID can be used to retrieve the next set of posts, thanks to the [max_id](https://developer.twitter.com/en/docs/tweets/timelines/api-reference/get-statuses-home_timeline.html) parameter in the Twitter API:

```java
public class TweetDataSource extends ItemKeyedDataSource<Long, Tweet> {

  // first type of ItemKeyedDataSource should match return type of getKey()
  @NonNull
  @Override
  public Long getKey(@NonNull Tweet item) {
      // item.getPostId() is a Long type
      return item.getPostId();
  }
```

Next, we should make sure to pass into the data constructor whatever dependencies are needed.  
```java
// pass whatever dependencies are needed to make the network call
TwitterClient mClient;

// define the type of data that will be emitted by this datasource
public TweetDataSource(TwitterClient client) {
    mClient = client;
}
```

Next, we need to define inside the data source the `loadInitial()` and `loadAfter()`. 
```java
public void loadInitial(@NonNull LoadInitialParams<Long> params, @NonNull final LoadInitialCallback<Tweet> callback) {

   JsonHttpResponseHandler jsonHttpResponseHandler = createTweetHandler(callback);
   // no max_id should be passed on initial load
   mClient.getHomeTimeline(params.requestedLoadSize, jsonHttpResponseHandler);
}

// Called repeatedly when more data needs to be set
@Override
public void loadAfter(@NonNull LoadParams<Long> params, @NonNull LoadCallback<Tweet> callback) {
   // fetch data synchronously
   JsonHttpResponseHandler jsonHttpResponseHandler = createTweetHandler(callback);

   // params.key & requestedLoadSize should be used
   // params.key will be the lowest Twitter post ID retrieved and should be used for the max_id= parameter in Twitter API.
   mClient.getHomeTimeline(params.key, params.requestedLoadSize, jsonHttpResponseHandler);
}
```

The `createTweetHandler()` method parses the JSON data and posts the data to the RecyclerView adapter and PagedList handler.  Keep in mind that all of these network calls are already done in a background thread, so the network call should be forced to run synchronously:

```java
public JsonHttpResponseHandler createTweetHandler(final LoadCallback<Tweet> callback) {

 JsonHttpResponseHandler handler = new JsonHttpResponseHandler() {
     @Override
     public void onSuccess(int statusCode, Header[] headers, JSONArray response) {
         ArrayList<Tweet> tweets = new ArrayList<Tweet>();
         tweets = Tweet.fromJson(response);

         // send back to PagedList handler           
         callback.onResult(tweets);
     }
 };

 // fetch data synchronously
 // For AsyncHttpClient, this workaround forces the callback to be run synchronously
 handler.setUseSynchronousMode(true);
 handler.setUsePoolThread(true);
```

#### Add a Data Source Factory

A data source factory simply creates the data source.  Because of the dependency on the Twitter client, we need to pass it here too:

```java
public class TweetDataSourceFactory extends DataSource.Factory<Long, Tweet> {
    TwitterClient client;

    public TweetDataSourceFactory(TwitterClient client) {
        this.client = client;
    }
    @Override
    public DataSource<Long, Tweet> create() {
        TweetDataSource dataSource = new TweetDataSource(this.client);
        return dataSource;
    }
}

```

#### Creating a PagedList

Once we have created the data source and data source factory, the final step is to create the PagedList and listen for updates.  We can then call the `submitList()` on our adapter with this next set of data:

```java		
public abstract class BaseTimelineFragment extends Fragment {

  LiveData<PagedList<Tweet>> tweets;

  public void onActivityCreated(Bundle savedInstanceState) {

    tweets = new LivePagedListBuilder(factory, config).build();

    tweets.observe(this, new Observer<PagedList<Tweet>>() {
			    @Override
			    public void onChanged(@Nullable PagedList<Tweet> tweets) {
				tweetAdapter.submitList(tweets);
			    }
		  });

  }
```

### Adding multiple data sources

The Paging Library requires that the data used in the RecyclerView, regardless of whether they are pulled from the network, database, or memory, be defined as **data sources**.  It also allows the flexibility to first pull data from the database or memory, and if there is no more data, to pull additional segments of data from the network.   To help showcase how the Paging Library works, you can also try out [this sample code](https://github.com/googlesamples/android-architecture-components/tree/master/PagingWithNetworkSample) from Google.

### References

* <https://developer.android.com/topic/libraries/architecture/paging/>
* <https://developer.android.com/topic/libraries/architecture/paging/#update-existing-app>
* <https://www.youtube.com/watch?v=QVMqCRs0BNA/>
* <https://codelabs.developers.google.com/codelabs/android-paging/index.html>
