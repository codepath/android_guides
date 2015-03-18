## Overview

RecyclerView is considered [the successor to ListView](https://www.youtube.com/watch?v=3TtVsy98ces&t=232). One of the reasons is that RecyclerView has a more extensible framework, especially since it provides the ability to implement both horizontal and vertical layouts.  

<img src="https://developer.android.com/training/material/images/RecyclerView.png" width="500" alt="RecyclerView" />

Furthermore, it provides animation support for ListView items whenever they are added or removed, which had been extremely difficult to do in the current implementation.  RecyclerView also begins to enforce the [ViewHolder pattern](http://guides.codepath.com/android/Using-an-ArrayAdapter-with-ListView#improving-performance-with-the-viewholder-pattern) too, which was already a recommended practice but now deeply integrated with this new framework.

For more details, see [this detailed overview](http://www.grokkingandroid.com/first-glance-androids-recyclerview/).

### Compared to ListView

`RecyclerView` differs from its predecessor `ListView` primarily because of the following features:

 * **Required ViewHolder in Adapters** - `ListView` adapters do not require the use of the ViewHolder pattern to improve performance. In contrast, implementing an adapter for `RecyclerView` requires the use of the ViewHolder pattern.
 * **Customizable Item Layouts** - `ListView` can only layout items in a vertical linear arrangement and this cannot be customized. In contrast, the `RecyclerView` has a `RecyclerView.LayoutManager` that allows any item layouts including horizontal lists or staggered grids. 
 * **Easy Item Animations** - `ListView` contains no special provisions through which one can animate the addition or deletion of items. In contrast, the `RecyclerView` has the `RecyclerView.ItemAnimator` class for handling item animations. 
 * **Manual Data Source** - `ListView` had adapters for different sources such as `ArrayAdapter` and `CursorAdapter` for arrays and database results respectively. In contrast, the `RecyclerView.Adapter` requires a custom implementation to supply the data to the adapter.
 * **Manual Item Decoration** - `ListView` has the `android:divider` property for easy dividers between items in the list. In contrast, `RecyclerView` requires the use of a `RecyclerView.ItemDecoration` object to setup much more manual divider decorations.
 * **Manual Click Detection** - `ListView` has a `AdapterView.OnItemClickListener` interface for binding to the click events for individual items in the list. In contrast, `RecyclerView` only has support for ` RecyclerView.OnItemTouchListener` which manages individual touch events but has no built-in click handling.

For a more detailed comparison, check out [this excellent article](http://www.truiton.com/2015/03/android-recyclerview-vs-listview-comparison/).

### Layout Managers

A RecyclerView needs to have a layout manager and an adapter to be instantiated. A layout manager positions item views inside a RecyclerView and determines when to reuse item views that are no longer visible to the user. 

[RecyclerView](https://developer.android.com/reference/android/support/v7/widget/RecyclerView.html) provides these built-in layout managers:

 * `LinearLayoutManager` shows items in a vertical or horizontal scrolling list.
 * `GridLayoutManager` shows items in a grid.
 * `StaggeredGridLayoutManager` shows items in a staggered grid.

To create a custom layout manager, extend the [RecyclerView.LayoutManager](https://developer.android.com/reference/android/support/v7/widget/RecyclerView.LayoutManager.html) class.

## Usage

### Installation

Add the following to your `app/build.gradle`:

```gradle
dependencies {
    ...
    compile 'com.android.support:recyclerview-v7:21.0.+'
}
```

Now we are ready to implement the `RecyclerView` starting with XML layout and then the adapter.

### Constructing the RecyclerView

First, let's add a `RecyclerView` to the layout:

```xml
<android.support.v7.widget.RecyclerView
        android:id="@+id/recyclerView"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        tools:context=".MainActivity"
        tools:listitem="@layout/item_demo_01" />
```

### Creating the RecyclerView Adapter

Next, let's define the adapter for our `RecyclerView` which will bind to the data source and populate the `RecyclerView` with item rows.

The interface for the adapter now aligns more closely with the `ViewHolder` pattern, which has a method to inflate new views and another to simply populate the information from a previously reused item.

```java
public class SimpleRecyclerAdapter extends RecyclerView.Adapter<SimpleItemViewHolder> {
		private List<String> items;

		// Provide a reference to the views for each data item
		// Provide access to all the views for a data item in a view holder
		public final static class SimpleItemViewHolder extends RecyclerView.ViewHolder {
			TextView label;

			public SimpleItemViewHolder(View itemView) {
				super(itemView);
				label = (TextView) itemView.findViewById(android.R.id.text1);
			}
		}
		
		// Provide a suitable constructor (depends on the kind of dataset)
		public SimpleRecyclerAdapter(List<String> items) {
			this.items = items;
		}
              
		// Return the size of your dataset (invoked by the layout manager)
		@Override
		public int getItemCount() {
			return this.items.size();
		}
               
                // Create new items (invoked by the layout manager)
                // Usually involves inflating a layout from XML and returning the holder
		@Override
		public SimpleItemViewHolder onCreateViewHolder(ViewGroup viewGroup, int viewType) {
	            View itemView = LayoutInflater.from(viewGroup.getContext()).
	                inflate(android.R.layout.simple_list_item_1, viewGroup, false);
	            return new SimpleItemViewHolder(itemView);
		}
		
		// Replace the contents of a view (invoked by the layout manager)
                // Involves populating data into the item through holder
		@Override
		public void onBindViewHolder(SimpleItemViewHolder viewHolder, int position) {
			String item = items.get(position);
			viewHolder.label.setText(item);
		}
}
```

### Constructing the RecyclerView

In the Java source file:

```java
public class MainActivity extends Activity {
	private RecyclerView recyclerView;

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		recyclerView = (RecyclerView) findViewById(R.id.recyclerView);
		// Setup layout manager for items
		LinearLayoutManager layoutManager = new LinearLayoutManager(this);
                // Control orientation of the items
		layoutManager.setOrientation(LinearLayoutManager.VERTICAL);
		layoutManager.scrollToPosition(0);
                // Attach layout manager
		recyclerView.setLayoutManager(layoutManager);
		// Bind adapter to recycler
		ArrayList<String> items = new ArrayList<String>();
		SimpleRecyclerAdapter aItems = new SimpleRecyclerAdapter(items);
		recyclerView.setAdapter(aItems);
     }
}
```

Setup item decorations with [DividerItemDecoration](https://gist.githubusercontent.com/alexfu/0f464fc3742f134ccd1e/raw/abe729359e5b3691f2fe56445644baf0e40b35ba/DividerItemDecoration.java):

```java
RecyclerView.ItemDecoration itemDecoration =
    new DividerItemDecoration(this, DividerItemDecoration.VERTICAL_LIST);
recyclerView.addItemDecoration(itemDecoration);
```

### Item Animations

The RecyclerView now provides support for an ItemAnimator listener pattern, which provides methods such as `notifyItemInserted()` and `notifyItemRemoved()`:

```java
// this is the default; 
// this call is actually only necessary with custom ItemAnimators
recyclerView.setItemAnimator(new DefaultItemAnimator());
```

### Item Touch Detection

```java
recyclerView.addOnItemTouchListener(new OnItemTouchListener() {
			
	@Override
	public void onTouchEvent(RecyclerView recycler, MotionEvent event) {
				
	}
			
	@Override
	public boolean onInterceptTouchEvent(RecyclerView recycler, MotionEvent event) {
		return false;
	}
});
```

### Better Performance with Fixed Size

```java
// allows for optimizations if all item views are of the same size:
recyclerView.setHasFixedSize(true);
```

### Attaching Click Handlers to Items

See [this detailed stackoverflow post](http://stackoverflow.com/a/24933117) which describes how to setup item-level click handlers when using `RecyclerView`. Note that there is no `onItemClickListener` equivalent. 

## References

* <http://www.grokkingandroid.com/first-glance-androids-recyclerview/>
* <http://www.grokkingandroid.com/statelistdrawables-for-recyclerview-selection/>
* <http://www.bignerdranch.com/blog/recyclerview-part-1-fundamentals-for-listview-experts/>
* <https://developer.android.com/training/material/lists-cards.html>