## Overview

A common application feature is to load automatically more items as the user scrolls through the items (aka infinite scroll). This is done by triggering a request for more data once the user crosses a threshold of remaining items before they've hit the end. 

The approaches for ListView, GridView and [[RecyclerView|Using-the-RecyclerView]] (the successor to ListView) are documented here.  Both are similar in code except that the [LayoutManager](http://developer.android.com/reference/android/support/v7/widget/RecyclerView.LayoutManager.html) in the RecyclerView needs to be passed in to provide the necessary information to implement infinite scrolling.  

In both cases, the information needed to implement the scrolling include determining the last visible item within the list and some type of threshold value to start fetching more data before the last item has been reached.  This data can be used to decide when to load more data from an external source:

<a href="https://imgur.com/6E7X1pr.png" target="_blank"><img src="https://imgur.com/6E7X1pr.png"/></a>

To provide the appearance of endless scrolling, it's important to fetch data before the user gets to the end of the list.  Adding a threshold value therefore helps anticipate the need to append more data. 

## Implementing with ListView or GridView

Every `AdapterView` (such as `ListView` and `GridView`) has support for binding to the `OnScrollListener` events which are triggered whenever a user scrolls through the collection. Using this system, we can define a basic `EndlessScrollListener` which supports most use cases by creating our own class that extends `OnScrollListener`:

```java
import android.widget.AbsListView; 

public abstract class EndlessScrollListener implements AbsListView.OnScrollListener {
	// The minimum number of items to have below your current scroll position
	// before loading more.
	private int visibleThreshold = 5;
	// The current offset index of data you have loaded
	private int currentPage = 0;
	// The total number of items in the dataset after the last load
	private int previousTotalItemCount = 0;
	// True if we are still waiting for the last set of data to load.
	private boolean loading = true;
	// Sets the starting page index
	private int startingPageIndex = 0;

	public EndlessScrollListener() {
	}

	public EndlessScrollListener(int visibleThreshold) {
		this.visibleThreshold = visibleThreshold;
	}

	public EndlessScrollListener(int visibleThreshold, int startPage) {
		this.visibleThreshold = visibleThreshold;
		this.startingPageIndex = startPage;
		this.currentPage = startPage;
	}

	
	@Override
	public void onScroll(AbsListView view, int firstVisibleItem, int visibleItemCount, int totalItemCount) 
        {
		// If the total item count is zero and the previous isn't, assume the
		// list is invalidated and should be reset back to initial state
		if (totalItemCount < previousTotalItemCount) {
			this.currentPage = this.startingPageIndex;
			this.previousTotalItemCount = totalItemCount;
			if (totalItemCount == 0) { this.loading = true; } 
		}
		// If it's still loading, we check to see if the dataset count has
		// changed, if so we conclude it has finished loading and update the current page
		// number and total item count.
		if (loading && (totalItemCount > previousTotalItemCount)) {
			loading = false;
			previousTotalItemCount = totalItemCount;
			currentPage++;
		}
		
		// If it isn't currently loading, we check to see if we have breached
		// the visibleThreshold and need to reload more data.
		// If we do need to reload some more data, we execute onLoadMore to fetch the data.
		if (!loading && (firstVisibleItem + visibleItemCount + visibleThreshold) >= totalItemCount ) {
		    loading = onLoadMore(currentPage + 1, totalItemCount);
		}
	}

	// Defines the process for actually loading more data based on page
	// Returns true if more data is being loaded; returns false if there is no more data to load.
	public abstract boolean onLoadMore(int page, int totalItemsCount);

	@Override
	public void onScrollStateChanged(AbsListView view, int scrollState) {
		// Don't take any action on changed
	}
}
```

Notice that this is an abstract class, and that in order to use this, you must extend this base class and define the `onLoadMore` method to actually retrieve the new data. We can define now an anonymous class within any activity that extends `EndlessScrollListener` and bind that to the AdapterView. For example:

```java
public class MainActivity extends Activity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        // ... the usual 
        ListView lvItems = (ListView) findViewById(R.id.lvItems);
        // Attach the listener to the AdapterView onCreate
        lvItems.setOnScrollListener(new EndlessScrollListener() {
          @Override
          public boolean onLoadMore(int page, int totalItemsCount) {
              // Triggered only when new data needs to be appended to the list
              // Add whatever code is needed to append new items to your AdapterView
              loadNextDataFromApi(page); 
              // or loadNextDataFromApi(totalItemsCount); 
              return true; // ONLY if more data is actually being loaded; false otherwise.
          }
        });
    }
    

   // Append the next page of data into the adapter
   // This method probably sends out a network request and appends new data items to your adapter. 
   public void loadNextDataFromApi(int offset) {
      // Send an API request to retrieve appropriate paginated data 
      //  --> Send the request including an offset value (i.e `page`) as a query parameter.
      //  --> Deserialize and construct new model objects from the API response
      //  --> Append the new data objects to the existing set of items inside the array of items
      //  --> Notify the adapter of the new items made with `notifyDataSetChanged()`
   }
}
```

Now as you scroll, items will be automatically filling in because the `onLoadMore` method will be triggered once the user crosses the `visibleThreshold`. This approach works equally well for a `GridView` and the listener gives access to both the `page` as well as the `totalItemsCount` to support both pagination and offset based fetching.

## Implementing with RecyclerView

We can use a similar approach with the [[RecyclerView|Using-the-RecyclerView]] by defining an interface `EndlessRecyclerViewScrollListener` that requires an `onLoadMore()` method to be implemented.  The [LayoutManager](http://developer.android.com/reference/android/support/v7/widget/RecyclerView.LayoutManager.html), which is responsible in the RecyclerView for rendering where items should be positioned and manages scrolling, provides information about the current scroll position relative to the adapter.  For this reason, we need to pass an instance of what LayoutManager is being used to collect the necessary information to ascertain when to load more data.  

Implementing endless pagination for `RecyclerView` requires the following steps:

1. Copy over the [EndlessRecyclerViewScrollListener.java](https://gist.github.com/nesquena/d09dc68ff07e845cc622) into your application.
2. Call `addOnScrollListener(...)` on a `RecyclerView` to enable endless pagination. Pass in an instance of `EndlessRecyclerViewScrollListener` and implement the `onLoadMore` which fires whenever a new page needs to be loaded to fill up the list.
3. Inside the aforementioned `onLoadMore` method, load additional items into the adapter either by sending out a network request or by loading from another source. 

To start handling the scroll events for steps 2 and 3, we need to use the `addOnScrollListener()` method in our `Activity` or `Fragment` and pass in the instance of the `EndlessRecyclerViewScrollListener` with the layout manager as shown below:

```java
public class MainActivity extends Activity {
    // Store a member variable for the listener
    private EndlessRecyclerViewScrollListener scrollListener;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
       // Configure the RecyclerView
       RecyclerView rvItems = (RecyclerView) findViewById(R.id.rvContacts);
       LinearLayoutManager linearLayoutManager = new LinearLayoutManager(this);
       rvItems.setLayoutManager(linearLayoutManager);
       // Retain an instance so that you can call `resetState()` for fresh searches
       scrollListener = new EndlessRecyclerViewScrollListener(linearLayoutManager) {
           @Override
           public void onLoadMore(int page, int totalItemsCount, RecyclerView view) {
               // Triggered only when new data needs to be appended to the list
               // Add whatever code is needed to append new items to the bottom of the list
               loadNextDataFromApi(page);
           }
      };
      // Adds the scroll listener to RecyclerView
      rvItems.addOnScrollListener(scrollListener);
  }

  // Append the next page of data into the adapter
  // This method probably sends out a network request and appends new data items to your adapter. 
  public void loadNextDataFromApi(int offset) {
      // Send an API request to retrieve appropriate paginated data 
      //  --> Send the request including an offset value (i.e `page`) as a query parameter.
      //  --> Deserialize and construct new model objects from the API response
      //  --> Append the new data objects to the existing set of items inside the array of items
      //  --> Notify the adapter of the new items made with `notifyItemRangeInserted()`
  }
}
```

### Resetting the Endless Scroll State

When you intend to perform a new search, make sure to clear the existing contents from the list and notify the adapter the contents have changed **as soon as possible**.  Make sure also to reset the state of the `EndlessRecyclerViewScrollListener` with the `resetState` method:

```java
// 1. First, clear the array of data
listOfItems.clear();
// 2. Notify the adapter of the update
recyclerAdapterOfItems.notifyDataSetChanged(); // or notifyItemRangeRemoved
// 3. Reset endless scroll listener when performing a new search
scrollListener.resetState();
```

You can refer to this [code sample for usage](https://gist.github.com/rogerhu/17aca6ad4dbdb3fa5892) and [this code sample](https://gist.github.com/nesquena/d09dc68ff07e845cc622) for the full endless scroll source code. 

All of the code needed is already incorporated in the `EndlessRecyclerViewScrollListener.java` code snippet above. However, if you wish to understand how the endless scrolling is calculated, the [detailed explanation is available here](https://hackmd.io/s/HyLTXPvH).

## Troubleshooting

If you are running into problems, please carefully consider the following suggestions:

* For the ListView, make sure to setup the `setOnScrollListener` listener in the `onCreate` method of the `Activity` or `onCreateView` in a Fragment and not much later otherwise you may encounter unexpected issues. 

* In order for the pagination system to continue working reliably, you should make sure to **clear the adapter** of items (or notify adapter after clearing the array) before appending new items to the list.  For RecyclerView, it is highly recommended to make more granular updates when notifying the adapter.  See this [video talk] (https://youtu.be/imsr8NrIAMs?t=8m27s) for more context.

* In order for this pagination system to trigger, keep in mind that as `loadNextDataFromApi` is called, new data needs to be **appended to the existing data source**. In other words, only clear items from the list when on the initial "page". Subsequent "pages" of data should be appended to the existing data.

* If you see `Cannot call this method in a scroll callback. Scroll callbacks might be run during a measure & layout pass where you cannot change the RecyclerView data.`, you need to do the following inside your `onLoadMore()` method as outlined in [this Stack Overflow](http://stackoverflow.com/questions/39445330/cannot-call-notifyiteminserted-method-in-a-scroll-callback-recyclerview-v724-2) article to delay the adapter update:

    ```java
    // Delay before notifying the adapter since the scroll listeners 
    // can be called while RecyclerView data cannot be changed.
    view.post(new Runnable() {
        @Override
        public void run() {
            // Notify adapter with appropriate notify methods
            adapter.notifyItemRangeInserted(curSize, allContacts.size() - 1);
        }
    });
    ```

## Displaying Progress with Custom Adapter

To display the last row as a ProgressBar indicating that the ListView is loading data, we do the trick in the Adapter. Having defined two types of views in `getItemViewType(int position)`, we can display the last row differently from a normal data row. It can be a ProgressBar or some text to indicate that the ListView has reached the last row by comparing the size of data List to the number of items on the server side. See [this gist](https://gist.github.com/nesquena/a988aac278cff59a9a69) for sample code.

## References

* <http://benjii.me/2010/08/endless-scrolling-listview-in-android/>
* <http://stackoverflow.com/questions/1080811/android-endless-list>
* <http://stackoverflow.com/questions/12583419/android-listview-automatically-load-more-data-when-scroll-to-the-bottom>
