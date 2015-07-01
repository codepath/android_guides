## Overview

The `RecyclerView` is a new `ViewGroup` that is prepared to render any adapter-based view in a similar way. It is supposed to be the successor of [ListView] (https://github.com/codepath/android_guides/wiki/Using-an-ArrayAdapter-with-ListView) and [GridView] (http://developer.android.com/guide/topics/ui/layout/gridview.html), and it can be found in the [latest support-v7 version] (https://developer.android.com/reference/android/support/v7/app/package-summary.html). One of the reasons is that `RecyclerView` has a more extensible framework, especially since it provides the ability to implement both horizontal and vertical layouts. Use the `RecyclerView` widget when you have data collections whose elements change at runtime based on user action or network events.

If you want to use a `RecyclerView`, you will need to work with the following:
* `RecyclerView.Adapter` - To handle the data collection and bind it to the view
* `LayoutManager` - Helps in positioning the items
* `ItemAnimator` - Helps with animating the items for common operations such as Addition or Removal of item

<img src="https://developer.android.com/training/material/images/RecyclerView.png" width="500" alt="RecyclerView" />

Furthermore, it provides animation support for `ListView` items whenever they are added or removed, which had been extremely difficult to do in the current implementation.  `RecyclerView` also begins to enforce the [ViewHolder pattern](http://guides.codepath.com/android/Using-an-ArrayAdapter-with-ListView#improving-performance-with-the-viewholder-pattern) too, which was already a recommended practice but now deeply integrated with this new framework.

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

## Components of a `RecyclerView`

### `LayoutManagers`

A `RecyclerView` needs to have a layout manager and an adapter to be instantiated. A layout manager positions item views inside a `RecyclerView` and determines when to reuse item views that are no longer visible to the user. 

[RecyclerView](https://developer.android.com/reference/android/support/v7/widget/RecyclerView.html) provides these built-in layout managers:

 * `LinearLayoutManager` shows items in a vertical or horizontal scrolling list.
 * `GridLayoutManager` shows items in a grid.
 * `StaggeredGridLayoutManager` shows items in a staggered grid.

To create a custom layout manager, extend the [RecyclerView.LayoutManager](https://developer.android.com/reference/android/support/v7/widget/RecyclerView.LayoutManager.html) class.

### `RecyclerView.Adapter`

`RecyclerView` includes a new kind of adapter. It’s a similar approach to the ones you already used, but with some peculiarities, such as a required `ViewHolder`. You will have to override two main methods: one to inflate the view and its view holder, and another one to bind data to the view. The good thing about this is that first method is called only when we really need to create a new view. No need to check if it’s being recycled.

### `ItemAnimator`

`RecyclerView.ItemAnimator` will animate `ViewGroup` modifications such as add/delete/select that are notified to adapter. `DefaultItemAnimator` can be used for basic default animations and works quite well.

# Using the `RecyclerView`

### Installation

Add the following dependencies to your `app/build.gradle`:

```gradle
dependencies {
    ...
    compile 'com.android.support:recyclerview-v7:22.0.0'
}
```

Click on "Sync Project with Gradle files" to let your IDE download the appropriate resources.

### Constructing the `RecyclerViewActivity`: 

Create a new activity and call it `RecyclerViewActivity`. 

Here's how your activity initially looks like with the `RecyclerView` object instantiated: 

```java
public class RecyclerViewActivity extends ActionBarActivity {

    private RecyclerView recyclerView;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_recycler_view);

        //Get Handle to your UI elements
        initUi();
    }
```

Before proceeding, let's add a `RecyclerView` widget to our layout file (`activity_recycler_view`):

```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent" >

    <android.support.v7.widget.RecyclerView
        android:id="@+id/recyclerView"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />

</RelativeLayout>
```

Next, get the handle to the `RecyclerView` in the activity in the `initUi` method:

```java
     private void initUi(View view) {
        //Get the handle to the recycler view
        recyclerView = (RecyclerView) view.findViewById(R.id.recyclerView);
        configureRecyclerView();
        //Initiate your other elements here
        // ...
    }
```

### Configuring the `RecyclerView` (read comments for what the lines do):

```java
      private void configureRecyclerView() {
        
        // Setup layout manager for items
        LinearLayoutManager layoutManager = new LinearLayoutManager(this);
        
        // Control orientation of the items
        layoutManager.setOrientation(LinearLayoutManager.VERTICAL);

        //Customize the position you want to default scroll to
        layoutManager.scrollToPosition(0);

        // Attach layout manager to the RecyclerView
        recyclerView.setLayoutManager(layoutManager);

        // allows for optimizations if all item views are of the same size:
        recyclerView.setHasFixedSize(true);

        // Reference : https://gist.githubusercontent.com/alexfu/0f464fc3742f134ccd1e/raw/abe729359e5b3691f2fe56445644baf0e40b35ba/DividerItemDecoration.java
        RecyclerView.ItemDecoration itemDecoration = new DividerItemDecoration(this, DividerItemDecoration.VERTICAL_LIST);
        recyclerView.addItemDecoration(itemDecoration);
        
        bindDataToAdapter();
    }
```

Optionally, you can also add the following properties to define animations and item touches:

```java

        // RecyclerView uses this by default. You can add custom animations by using RecyclerView.ItemAnimator()
        recyclerView.setItemAnimator(new DefaultItemAnimator());

        //Handle item touch events
        recyclerView.addOnItemTouchListener(new RecyclerView.OnItemTouchListener() {

            @Override
            public void onTouchEvent(RecyclerView recycler, MotionEvent event) {
                //Handle on touch events
            }

            @Override
            public boolean onInterceptTouchEvent(RecyclerView recycler, MotionEvent event) {
                return false;
            }
        });

```

### Creating the `RecyclerView.Adapter`

Next, let's define the adapter for our `RecyclerView` which will bind to the data source and populate the `RecyclerView` with item rows.

The interface for the adapter now aligns more closely with the `ViewHolder` pattern, which has a method to inflate new views and another to simply populate the information from a previously reused item.

```java
public class SimpleItemRecyclerViewAdapter extends RecyclerView.Adapter<RecyclerViewSimpleTextViewHolder> {

    private Context context;
    // The items to display in your RecyclerView
    private List<String> items;
    // Allows to remember the last item shown on screen
    private int lastPosition = -1;

    // Provide a suitable constructor (depends on the kind of dataset)
    public SimpleItemRecyclerViewAdapter(List<String> items, Context context) {
        this.items = items;
        this.context = context;
    }

    // Return the size of your dataset (invoked by the layout manager)
    @Override
    public int getItemCount() {
        return this.items.size();
    }

    // Create new items (invoked by the layout manager)
    // Usually involves inflating a layout from XML and returning the holder
    @Override
    public RecyclerViewSimpleTextViewHolder onCreateViewHolder(ViewGroup viewGroup, int viewType) {
        View itemView = LayoutInflater.from(viewGroup.getContext()).
                inflate(android.R.layout.simple_list_item_1, viewGroup, false);
        return new RecyclerViewSimpleTextViewHolder(itemView);
    }

    // Replace the contents of a view (invoked by the layout manager)
    // Involves populating data into the item through holder
    @Override
    public void onBindViewHolder(RecyclerViewSimpleTextViewHolder viewHolder, int position) {
        String item = items.get(position);
        viewHolder.getLabel().setText(item);
        // Here you apply the animation when the view is bound
        setAnimation(viewHolder.itemView, position);
    }

    // Apply some custom animations to loading the items in the list
    private void setAnimation(View viewToAnimate, int position)
    {
        // If the bound view wasn't previously displayed on screen, it's animated
        if (position > lastPosition)
        {
            Animation animation = AnimationUtils.loadAnimation(context, android.R.anim.slide_in_left);
            viewToAnimate.startAnimation(animation);
            lastPosition = position;
        }
    }
}

```

### Defining the `RecyclerView.ViewHolder`:

```java
public class RecyclerViewSimpleTextViewHolder extends RecyclerView.ViewHolder {
    private TextView label;

    public RecyclerViewSimpleTextViewHolder(View v) {
        super(v);
        label = (TextView) v.findViewById(android.R.id.text1);
    }

    public TextView getLabel() {
        return label;
    }

    public void setLabel(TextView label) {
        this.label = label;
    }
}
```

### Binding the data to the adapter

In the `RecyclerViewFragment`, we will create a list which generates random strings and pass that to the adapter. We then set the adapter to the `RecyclerView` object:

```java
     private void bindDataToAdapter() {
        // Bind adapter to recycler view object
        recyclerView.setAdapter(new SimpleItemRecyclerViewAdapter(getSampleArrayList(), getActivity()));
     }

    private ArrayList<String> getSampleArrayList() {
        SecureRandom random = new SecureRandom();
        ArrayList<String> items = new ArrayList<>();

        for (int i = 0; i < 50; i++) {
            items.add(getRandomString(i, random));
        }

        return items;
    }

    private String getRandomString(int i, SecureRandom random) {
        return (i+1) + ". " + new BigInteger(130, random).toString(32).toUpperCase();
    }
```

That's it, we're good to go. Now, compile and run the app and you should see something like the screenshot below. Scroll down the list to see the custom animations while loading the item. If you scroll back up, the views will be recycled and far smoother than the `ListView` widget.

<img src="http://i105.photobucket.com/albums/m232/purplehaze0077/Screen%20Shot%202015-06-26%20at%207.25.14%20AM.png" width="500" alt="Screenshot" />



### Attaching Click Handlers to Items

See [this detailed stackoverflow post](http://stackoverflow.com/a/24933117) which describes how to setup item-level click handlers when using `RecyclerView`. Note that there is no `onItemClickListener` equivalent. 

In certain cases, you'd want to setup click handlers for views within the `RecyclerView` but define the click logic within the containing `Activity` or `Fragment` (i.e bubble up events from the adapter). To achieve this, [[create a custom listener|Creating-Custom-Listeners]] within the adapter and then fire the events upwards to an interface implementation defined within the parent.

### Heterogenous Views

See [this guide](https://github.com/codepath/android_guides/wiki/Heterogenous-Layouts-inside-RecyclerView) if you want to inflate multiple layouts inside a single `RecyclerView`.

## References

* <https://developer.android.com/reference/android/support/v7/widget/RecyclerView.html>
* <http://www.grokkingandroid.com/first-glance-androids-recyclerview/>
* <http://www.grokkingandroid.com/statelistdrawables-for-recyclerview-selection/>
* <http://www.bignerdranch.com/blog/recyclerview-part-1-fundamentals-for-listview-experts/>
* <https://developer.android.com/training/material/lists-cards.html>
* <http://antonioleiva.com/recyclerview/>