### Overview


### Objective
We're going to create our own implementation of a determinate progress bar. The progress bar will have an indicator that represents a goal. We will define custom attributes, handle measuring and drawing, and animations. 


### Prepare app skeleton
Clone [this repository](https://github.com/codepath/android-custom-progress-bar), and import the project in Android Studio. Build and run the app to see the following screen: 

<img src="http://i.imgur.com/n6LW1jL.png" width="300">

The skeleton app contains a `GoalProgressBar` class which extends Android's [ProgressBar](http://developer.android.com/reference/android/widget/ProgressBar.html), but doesn't add any additional functionality. Clicking on the `Reset Progress` button will update the progress to a new value, which will help us test our changes during development. 


### Initialization
We won't need to extend android's `ProgressBar` as we will be drawing the progress manually. Change `GoalProgressBar` to extend `View` instead of `ProgressBar`, and remove the `style` and `android:indeterminite` attributes from our custom view's definition in `activity_main.xml`. 

In order to draw the progress we'll need to use a Paint object. Define a new paint object that is initialized after view creation: 
```java
private Paint progressPaint;

public GoalProgressBar(Context context, AttributeSet attrs) {
    super(context, attrs);
    init();
}

private void init() {
    progressPaint = new Paint();
    progressPaint.setStyle(Paint.Style.FILL_AND_STROKE);
}
```

It's important that the `Paint` is only created once, as reinitializing the object each time it is used in drawing would affect performance. 

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

### Custom Attributes
We're going to create custom attributes to allow users to customize different components of our progress bar. To start out, we'll define member variables for each customizable attribute: 
```java
// bar color when the goal has been reached
private int goalReachedColor;

// bar color when the goal has not been reached
private int goalNotReachedColor;

// bar color for the unfilled section (remaining progress)
private int unfilledSectionColor;

// thickness of the progress bar
private float barThickness;

// height of the goal indicator 
private float goalIndicatorHeight;

// thickness of the goal indicator
private float goalIndicatorThickness;
```

### Measuring 
Next we will override `onMeasure`, so we can tell our parent view what size we'd like the view to be. For the width of the view, we can use whatever width imposed by our parent (provided in `widthMeasureSpec`), but we'll add some custom handling for determining our height. 

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

### Drawing the view
// TODO 

