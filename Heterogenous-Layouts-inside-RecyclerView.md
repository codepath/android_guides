## Prerequisite

Make sure you are familiar with [RecyclerView] (https://developer.android.com/reference/android/support/v7/widget/RecyclerView.html) by going through the following guide for [basic usage of a `RecyclerView`] (https://github.com/codepath/android_guides/wiki/Using-the-RecyclerView). We will be building on top of the classes from the above guide so it is very important that you have the basic `RecyclerView` up and running.

## Overview

`RecyclerView` can also be used to inflate multiple view types in situations where your list might be heterogeneous, in the sense, based on the response from the server, there might be a requirement for inflating different types of layouts (example: Consider facebook home feed where there are a variety of stories such as a status update, location update, single image, image album, video, etc). This guide will explain how to inflate multiple view types inside your `RecyclerView` widget based on the item type. 

Note: Refer [Implementing a Heterogeneous ListView](https://github.com/codepath/android_guides/wiki/Implementing-a-Heterogenous-ListView) guide on how to inflate multiple item types within a `ListView`.

To implement heterogeneous layouts inside the `RecyclerView`, most of the work is done within the `RecyclerView.Adapter`. In particular, there are special methods to be overridden within the adapter: 
* `getItemViewType()`
* `onCreateViewHolder()`
* `onBindViewHolder()` 

## Implementation 

Building on top of the basic `RecyclerView` [usage project](https://github.com/codepath/android_guides/wiki/Using-the-RecyclerView), we will now replace the `SimpleItemRecyclerViewAdapter` with a more `ComplexRecyclerViewAdapter` which does all the heavy-lifting for inflating different types of layouts based on the item view type. The following example will be inflating two different layouts for based on the object that the List<Object> holds.  `layout_viewholder1.xml` will be used for `User` objects and `layout_viewholder2.xml` will be used for `String` objects. 

For the purpose of this exercise, we will modify our sample data set in `RecyclerViewActivity` to contain a list of objects as shown:

```java
    private ArrayList<Object> getSampleArrayList() {
        ArrayList<Object> items = new ArrayList<>();
        items.add(new User("Dany Targaryen", "Valyria"));
        items.add(new User("Rob Stark", "Winterfell"));
        items.add("image");
        items.add(new User("Jon Snow", "Castle Black"));
        items.add("image");
        items.add(new User("Tyrion Lanister", "King's Landing"));
        return items;
    }
```

Next, you need to create the classes (and layouts) for `ViewHolder1` (`layout_viewholder1.xml`) and `ViewHolder2` (`layout_viewholder2.xml`).

#### ViewHolder1.java

```java
public class ViewHolder1 extends RecyclerView.ViewHolder {

    private TextView label1, label2;

    public ViewHolder1(View v) {
        super(v);
        label1 = (TextView) v.findViewById(R.id.text1);
        label2 = (TextView) v.findViewById(R.id.text2);
    }

    public TextView getLabel1() {
        return label1;
    }

    public void setLabel1(TextView label1) {
        this.label1 = label1;
    }

    public TextView getLabel2() {
        return label2;
    }

    public void setLabel2(TextView label2) {
        this.label2 = label2;
    }
}
```

#### layout_viewholder1.xml

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/llContainer"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:padding="5dp">

    <TextView
        android:id="@+id/text1"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:textStyle="bold"
        android:textAppearance="?android:attr/textAppearanceListItemSmall"
        android:gravity="center_vertical"/>

    <TextView
        android:id="@+id/text2"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:textAppearance="?android:attr/textAppearanceListItemSmall"
        android:gravity="center_vertical" />

</LinearLayout>
```

#### ViewHolder2.java

```java
public class ViewHolder2 extends RecyclerView.ViewHolder {

    private ImageView ivExample;

    public ViewHolder2(View v) {
        super(v);
        ivExample = (ImageView) v.findViewById(R.id.ivExample);
    }

    public ImageView getImageView() {
        return ivExample;
    }

    public void setImageView(ImageView ivExample) {
        this.ivExample = ivExample;
    }
}
```

#### layout_viewholder2.xml

```xml
<ImageView xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/ivExample"
    android:adjustViewBounds="true"
    android:scaleType="fitXY"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"/>
```

_Optional_: The asset that was used is attached below. You may choose your own asset instead.

<img src="http://i105.photobucket.com/albums/m232/purplehaze0077/sample_golden_gate.jpeg" width="500" alt="ss1" />

### Creating the `ComplexRecyclerViewAdapter`

```java
public class ComplexRecyclerViewAdapter extends RecyclerView.Adapter<RecyclerView.ViewHolder> {

    // The items to display in your RecyclerView
    private List<Object> items;

    private final int USER = 0, IMAGE = 1;

    // Provide a suitable constructor (depends on the kind of dataset)
    public ComplexRecyclerViewAdapter(List<Object> items) {
        this.items = items;
    }

    // Return the size of your dataset (invoked by the layout manager)
    @Override
    public int getItemCount() {
        return this.items.size();
    }

    @Override
    public int getItemViewType(int position) {
         //More to come
    }

    @Override
    public RecyclerView.ViewHolder onCreateViewHolder(ViewGroup viewGroup, int viewType) {
         //More to come
    }

    @Override
    public void onBindViewHolder(RecyclerView.ViewHolder viewHolder, int position) {
         //More to come
    }
}
```

Now, you need to override the `getItemViewType` method to tell the `RecyclerView` about the type of view to inflate based on the position. We will return USER or IMAGE based on the type of object in the data we have.

```java
    //Returns the view type of the item at position for the purposes of view recycling.
    @Override
    public int getItemViewType(int position) {
        if (items.get(position) instanceof User) {
            return USER;
        } else if (items.get(position) instanceof String) {
            return IMAGE;
        }
        return -1;
    }
```

Next, you need to override the `onCreateViewHolder` method to tell the `RecyclerView.Adapter` about which `RecyclerView.ViewHolder` object to create based on the `viewType` returned. Create `ViewHolder1` with view `layout_viewholder1` for ODD items and `ViewHolder2` with view `layout_viewholder2` for EVEN ones

```java
     /**
     * This method creates different RecyclerView.ViewHolder objects based on the item view type.\
     *
     * @param viewGroup ViewGroup container for the item
     * @param viewType type of view to be inflated
     * @return viewHolder to be inflated
     */
    @Override
    public RecyclerView.ViewHolder onCreateViewHolder(ViewGroup viewGroup, int viewType) {

        RecyclerView.ViewHolder viewHolder;
        LayoutInflater inflater = LayoutInflater.from(viewGroup.getContext());

        switch (viewType) {
            case USER:
                View v1 = inflater.inflate(R.layout.layout_viewholder1, viewGroup, false);
                viewHolder = new ViewHolder1(v1);
                break;
            case IMAGE:
                View v2 = inflater.inflate(R.layout.layout_viewholder2, viewGroup, false);
                viewHolder = new ViewHolder2(v2);
                break;
            default:
                View v = inflater.inflate(android.R.layout.simple_list_item_1, viewGroup, false);
                viewHolder = new RecyclerViewSimpleTextViewHolder(v);
                break;
        }
        return viewHolder;
    }
```

Next, override the `onBindViewHolder` method to configure the `ViewHolder` with actual data that needs to be displayed. Distinguish the two different layouts and load them with sample text and image as follows.

```java
    /**
     * This method internally calls onBindViewHolder(ViewHolder, int) to update the
     * RecyclerView.ViewHolder contents with the item at the given position
     * and also sets up some private fields to be used by RecyclerView.
     *
     * @param viewHolder The type of RecyclerView.ViewHolder to populate
     * @param position Item position in the viewgroup.
     */
    @Override
    public void onBindViewHolder(RecyclerView.ViewHolder viewHolder, int position) {
        switch (viewHolder.getItemViewType()) {
            case USER:
                ViewHolder1 vh1 = (ViewHolder1) viewHolder;
                configureViewHolder1(vh1, position);
                break;
            case IMAGE:
                ViewHolder2 vh2 = (ViewHolder2) viewHolder;
                configureViewHolder2(vh2);
                break;
            default:
                RecyclerViewSimpleTextViewHolder vh = (RecyclerViewSimpleTextViewHolder) viewHolder;
                configureDefaultViewHolder(vh, position);
                break;
        }
    }
```

The following methods are used for configuring the individual `RecyclerView.ViewHolder` objects:

```java 
    private void configureDefaultViewHolder(RecyclerViewSimpleTextViewHolder vh, int position) {
        vh.getLabel().setText((CharSequence) items.get(position));
    }

    private void configureViewHolder1(ViewHolder1 vh1, int position) {
        User user = (User) items.get(position);
        if (user != null) {
            vh1.getLabel1().setText("Name: " + user.name);
            vh1.getLabel2().setText("Hometown: " + user.hometown);
        }
    }

    private void configureViewHolder2(ViewHolder2 vh2) {
        vh2.getImageView().setImageResource(R.drawable.sample_golden_gate);
    }
```

One ***final and important*** change before you can run the program would be to change the `bindDataToAdapter` method in our `RecyclerViewActivity` to set the `ComplexRecyclerViewAdapter` instead of the `SimpleItemRecyclerViewAdapter` as follows:

```java
    private void bindDataToAdapter() {
        // Bind adapter to recycler view object
        recyclerView.setAdapter(new ComplexRecyclerViewAdapter(getSampleArrayList()));
    }
```

Rest of the implementation remains the same. After compiling and running your app, here's the output you should be looking at:

<img src="http://i105.photobucket.com/albums/m232/purplehaze0077/heterorecycler1.png" width="300" alt="ss1" /> <img src="http://i105.photobucket.com/albums/m232/purplehaze0077/heterorecycler2.png" width="300" alt="ss2" />

### References:

* [RecyclerView] (https://developer.android.com/reference/android/support/v7/widget/RecyclerView.html)
* [StackOverflow post](http://stackoverflow.com/questions/26245139/how-to-create-recyclerview-with-multiple-view-type).
* [Stackoverflow post](http://stackoverflow.com/questions/25914003/recyclerview-and-handling-different-type-of-row-inflation/29362643#29362643)


