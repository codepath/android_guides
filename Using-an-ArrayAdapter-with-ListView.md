In Android development, any time you want to show a list of items vertically you will want to use a ListView and an Adapter. The simplest adapter is called an ArrayAdapter because the adapter takes an Array and converts the items into View objects to be loaded into the ListView container.

The ArrayAdapter fits in the middle between the Array (data source) and the List View (representation) and describes two things:

 * Which array to use as the data source
 * How to convert any item in the array to a View object

## Using a Basic ArrayAdapter

To use a basic ArrayAdapter, you just need to initialize the adapter and attach the adapter to the ListView. First, initialize the adapter:

```java
ArrayAdapter<String> itemsAdapter = new ArrayAdapter<String>(this, android.R.layout.simple_list_item_1, items);
```

The ArrayAdapter requires a type of the item to be converted to a View to be specified and then accepts three arguments: context (activity), XML Resource, and the data array.

By default, this will convert the items in the array into View by calling toString on the item and then inserting the result into a TextView that is displayed as the list item for that data. If the app requires a more complex translation between item and View, we need to create a custom ArrayAdapter.

## Using a Custom ArrayAdapter

### Defining the Model

Given a Java object that has certain fields:

```java
public class User {
    public Long id;
    public String name;
    public String hometown;
}
```

We can create a custom listview of user objects by subclassing ArrayAdapter, describing how to translate the object into a view within that class and then using it like any other adapter.

### Constructing Models

In order to create models, you will likely be loading the data from a source (i.e database or JSON API), so you should create two additional methods in each model to allow for construction of a list or a singular item:

```java
public class User {
    public User(JSONObject object){
        try {
            this.id = object.getLong("id");
            this.name = object.getString("name");
            this.hometown = object.getString("hometown");
       } catch (JSONException e) {
            e.printStackTrace();
       }
    }

    public static ArrayList<User> fromJson(JSONArray jsonObjects) {
           ArrayList<User> users = new ArrayList<User>();
           for (int i = 0; i < jsonObjects.length; i++) {
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

### Creating the View Template

First, we need to create an XML layout (item_user.xml) that represents the template for the item:

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

Next, we need to define the Adapter to describe the process of converting the Java object to a View (in the getView method):

```java
public class UsersAdapter extends ArrayAdapter<User> {
    public UsersAdapter(Context context, List<User> users) {
       super(context, R.layout.item_user, users);
    }
    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
       // Get the data item for this position
       User user = getItem(position); 
       // Check if an existing view is being reused, otherwise inflate the view
        View view = convertView;
        if (view == null) {
           LayoutInflater inflater = LayoutInflater.from(getContext());
           view = inflater.inflate(R.layout.item_user, null);
        }
       // Populate the data into the template view using the data object
       TextView tvName = (TextView) view.findViewById(R.id.tvName);
       TextView tvHome = (TextView) view.findViewById(R.id.tvHome);
       tvName.setText(user.name);
       tvHome.setText(user.hometown);
       // Return the completed view to render on screen
       return view;
   }
}
```

That adapter has a constructor and a "getView" method to describe the translation between the data item and the View to display.

## Attaching the Adapter to a ListView

Now, we can use that adapter in the Activity to display an array of items into the ListView:

```java
// Construct the data source
ArrayList<User> arrayOfUsers = new ArrayList<User>();
// Create the adapter to convert the array to views
adapter = new UsersAdapter(this, arrayOfUsers);
// Attach the adapter to a ListView
ListView listView = (ListView) findViewById(R.id.lvItems);
listView.setAdapter(adapter);
```

At this point, the ListView is now successfully bound to the users array data.

## Populating Data into ListView

Once the adapter is attached, items will automatically be populated into the ListView based on the contents of the array. You can add new items to the adapter at any time with:

```java
// Add item to adapter
User newUser = new User(jsonObject);
adapter.add(newUser);
// Or even append an entire new collection
// Fetching some data, data has now returned
// If data was JSON, convert to ArrayList of User objects.
adapter.addAll(newUsers);
```

which will append the new items to the list. You can also clear the entire list at any time with:

```
adapter.clear()
```

Using the adapter now, you can add, remove and modify users and the items within the ListView will automatically reflect any changes.