## Overview

RecyclerView is considered [the successor to ListView](https://www.youtube.com/watch?v=3TtVsy98ces&t=232).  One of the reasons is that RecyclerView has a more extensible framework, especially since it provides the ability to implement both horizontal and vertical layouts.  

Furthermore, it provides animation support for ListView items whenever they are added or removed, which had been extremely difficult to do in the current implementation.  RecyclerView also begins to enforce the [ViewHolder pattern](http://guides.codepath.com/android/Using-an-ArrayAdapter-with-ListView#improving-performance-with-the-viewholder-pattern) too, which was already a recommended practice but now deeply integrated with this new framework.

For more details, see [this detailed overview](http://www.grokkingandroid.com/first-glance-androids-recyclerview/).

### Layout Managers

A RecyclerView needs to have a layout manager and an adapter to be instantiated.  There are currently several built-in layout managers including `LinearLayoutManager`, `GridLayoutManager`, and `StaggeredGridLayoutManager`.  

### Adapters 

The interface for the adapter now aligns more closely with the `ViewHolder` pattern, which has a method to inflate new views and another to simply populate the information from a previously reused item.

### Animations

The RecyclerView now provides support for an ItemAnimator listener pattern, which provides methods such as `notifyItemInserted()` and `notifyItemRemoved()`.

## Usage

### Installation

Add the following to your `app/build.gradle`:

```gradle
dependencies {
    ...
    compile 'com.android.support:recyclerview-v7:21.+'
}
```

Now we are ready to implement the `RecyclerView` starting with the adapter.

### Creating the RecyclerView Adapter

```java
public class SimpleRecyclerAdapter extends RecyclerView.Adapter<SimpleItemViewHolder> {
		private List<String> items;
		
		public SimpleRecyclerAdapter(List<String> items) {
			this.items = items;
		}

		@Override
		public int getItemCount() {
			return this.items.size();
		}

		@Override
		public SimpleItemViewHolder onCreateViewHolder(ViewGroup viewGroup, int viewType) {
	            View itemView = LayoutInflater.from(viewGroup.getContext()).
	                inflate(android.R.layout.simple_list_item_1, viewGroup, false);
	            return new SimpleItemViewHolder(itemView);
		}
		

		@Override
		public void onBindViewHolder(SimpleItemViewHolder viewHolder, int position) {
			String item = items.get(position);
			viewHolder.label.setText(item);
		}

		public final static class SimpleItemViewHolder extends RecyclerView.ViewHolder {
			TextView label;

			public SimpleItemViewHolder(View itemView) {
				super(itemView);
				label = (TextView) itemView.findViewById(android.R.id.text1);
			}
		}
}
```

### Constructing the RecyclerView

In layout XML file:

```xml
<android.support.v7.widget.RecyclerView
        android:id="@+id/recyclerView"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        tools:context=".MainActivity"
        tools:listitem="@layout/item_demo_01" />
```

In the Java source file:

```java
public class MainActivity extends Activity {
	private RecyclerView recyclerView;

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		recyclerView = (RecyclerView) findViewById(R.id.recyclerView);
		// Setup item layout
		LinearLayoutManager layoutManager = new LinearLayoutManager(this);
		layoutManager.setOrientation(LinearLayoutManager.VERTICAL);
		layoutManager.scrollToPosition(0);
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

Setup animator:

```java
// this is the default; 
// this call is actually only necessary with custom ItemAnimators
recyclerView.setItemAnimator(new DefaultItemAnimator());
```

Touch detection:

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

Faster with Fixed Size:

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