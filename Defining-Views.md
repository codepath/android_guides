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

## References

 * ...