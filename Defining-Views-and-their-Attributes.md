## Overview

Defining view components within a layout is a critical part of Android development. Let's take a look at common views as well as common attributes of views.

### Common Views

The six most common views are:

 * **TextView** displays a formatted text label
 * **ImageView** displays an image resource
 * **Button** can be clicked to perform an action
 * **ImageButton** displays a clickable image
 * **EditText** is an editable text field for user input
 * **ListView** is a scrollable list of items containing other views

### View Identifiers

Any view can have an identifier attached that uniquely names that view for later access. You can assign a view an id within the XML layout:

```xml
<Button android:id="@+id/my_button" />
```

This id can then be accessed within the Java code for the corresponding activity (in `onCreate` of Activity for example):

```java
Button myButton = (Button) findViewById(R.id.my_button);
```

Another important note is that any view with an id specified will automatically retain its state on a configuration change (i.e phone orientation switch).

### View Height and Width

Every view has height and width properties controlling the size of the view. Height and width have to be defined in the XML for every view with:

```xml
<TextView
  android:layout_width="165dp" 
  android:layout_height="wrap_content" />
```

This can take the form of `wrap_content` (adjust height and width to the content size), `match_parent` (adjust height and width to the full size of the parent container), and a dimensions value such as `120dp`. This can be changed at runtime in a number of ways:

```java
// Change the width or height
int newInPixels = 50;
view.setLayoutParams(new LayoutParams(newInPixels, newInPixels));
// Trigger invalidation of the view to force adjustment
view.requestLayout();
```

Or we can change just the width or height individually:

```java
int newDimensionInPixels = 50;
view.getLayoutParams().width = newDimensionInPixels;
view.getLayoutParams().height = newDimensionInPixels;
// Trigger invalidation of the view to force adjustment
view.requestLayout();
```

We can also set these dimensions in `dp` rather than pixels with:

```java
int newDimensionInPixels = 50;
// convert to 50dp
int dimensionInDp = (int) TypedValue.applyDimension(TypedValue.COMPLEX_UNIT_DIP, newDimensionInPixels, 
    getResources().getDisplayMetrics());
view.getLayoutParams().width = newDimensionInPixels;
view.getLayoutParams().height = dimensionInDp;
// Trigger invalidation of the view to force adjustment
view.requestLayout();
```

### Views Margin and Padding

Margins and padding values for views allows us to position and space elements in a layout.

* **Layout Margin** defines the amount of space around the outside of a view
* **Padding** defines the amount of space around the contents or children of a view.

An example of setting margins and padding:

```xml
<LinearLayout>
   <TextView android:layout_margin="5dp" android:padding="5dp">
   <Button layout_marginBottom="5dp">
</LinearLayout>
```

### View Gravity

Gravity can be used to define the direction of the contents of a view.

* **gravity** determines the direction that the contents of a view will align (like CSS text-align).
* **layout_gravity** determines the direction of the view within it's parent (like CSS float).

An example of applying gravity:

```xml
<TextView
  android:gravity="left"
  android:layout_gravity="right"
  android:layout_width="165dp" android:layout_height="wrap_content"
  android:textSize="12sp" android:text="@string/hello_world" />
```

### View Attributes

Every view has many different attributes which can be applied to manage various properties.

* Certain properties are shared across many views such as `android:layout_width`
* Other properties are based on a view's function such as `android:textColor`

Common view attributes include:

| Attribute             | Description                     | Example Value     |
| ---------             | ------------                    | ------------      |
| `android:background`  | Background for the view         | `#ffffff`         |
| `android:onClick`     | Method to invoke when clicked   | `onButtonClicked` | 
| `android:visibility`  | Controls how view appears       | `invisible`       |
| `android:hint`        | Hint text to display when empty | `@string/hint`    |
| `android:text`        | Text to display in view         | `@string/foo`     |
| `android:textColor`   | Color of the text               | `#000000`         |
| `android:textSize`    | Size of the text                | `21sp`            |
| `android:textStyle`   | Style of the text formatting    | `bold`            |

Review the [View docs](http://developer.android.com/reference/android/view/View.html) and [TextView docs](http://developer.android.com/reference/android/widget/TextView.html) for a list of additional attributes. An example of setting view attributes:

```xml
<TextView
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"
   android:text="@string/hello_world"
   android:background="#000"
   android:textColor="#fff"
   android:layout_centerHorizontal="true"
/>
```

## References

 * <http://developer.android.com/reference/android/view/View.html>
 * <http://developer.android.com/reference/android/widget/TextView.html>
 * <http://mobile.tutsplus.com/tutorials/android/android-sdk-user-interface-design/>