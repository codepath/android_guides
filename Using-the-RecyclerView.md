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

## Using the RecyclerView

### Installation

Make sure the recyclerview support library is listed as a dependency in your `app/build.gradle`:

```gradle
dependencies {
    ...
    compile 'com.android.support:recyclerview-v7:22.2.+'
}
```

Click on "Sync Project with Gradle files" to let your IDE download the appropriate resources.

### Defining a Model

Every RecyclerView is backed by a source for data. In this case, we will define a `User` class which represents the data model being represented by the RecyclerView:

```java
public class User {
    // Define attributes of a user
    public String name;
    public String hometown;
    // Create a constructor 
    public User(String name, String hometown) {
       this.name = name;
       this.hometown = hometown;
    }
}
```

### Create the RecyclerView within Layout

Inside the desired activity layout XML file in `res/layout/activity_users.xml`, let's add the `RecyclerView` from the support library:

```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent" >
    
    <android.support.v7.widget.RecyclerView
      android:id="@+id/rvUsers"
      android:layout_width="match_parent"
      android:layout_height="match_parent" />

</RelativeLayout>
```

In the layout, preview we can see the `RecyclerView` within the activity:

<img src="http://i.imgur.com/Qf5fQ8X.png" width="300" />

Now the `RecyclerView` is embedded within our activity layout file. Next, we can define the layout for each item within our list.

### Creating the Custom Row Layout

Before we create the adapter, let's define the XML layout file that will be used for each row within the list. This item layout for now should contain a horizontal linear layout with a textview for the name and hometown as shown below:

<img src="http://i.imgur.com/MmY8zqI.png" width="300" />
<img src="http://i.imgur.com/fu3FzsV.png" width="300" />

This layout file can be created in `res/layout/item_user.xml` and will be rendered for each item row:

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="horizontal" android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="?android:attr/selectableItemBackground">
    
    <ImageView
        android:layout_width="50dp"
        android:layout_height="50dp"
        android:id="@+id/imageView"
        android:src="@mipmap/ic_launcher"
        android:layout_gravity="center_vertical" />    

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textAppearance="?android:attr/textAppearanceLarge"
        android:text="Dennis"
        android:id="@+id/tvName"
        android:layout_gravity="center_vertical" />

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textAppearance="?android:attr/textAppearanceMedium"
        android:text="Winterfell"
        android:id="@+id/tvHometown"
        android:layout_marginLeft="10dp"
        android:layout_gravity="center_vertical" />
