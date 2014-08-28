## Overview

A common application feature is to have an `AdapterView` (such as a `ListView` or `GridView`) that automatically loads more items as the user scrolls through the items (aka infinite scroll). This is done by triggering a request for more data once the user crosses a threshold of remaining items before they've hit the end. 

Every `AdapterView` has support for binding to the `OnScrollListener` events which are triggered whenever a user scrolls through the collection. Using this system, we can define a basic `EndlessScrollListener` which supports most use cases by creating our own class that extends `OnScrollListener`:

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
	public void onScroll(AbsListView view,int firstVisibleItem,int visibleItemCount,int totalItemCount) 
        {
		// If the total item count is zero and the previous isn't, assume the
		// list is invalidated and should be reset back to initial state
		if (totalItemCount < previousTotalItemCount) {
			this.currentPage = this.startingPageIndex;
			this.previousTotalItemCount = totalItemCount;
			if (totalItemCount == 0) { this.loading = true; } 
		}

		// If it’s still loading, we check to see if the dataset count has
		// changed, if so we conclude it has finished loading and update the current page
		// number and total item count.
		if (loading && (totalItemCount > previousTotalItemCount)) {
			loading = false;
			previousTotalItemCount = totalItemCount;
			currentPage++;
		}
		
		// If it isn’t currently loading, we check to see if we have breached
		// the visibleThreshold and need to reload more data.
		// If we do need to reload some more data, we execute onLoadMore to fetch the data.
		if (!loading && (totalItemCount - visibleItemCount)<=(firstVisibleItem + visibleThreshold)) {
		    onLoadMore(currentPage + 1, totalItemCount);
		    loading = true;
		}
	}

	// Defines the process for actually loading more data based on page
	public abstract void onLoadMore(int page, int totalItemsCount);

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
	    public void onLoadMore(int page, int totalItemsCount) {
                // Triggered only when new data needs to be appended to the list
                // Add whatever code is needed to append new items to your AdapterView
	        customLoadMoreDataFromApi(page); 
                // or customLoadMoreDataFromApi(totalItemsCount); 
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

## Troubleshooting

If you are running into problems, please carefully consider the following suggestions:

* Make sure to setup the `setOnScrollListener` listener in the `onCreate` method of the `Activity` and not later on otherwise you may encounter unexpected issues. 

* In order for the pagination system to continue working reliably, you should make sure to **clear the adapter** of items (or notify adapter after clearing the array) before appending new items to the list.

* In order for this pagination system to trigger, keep in mind that as `customLoadMoreDataFromApi` is called, new data needs to be **appended to the existing data source**. In other words, only clear items from the list when on the initial "page". Subsequent "pages" of data should be appended to the existing data.

## Displaying Progress with Custom Adapter

To display the last row as a ProgressBar indicating that the ListView is loading data, we do the trick in Adapter. Having defined two types of views in `getItemViewType(int position)`, we can display the last row differently from a normal data row. It can be a ProgressBar or some text to indicate that the ListView has reached the last row by comparing the size of data List to the number of items on the server side.

```java
/**
 *  A child class shall subclass this Adapter and 
 *  implement method getDataRow(int position, View convertView, ViewGroup parent),
 *  which supplies a View present data in a ListRow.
 *  
 *  This parent Adapter takes care of displaying ProgressBar in a row or 
 *  indicating that it has reached the last row.
 * 
 */
public abstract class GenericAdapter<T> extends BaseAdapter {
	
	// the main data list to save loaded data
	protected List<T> dataList;
	
	protected Activity mActivity;
	
	// the serverListSize is the total number of items on the server side,
	// which should be returned from the web request results
	protected int serverListSize = -1;
	
	// Two view types which will be used to determine whether a row should be displaying 
	// data or a Progressbar
	public static final int VIEW_TYPE_LOADING = 0;
	public static final int VIEW_TYPE_ACTIVITY = 1;
	
	
	public GenericAdapter(Activity activity, List<T> list) {
		mActivity = activity;
		dataList = list;
	}
	
	
	public void setServerListSize(int serverListSize){
		this.serverListSize = serverListSize;
	}


/**
	 * disable click events on indicating rows
	 */
	@Override
	public boolean isEnabled(int position) {
		 
		return getItemViewType(position) == VIEW_TYPE_ACTIVITY;
	}

	/**
	 * One type is normal data row, the other type is Progressbar
	 */
	@Override
	public int getViewTypeCount() {
		return 2;
	}
	
	
	/**
	 * the size of the List plus one, the one is the last row, which displays a Progressbar
	 */
	@Override
	public int getCount() {
		return dataList.size() + 1;
	}

	
	/**
	 * return the type of the row, 
	 * the last row indicates the user that the ListView is loading more data
	 */
	@Override
	public int getItemViewType(int position) {
		// TODO Auto-generated method stub
		return (position >= dataList.size()) ? VIEW_TYPE_LOADING
				: VIEW_TYPE_ACTIVITY;
	}

	@Override
	public T getItem(int position) {
		// TODO Auto-generated method stub
		return (getItemViewType(position) == VIEW_TYPE_ACTIVITY) ? dataList
				.get(position) : null;
	}

	@Override
	public long getItemId(int position) {
		// TODO Auto-generated method stub
		return (getItemViewType(position) == VIEW_TYPE_ACTIVITY) ? position
				: -1;
	}

	/**
	 *  returns the correct view 
	 */
	@Override
	public  View getView(int position, View convertView, ViewGroup parent){
		if (getItemViewType(position) == VIEW_TYPE_LOADING) {
			// display the last row
			return getFooterView(position, convertView, parent);
		}
		View dataRow = convertView;
		dataRow = getDataRow(position, convertView, parent);
		
		return dataRow;
	};
	
	/**
	 * A subclass should override this method to supply the data row.
	 * @param position
	 * @param convertView
	 * @param parent
	 * @return
	 */
	public abstract View getDataRow(int position, View convertView, ViewGroup parent);

	
	/**
	 * returns a View to be displayed in the last row.
	 * @param position
	 * @param convertView
	 * @param parent
	 * @return
	 */
	public View getFooterView(int position, View convertView,
			ViewGroup parent) {
		if (position >= serverListSize && serverListSize > 0) {
			// the ListView has reached the last row
			TextView tvLastRow = new TextView(mActivity);
			tvLastRow.setHint("Reached the last row.");
			tvLastRow.setGravity(Gravity.CENTER);
			return tvLastRow;
		}
		
		View row = convertView;
		if (row == null) {
			row = mActivity.getLayoutInflater().inflate(
					R.layout.progress, parent, false);
		}

		return row;
	}

}
```

## References

* <http://benjii.me/2010/08/endless-scrolling-listview-in-android/>
* <http://stackoverflow.com/questions/1080811/android-endless-list>
* <http://stackoverflow.com/questions/12583419/android-listview-automatically-load-more-data-when-scroll-to-the-bottom>