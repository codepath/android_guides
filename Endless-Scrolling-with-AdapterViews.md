## Overview

A common application feature is to have an AdapterView (such as a ListView or GridView) that automatically loads more items as the user scrolls through the items (aka infinite scroll). This is done by triggering a request for more data once the user crosses a threshold of remaining items before they've hit the end. 

Every AdapterView has support for binding to the `OnScrollListener` events which are triggered whenever a user scrolls through the collection. Using this system, we can define a basic `EndlessScrollListener` which supports most use cases by creating our own class that extends `OnScrollListener`:

```java
public abstract class EndlessScrollListener implements OnScrollListener {
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
	public void onScroll(AbsListView view, int firstVisibleItem, int visibleItemCount, int totalItemCount) {
		// If the total item count is zero and the previous isn't, assume the
		// list is invalidated and should be reset back to initial state
		if (!loading && (totalItemCount < previousTotalItemCount)) {
			this.currentPage = this.startingPageIndex;
			this.previousTotalItemCount = totalItemCount;
			this.loading = true;
		}

		// If it’s still loading, we check to see if the dataset count has
		// changed, if so we conclude it has finished loading and update the current page
		// number and total item count.
		if (loading) {
			if (totalItemCount > previousTotalItemCount) {
				loading = false;
				previousTotalItemCount = totalItemCount;
				currentPage++;
			}
		}
		// If it isn’t currently loading, we check to see if we have breached
		// the visibleThreshold and need to reload more data.
		// If we do need to reload some more data, we execute onLoadMore to fetch the data.
		if (!loading && (totalItemCount - visibleItemCount) <= (firstVisibleItem + visibleThreshold)) {
			onLoadMore(currentPage + 1, totalItemCount);
			loading = true;
		}
	}

	// Defines the process for actually loading more data based on page
	public abstract void onLoadMore(int page, int totalItemsCount);

	@Override
	public void onScrollStateChanged(AbsListView view, int scrollState) {
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
        lvItems.setOnScrollListener(new EndlessScrollListener() {
	    @Override
	    public void onLoadMore(int page, int totalItemsCount) {
                // Whatever code is needed to append new items to your AdapterView
                // probably sending out a network request and appending items to your adapter. 
                // Use the page or the totalItemsCount to retrieve correct data.
	        customLoadMoreDataFromApi(page); 
                // or customLoadMoreDataFromApi(totalItemsCount); 
	    }
        });
    }
}
```

Now as you scroll, items will be automatically filling in because the `onLoadMore` method will be triggered once the user crosses the `visibleThreshold`. This approach works equally well for a `GridView` and the listener gives access to both the `page` as well as the `totalItemsCount` to support both pagination and offset based fetching.

## References

* <http://benjii.me/2010/08/endless-scrolling-listview-in-android/>
* <http://stackoverflow.com/questions/1080811/android-endless-list>
* <http://stackoverflow.com/questions/12583419/android-listview-automatically-load-more-data-when-scroll-to-the-bottom>