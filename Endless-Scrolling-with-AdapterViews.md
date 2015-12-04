## Overview

A common application feature is to load automatically more items as the user scrolls through the items (aka infinite scroll). This is done by triggering a request for more data once the user crosses a threshold of remaining items before they've hit the end. 

The approaches for ListView and [[RecyclerView|Using-the-RecyclerView]] (the successor to ListView) are documented here.  Both are similar in code except that the [LayoutManager](http://developer.android.com/reference/android/support/v7/widget/RecyclerView.LayoutManager.html) in the RecyclerView needs to be passed in to provide the necessary information to implement infinite scrolling.

### Implementing with ListView 

Every `AdapterView` has support for binding to the `OnScrollListener` events which are triggered whenever a user scrolls through the collection. Using this system, we can define a basic `EndlessScrollListener` which supports most use cases by creating our own class that extends `OnScrollListener`:

```java
import android.widget.AbsListView; 

public abstract class EndlessScrollListener implements AbsListView.OnScrollListener {
	// The minimum amount of items to have below your current scroll position
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

	// This happens many times a second during a scroll, so be wary of the code you place here.
	// We are given a few useful parameters to help us work out if we need to load some more data,
	// but first we check if we are waiting for the previous load to finish.
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
		if (!loading && (totalItemCount - visibleItemCount) <= (firstVisibleItem + visibleThreshold)) {
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
              customLoadMoreDataFromApi(page); 
              // or customLoadMoreDataFromApi(totalItemsCount); 
              return true; // ONLY if more data is actually being loaded; false otherwise.
          }
        });
    }
    
    // Append more data into the adapter
    public void customLoadMoreDataFromApi(int offset) {
      // This method probably sends out a network request and appends new data items to your adapter. 
      // Use the offset value and add it as a parameter to your API request to retrieve paginated data.
      // Deserialize API response and then construct new objects to append to the adapter
    }
}
```

Now as you scroll, items will be automatically filling in because the `onLoadMore` method will be triggered once the user crosses the `visibleThreshold`. This approach works equally well for a `GridView` and the listener gives access to both the `page` as well as the `totalItemsCount` to support both pagination and offset based fetching.

### Implementing with RecyclerView

We can also use a similar approach with the RecyclerView by defining an interface `EndlessRecyclerViewScrollListener` that requires an `onLoadMore()` method to be implemented.  The [LayoutManager](http://developer.android.com/reference/android/support/v7/widget/RecyclerView.LayoutManager.html), which is responsible in the RecyclerView for rendering where items should be positioned and manages scrolling, provides information about the current scroll position relative to the adapter.  For this reason, we need to pass an instance of what LayoutManager is being used to collect the necessary information to ascertain when to load more data:

```java
import android.support.v7.widget.LinearLayoutManager;
import android.support.v7.widget.RecyclerView;

public abstract class EndlessRecyclerViewScrollListener extends RecyclerView.OnScrollListener {
    // The minimum amount of items to have below your current scroll position
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

    private LinearLayoutManager mLinearLayoutManager;

    public EndlessRecyclerViewScrollListener(LinearLayoutManager layoutManager) {
        this.mLinearLayoutManager = layoutManager;
    }

    // This happens many times a second during a scroll, so be wary of the code you place here.
    // We are given a few useful parameters to help us work out if we need to load some more data,
    // but first we check if we are waiting for the previous load to finish.
    @Override
    public void onScrolled(RecyclerView view, int dx, int dy) {
        int firstVisibleItem = mLinearLayoutManager.findFirstVisibleItemPosition();
        int visibleItemCount = view.getChildCount();
        int totalItemCount = mLinearLayoutManager.getItemCount();

        // If the total item count is zero and the previous isn't, assume the
        // list is invalidated and should be reset back to initial state
        if (totalItemCount < previousTotalItemCount) {
            this.currentPage = this.startingPageIndex;
            this.previousTotalItemCount = totalItemCount;
            if (totalItemCount == 0) {
                this.loading = true;
            }
        }
        // If it’s still loading, we check to see if the dataset count has
        // changed, if so we conclude it has finished loading and update the current page
        // number and total item count.
        if (loading && (totalItemCount > previousTotalItemCount)) {
            loading = false;
            previousTotalItemCount = totalItemCount;
        }

        // If it isn’t currently loading, we check to see if we have breached
        // the visibleThreshold and need to reload more data.
        // If we do need to reload some more data, we execute onLoadMore to fetch the data.
        if (!loading && (totalItemCount - visibleItemCount) <= (firstVisibleItem + visibleThreshold)) {
            currentPage++;
            onLoadMore(currentPage, totalItemCount);
            loading = true;
        }
    }

    // Defines the process for actually loading more data based on page
    public abstract void onLoadMore(int page, int totalItemsCount);

}
```

Assuming we are using a `LinearLayoutManager`, we simply need to use the `addOnScrollListener()` method and pass in an instance of the `EndlessRecyclerViewScrollListener` with the layout manager:

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
   RecyclerView rvItems = (RecyclerView) findViewById(R.id.rvContacts);

    // fetch data here
    final List<Item> items = Contact.createContactsList(10);
    final ContactsAdapter adapter = new ContactsAdapter(items);

    // set adapter
    rvItems.setAdapter(adapter);
    final LinearLayoutManager linearLayoutManager = new LinearLayoutManager(this);
    rvItems.setLayoutManager(linearLayoutManager);

    // add scroll listener
    rvItems.addOnScrollListener(new EndlessRecyclerViewScrollListener(linearLayoutManager) {
       @Override
       public void onLoadMore(int page, int totalItemsCount) {
          // fetch data here

          // update the adapter, saving the last known size
          int curSize = adapter.getItemCount(); 
          items.addAll(moreContacts);
    
          // for efficiency purposes, only notify the adapter of what elements that got changed
          // curSize will equal to the index of the first element inserted because the list is 0-indexed
          adapter.notifyItemRangeInserted(curSize, items.size() - 1);
     }
  });
}
```

## Troubleshooting

If you are running into problems, please carefully consider the following suggestions:

* Make sure to setup the `setOnScrollListener` listener in the `onCreate` method of the `Activity` or `onCreateView` in a Fragment and not much later otherwise you may encounter unexpected issues. 

* In order for the pagination system to continue working reliably, you should make sure to **clear the adapter** of items (or notify adapter after clearing the array) before appending new items to the list.

* In order for this pagination system to trigger, keep in mind that as `customLoadMoreDataFromApi` is called, new data needs to be **appended to the existing data source**. In other words, only clear items from the list when on the initial "page". Subsequent "pages" of data should be appended to the existing data.

## Displaying Progress with Custom Adapter

To display the last row as a ProgressBar indicating that the ListView is loading data, we do the trick in the Adapter. Having defined two types of views in `getItemViewType(int position)`, we can display the last row differently from a normal data row. It can be a ProgressBar or some text to indicate that the ListView has reached the last row by comparing the size of data List to the number of items on the server side. See [this gist](https://gist.github.com/nesquena/a988aac278cff59a9a69) for sample code.

## References

* <http://benjii.me/2010/08/endless-scrolling-listview-in-android/>
* <http://stackoverflow.com/questions/1080811/android-endless-list>
* <http://stackoverflow.com/questions/12583419/android-listview-automatically-load-more-data-when-scroll-to-the-bottom>