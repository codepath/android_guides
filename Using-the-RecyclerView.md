## Overview

**Needs Attention**

See [this detailed overview](http://www.grokkingandroid.com/first-glance-androids-recyclerview/).

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

```xml
<android.support.v7.widget.RecyclerView
        android:id="@+id/recyclerView"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        tools:context=".MainActivity"
        tools:listitem="@layout/item_demo_01" />
```

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

Setup item decorations with [DivierItemDecoration](https://gist.githubusercontent.com/alexfu/0f464fc3742f134ccd1e/raw/abe729359e5b3691f2fe56445644baf0e40b35ba/DividerItemDecoration.java):

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

## References

* <http://www.grokkingandroid.com/first-glance-androids-recyclerview/>
* <http://www.grokkingandroid.com/statelistdrawables-for-recyclerview-selection/>