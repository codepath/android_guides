### Objective

We're going to create our own implementation of a determinate progress bar by creating a custom view. The progress bar will have an indicator that represents a goal. We will define custom attributes, handle measuring and drawing, and animations. 

![](http://i.imgur.com/rMW3K4A.gif)

Review the [creating a custom view](http://guides.codepath.com/android/Defining-Custom-Views) cliffnotes for a broader overview of what is required in building custom views.

### Prepare app skeleton

Clone [this repository](https://github.com/codepath/android-custom-progress-bar), and import the project in Android Studio. Build and run the app to see the following screen: 

<img src="http://i.imgur.com/n6LW1jL.png" width="300">

The skeleton app contains a `GoalProgressBar` class which extends Android's [ProgressBar](http://developer.android.com/reference/android/widget/ProgressBar.html), but doesn't add any additional functionality. Clicking on the `Reset Progress` button will update the progress to a new value, which will help us test our changes during development. 


### Initialization

We won't need to extend android's `ProgressBar` as we will be drawing the progress manually. Change `GoalProgressBar` to extend `View` instead of `ProgressBar`, and remove the `style` and `android:indeterminate` attributes from our custom view's definition in `activity_main.xml`. 

Our progress bar will have an indicator for a given 'goal'. The goal indicator can appear at any given position on the progress bar, and will have an adjustable size.

Create public setters for an `int progress`, as well as an `int goal`. When either of these methods are called, we'll update a `boolean isGoalReached` method that we'll use to determine which color to draw the progress bar. 

```java
public void setProgress(int progress) {
  this.progress = progress;
  updateGoalReached();
  invalidate();
}

public void setGoal(int goal) {
  this.goal = goal;
  updateGoalReached();
  invalidate();
}

private void updateGoalReached() {
  isGoalReached = progress >= goal;
}
```

### Customizable fields

We're going to expose customizable fields to allow different components in our progress bar. To start out, we'll define member variables for each customizable attribute: 

```java
// height of the goal indicator 
private float goalIndicatorHeight;

// thickness of the goal indicator
private float goalIndicatorWidth;

// bar color when the goal has been reached
private int goalReachedColor;

// bar color when the goal has not been reached
private int goalNotReachedColor;

// bar color for the unfilled section (remaining progress)
private int unfilledSectionColor;

// height of the progress bar
private float barHeight;
```

In order to be able to set these variables from Java code, create setters for these member variables. When any of these setters are called, be sure to call `postInvalidate();` after updating the field so the view is redrawn. 

In order to set these variables from XML, we first have to declare a styleable resource in `res/values/attrs.xml`: 

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
  <declare-styleable name="GoalProgressBar">
    <attr name="goalIndicatorHeight" format="dimension"/>
    <attr name="goalIndicatorWidth" format="dimension"/>
    <attr name="goalReachedColor" format="color"/>
    <attr name="goalNotReachedColor" format="color"/>
    <attr name="unfilledSectionColor" format="color"/>
    <attr name="barHeight" format="dimension"/>
  </declare-styleable>
</resources>
```

The variable names and defined `<attr` names do not have to be the same, but are doing so here for the sake of consistency. 

Next we'll add these new attributes to our existing `GoalProgressBar` in `activity_main.xml`. Before we can start defining custom attributes, we need to add an additional XML namespace on the root element: 
`xmlns:app="http://schemas.android.com/apk/res-auto"`

Once the new `app` namespace is defined (we can call it anything, it doesn't have to be `app`) we can start adding these attributes to our `GoalProgressBar`: 

```xml
<com.codepath.customprogressbar.GoalProgressBar
    android:id="@+id/progressBar"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    app:goalReachedColor="@color/green"
    app:goalNotReachedColor="@color/dark_gray"
    app:unfilledSectionColor="@color/gray"
    app:barHeight="4dp"
    app:goalIndicatorHeight="16dp"
    app:goalIndicatorWidth="4dp"/>
```

Create a new method for initialization to be called from the constructor of our `View`: 
```java
private void init(AttributeSet attrs) {
   // extract our custom attributes
}
```

We will extract our custom attribute values from the given [AttributeSet](http://developer.android.com/reference/android/util/AttributeSet.html). 

To extract the attributes from the `AttributeSet`, we'll use a [TypedArray](http://developer.android.com/reference/android/content/res/TypedArray.html). A `TypedArray` is basically a container to hold onto defined attributes. This object must be recycled once we're done using it, so we'll interact with it in a `try` block so we can be sure to call `recycle()` on it once we're finished: 
```java
TypedArray typedArray = getContext().getTheme().obtainStyledAttributes(attrs, R.styleable.GoalProgressBar, 0, 0);
try {
  // extract attributes to member variables from typedArray
} finally {
  typedArray.recycle(); 
}
``` 

Extract each of our defined attributes from the typed array by providing the `styleable` ID to the `typedArray`. Be sure to set each member variable via it's setter for consistent behavior: 
```java
setGoalReachedColor(typedArray.getColor(R.styleable.GoalProgressBar_goalReachedColor, Color.BLUE));
```

### Measuring 

Now that we have the framework in place for setting the required fields, we can implement the measuring logic. To do this we will override `onMeasure`, so we can tell our parent view what size we'd like the view to be. 

For the width of the view, we can use whatever width imposed by our parent (provided in `widthMeasureSpec`), but we'll add some custom handling for determining our height. 

Since the tallest component of our view will be the goal indicator, we'll use it's height to determine the height of the entire view: 

```java 
@Override
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
  // set width
  int width = MeasureSpec.getSize(widthMeasureSpec);

  // set height
  int heightSize = MeasureSpec.getSize(heightMeasureSpec);
  int height;
  switch (MeasureSpec.getMode(heightMeasureSpec)) {
    case MeasureSpec.EXACTLY:
      // we must be exactly the given size 
      height = heightSize;
      break;
    case MeasureSpec.AT_MOST:
      // we can not be bigger than the specified height
      height = (int) Math.min(goalIndicatorHeight, heightSize);
      break;
    default:
      // we can be whatever height want
      height = (int) goalIndicatorHeight;
      break;
  }

  setMeasuredDimension(width, height);
}
```

This will properly handle the sizing of our view, even when we allow a customizable sized goal indicator. 

### Drawing the View

Now our view has set it's appropriate height and width, we're ready to draw the content of the view to the screen. We do this by overriding [onDraw](http://developer.android.com/reference/android/view/View.html#onDraw(android.graphics.Canvas)). 

In order to draw the progress we'll need to use a Paint object. Define a new paint object that is initialized in `init`: 
```java
private Paint progressPaint;

...

private void init(AttributeSet attrs) {
  progressPaint = new Paint();
  progressPaint.setStyle(Paint.Style.FILL_AND_STROKE);
}
```

It's important that the `Paint` is only created once, as reinitializing the object each time it is used in drawing would affect performance. 

`onDraw` provides a `Canvas` onto which we can use our `Paint` to draw. We'll start with three different components to draw, so we'll have three separate calls to draw onto the `Canvas`. 

We need to draw the section of the progress bar that is 'filled' - so if the maximum is set to 100, and `progress` is set to 70, we'll the filled section will be drawn to 7/10 the width. The color of this section will depend on whether or not `progress` meets/exceeds our `goal`. 

We'll also have to draw the remaining/unfilled section of the progress bar, which in the case of 70% progress would be 3/10 of the width, starting where the filled section left off. 

We also have to draw the 'goal' indicator, which will be at a position defined by the user. 

```java
@Override
protected void onDraw(Canvas canvas) {
  int halfHeight = getHeight() / 2;
  int progressEndX = (int) (getWidth() * progress / 100f);

  // draw the part of the bar that's filled
  progressPaint.setStrokeWidth(barHeight);
  progressPaint.setColor(isGoalReached ? goalReachedColor : goalNotReachedColor);
  canvas.drawLine(0, halfHeight, progressEndX, halfHeight, progressPaint);

  // draw the unfilled section
  progressPaint.setColor(unfilledSectionColor);
  canvas.drawLine(progressEndX, halfHeight, getWidth(), halfHeight, progressPaint);

  // draw the goal indicator
  float indicatorPosition = getWidth() * goal / 100f;
  progressPaint.setColor(goalReachedColor);
  progressPaint.setStrokeWidth(goalIndicatorWidth);
  canvas.drawLine(indicatorPosition, halfHeight - goalIndicatorHeight / 2,
                indicatorPosition, halfHeight + goalIndicatorHeight / 2, progressPaint);
}
```

### Custom Goal Indicator

Next we're going to create some different options for the goal indicator. We can allow users to choose between different shapes to use as the goal indicator. We'll do this by creating a new XML attribute that has a predefined set of values, to make the API as user friendly as possible:
 
<img src="http://i.imgur.com/M3yEzH9.png" width="300">

We'll need to add a new `attr` to our existing declared styleable in `attrs.xml`, this time it will be of type `enum`: 

```xml
<attr name="indicatorType" format="enum">
    <enum name="line" value="0"/>
    <enum name="circle" value="1"/>
    <enum name="square" value="2"/>
</attr>
```

Create an `enum` in `GoalProgressBar` with the same values as we defined in the new styleable: 

```java
public enum IndicatorType {
  Line, Circle, Square
} 
```

Add a member variable of this new enum, and allow it to be set via a public setter. Use this setter in extracting the attribute from our `typedArray`:

```java
setIndicatorType(IndicatorType.values()[typedArray.getInt(R.styleable.GoalProgressBar_indicatorType, IndicatorType.Line.ordinal())]);
```

Now we'll need to add logic to `onDraw` to draw the differently styled indicator. Use a `switch` on the `indicatorType` to determine how to draw the goal indicator: 

```java
// draw the goal indicator
float indicatorPosition = getIndicatorPosition();
progressPaint.setColor(goalReachedColor);
progressPaint.setStrokeWidth(goalIndicatorWidth);
switch (indicatorType) {
  case Line:
    canvas.drawLine(indicatorPosition, 
                    halfHeight - goalIndicatorHeight / 2,
                    indicatorPosition, 
                    halfHeight + goalIndicatorHeight / 2, 
                    progressPaint);
    break;
  case Circle:
    canvas.drawCircle(indicatorPosition, 
                      goalIndicatorHeight / 2, 
                      goalIndicatorHeight / 2, 
                      progressPaint);
    break;
  case Square:
    canvas.drawRect(indicatorPosition - (goalIndicatorHeight / 2), 
                    0, 
                    indicatorPosition + (goalIndicatorHeight / 2), 
                    goalIndicatorHeight, 
                    progressPaint);
    break;
}
```

### Saving Instance State

In order to maintain state across screen rotation, we'll need to handle saving and restoring instance state. To do so we'll override [onSaveInstanceState](http://developer.android.com/reference/android/view/View.html#onSaveInstanceState()) and [onRestoreInstanceState](http://developer.android.com/reference/android/view/View.html#onRestoreInstanceState(android.os.Parcelable)): 

```java
@Override
public Parcelable onSaveInstanceState() {
  Bundle bundle = new Bundle();

  // save our progress
  bundle.putInt("progress", progress);

  // be sure to save all other instance state that may exist
  bundle.putParcelable("superState", super.onSaveInstanceState());

  return bundle;
}

@Override
public void onRestoreInstanceState(Parcelable state) {
  if (state instanceof Bundle) {
    Bundle bundle = (Bundle) state;

    // restore our progress
    setProgress(bundle.getInt("progress"));

    // restore all other state
    state = bundle.getParcelable("superState");
  }
  super.onRestoreInstanceState(state);
}
```

### Animation

To make our progress bar animate when set, we'll overload `setProgress` to take an additional boolean flag for whether or not to animate filling of the progress bar. 

We'll define a new member `ValueAnimator barAnimator;` to animate our progress value. When `setProgress` is called, and the flag for animation is set, we'll initialize and start the animator with the new progress value: 

```java
barAnimator = ValueAnimator.ofFloat(0, 1);

// animate over the course of 700 milliseconds
barAnimator.setDuration(700);

// reset progress to zero so it can animate up
setProgress(0, false);

// use a decelerate interpolator which will slow down towards the end of animation
barAnimator.setInterpolator(new DecelerateInterpolator());

// call setProgress (without animating) to 
// update the progress and invalidate the view
barAnimator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
  @Override
    public void onAnimationUpdate(ValueAnimator animation) {
      float interpolation = (float) animation.getAnimatedValue();
      setProgress((int) (interpolation * progress), false);
  }
});

if (!barAnimator.isStarted()) {
  barAnimator.start();
}
```


### Full Implementation
To view the the complete implementation of this custom view, see [this branch](https://github.com/codepath/android-custom-progress-bar/tree/eric/solution). 