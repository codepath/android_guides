## Overview

Android UI elements are all based on `View` (single element on screen) and `ViewGroup` (collection of elements on screen). There are many "widgets" and "layouts" built-in that can be used to build the UI such as views like [Button](http://developer.android.com/reference/android/widget/Button.html) and [TextView](http://developer.android.com/reference/android/widget/TextView.html), and layouts like [RelativeLayout](http://developer.android.com/reference/android/widget/RelativeLayout.html)

In some apps though we need to be able to customize views to suit our own needs. This might mean extending an existing view, creating your own [View](http://developer.android.com/reference/android/view/View.html) subclass or doing more complicated drawing with a [SurfaceView](http://developer.android.com/reference/android/view/SurfaceView.html).

Customizing your own views involves extending `View` or an existing subclass, overriding the view behavior by writing methods such as `onDraw` or `onTouchEvent` and then using your new view in an activity.

## Creating Fully Custom Components

Creating custom views is centered around five primary aspects that we may need to control or modify:

 * [Drawing](http://developer.android.com/training/custom-views/custom-drawing.html) - Control the rendering of the view on screen visually by overriding the <a href="http://developer.android.com/reference/android/view/View.html#onDraw(android.graphics.Canvas)">onDraw</a> method. 
 * [Interaction](http://developer.android.com/training/custom-views/making-interactive.html) - Control the ways the user can interact with the view with the <a href="http://developer.android.com/reference/android/view/View.html#onTouchEvent(android.view.MotionEvent)">onTouchEvent</a> and gestures.
 * [Measurement](http://developer.android.com/guide/topics/ui/custom-components.html#custom) - Control the content dimensions of the view on screen by overriding the <a href="http://developer.android.com/reference/android/view/View.html#onMeasure(int, int)">onMeasure</a> method.
 * [Attributes](http://developer.android.com/training/custom-views/create-view.html#customattr) - Defining custom XML attributes for your view and using them to control behavior with [TypedArray](http://developer.android.com/reference/android/content/res/TypedArray.html)
 * [Persistence](http://stackoverflow.com/questions/3542333/how-to-prevent-custom-views-from-losing-state-across-screen-orientation-changes) - Storing and restoring state on configuration changes to avoid losing the state with [onSaveInstanceState](http://developer.android.com/reference/android/view/View.html#onSaveInstanceState\(\)) and [onRestoreInstanceState](http://developer.android.com/reference/android/view/View.html#onRestoreInstanceState(android.os.Parcelable))

To take a closer look, suppose we want to **create our own view control that allows the user to select between different shapes**. The view will display a single shape (square, circle or triangle) and clicking on the view will toggle the shape selected between the different options.

![ShapeView 1](http://i.imgur.com/WrZx6wE.png)

### Define Custom View Class

To create our own custom toggle-able shape selector from scratch, we start by defining a `ShapeSelectorView` which extends from `View` and implements the required constructor:

```java
public class ShapeSelectorView extends View {
  // We must provide a constructor that takes a Context and an AttributeSet.
  // This constructor allows the UI to create and edit an instance of your view.
  public ShapeSelectorView(Context context, AttributeSet attrs) {
        super(context, attrs);
    }
}
```

### Add View to Layout

Next, let's add this view to our activity layout along with a caption and a button for selecting the shape after choosing:

```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent" >
    <com.codepath.example.customviewdemo.ShapeSelectorView
        android:id="@+id/shapeSelector"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_alignParentRight="true"
        android:layout_alignParentTop="true"
        android:layout_alignParentLeft="true" />
</RelativeLayout>
```

Note how we define a custom namespace `app`.  This namespace allows you to allow Android to auto-resolve the namespace, avoiding the necessity for specifying the package name in this file.  See this [blog post](http://slodge.blogspot.com/2013/04/auto-resource-namespace-resolution-for.html) for more information.

### Define Custom Attributes

Well-written custom views can be configured and styled via XML attributes. You need to ask yourself which aspects of your view should be customizable. For example, we might want to let the user select the **color of the shape** as well as give the user the option to **display the name of the shape** in the view as well. We might want the view to be configurable in XML as follows:

```xml
<com.codepath.example.customviewdemo.ShapeSelectorView
    app:shapeColor="#7f0000"
    app:displayShapeName="true"
    android:id="@+id/shapeSelector"
    ... />
```

In order to be able to define `shapeColor` and `displayShapeName`, we need to define these as attributes within `res/values/attrs.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
   <declare-styleable name="ShapeSelectorView">
       <attr name="shapeColor" format="color" />
       <attr name="displayShapeName" format="boolean" />
   </declare-styleable>
</resources>
```

Notice we define the `attr` node along with the `name` and `format` for each custom attribute we'd like to be able to define. The `format` is the expected type of value for that property and valid options include string, color, dimension, boolean, integer, float, enum, and [several others](http://chaosinmotion.com/blog/?p=179).

Once you define the custom attributes, you can use them in layout XML files just like built-in attributes. The only difference is that your custom attributes belong to a different namespace. You can define the namespace within the root view of the layout and configure the properties for the view.  Normally you would need to specify a namespace such as `http://schemas.android.com/apk/res/<package_name>` (i.e. 
com.codepath.example.customviewdemo) but the namespace `http://schemas.android.com/apk/res-auto` will auto-resolve for you.

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent" >
    <com.codepath.example.customviewdemo.ShapeSelectorView
           app:shapeColor="#7f0000"
           app:displayShapeName="true"
           android:id="@+id/shapeSelector"
           android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:layout_above="@+id/btnSelect"
           android:layout_alignParentLeft="true"
           android:layout_below="@+id/tvPrompt" />
</RelativeLayout>
```

### Apply Custom Attributes

Now that we have set custom properties such as `shapeColor` and `displayShapeName`, we need to extract those properties to be used within our custom View within the constructor. To extract the custom attributes, we can use a `TypedArray` and the `obtainStyledAttributes` on the `AttributeSet`:

```java
public class ShapeSelectorView extends View {
  private int shapeColor;
  private boolean displayShapeName;

  public ShapeSelectorView(Context context, AttributeSet attrs) {
    super(context, attrs);
    setupAttributes(attrs);
  }

  private void setupAttributes(AttributeSet attrs) {
    // Obtain a typed array of attributes
    TypedArray a = getContext().getTheme().obtainStyledAttributes(attrs, R.styleable.ShapeSelectorView, 0, 0);
    // Extract custom attributes into member variables
    try {
      shapeColor = a.getColor(R.styleable.ShapeSelectorView_shapeColor, Color.BLACK);
      displayShapeName = a.getBoolean(R.styleable.ShapeSelectorView_displayShapeName, false);
    } finally {
      // TypedArray objects are shared and must be recycled.
      a.recycle();
    }
  }
}
```

### Add Property Methods

Let's expose property methods to allow us to get and set the important properties after a view has been created:

```java
public class ShapeSelectorView extends View {
  // ...
  public boolean isDisplayingShapeName() {
    return displayShapeName;
  }

  public void setDisplayingShapeName(boolean state) {
    this.displayShapeName = state;
    invalidate();
    requestLayout();
  }

  public int getShapeColor() {
    return shapeColor;
  }

  public void setShapeColor(int color) {
    this.shapeColor = color;
    invalidate();
    requestLayout();
  }
}
```

Notice that when the view properties are changed and might require a redraw, be sure to call `invalidate()` and `requestLayout()` to update the appearance.

### Drawing a Shape

Next, let's actually draw a square taking into account the properties defined above for shape color and shape name. All view drawing happens within the `onDraw` method using the `Canvas` object to paint onto the view. Let's draw a square shape:

```java
public class ShapeSelectorView extends View {
  // ...
  private int shapeWidth = 100;
  private int shapeHeight = 100;
  private int textXOffset = 0;
  private int textYOffset = 30;
  private Paint paintShape;

  // ...
  public ShapeSelectorView(Context context, AttributeSet attrs) {
    super(context, attrs);
    setupAttributes(attrs);
    setupPaint();
  }

  @Override
  protected void onDraw(Canvas canvas) {
    super.onDraw(canvas);
    canvas.drawRect(0, 0, shapeWidth, shapeHeight, paintShape);
    if (displayShapeName) {
      canvas.drawText("Square", shapeWidth + textXOffset, shapeHeight + textXOffset, paintShape);
    }
  }

  private void setupPaint() { 
      paintShape = new Paint();
      paintShape.setStyle(Style.FILL);
      paintShape.setColor(shapeColor);
      paintShape.setTextSize(30);
   }
}
```

This will paint the square based on the `shapeColor` specified in the XML and will paint the shape name if specified within the `displayShapeName` property. Result looks like:

![ShapeView 1](http://i.imgur.com/CUJnh0b.png)

You can read more about drawing onto a canvas on the official [Custom 2D Drawing Tutorial](http://developer.android.com/guide/topics/graphics/2d-graphics.html).

### Calculating the Dimensions

In order to understand the width and height of a view that is being custom drawn, we need to define the `onMeasure` method which determines the width and height of the view based on it's contents. In this case, the height and width are determined by the shape and text drawn within the view. Let's define the `onMeasure` as follows:

```java
public class ShapeSelectorView extends View {
  @Override
  protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    // Defines the extra padding for the shape name text
    int textPadding = 10;
    int contentWidth = shapeWidth;
    
    // Resolve the width based on our minimum and the measure spec
    int minw = contentWidth + getPaddingLeft() + getPaddingRight();
    int w = resolveSizeAndState(minw, widthMeasureSpec, 0);
    
    // Ask for a height that would let the view get as big as it can
    int minh = shapeHeight + getPaddingBottom() + getPaddingTop();
    if (displayShapeName) { 
        minh += textYOffset + textPadding;
    }
    int h = resolveSizeAndState(minh, heightMeasureSpec, 0);
    
    // Calling this method determines the measured width and height
    // Retrieve with getMeasuredWidth or getMeasuredHeight methods later
    setMeasuredDimension(w, h);
  }
}
```

Note that the calculations take into account the view's padding and calculate the content size. Also note that the `onMeasure` method must call the <a href="http://developer.android.com/reference/android/view/View.html#setMeasuredDimension(int, int)">setMeasuredDimension</a>. Widths and heights are discussed using the <a href="http://developer.android.com/reference/android/view/View.MeasureSpec.html">MeasureSpec</a> which encapsulates all the different types of constraints imposed by the parent layout for a view. The helper method <a href="http://developer.android.com/reference/android/view/View.html#resolveSizeAndState(int, int, int)">resolveSizeAndState()</a> returns an appropriate value by comparing the view's desired size to the spec passed into the method.

### Toggling Between Shapes

We have the square drawing, but we want the shape to toggle each time the view is clicked. Let's setup a touch handler to ensure the shape changes as specified using the `onTouchEvent` method:

```java
public class ShapeSelectorView extends View {
  // ...
  private String[] shapeValues = { "square", "circle", "triangle" };
  private int currentShapeIndex = 0;

  // Change the currentShapeIndex whenever the shape is clicked
  @Override
  public boolean onTouchEvent(MotionEvent event) {
    boolean result = super.onTouchEvent(event);
    if (event.getAction() == MotionEvent.ACTION_DOWN) {
      currentShapeIndex ++;
      if (currentShapeIndex > (shapeValues.length - 1)) {
        currentShapeIndex = 0;
      }
      postInvalidate();
      return true;
    }
    return result;
  }
}
```

Now whenever the shape is clicked, the selected shape index will change and a different shape should be drawn after `postInvalidate` is called. Let's update the `onDraw` method to paint the correct shape according to the selected index:

```java
public class ShapeSelectorView extends View {
  // ...

  @Override
  protected void onDraw(Canvas canvas) {
    super.onDraw(canvas);
    String shapeSelected = shapeValues[currentShapeIndex];
    if (shapeSelected.equals("square")) {
      canvas.drawRect(0, 0, shapeWidth, shapeHeight, paintShape);
      textXOffset = 0;
    } else if (shapeSelected.equals("circle")) {
      canvas.drawCircle(shapeWidth / 2, shapeHeight / 2, shapeWidth / 2, paintShape);
      textXOffset = 12;
    } else if (shapeSelected.equals("triangle")) {
      canvas.drawPath(getTrianglePath(), paintShape);
      textXOffset = 0;
    }
    if (displayShapeName) {
      canvas.drawText(shapeSelected, 0 + textXOffset, shapeHeight + textYOffset, paintShape);
    }
  }

  protected Path getTrianglePath() {
    Point p1 = new Point(0, shapeHeight), p2 = null, p3 = null;
    p2 = new Point(p1.x + shapeWidth, p1.y);
    p3 = new Point(p1.x + (shapeWidth / 2), p1.y - shapeHeight);
    Path path = new Path();
    path.moveTo(p1.x, p1.y);
    path.lineTo(p2.x, p2.y);
    path.lineTo(p3.x, p3.y);
    return path;
  }

  // ...
}
```

Now every time that we click the view, a different shape appears rotating between the three available options. Result looks like:

![ShapeView 2](http://i.imgur.com/uJnup7s.png)

For more advanced view user interaction, check out the [Making the View Interactive](http://developer.android.com/training/custom-views/making-interactive.html) official docs.

### Finishing The View

The final touch might be to add a property to allow the activity to access the selected shape from within the view. First, let's add the method to expose the selected shape:

```java
public class ShapeSelectorView extends View {
  // ...
  // Returns selected shape name
  public String getSelectedShape() {
    return shapeValues[currentShapeIndex];
  }
}
```

and then now within the activity, we might be able to display the selected shape with a toast when a button is pressed:

```java
public class MainActivity extends Activity {
  private Button btnSelect;
  private ShapeSelectorView shapeSelector;

  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    shapeSelector = (ShapeSelectorView) findViewById(R.id.shapeSelector);
    btnSelect = (Button) findViewById(R.id.btnSelect);
    btnSelect.setOnClickListener(new OnClickListener() {
      @Override
      public void onClick(View v) {
        Toast.makeText(MainActivity.this, "You selected: " +
             shapeSelector.getSelectedShape(), Toast.LENGTH_LONG).show();
      }
    });
  }
}
```

The result of this is the following:

![Shape 3](http://i.imgur.com/c0qMRel.png)

There are many events which can be customized for a view, check out the [Custom Components](http://developer.android.com/guide/topics/ui/custom-components.html#custom) guide for a more details.

### Saving View Instance State

Views are responsible for maintaining their own state when configuration changes (i.e phone is rotated) occur. You can do this by implementing [View#onSaveInstanceState](http://developer.android.com/reference/android/view/View.html#onSaveInstanceState%28%29) and [View#onRestoreInstanceState](http://developer.android.com/reference/android/view/View.html#onRestoreInstanceState%28android.os.Parcelable%29) in order to save and then restore the view state. For example, to maintain the selected shape index for our shape selector:

```java
public class ShapeSelectorView extends View {
  // This is the view state for this shape selector
  private int currentShapeIndex = 0;

  @Override
  public Parcelable onSaveInstanceState() {
    // Construct bundle
    Bundle bundle = new Bundle();
    // Store base view state
    bundle.putParcelable("instanceState", super.onSaveInstanceState());
    // Save our custom view state to bundle
    bundle.putInt("currentShapeIndex", this.currentShapeIndex);
    // ... store any other custom state here ...
    // Return the bundle
    return bundle;
  }

  @Override
  public void onRestoreInstanceState(Parcelable state) {
    // Checks if the state is the bundle we saved
    if (state instanceof Bundle) {
      Bundle bundle = (Bundle) state;
      // Load back our custom view state
      this.currentShapeIndex = bundle.getInt("currentShapeIndex");
      // ... load any other custom state here ...
      // Load base view state back
      state = bundle.getParcelable("instanceState");
    }
    // Pass base view state on to super
    super.onRestoreInstanceState(state);
  }
}
```

Once you've defined these saving and restoring methods, your view will be capable of automatically persisting state when configuration changes occur.

## Extending Existing Views

There is an even easier option for creating a custom View which is useful in certain circumstances. If there is a component that is already very similar to what you want, you can simply extend that component and just override the behavior that you want to change and get the rest of the behavior for free.

**Incomplete: Fill this in with a relevant example**

Read more about this in the [Extending View Types](http://developer.android.com/guide/topics/ui/custom-components.html#modifying) guide for more details.
 
## Creating Compound Views

If you don't want to create a completely customized component, but instead are looking to put together a reusable component that consists of a group of existing controls, then you may want to simply create a compound control. You might also want to create your own `ViewGroup` to act as a container for views or create a custom layout.

**Incomplete: Fill this in with a relevant example**

Read more about this in the [Custom Compound Components](http://developer.android.com/guide/topics/ui/custom-components.html#compound) guide for more details.

## Libraries

There are many libraries for Android that contain custom views such as:

* [Caldroid](https://github.com/roomorama/Caldroid) - A better calendar widget
* [PullToRefresh-ListView](https://github.com/erikwt/PullToRefresh-ListView) - Pull to refresh enabled ListView
* [RoundedImageView](https://github.com/vinc3m1/RoundedImageView) - ImageView extension that rounds the image with a border

## References

 * <http://developer.android.com/guide/topics/ui/custom-components.html>
 * <http://developer.android.com/training/custom-views/index.html>
 * <http://www.vogella.com/articles/AndroidCustomViews/article.html>
 * <http://danielnadeau.blogspot.com/2013/02/creating-custom-views-on-android.html>
 * <http://mobile.tutsplus.com/tutorials/android/android-sdk-creating-custom-views-2/>
 * <http://arpitonline.com/blog/2012/07/01/creating-custom-layouts-for-android/>
 * <http://javatechig.com/android/creating-custom-and-compound-views-in-android-tutorial>
 * <http://staticallytyped.wordpress.com/2011/04/16/android-custom-xml-attributes-and-views/>
 * <http://www.jayway.com/2012/12/12/creating-custom-android-views-part-4-measuring-and-how-to-force-a-view-to-be-square/>
 * <https://www.dropbox.com/s/1l55125fnuhdvti/Custom%20Views%20-%20Arun.pdf?dl=0>