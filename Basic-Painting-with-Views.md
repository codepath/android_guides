## Creating our Custom View

Create a simple class for drawing that extends View called `SimpleDrawingView`:

```java
public class SimpleDrawingView extends View {
  public SimpleDrawingView(Context context, AttributeSet attrs) {
    super(context, attrs);
  }
}
```

Add this to the XML layout for activity so our custom view is embedded within:

```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity" >

    <com.codepath.example.simpledrawapp.SimpleDrawingView
        android:id="@+id/simpleDrawingView1"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentBottom="true"
        android:layout_alignParentLeft="true"
        android:layout_alignParentRight="true"
        android:layout_alignParentTop="true" />

</RelativeLayout>
```

### Simple Drawing with Canvas

Let's try drawing a couple of circle on screen. This requires us to define a `Paint` object which controls the styling and color of what is drawn. Let's start by preparing the paint:

```java
public class SimpleDrawingView extends View {
  // setup initial color
  private final int paintColor = Color.BLACK;
  // defines paint and canvas
  private Paint drawPaint;

  public SimpleDrawingView(Context context, AttributeSet attrs) {
    super(context, attrs);
    setFocusable(true);
    setFocusableInTouchMode(true);
    setupPaint();
  }
  
  // Setup paint with color and stroke styles
  private void setupPaint() {
    drawPaint = new Paint();
    drawPaint.setColor(paintColor);
    drawPaint.setAntiAlias(true);
    drawPaint.setStrokeWidth(5);
    drawPaint.setStyle(Paint.Style.STROKE);
    drawPaint.setStrokeJoin(Paint.Join.ROUND);
    drawPaint.setStrokeCap(Paint.Cap.ROUND);
  }
}
```

Now that we have the paint setup to have a black color and configured a particular stroke style, let's try to draw a few circles with different colors. All drawing that happens in a view should take place within the `onDraw` method which is automatically called when a view is rendered:

```java
public class SimpleDrawingView extends View {
    // ...variables and setting up paint... 
    // Let's draw three circles
    @Override
    protected void onDraw(Canvas canvas) {
      canvas.drawCircle(50, 50, 20, drawPaint);
      drawPaint.setColor(Color.GREEN);
      canvas.drawCircle(50, 150, 20, drawPaint);
      drawPaint.setColor(Color.BLUE);
      canvas.drawCircle(50, 250, 20, drawPaint);
    }
}
```

Notice that `onDraw` passes us a `canvas` object which we use to draw leveraging the `Paint` we defined earlier. The `drawCircle` method accepts the x, y and radius of the circle in addition to the paint. This renders the following:

![Circle](http://i.imgur.com/fK8Vfuz.png)

### Handling Touch Interactions

Suppose now we wanted to draw a circle every time the user touches down on the drawing view. This would require us to keep track of an array of points for our circles and then append a point for each `onTouch` event triggered:

```java
public class SimpleDrawingView extends View {
  // setup initial color
  private final int paintColor = Color.BLACK;
  // defines paint and canvas
  private Paint drawPaint;
  // Store circles to draw each time the user touches down
  private List<Point> circlePoints;

  public SimpleDrawingView(Context context, AttributeSet attrs) {
    super(context, attrs);
    setupPaint(); // same as before
    circlePoints = new ArrayList<Point>();
  }

  // Draw each circle onto the view
  @Override
  protected void onDraw(Canvas canvas) {
    for (Point p : circlePoints) {
      canvas.drawCircle(p.x, p.y, 5, drawPaint);
    }
  }

  // Append new circle each time user presses on screen
  @Override
  public boolean onTouchEvent(MotionEvent event) {
    float touchX = event.getX();
    float touchY = event.getY();
    circlePoints.add(new Point(Math.round(touchX), Math.round(touchY)));
    // redraw
    invalidate();
    return true;
  }

  private void setupPaint() {
    // same as before
    drawPaint.setStyle(Paint.Style.FILL); // change to fill
    // ...
  }
}
```

with this, a black circle is drawn each time we press down:

![Circle 2](http://i.imgur.com/MmxcjsV.png)

### Drawing with Paths

So far we have explored the `onDraw` method of a view and we were able to draw circles onto the view based on touch interactions with the view. Next, let's improve our drawing application by removing the list of circles and instead drawing with paths. The [Path](http://developer.android.com/reference/android/graphics/Path.html) class is ideal for allowing the user to draw on screen. A path can contain many lines, contours and even other shapes. First, let's add a `Path` variable to track our drawing:

```java
public class SimpleDrawingView extends View {
  // ...
  private Path path = new Path();
  // ...
}
```

Next, let's append points to the path as the user touches the screen. When the user presses down, let's start a path and then when they drag let's connect the points together. To do this, we need modify the `onTouchEvent` to append these points to our Path object:

```java
public class SimpleDrawingView extends View {
    private Path path = new Path();

    // Get x and y and append them to the path
    public boolean onTouchEvent(MotionEvent event) {
        float pointX = event.getX();
        float pointY = event.getY();
        // Checks for the event that occurs
        switch (event.getAction()) {
        case MotionEvent.ACTION_DOWN:
            // Starts a new line in the path
            path.moveTo(pointX, pointY);
            return true;
        case MotionEvent.ACTION_MOVE:
            // Draws line between last point and this point
            path.lineTo(pointX, pointY);
            break;
        default:
            return false;
        }
        // Forces a view to redraw
        postInvalidate();
        return false;
    }

   // ...
}
```

and then let's alter the `onDraw` to remove the circles and instead to render the lines we have plotted in our path:

```java
public class SimpleDrawingView extends View {
  // ... onTouchEvent ...

  // Draws the path created during the touch events
  @Override
  protected void onDraw(Canvas canvas) {
      canvas.drawPath(path, drawPaint);
  }

  private void setupPaint() {
    // same as before
    drawPaint.setStyle(Paint.Style.STROKE); // change back to stroke
    // ...
  }
}
```

and with that, we have a very basic painting app that looks like:

![Circle 3](http://i.imgur.com/RixrXoy.png)

You can also check out the [full source code for SimpleDrawingView](https://gist.github.com/nesquena/a1df61dce24b636edb0f) for reference.