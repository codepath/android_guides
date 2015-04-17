In Android development, any time we want to show a vertical list of scrollable items we will use a `ListView` which has data populated using an `Adapter`. The simplest adapter to use is called an `ArrayAdapter` because the adapter converts an `ArrayList` of objects into `View` items loaded into the `ListView` container.

<img src="http://i.imgur.com/mk82Jd2.jpg" width="600" />

The `ArrayAdapter` fits in between an `ArrayList` (data source) and the `ListView` (visual representation) and configures two aspects:

 * Which array to use as the data source for the list
 * How to convert any given item in the array into a corresponding View object

Note as shown above that there are other data sources besides an `ArrayAdapter` such as the [[CursorAdapter|Populating-a-ListView-with-a-CursorAdapter]] which instead binds directly to a result set from a [[Local SQLite Database|Local-Databases-with-SQLiteOpenHelper]].

## Using a Basic ArrayAdapter

To use a basic `ArrayAdapter`, you just need to initialize the adapter and attach the adapter to the ListView. First, we initialize the adapter:

```java
ArrayAdapter<String> itemsAdapter = 
    new ArrayAdapter<String>(this, android.R.layout.simple_list_item_1, items);
```

The `ArrayAdapter` requires a declaration of the type of the item to be converted to a `View` (a `String` in this case) and then accepts three arguments: `context` (activity instance), XML item layout, and the array of data. Note that we've chosen [simple_list_item_1.xml](https://github.com/android/platform_frameworks_base/blob/master/core/res/res/layout/simple_list_item_1.xml) which is a simple `TextView` as the layout for each of the items.

Now, we just need to connect this adapter to a `ListView` to be populated:

```java
ListView listView = (ListView) findViewById(R.id.lvItems);
listView.setAdapter(itemsAdapter);
```