</LinearLayout>
```

With the custom item layout complete, let's create the adapter to populate the data into the recycler view.

### Creating the `RecyclerView.Adapter`

Here we need to create the adapter which will actually populate the data into the RecyclerView. The adapter's role is to **convert an object at a position into a list row item** to be inserted. 

However with a `RecyclerView` the adapter requires the existence of a "ViewHolder" object which describes and provides access to all the views within each item row.  We can create the basic empty adapter and holder together in `UserRecyclerViewAdapter.java` as follows:

```java
// Create the basic adapter extending from RecyclerView.Adapter
// Note that we specify the custom ViewHolder which gives us access to our views
public class UserRecyclerViewAdapter extends 
    RecyclerView.Adapter<UserRecyclerViewAdapter.ViewHolder> {

    // Provide a direct reference to each of the views within a data item
    // Used to cache the views within the item layout for fast access
    public static class ViewHolder extends RecyclerView.ViewHolder {
        // Your holder should contain a member variable
        // for any view that will be set as you render a row
        public TextView tvName;
        public TextView tvHometown;
        // We also create a constructor that accepts the entire item row
        // and does the view lookups to find each subview
        public ViewHolder(View itemView) {
            super(itemView);
            this.tvName = (TextView) itemView.findViewById(R.id.tvName);
            this.tvHometown = (TextView) itemView.findViewById(R.id.tvHometown);
        }
    }

}
```

Now that we've defined the basic adapter and `ViewHolder`, we need to begin filling in our adapter. First, let's store a member variable for the list of users and pass the list in through our constructor:

```java
public class UserRecyclerViewAdapter extends 
    RecyclerView.Adapter<UserRecyclerViewAdapter.ViewHolder> {

    // ... view holder defined above...

    // Store a member variable for the users
    private ArrayList<User> users;

    // Pass in the users array into the constructor
    public UserRecyclerViewAdapter(ArrayList<User> users) {
        this.users = users;
    }
}
```

Every adapter has three primary methods: `onCreateViewHolder` to inflate the item layout and create the holder, `onBindViewHolder` to set the view attributes based on the data and `getItemCount` to determine the number of items. We need to implement all three to finish the adapter:

```java
public class UserRecyclerViewAdapter extends 
    RecyclerView.Adapter<UserRecyclerViewAdapter.ViewHolder> {

    // ... constructor and member variables

    // Usually involves inflating a layout from XML and returning the holder
    @Override
    public UserRecyclerViewAdapter.ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        // Inflate the custom layout
        View itemView = LayoutInflater.from(parent.getContext()).
            inflate(R.layout.item_user, parent, false);
        // Return a new holder instance
        return new UserRecyclerViewAdapter.ViewHolder(itemView);
    }
    
    // Involves populating data into the item through holder
    @Override
    public void onBindViewHolder(UserRecyclerViewAdapter.ViewHolder holder, int position) {
        // Get the data model based on position
        User user = users.get(position);
        // Set item views based on the data model
        holder.tvName.setText(user.name);
        holder.tvHometown.setText(user.hometown);
    }

    // Return the total count of items
    @Override
    public int getItemCount() {
        return users.size();
    }
}
```

With the adapter completed, all that is remaining is to bind the data from the adapter into the RecyclerView.

### Binding the Adapter to the RecyclerView

In our activity, we will populate a set of sample users which should be displayed in the `RecyclerView`. 

```java
public class UserListActivity extends ActionBarActivity {

     @Override
     protected void onCreate(Bundle savedInstanceState) {
         // ...
         // Lookup the recyclerview in activity layout
         RecyclerView rvUsers = (RecyclerView) findViewById(R.id.rvUsers);
         // Create adapter passing in the sample user data
         UserRecyclerViewAdapter adapter = new UserRecyclerViewAdapter(getThronesCharacters());
         // Attach the adapter to the recyclerview to populate items
         rvUsers.setAdapter(adapter);
         // Set layout manager to position the items
         rvUsers.setLayoutManager(new LinearLayoutManager(this));
         // That's all!
     }

     private ArrayList<User> getThronesCharacters() {
        ArrayList<User> items = new ArrayList<>();
        items.add(new User("Dany Targaryen", "Valyria"));
        items.add(new User("Rob Stark", "Winterfell"));
        items.add(new User("Jon Snow", "Castle Black"));
        items.add(new User("Tyrion Lanister", "King's Landing"));
        return items;
    }

}
```

Finally, compile and run the app and you should see something like the screenshot below. If you create enough items and scroll through the list, the views will be recycled and far smoother by default than the `ListView` widget:

<img src="http://i.imgur.com/LUPAekZ.png" width="400" alt="Screenshot" />

### Notifying the Adapter

With `RecyclerView` as the data source changes, we need to keep the adapter notified of any changes. There are many method available to use when notifying the adapter of different changes:

| Method | Description |
| ------ | ----------  |
| `notifyDataSetChanged()` | Notify that the dataset has changed. |
| `notifyItemChanged(int pos)` | Notify that item at position has changed. |
| `notifyItemInserted(int pos)` | Notify that item reflected at position has been newly inserted. |
| `notifyItemRemoved(int pos)` | Notify that items previously located at position has been removed from the data set. |

We can use these from the activity or fragment:

```java
// Add a new user
users.set(0, new User(...));
// Notify the adapter
adapter.notifyItemInserted(0);
```

Every time we want to add or remove items from the recyclerview, we will need to explicitly inform to the adapter of the event. 

## Configuring the RecyclerView

The `RecyclerView` is quite flexible and customizable. Several of the options available are shown below.

### Performance

We can also enable optimizations if all item views are of the same height and width for significantly smoother scrolling:

```java
recyclerView.setHasFixedSize(true);
```

### Decorations

We can decorate the items using various decorators attached to the recycler such as the [DividerItemDecoration](https://gist.githubusercontent.com/alexfu/0f464fc3742f134ccd1e/raw/abe729359e5b3691f2fe56445644baf0e40b35ba/DividerItemDecoration.java):

```java
RecyclerView.ItemDecoration itemDecoration = new 
    DividerItemDecoration(this, DividerItemDecoration.VERTICAL_LIST);
