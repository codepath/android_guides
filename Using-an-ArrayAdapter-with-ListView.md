In Android development, any time you want to show a list of items vertically you will want to use a ListView and an Adapter. The simplest adapter is called an ArrayAdapter because the adapter takes an Array and converts the items into View objects to be loaded into the ListView container.

The ArrayAdapter fits in the middle between the Array (data source) and the List View (representation) and describes two things:

 * Which array to use as the data source
 * How to convert any item in the array to a View object

## Using a Basic ArrayAdapter

To use a basic ArrayAdapter, you just need to initialize the adapter and attach the adapter to the ListView. First, initialize the adapter:

```java
ArrayAdapter<String> itemsAdapter = 
    new ArrayAdapter<String>(this, android.R.layout.simple_list_item_1, items);
```

The ArrayAdapter requires a type of the item to be converted to a View to be specified and then accepts three arguments: context (activity), XML Resource, and the data array.

By default, this will convert the items in the array into View by calling toString on the item and then inserting the result into a TextView that is displayed as the list item for that data. If the app requires a more complex translation between item and View, we need to create a custom ArrayAdapter.

## Using a Custom ArrayAdapter

### Defining the Model

Given a Java object that has certain fields:

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

We can create a custom listview of user objects by subclassing ArrayAdapter, describing how to translate the object into a view within that class and then using it like any other adapter.

### Constructing Models

In order to create models, you will likely be loading the data from a source (i.e database or JSON API), so you should create two additional methods in each model to allow for construction of a list or a singular item if the data is coming from a JSON API:

```java
public class User {
    public User(JSONObject object){
        try {
            this.name = object.getString("name");
            this.hometown = object.getString("hometown");
       } catch (JSONException e) {
            e.printStackTrace();
       }
    }

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

For more details, check out our guide on [[converting JSON into a model|Converting JSON to Models]].

### Creating the View Template

Next, we need to create an XML layout that represents the template for the item in `res/layout/item_user.xml`:

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

Next, we need to define the Adapter to describe the process of converting the Java object to a View (in the getView method). The simpler approach to this (without view caching) is the following:

```java
public class UsersAdapter extends ArrayAdapter<User> {
    public UsersAdapter(Context context, ArrayList<User> users) {
       super(context, R.layout.item_user, users);
    }

    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
       // Get the data item for this position
       User user = getItem(position);    
       // Check if an existing view is being reused, otherwise inflate the view
       if (convertView == null) {
          convertView = LayoutInflater.from(getContext()).inflate(R.layout.item_user, null);
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

That adapter has a constructor and a `getView()` method to describe the **translation between the data item and the View** to display.  `getView()` is the method that returns the actual view used as a row within the ListView at a particular position.  

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

```
adapter.clear()
```

Using the adapter now, you can add, remove and modify users and the items within the ListView will automatically reflect any changes.

## Improving Performance with the ViewHolder Pattern

To improve performance, we should modify the custom adapter by applying the **ViewHolder** pattern which speeds up the population of the ListView considerably by caching view lookups for smoother, faster loading:

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
          convertView = inflater.inflate(R.layout.item_user, null);
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

## References

 * <http://lucasr.org/2012/04/05/performance-tips-for-androids-listview/>