By default, this will now convert each item in the data array into a view by calling `toString` on the item and then assigning the result as the value of a `TextView` ([simple_list_item_1.xml](https://github.com/android/platform_frameworks_base/blob/master/core/res/res/layout/simple_list_item_1.xml)) that is displayed as the row for that data item. If the app requires a more complex translation between item and `View` then we need to create a custom `ArrayAdapter` instead. 

## Using a Custom ArrayAdapter

When we want to display a series of items into a list using a custom representation of the items, we need to use our own custom XML layout for each item. To do this, we need to create our own custom `ArrayAdapter` class. See [this repo for the source code](https://github.com/codepath/android-custom-array-adapter-demo). First, we often need to define a model to represent the data within each list item.

### Defining the Model

Given a Java object that has certain fields defined such as a `User` class:

```java
public class User {
    public String name;
    public String hometown;

    public User(String name, String hometown) {
       this.name = name;
       this.hometown = hometown;
    }
}
```

We can create a custom `ListView` of `User` objects by subclassing `ArrayAdapter` to describe how to translate the object into a view within that class and then using it like any other adapter.

### Creating the View Template

Next, we need to create an XML layout that represents the view template for each item in `res/layout/item_user.xml`:

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
 android:layout_width="match_parent"
 android:layout_height="match_parent" >
    <TextView
      android:id="@+id/tvName"
      android:layout_width="wrap_content"
      android:layout_height="wrap_content"
      android:text="Name" />
   <TextView
      android:id="@+id/tvHome"
      android:layout_width="wrap_content"
      android:layout_height="wrap_content"
      android:text="HomeTown" />
</LinearLayout>
```

### Defining the Adapter

Next, we need to define the adapter to describe the process of converting the Java object to a View (in the `getView` method). The naive approach to this (without any view caching) looks like the following:

```java
public class UsersAdapter extends ArrayAdapter<User> {
    public UsersAdapter(Context context, ArrayList<User> users) {
       super(context, 0, users);
    }

    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
       // Get the data item for this position
       User user = getItem(position);    
       // Check if an existing view is being reused, otherwise inflate the view
       if (convertView == null) {
          convertView = LayoutInflater.from(getContext()).inflate(R.layout.item_user, parent, false);
       }
       // Lookup view for data population
       TextView tvName = (TextView) convertView.findViewById(R.id.tvName);
       TextView tvHome = (TextView) convertView.findViewById(R.id.tvHome);
       // Populate the data into the template view using the data object
       tvName.setText(user.name);
       tvHome.setText(user.hometown);
       // Return the completed view to render on screen
       return convertView;
   }
}
```

That adapter has a constructor and a `getView()` method to describe the **translation between the data item and the View** to display.  `getView()` is the method that returns the actual view used as a row within the `ListView` at a particular position.  

## Attaching the Adapter to a ListView

Now, we can use that adapter in the Activity to display an array of items into the ListView:

```java
// Construct the data source
ArrayList<User> arrayOfUsers = new ArrayList<User>();
// Create the adapter to convert the array to views
UsersAdapter adapter = new UsersAdapter(this, arrayOfUsers);
// Attach the adapter to a ListView
ListView listView = (ListView) findViewById(R.id.lvItems);
listView.setAdapter(adapter);
```

At this point, the ListView is now successfully bound to the users array data.

## Populating Data into ListView

Once the adapter is attached, items will automatically be populated into the ListView based on the contents of the array. You can add new items to the adapter at any time with:

```java
// Add item to adapter
User newUser = new User("Nathan", "San Diego");
adapter.add(newUser);
// Or even append an entire new collection
// Fetching some data, data has now returned
// If data was JSON, convert to ArrayList of User objects.
JSONArray jsonArray = ...;
ArrayList<User> newUsers = User.fromJson(jsonArray)
adapter.addAll(newUsers);
```

which will append the new items to the list. You can also clear the entire list at any time with:

```java
adapter.clear();
```

Using the adapter now, you can add, remove and modify users and the items within the ListView will automatically reflect any changes.

### Constructing Models from External Source

In order to create model instances, you will likely be loading the data from an external source (i.e database or REST JSON API), so you should create two additional methods in each model to allow for construction of a list or a singular item **if the data is coming from a JSON API**:

```java
public class User {
    // Constructor to convert JSON object into a Java class instance
    public User(JSONObject object){
        try {
            this.name = object.getString("name");
            this.hometown = object.getString("hometown");
       } catch (JSONException e) {
            e.printStackTrace();
       }
    }

    // Factory method to convert an array of JSON objects into a list of objects
    // User.fromJson(jsonArray);
    public static ArrayList<User> fromJson(JSONArray jsonObjects) {
           ArrayList<User> users = new ArrayList<User>();
           for (int i = 0; i < jsonObjects.length(); i++) {
               try {
                  users.add(new User(jsonObjects.getJSONObject(i)));
               } catch (JSONException e) {
                  e.printStackTrace();
               }
          }
          return users;
    }
}
```

For more details, check out our guide on [[converting JSON into a model|Converting JSON to Models]]. If you are not using a JSON source for your data, you can safely skip this step.

## Improving Performance with the ViewHolder Pattern

To improve performance, we should modify the custom adapter by applying the **ViewHolder** pattern which speeds up the population of the ListView considerably by caching view lookups for smoother, faster item loading:

```java
public class UsersAdapter extends ArrayAdapter<User> {
    // View lookup cache
    private static class ViewHolder {
        TextView name;
        TextView home;
    }

    public UsersAdapter(Context context, ArrayList<User> users) {
       super(context, R.layout.item_user, users);
    }

    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
       // Get the data item for this position
       User user = getItem(position);    
       // Check if an existing view is being reused, otherwise inflate the view
       ViewHolder viewHolder; // view lookup cache stored in tag
       if (convertView == null) {
          viewHolder = new ViewHolder();
          LayoutInflater inflater = LayoutInflater.from(getContext());
          convertView = inflater.inflate(R.layout.item_user, parent, false);
          viewHolder.name = (TextView) convertView.findViewById(R.id.tvName);
          viewHolder.home = (TextView) convertView.findViewById(R.id.tvHome);
          convertView.setTag(viewHolder);
       } else {
           viewHolder = (ViewHolder) convertView.getTag();
       }
       // Populate the data into the template view using the data object
       viewHolder.name.setText(user.name);
       viewHolder.home.setText(user.hometown);
       // Return the completed view to render on screen
       return convertView;
   }
}
```

In this example we also have a private static class called `ViewHolder`.  Making calls to `findViewById()` is really slow in practice, and if your adapter has to call it for each View in your row for every single row then you will quickly run into performance issues.  What the ViewHolder class does is cache the call to `findViewById()`.  Once your ListView has reached the max amount of rows it can display on a screen, Android is smart enough to begin recycling those row Views.  We check if a View is recycled with `if (convertView == null)`.  If it is not null then we have a recycled View and can just change its values, otherwise we need to create a new row View.  The magic behind this is the `setTag()` method which lets us attach an arbitrary object onto a View object, which is how we save the already inflated View for future reuse.

### Beyond ViewHolders

[Customizing Android ListView Rows by Subclassing](http://www.bignerdranch.com/blog/customizing-android-listview-rows-subclassing/) describes a strategy for obtaining instances of child views using a similar approach as a ViewHolder but without the explicit ViewHolder subclass.

## References

 * <http://lucasr.org/2012/04/05/performance-tips-for-androids-listview/>
 * <http://www.doubleencore.com/2013/05/layout-inflation-as-intended/>
 * <http://www.bignerdranch.com/blog/customizing-android-listview-rows-subclassing/>