recyclerView.addItemDecoration(itemDecoration);
```

This will display dividers between each item within the list as shown below:

<img src="http://i.imgur.com/penvJxw.png" width="400" alt="Screenshot" />

### Layouts

The positioning of the items is configured using the layout manager. By default, we can choose between `LinearLayoutManager`, `GridLayoutManager`, and `StaggeredGridLayoutManager`. Linear displays items either vertically or horizontally:

```java
// Setup layout manager for items
LinearLayoutManager layoutManager = new LinearLayoutManager(this);
// Control orientation of the items
// also supports LinearLayoutManager.HORIZONTAL
layoutManager.setOrientation(LinearLayoutManager.VERTICAL);
// Optionally customize the position you want to default scroll to
layoutManager.scrollToPosition(0);
// Attach layout manager to the RecyclerView
recyclerView.setLayoutManager(layoutManager);
```

Displaying items in a grid or staggered grid works similarly:

```java
// First param is number of columns and second param is orientation i.e Vertical or Horizontal
StaggeredGridLayoutManager gridLayoutManager = 
    new StaggeredGridLayoutManager(2, StaggeredGridLayoutManager.VERTICAL);
// Attach the layout manager to the recycler view
recyclerView.setLayoutManager(gridLayoutManager);
```

For example, a staggered grid might look like:

<img src="http://i.imgur.com/AlANFgj.png" width="300" alt="Screenshot" />

We can build [our own custom layout managers](http://wiresareobsolete.com/2014/09/building-a-recyclerview-layoutmanager-part-1/) as outlined there.

### Animators

RecyclerView supports custom animations for items as they enter, move, or get deleted. If you want to setup custom animations, first load the [third-party recyclerview-animators library](https://github.com/wasabeef/recyclerview-animators) into `app/build.gradle`:

```gradle
repositories {
    jcenter()
}

dependencies {
    compile 'jp.wasabeef:recyclerview-animators:1.2.0@aar'
}
```

Next, we can use any of the defined animators to change the behavior of our RecyclerView:

```java
recyclerView.setItemAnimator(new SlideInUpAnimator());
```

For example, here's scrolling through a list after customizing the animation:

<img src="http://i.imgur.com/v0VyQS8.gif" width="300" alt="Screenshot" />

### Heterogeneous Views

See [this guide](https://github.com/codepath/android_guides/wiki/Heterogenous-Layouts-inside-RecyclerView) if you want to inflate multiple layouts inside a single `RecyclerView`.

### Handling Touch Events

RecyclerView allows us to handle touch events with:

```java
recyclerView.addOnItemTouchListener(new RecyclerView.OnItemTouchListener() {

    @Override
    public void onTouchEvent(RecyclerView recycler, MotionEvent event) {
        // Handle on touch events here
    }

    @Override
    public boolean onInterceptTouchEvent(RecyclerView recycler, MotionEvent event) {
        return false;
    }
    
});
```

### Attaching Click Handlers to Items

RecyclerView does not have special provisions for attaching click handlers to items unlike ListView which has the method `setOnItemClickListener`. To achieve a similar effect, we can attach click events within the `ViewHolder` within our adapter:

```java
public class UserRecyclerViewAdapter extends RecyclerView.Adapter<UserRecyclerViewAdapter.ViewHolder> {
    // ...

