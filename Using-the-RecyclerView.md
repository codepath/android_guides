## Overview

The `RecyclerView` is a new `ViewGroup` that is prepared to render any adapter-based view in a similar way. It is supposed to be the successor of [[ListView|Using-an-ArrayAdapter-with-ListView]] and [GridView](http://developer.android.com/guide/topics/ui/layout/gridview.html), and it can be found in the [latest support-v7 version](https://developer.android.com/reference/android/support/v7/app/package-summary.html). One of the reasons is that `RecyclerView` has a more extensible framework, especially since it provides the ability to implement both horizontal and vertical layouts. Use the `RecyclerView` widget when you have data collections whose elements change at runtime based on user action or network events.

If you want to use a `RecyclerView`, you will need to work with the following:
* `RecyclerView.Adapter` - To handle the data collection and bind it to the view
* `LayoutManager` - Helps in positioning the items
* `ItemAnimator` - Helps with animating the items for common operations such as Addition or Removal of item

<img src="https://developer.android.com/training/material/images/RecyclerView.png" width="500" alt="RecyclerView" />

Furthermore, it provides animation support for `ListView` items whenever they are added or removed, which had been extremely difficult to do in the current implementation.  `RecyclerView` also begins to enforce the [[ViewHolder pattern|Using-an-ArrayAdapter-with-ListView#improving-performance-with-the-viewholder-pattern]] too, which was already a recommended practice but now deeply integrated with this new framework.

For more details, see [this detailed overview](http://www.grokkingandroid.com/first-glance-androids-recyclerview/).

### Compared to ListView

`RecyclerView` differs from its predecessor `ListView` primarily because of the following features:

 * **Required ViewHolder in Adapters** - `ListView` adapters do not require the use of the ViewHolder pattern to improve performance. In contrast, implementing an adapter for `RecyclerView` requires the use of the ViewHolder pattern for which it uses `RecyclerView.Viewholder`.
 * **Customizable Item Layouts** - `ListView` can only layout items in a vertical linear arrangement and this cannot be customized. In contrast, the `RecyclerView` has a `RecyclerView.LayoutManager` that allows any item layouts including horizontal lists or staggered grids.
 * **Easy Item Animations** - `ListView` contains no special provisions through which one can animate the addition or deletion of items. In contrast, the `RecyclerView` has the `RecyclerView.ItemAnimator` class for handling item animations.
 * **Manual Data Source** - `ListView` had adapters for different sources such as `ArrayAdapter` and `CursorAdapter` for arrays and database results respectively. In contrast, the `RecyclerView.Adapter` requires a custom implementation to supply the data to the adapter.
 * **Manual Item Decoration** - `ListView` has the `android:divider` property for easy dividers between items in the list. In contrast, `RecyclerView` requires the use of a `RecyclerView.ItemDecoration` object to setup much more manual divider decorations.
 * **Manual Click Detection** - `ListView` has a `AdapterView.OnItemClickListener` interface for binding to the click events for individual items in the list. In contrast, `RecyclerView` only has support for ` RecyclerView.OnItemTouchListener` which manages individual touch events but has no built-in click handling.

## Components of a `RecyclerView`

### `LayoutManagers`

A `RecyclerView` needs to have a layout manager and an adapter to be instantiated. A layout manager positions item views inside a `RecyclerView` and determines when to reuse item views that are no longer visible to the user.

[RecyclerView](https://developer.android.com/reference/android/support/v7/widget/RecyclerView.html) provides these built-in layout managers:

 * `LinearLayoutManager` shows items in a vertical or horizontal scrolling list.
 * `GridLayoutManager` shows items in a grid.
 * `StaggeredGridLayoutManager` shows items in a staggered grid.

To create a custom layout manager, extend the [RecyclerView.LayoutManager](https://developer.android.com/reference/android/support/v7/widget/RecyclerView.LayoutManager.html) class.

Here is [Dave Smith's talk](https://www.youtube.com/watch?v=gs_C1E8HwvE&index=22&list=WL) on custom layout manager
> **Notes**: In recent version of the Support Library, if you don't explicitly set the LayoutManager, the RecyclerView will not show! There is a Logcat error though `E/RecyclerView: No layout manager attached; skipping layout`

### `RecyclerView.Adapter`

`RecyclerView` includes a new kind of adapter. It’s a similar approach to the ones you already used, but with some peculiarities, such as a required `ViewHolder`. You will have to override two main methods: one to inflate the view and its view holder, and another one to bind data to the view. The good thing about this is that first method is called only when we really need to create a new view. No need to check if it’s being recycled.

### `ItemAnimator`

`RecyclerView.ItemAnimator` will animate `ViewGroup` modifications such as add/delete/select that are notified to adapter. `DefaultItemAnimator` can be used for basic default animations and works quite well.  See the [[section|Using-the-RecyclerView#animators]] of this guide for more information.

## Using the RecyclerView

Using a `RecyclerView` has the following key steps:

1. Add `RecyclerView` support library to the gradle build file
2. Define a model class to use as the data source
3. Add a `RecyclerView` to your activity to display the items
4. Create a custom row layout XML file to visualize the item
5. Create a `RecyclerView.Adapter` and `ViewHolder` to render the item
6. Bind the adapter to the data source to populate the `RecyclerView`

The steps are explained in more detail below.

### Installation

Make sure the RecyclerView support library is listed as a dependency in your `app/build.gradle`:

```gradle
dependencies {
    ...
    compile 'com.android.support:recyclerview-v7:25.3.1'
}
```

Click on "Sync Project with Gradle files" to let your IDE download the appropriate resources.

### Defining a Model

Every RecyclerView is backed by a source for data. In this case, we will define a `Contact` class which represents the data model being displayed by the RecyclerView:

```java
public class Contact {
    private String mName;
    private boolean mOnline;

    public Contact(String name, boolean online) {
        mName = name;
        mOnline = online;
    }

    public String getName() {
        return mName;
    }

    public boolean isOnline() {
        return mOnline;
    }

    private static int lastContactId = 0;

    public static ArrayList<Contact> createContactsList(int numContacts) {
        ArrayList<Contact> contacts = new ArrayList<Contact>();

        for (int i = 1; i <= numContacts; i++) {
            contacts.add(new Contact("Person " + ++lastContactId, i <= numContacts / 2));
        }

        return contacts;
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
      android:id="@+id/rvContacts"
      android:layout_width="match_parent"
      android:layout_height="match_parent" />

</RelativeLayout>
```

In the layout, preview we can see the `RecyclerView` within the activity:

<img src="https://i.imgur.com/Qf5fQ8X.png" width="300" />

Now the `RecyclerView` is embedded within our activity layout file. Next, we can define the layout for each item within our list.

### Creating the Custom Row Layout

Before we create the adapter, let's define the XML layout file that will be used for each row within the list. This item layout for now should contain a horizontal linear layout with a textview for the name and a button to message the person:

<img src="https://i.imgur.com/wPRTc76.png" width="300" />
<img src="https://i.imgur.com/fu3FzsV.png" width="300" />

This layout file can be created in `res/layout/item_contact.xml` and will be rendered for each item row.  **Note** that you should be using `wrap_content` for the `layout_height` because `RecyclerView` versions prior to `23.2.1` previously ignored layout parameters.  See [this link](http://android-developers.blogspot.com/2016/02/android-support-library-232.html) for more context.

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
        xmlns:android="http://schemas.android.com/apk/res/android"
        android:orientation="horizontal"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:paddingTop="10dp"
        android:paddingBottom="10dp"
        >

    <TextView
        android:id="@+id/contact_name"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_weight="1"
        />

    <Button
        android:id="@+id/message_button"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:paddingLeft="16dp"
        android:paddingRight="16dp"
        android:textSize="10sp"
        />
</LinearLayout>
```

With the custom item layout complete, let's create the adapter to populate the data into the recycler view.

### Creating the `RecyclerView.Adapter`

Here we need to create the adapter which will actually populate the data into the RecyclerView. The adapter's role is to **convert an object at a position into a list row item** to be inserted.

However with a `RecyclerView` the adapter requires the existence of a "ViewHolder" object which describes and provides access to all the views within each item row.  We can create the basic empty adapter and holder together in `ContactsAdapter.java` as follows:

```java
// Create the basic adapter extending from RecyclerView.Adapter
// Note that we specify the custom ViewHolder which gives us access to our views
public class ContactsAdapter extends
    RecyclerView.Adapter<ContactsAdapter.ViewHolder> {

    // Provide a direct reference to each of the views within a data item
    // Used to cache the views within the item layout for fast access
    public class ViewHolder extends RecyclerView.ViewHolder {
        // Your holder should contain a member variable
        // for any view that will be set as you render a row
        public TextView nameTextView;
        public Button messageButton;

        // We also create a constructor that accepts the entire item row
        // and does the view lookups to find each subview
        public ViewHolder(View itemView) {
            // Stores the itemView in a public final member variable that can be used
            // to access the context from any ViewHolder instance.
            super(itemView);

            nameTextView = (TextView) itemView.findViewById(R.id.contact_name);
            messageButton = (Button) itemView.findViewById(R.id.message_button);
        }
    }
}
```

Now that we've defined the basic adapter and `ViewHolder`, we need to begin filling in our adapter. First, let's store a member variable for the list of contacts and pass the list in through our constructor:

```java
public class ContactsAdapter extends
    RecyclerView.Adapter<ContactsAdapter.ViewHolder> {

    // ... view holder defined above...

    // Store a member variable for the contacts
    private List<Contact> mContacts;
    // Store the context for easy access
    private Context mContext;

    // Pass in the contact array into the constructor
    public ContactsAdapter(Context context, List<Contact> contacts) {
        mContacts = contacts;
        mContext = context;
    }

    // Easy access to the context object in the recyclerview
    private Context getContext() {
       return mContext;
    }
}
```

Every adapter has three primary methods: `onCreateViewHolder` to inflate the item layout and create the holder, `onBindViewHolder` to set the view attributes based on the data and `getItemCount` to determine the number of items. We need to implement all three to finish the adapter:

```java
public class ContactsAdapter extends
    RecyclerView.Adapter<ContactsAdapter.ViewHolder> {

    // ... constructor and member variables

    // Usually involves inflating a layout from XML and returning the holder
    @Override
    public ContactsAdapter.ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        Context context = parent.getContext();
        LayoutInflater inflater = LayoutInflater.from(context);

        // Inflate the custom layout
        View contactView = inflater.inflate(R.layout.item_contact, parent, false);

        // Return a new holder instance
        ViewHolder viewHolder = new ViewHolder(contactView);
        return viewHolder;
    }

    // Involves populating data into the item through holder
    @Override
    public void onBindViewHolder(ContactsAdapter.ViewHolder viewHolder, int position) {
        // Get the data model based on position
        Contact contact = mContacts.get(position);

        // Set item views based on your views and data model
        TextView textView = viewHolder.nameTextView;
        textView.setText(contact.getName());
        Button button = viewHolder.messageButton;
        button.setText(contact.isOnline() ? "Message" : "Offline");
        button.setEnabled(contact.isOnline());
    }

    // Returns the total count of items in the list
    @Override
    public int getItemCount() {
        return mContacts.size();
    }
}
```

With the adapter completed, all that is remaining is to bind the data from the adapter into the RecyclerView.

### Binding the Adapter to the RecyclerView

In our activity, we will populate a set of sample users which should be displayed in the `RecyclerView`.

```java
public class UserListActivity extends AppCompatActivity {

     ArrayList<Contact> contacts;

     @Override
     protected void onCreate(Bundle savedInstanceState) {
         // ...
         // Lookup the recyclerview in activity layout
         RecyclerView rvContacts = (RecyclerView) findViewById(R.id.rvContacts);

         // Initialize contacts
         contacts = Contact.createContactsList(20);
         // Create adapter passing in the sample user data
         ContactsAdapter adapter = new ContactsAdapter(this, contacts);
         // Attach the adapter to the recyclerview to populate items
         rvContacts.setAdapter(adapter);
         // Set layout manager to position the items
         rvContacts.setLayoutManager(new LinearLayoutManager(this));
         // That's all!
     }
}
```

Finally, compile and run the app and you should see something like the screenshot below. If you create enough items and scroll through the list, the views will be recycled and far smoother by default than the `ListView` widget:

<img src="https://i.imgur.com/vbIL5HA.gif" width="400" alt="Screenshot" />

### Notifying the Adapter

Unlike ListView, there is no way to add or remove items directly through the `RecyclerView` adapter.  You need to make changes to the data source directly and notify the adapter of any changes.   Also, whenever adding or removing elements, always make changes to the **existing** list.  For instance, reinitializing the list of Contacts such as the following will not affect the adapter, since it has a memory reference to the old list:

```java
// do not reinitialize an existing reference used by an adapter
contacts = Contact.createContactsList(5);
```

Instead, you need to act directly on the existing reference:

```java
// add to the existing list
contacts.addAll(Contact.createContactsList(5));
```

There are many method available to use when notifying the adapter of different changes:

| Method | Description |
| ------ | ----------  |
| `notifyItemChanged(int pos)` | Notify that item at position has changed. |
| `notifyItemInserted(int pos)` | Notify that item reflected at position has been newly inserted. |
| `notifyItemRemoved(int pos)` | Notify that items previously located at position has been removed from the data set. |
| `notifyDataSetChanged()` | Notify that the dataset has changed.  Use only as last resort.|

We can use these from the activity or fragment:

```java
// Add a new contact
contacts.add(0, new Contact("Barney", true));
// Notify the adapter that an item was inserted at position 0
adapter.notifyItemInserted(0);
```

Every time we want to add or remove items from the RecyclerView, we will need to explicitly inform to the adapter of the event.  Unlike the ListView adapter, a RecyclerView adapter should not rely on `notifyDataSetChanged()` since the more granular actions should be used.  See the [API documentation](https://developer.android.com/reference/android/support/v7/widget/RecyclerView.Adapter.html) for more details.

Also, if you are intending to update an existing list, make sure to get the current count of items before making any changes.  For instance, a `getItemCount()` on the adapter should be called to record the first index that will be changed.

```java
// record this value before making any changes to the existing list
int curSize = adapter.getItemCount();

// replace this line with wherever you get new records
ArrayList<Contact> newItems = Contact.createContactsList(20);

// update the existing list
contacts.addAll(newItems);
// curSize should represent the first element that got added
// newItems.size() represents the itemCount
adapter.notifyItemRangeInserted(curSize, newItems.size());
```

### Diffing Larger Changes

Often times there are cases when changes to your list are more complex (i.e. sorting an existing list) and it cannot be easily determined whether each row changed.  In this cases, you would normally have to call `notifyDataSetChanged()` on the entire adapter to update the entire screen, which eliminates the ability to perform animation sequences to showcase what changed.

A new `DiffUtil` class has been added in the v24.2.0 of the support library to help compute the difference between the old and new list.   This class uses the same algorithm used for computing line changes in source code (the [diff utility](https://en.wikipedia.org/wiki/Diff_utility) program), so it usually fairly fast.  It is recommended however for larger lists that you execute this computation in a background thread.

To use the `DiffUtil` class, you need to first implement a class that implements the `DiffUtil.Callback` that accepts the old and new list:

```java

public class ContactDiffCallback extends DiffUtil.Callback {

    private List<Contact> mOldList;
    private List<Contact> mNewList;

    public ContactDiffCallback(List<Contact> oldList, List<Contact> newList) {
        this.mOldList = oldList;
        this.mNewList = newList;
    }
    @Override
    public int getOldListSize() {
        return mOldList.size();
    }

    @Override
    public int getNewListSize() {
        return mNewList.size();
    }

    @Override
    public boolean areItemsTheSame(int oldItemPosition, int newItemPosition) {
        // add a unique ID property on Contact and expose a getId() method
        return mOldList.get(oldItemPosition).getId() == mNewList.get(newItemPosition).getId();
    }

    @Override
    public boolean areContentsTheSame(int oldItemPosition, int newItemPosition) {
        Contact oldContact = mOldList.get(oldItemPosition);
        Contact newContact = mNewList.get(newItemPosition);

        if (oldContact.getName() == newContact.getName() && oldContact.isOnline() == newContact.isOnline()) {
            return true;
        }
        return false;
    }
}
```

Next, you would implement a `swapItems()` method on your adapter to perform the diff and then invoke `dispatchUpdates()` to notify the adapter whether the element was inserted, removed, moved, or changed:

```java
public class ContactsAdapter extends
        RecyclerView.Adapter<ContactsAdapter.ViewHolder> {

  public void swapItems(List<Contact> contacts) {
        // compute diffs
        final ContactDiffCallback diffCallback = new ContactDiffCallback(this.mContacts, contacts);
        final DiffUtil.DiffResult diffResult = DiffUtil.calculateDiff(diffCallback);

        // clear contacts and add
        this.mContacts.clear();
        this.mContacts.addAll(contacts);

        diffResult.dispatchUpdatesTo(this); // calls adapter's notify methods after diff is computed
    }
}
```

For a working example, see this [sample code](https://github.com/mrmike/DiffUtil-sample).

### Scrolling to New Items

If we are inserting elements to the front of the list and wish to maintain the position at the top, we can set the scroll position to the 1st element:

```java
adapter.notifyItemInserted(0);
rvContacts.scrollToPosition(0);   // index 0 position
```

If we are adding items to the end and wish to scroll to the bottom as items are added, we can notify the adapter that an additional element has been added and can call `smoothScrollToPosition()` on the RecyclerView:

```java
adapter.notifyItemInserted(contacts.size() - 1);  // contacts.size() - 1 is the last element position
rvContacts.scrollToPosition(mAdapter.getItemCount() - 1); // update based on adapter
```

### Implementing Endless Scrolling

To implement fetching more data and appending to the end of the list as the user scrolls towards the bottom,  use the `addOnScrollListener()` from the `RecyclerView` and add an `onLoadMore` method leveraging the [[EndlessScrollViewScrollListener|Endless-Scrolling-with-AdapterViews-and-RecyclerView#implementing-with-recyclerview]] document in the guide.

## Configuring the RecyclerView

The `RecyclerView` is quite flexible and customizable. Several of the options available are shown below.

### Performance

We can also enable optimizations if the items are static and will not change for significantly smoother scrolling:

```java
recyclerView.setHasFixedSize(true);
```

### Layouts

The positioning of the items is configured using the layout manager. By default, we can choose between `LinearLayoutManager`, `GridLayoutManager`, and `StaggeredGridLayoutManager`. Linear displays items either vertically or horizontally:

```java
// Setup layout manager for items with orientation
// Also supports `LinearLayoutManager.HORIZONTAL`
LinearLayoutManager layoutManager = new LinearLayoutManager(this, LinearLayoutManager.VERTICAL, false);
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

<img src="https://i.imgur.com/AlANFgj.png" width="300" alt="Screenshot" />

We can build [our own custom layout managers](http://wiresareobsolete.com/2014/09/building-a-recyclerview-layoutmanager-part-1/) as outlined there.

### Decorations

We can decorate the items using various decorators attached to the recyclerview such as the [DividerItemDecoration](https://developer.android.com/reference/android/support/v7/widget/DividerItemDecoration.html):

```java
RecyclerView.ItemDecoration itemDecoration = new
    DividerItemDecoration(this, DividerItemDecoration.VERTICAL);
recyclerView.addItemDecoration(itemDecoration);
```

This decorator displays dividers between each item within the list as illustrated below:

<img src="https://i.imgur.com/penvJxw.png" width="400" alt="Screenshot" />

#### Grid Spacing Decorations

Decorators can also be used for adding consistent spacing around items displayed in a grid layout or staggered grid. Copy over this [SpacesItemDecoration.java decorator](https://gist.github.com/nesquena/db922669798eba3e3661) into your project and apply to a `RecyclerView` using the `addItemDecoration` method. Refer to [this staggered grid tutorial](http://blog.grafixartist.com/pinterest-masonry-layout-staggered-grid/) for a more detailed outline.

### Animators

RecyclerView supports custom animations for items as they enter, move, or get deleted using [ItemAnimator](https://developer.android.com/reference/android/support/v7/widget/RecyclerView.ItemAnimator.html).  The default animation effects is defined by [DefaultItemAnimator](https://developer.android.com/reference/android/support/v7/widget/DefaultItemAnimator.html), and the complex implementation (see [source code)](https://android.googlesource.com/platform/frameworks/support/+/refs/heads/master/v7/recyclerview/src/android/support/v7/widget/DefaultItemAnimator.java) shows that the logic necessary to ensure that animation effects are performed in a specific sequence (remove, move, and add).

Currently, the fastest way to implement animations with RecyclerView is to use third-party libraries.  The [third-party recyclerview-animators library](https://github.com/wasabeef/recyclerview-animators) contains a lot of animations that you can use without needing to build your own.  Simply edit your  `app/build.gradle`:

```gradle
repositories {
    jcenter()
}

//If you are using a RecyclerView 23.1.0 or higher.
dependencies {
    compile 'jp.wasabeef:recyclerview-animators:2.2.3'
}

//If you are using a RecyclerView 23.0.1 or below.
dependencies {
    compile 'jp.wasabeef:recyclerview-animators:1.3.0'
}
```

Next, we can use any of the defined animators to change the behavior of our RecyclerView:

```java
recyclerView.setItemAnimator(new SlideInUpAnimator());
```

For example, here's scrolling through a list after customizing the animation:

<img src="https://i.imgur.com/v0VyQS8.gif" width="300" alt="Screenshot" />

For a further look into defining custom item animators, check out this [custom RecyclerView item animation post](https://hackmd.io/s/r1IEQ-jAl).

#### New ItemAnimator interface

Starting in the [support v23.1.0](https://developer.android.com/tools/support-library/index.html#revisions) library for `RecyclerView`, there is also a new interface for the [ItemAnimator](https://developer.android.com/reference/android/support/v7/widget/RecyclerView.ItemAnimator.html#pubmethods) interface.   The old interface has now been deprecated to `SimpleItemAnimator` .  This library adds a [ItemHolderInfo](https://developer.android.com/reference/android/support/v7/widget/RecyclerView.ItemAnimator.ItemHolderInfo.html) class, which appears to be similar to the [MoveInfo](https://github.com/android/platform_frameworks_support/blob/master/v7/recyclerview/src/android/support/v7/widget/DefaultItemAnimator.java#L53-L63) class defined by `DefaultItemAnimator` but used more generically to pass state information between animation transition states.  It is likely that the next version of `DefaultItemAnimator` will be simplified to use this new class and revised interface.

### Heterogeneous Views

See [[this guide|Heterogenous-Layouts-inside-RecyclerView]] if you want to inflate multiple types of rows inside a single `RecyclerView`:

<img src="https://i.imgur.com/HyOAOOu.png" width="300" alt="ss2" />

This is useful for feeds which contain various different types of items within a single list.

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

### Snap to Center Effect

In certain cases, we might want a horizontal `RecyclerView` that allows the user to scroll through a list of items. As the user scrolls, we might want items to "snap to center" as they are revealed. Such as in this example:

<img src="https://i.imgur.com/D5crJK4.gif" width="300" />

#### LinearSnapHelper

To achieve this snapping to center effect as the user scrolls, starting with support library 24.2.0 version and greater, we can use the built-in [LinearSnapHelper](https://developer.android.com/reference/android/support/v7/widget/LinearSnapHelper.html) as follows:

```java
SnapHelper snapHelper = new LinearSnapHelper();
snapHelper.attachToRecyclerView(recyclerView);
```

For more sophisticated snapping behavior, [read more about customizing these helpers](https://rubensousa.github.io/2016/08/recyclerviewsnap) and [review related sample code here](https://github.com/rubensousa/RecyclerViewSnap/).

#### SnappyRecyclerView

For a more manual approach, we can create a custom extension to `RecyclerView` called `SnappyRecyclerView` which will snap items to center as the user scrolls:

1. Copy over the code from [SnappyRecyclerView.java](https://gist.github.com/nesquena/b26f9a253bbbb6a2e4890891e8a57eb9) to your project.
2. Configure your new `SnappyRecyclerView` with a horizontal `LinearLayoutManager`:

   ```java
   LinearLayoutManager layoutManager = new LinearLayoutManager(this, LinearLayoutManager.HORIZONTAL, false);
   snappyRecyclerView.setLayoutManager(layoutManager);
   ```

3. Attach your adapter to the `RecyclerView` to populate the data into the horizontal list as normal.
4. You can access the currently "snapped" item position with `snappyRecyclerView.getFirstVisibleItemPosition()`.

That's all, you should be set for a snap-to-center horizontal scrolling list!

### Attaching Click Handlers to Items

#### Attaching Click Listeners with Decorators

Then easiest solution for setting up item click handlers within a `RecyclerView` is to use a decorator class to manage the item click listener. With [this clever `ItemClickSupport` decorator](https://gist.github.com/nesquena/231e356f372f214c4fe6), attaching a click handler can be done with:

```java
ItemClickSupport.addTo(mRecyclerView).setOnItemClickListener(
  new ItemClickSupport.OnItemClickListener() {
      @Override
      public void onItemClicked(RecyclerView recyclerView, int position, View v) {
          // do it
      }
  }
);
```

Under the covers, this is wrapping the interface pattern described in detail below. If you apply this code above, you **don't need to any of the manual item click handling below**. This technique was originally [outlined in this article](http://www.littlerobots.nl/blog/Handle-Android-RecyclerView-Clicks/).

#### Simple Click Handler within ViewHolder

RecyclerView does not have special provisions for attaching click handlers to items unlike ListView which has the method `setOnItemClickListener`. To achieve a similar effect manually (instead of using the decorator utility above), we can attach click events within the `ViewHolder` inside our adapter:

```java
public class ContactsAdapter extends RecyclerView.Adapter<ContactsAdapter.ViewHolder> {
    // ...

    // Used to cache the views within the item layout for fast access
    public class ViewHolder extends RecyclerView.ViewHolder implements View.OnClickListener {
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
            int position = getAdapterPosition(); // gets item position
            if (position != RecyclerView.NO_POSITION) { // Check if an item was deleted, but the user clicked it before the UI removed it
                User user = users.get(position);
                // We can access the data within the views
                Toast.makeText(context, tvName.getText(), Toast.LENGTH_SHORT).show();
            }
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

<img src="https://i.imgur.com/olMUglF.gif" width="400" alt="Screenshot" />

#### Attaching Click Handlers using Listeners

In certain cases, you'd want to setup click handlers for views within the `RecyclerView` but define the click logic within the containing `Activity` or `Fragment` (i.e bubble up events from the adapter). To achieve this, [[create a custom listener|Creating-Custom-Listeners]] within the adapter and then fire the events upwards to an interface implementation defined within the parent:

```java
public class ContactsAdapter extends RecyclerView.Adapter<ContactsAdapter.ViewHolder> {
    // ...

    /***** Creating OnItemClickListener *****/

    // Define listener member variable
    private OnItemClickListener listener;
    // Define the listener interface
    public interface OnItemClickListener {
        void onItemClick(View itemView, int position);
    }
    // Define the method that allows the parent activity or fragment to define the listener
    public void setOnItemClickListener(OnItemClickListener listener) {
        this.listener = listener;
    }

    public class ViewHolder extends RecyclerView.ViewHolder {
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
                    if (listener != null) {
                        int position = getAdapterPosition();
                        if (position != RecyclerView.NO_POSITION) {
                            listener.onItemClick(itemView, position);
                        }
                    }
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
ContactsAdapter adapter = ...;
adapter.setOnItemClickListener(new ContactsAdapter.OnItemClickListener() {
    @Override
    public void onItemClick(View view, int position) {
        String name = users.get(position).name;
        Toast.makeText(UserListActivity.this, name + " was clicked!", Toast.LENGTH_SHORT).show();
    }
});
```

See [this detailed stackoverflow post](http://stackoverflow.com/a/24933117/313399) which describes how to setup item-level click handlers when using `RecyclerView`.

## Implementing Pull to Refresh

The `SwipeRefreshLayout` should be used to refresh the contents of a `RecyclerView` via a vertical swipe gesture. See our detailed [[RecyclerView with SwipeRefreshLayout|Implementing-Pull-to-Refresh-Guide#recyclerview-with-swiperefreshlayout]] guide for a step-by-step tutorial on implementing pull to refresh.

## Swipe Detection

RecyclerView (since the release of v24.2.0) now has an `OnFlingListener` method that can be used to implement custom fling behavior.  Download this [RecyclerViewSwipeListener](https://gist.github.com/rogerhu/9e769149f9550c0a6ddb4987b94caee8) and you can handle custom swipe detection by adding this class to your RecyclerView:

```java
RecyclerView rvMyList;

rvMyList.setOnFlingListener(new RecyclerViewSwipeListener(true) {
   @Override
   public void onSwipeDown() {
      Toast.makeText(MainActivity.this, "swipe down", Toast.LENGTH_SHORT).show();
   }

   @Override
   public void onSwipeUp() {
      Toast.makeText(MainActivity.this, "swipe up", Toast.LENGTH_SHORT).show();
   }
});
```
## Using Uncover library

If your use RecyclerView to implement 'infinite scrolling' over output of the web service, or some other resource that requires slow background calls with multiple items at once, it may be reasonable to try the [Uncover library](https://github.com/andviane/google-books-android-viewer) from Maven central:

```java
dependencies {
    compile ('io.github.andviane:uncover:2.0.1@aar')
}    
```
This library requires you to implement the primary data fetcher, mediating between fast single item UI-thread calls on model and slow chunked calls on the background on your fetcher:

```java
    final RecyclerView recyclerView = (RecyclerView) findViewById(R.id.recycler);
    final UncoveringDataModel<String> model = new UncoveringDataModel<>();

    model.setPrimaryDataProvider(new PrimaryDataProvider<String>() {

      @Override
      public PrimaryResponse fetch(PrimaryRequest primaryRequest) {
        Log.i("Fetch", "Service call to fetch items" + 
          primaryRequest.getFrom() + "- " + primaryRequest.getTo());
        ...
        ArrayList<String> data = new ArrayList<String>();
        ...
        return new PrimaryResponse<String>(data, Integer.MAX_VALUE);
      }
    });

    RecyclerView.Adapter adapter = new RecyclerView.Adapter() {
      public int getItemCount() {
        return model.size();
      ...
    };
    model.install(recyclerView, adapter);
```

The `model.install` glues model, view and adapter into working implementation. When the user swipes forward quickly, the library skips unneeded fetches of data between the new and old position, last requested data are fetched first, and any pending requests are dropped from the queue if they data are no longer visible. 

## References

* <https://developer.android.com/reference/android/support/v7/widget/RecyclerView.html>
* <http://www.grokkingandroid.com/first-glance-androids-recyclerview/>
* <http://www.grokkingandroid.com/statelistdrawables-for-recyclerview-selection/>
* <http://www.bignerdranch.com/blog/recyclerview-part-1-fundamentals-for-listview-experts/>
* <https://developer.android.com/training/material/lists-cards.html>
* <http://antonioleiva.com/recyclerview/>
* <https://code.tutsplus.com/tutorials/getting-started-with-recyclerview-and-cardview-on-android--cms-23465>
* <https://code.tutsplus.com/tutorials/introduction-to-the-new-lollipop-activity-transitions--cms-23711>
