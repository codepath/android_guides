## Overview

In certain situations, we need to implement a ListView where there are **different types of rows** in the same list. In other words, different items in the list need to be represented differently. Examples include a Tumblr client where each post might be an image, text or a video. Another example would be Facebook with the many different types of feed items.

<img src="http://i.imgur.com/9cfECVP.png" width="300" />

To implement a heterogenous list of items, most of the work is done **within the adapter**. In particular, there are special methods to be overridden within an adapter such as `getItemViewType`, `getViewTypeCount` specifically for these situations.

## Implementation

Suppose we wanted to implement a heterogenous list of items in an application. Keep in mind that each item in this list should be implemented with a common base model.  The first step therefore is to create a model flexible enough to support all the types (i.e `Post`) and embed a mechanism to determine the type.   Let's pick a simple example such as list of colors in which each color is represented differently:

```java
public class SimpleColor {
	public enum ColorValues {
		RED, BLUE, GREEN
	}

	public SimpleColor(String label, ColorValues color) {
		super();
		this.label = label;
		this.color = color;
	}

	public String label;
	public ColorValues color;

}
```

Here we have an enum and a field that represents the color type. We can also have additional fields and information for each type. Now, we need to create an adapter that supports this heterogenous set of colors each represented differently. The skeleton for this type of heterogenous list is a few key methods such as [getViewTypeCount](http://developer.android.com/reference/android/widget/Adapter.html#getViewTypeCount\(\)) and [getItemViewType](http://developer.android.com/reference/android/widget/Adapter.html#getItemViewType\(int\)):

```java
public class ColorArrayAdapter extends ArrayAdapter<SimpleColor> {
    public ColorArrayAdapter(Context context, ArrayList<SimpleColor> colors) {
        super(context, 0, colors);
    }
    
    // Returns the number of types of Views that will be created by getView(int, View, ViewGroup)
    @Override
    public int getViewTypeCount() {
       // Returns the number of types of Views that will be created by this adapter
       // Each type represents a set of views that can be converted
    }
    
    // Get the type of View that will be created by getView(int, View, ViewGroup) 
    // for the specified item.
    @Override
    public int getItemViewType(int position) {
       // Return an integer here representing the type of View.
       // Note: Integers must be in the range 0 to getViewTypeCount() - 1
    }
     
    // Get a View that displays the data at the specified position in the data set. 
    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
       // View should be created based on the type returned from `getItemViewType(int position)`
       // convertView is guaranteed to be the "correct" recycled type
    }
}
```

For example, here's how the adapter might look for an adapter responsible for creating a list of items that are represented differently based on their color:

```java
public class ColorArrayAdapter extends ArrayAdapter<SimpleColor> {
	public ColorArrayAdapter(Context context, ArrayList<SimpleColor> colors) {
		super(context, 0, colors);
	}
         
        // Return an integer representing the type by fetching the enum type ordinal
	@Override
	public int getItemViewType(int position) {
		return getItem(position).color.ordinal();
	}
        
        // Total number of types is the number of enum values
	@Override
	public int getViewTypeCount() {
		return SimpleColor.ColorValues.values().length;
	}

	@Override
	public View getView(int position, View convertView, ViewGroup parent) {
		// Get the data item for this position
		SimpleColor color = getItem(position);
		// Check if an existing view is being reused, otherwise inflate the view
		if (convertView == null) {
			// Get the data item type for this position
			int type = getItemViewType(position);
			// Inflate XML layout based on the type     
			convertView = getInflatedLayoutForType(type);
		}
		// Lookup view for data population
		TextView tvLabel = (TextView) convertView.findViewById(R.id.tvLabel);
		if (tvLabel != null) {
			// Populate the data into the template view using the data object
			tvLabel.setText(color.label);
		}
		// Return the completed view to render on screen
		return convertView;
	}
        
        // Given the item type, responsible for returning the correct inflated XML layout file
	private View getInflatedLayoutForType(int type) {
		if (type == ColorValues.BLUE.ordinal()) {
			return LayoutInflater.from(getContext()).inflate(R.layout.item_blue_color, null);
		} else if (type == ColorValues.RED.ordinal()) {
			return LayoutInflater.from(getContext()).inflate(R.layout.item_red_color, null);
		} else if (type == ColorValues.GREEN.ordinal()) {
			return LayoutInflater.from(getContext()).inflate(R.layout.item_green_color, null);
		} else {
			return null;
		}
	}
}
```

Note that **once `getItemViewType` is defined**, the `convertView` object is guaranteed to be the "correct" type to ensure that recycling views is relatively straightforward. Now we can attach this together within an activity and populate a ListView with:

```java
public class HeterogenousListActivity extends Activity {
	private ListView lvColors;

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		lvColors = (ListView) findViewById(R.id.lvColors);
		ArrayList<SimpleColor> aColors = new ArrayList<SimpleColor>();
		// Populate colors into the array
		aColors.add(new SimpleColor("Blue", ColorValues.BLUE)); 
		aColors.add(new SimpleColor("Green", ColorValues.GREEN));
		// Attach the adapter
		ColorArrayAdapter adapterColors = new ColorArrayAdapter(this, aColors);
		lvColors.setAdapter(adapterColors);
	}
}
```

Here we've attached the array into an adapter and then populated the ListView as always. The result might look like this:

<img src="http://i.imgur.com/9cfECVP.png" width="450" />

This is a simple example for demonstration purposes. See [this article](https://edisonthk.wordpress.com/2014/06/12/constraints-of-multiple-layouts-listview-for-android/) for a more detailed overview.

### ViewHolder for Performance

As with any list within a production application, we will want to apply the [[ViewHolder Pattern|Using-an-ArrayAdapter-with-ListView#improving-performance-with-the-viewholder-pattern]] for smooth scrolling by caching the subview lookups. In this case, you'll need multiple ViewHolder classes (one per type) as [outlined in this stackoverflow post](http://stackoverflow.com/a/3515221/313399).

## References

 * <https://edisonthk.wordpress.com/2014/06/12/constraints-of-multiple-layouts-listview-for-android/>
 * <http://sparetimedev.blogspot.com/2012/10/recycling-of-views-with-heterogeneous.html>
 * <http://porcupineprogrammer.blogspot.com/2012/09/android-heterogeneous-adapters-gotcha.html>
 * <http://stackoverflow.com/questions/3514548/creating-viewholders-for-listviews-with-different-item-layouts>