    // Used to cache the views within the item layout for fast access
    public static class ViewHolder extends RecyclerView.ViewHolder implements View.OnClickListener {
        public TextView tvName;
        public TextView tvHometown;
        private Context context;

        public ViewHolder(Context context, View itemView) {
            super(itemView);
            this.tvName = (TextView) itemView.findViewById(R.id.tvName);
            this.tvHometown = (TextView) itemView.findViewById(R.id.tvHometown);
            // Store the context
            this.context = context;
            // Attach a click listener to the entire row view
            itemView.setOnClickListener(this);
        }

        // Handles the row being being clicked
        @Override
        public void onClick(View view) {
            int position = getLayoutPosition(); // gets item position
            // We can access the data within the views
            Toast.makeText(context, tvName.getText(), Toast.LENGTH_SHORT).show();
        }
    }
    
    // ...
}
```

If we want the item to show a "selected" effect when pressed, we can set the `android:background` of **the root layout for the row** to `?android:attr/selectableItemBackground`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="horizontal" android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="?android:attr/selectableItemBackground"> 
  <!-- ... -->
</LinearLayout>
```

This creates the following effect:

<img src="http://i.imgur.com/olMUglF.gif" width="400" alt="Screenshot" />

In certain cases, you'd want to setup click handlers for views within the `RecyclerView` but define the click logic within the containing `Activity` or `Fragment` (i.e bubble up events from the adapter). To achieve this, [[create a custom listener|Creating-Custom-Listeners]] within the adapter and then fire the events upwards to an interface implementation defined within the parent:

```java
public class UserRecyclerViewAdapter extends RecyclerView.Adapter<UserRecyclerViewAdapter.ViewHolder> {
    // ...

    /***** Creating OnItemClickListener *****/
    
    // Define listener member variable    
    private OnItemClickListener listener;
    // Define the listener interface
    public interface OnItemClickListener {
        void onItemClick(View view, int position);
    }
    // Define the method that allows the parent activity or fragment to define the listener
    public void setOnItemClickListener(OnItemClickListener listener) {
        this.listener = listener;
    }

    public static class ViewHolder extends RecyclerView.ViewHolder {
        public TextView tvName;
        public TextView tvHometown;

        public ViewHolder(final View itemView) {
            super(itemView);
            this.tvName = (TextView) itemView.findViewById(R.id.tvName);
            this.tvHometown = (TextView) itemView.findViewById(R.id.tvHometown);
            // Setup the click listener
            itemView.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                    // Triggers click upwards to the adapter on click
                    if (listener != null)
                        listener.onItemClick(itemView, getLayoutPosition());
                }
            });
        }
    }
    
    // ...
}
```

Then we can attach a click handler to the adapter with:

```java
// In the activity or fragment
UserRecyclerViewAdapter adapter = ...;
adapter.setOnItemClickListener(new UserRecyclerViewAdapter.OnItemClickListener() {
    @Override
    public void onItemClick(View view, int position) {
        String name = users.get(position).name;
        Toast.makeText(UserListActivity.this, name + " was clicked!", Toast.LENGTH_SHORT).show();
    }
});
```

See [this detailed stackoverflow post](http://stackoverflow.com/a/24933117/313399) which describes how to setup item-level click handlers when using `RecyclerView`. 

## References

* <https://developer.android.com/reference/android/support/v7/widget/RecyclerView.html>
* <http://www.grokkingandroid.com/first-glance-androids-recyclerview/>
* <http://www.grokkingandroid.com/statelistdrawables-for-recyclerview-selection/>
* <http://www.bignerdranch.com/blog/recyclerview-part-1-fundamentals-for-listview-experts/>
* <https://developer.android.com/training/material/lists-cards.html>
* <http://antonioleiva.com/recyclerview/>