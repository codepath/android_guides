## Overview

Reveal is a new animation introduced in Android L that animates the view's clipping boundaries. Reveal animations provide users visual continuity when you show or hide a group of UI elements. This animation is often times used in conjunction with a floating action button.

![reveal](http://i.imgur.com/8jzWpX1.gif)

### Setting Up

The `ViewAnimationUtils.createCircularReveal()` method enables you to animate a clipping circle to reveal or hide a view.

To reveal a previously invisible view using this effect:

```java
void enterReveal() {
    // previously invisible view
    final View myView = findViewById(R.id.my_view);

    // get the center for the clipping circle
    int cx = myView.getMeasuredWidth() / 2;
    int cy = myView.getMeasuredHeight() / 2;

    // get the final radius for the clipping circle
    int finalRadius = Math.max(myView.getWidth(), myView.getHeight()) / 2;

    // create the animator for this view (the start radius is zero)
    Animator anim =
        ViewAnimationUtils.createCircularReveal(myView, cx, cy, 0, finalRadius);

    // make the view visible and start the animation
    myView.setVisibility(View.VISIBLE);
    anim.start();
}
```

To hide a previously visible view using this effect:

```java
void exitReveal() {
    // previously visible view
    final View myView = findViewById(R.id.my_view);

    // get the center for the clipping circle
    int cx = myView.getMeasuredWidth() / 2;
    int cy = myView.getMeasuredHeight() / 2;

    // get the initial radius for the clipping circle
    int initialRadius = myView.getWidth() / 2;

    // create the animation (the final radius is zero)
    Animator anim =
        ViewAnimationUtils.createCircularReveal(myView, cx, cy, initialRadius, 0);

    // make the view invisible when the animation is done
    anim.addListener(new AnimatorListenerAdapter() {
        @Override
        public void onAnimationEnd(Animator animation) {
            super.onAnimationEnd(animation);
            myView.setVisibility(View.INVISIBLE);
        }
    });

    // start the animation
    anim.start();
}
```

### View Transition

You want to call `enterReveal()` after the enter transition of the activity has been completed.

```java
private Transition.TransitionListener mEnterTransitionListener;

@Override
protected void onCreate(Bundle savedInstanceState) {

  ...

  mEnterTransitionListener = new Transition.TransitionListener() {
      @Override
      public void onTransitionStart(Transition transition) {

      }

      @Override
      public void onTransitionEnd(Transition transition) {
          enterReveal();
      }

      @Override
      public void onTransitionCancel(Transition transition) {

      }

      @Override
      public void onTransitionPause(Transition transition) {

      }

      @Override
      public void onTransitionResume(Transition transition) {

      }
  };
  getWindow().getEnterTransition().addListener(mEnterTransitionListener);
}
```

Update `enterReveal()` method to remove this listener from the set listening to this animation. This is done to disable reveal animation when the activity ends.

```java
private void enterReveal() {
    // previously invisible view
    final View myView = findViewById(R.id.my_view);

    // get the center for the clipping circle
    int cx = myView.getMeasuredWidth() / 2;
    int cy = myView.getMeasuredHeight() / 2;

    // get the final radius for the clipping circle
    int finalRadius = Math.max(myView.getWidth(), myView.getHeight()) / 2;
    Animator anim = ViewAnimationUtils.createCircularReveal(myView, cx, cy, 0, finalRadius);
    myView.setVisibility(View.VISIBLE);
    anim.addListener(new Animator.AnimatorListener() {
        @Override
        public void onAnimationStart(Animator animation) {

        }

        @Override
        public void onAnimationEnd(Animator animation) {
            getWindow().getEnterTransition().removeListener(mEnterTransitionListener);
        }

        @Override
        public void onAnimationCancel(Animator animation) {

        }

        @Override
        public void onAnimationRepeat(Animator animation) {

        }
    });
    anim.start();
}
```

While exiting the activity after the reveal transition, you want to finish the activity after completing the exit reveal animation.

```java
anim.addListener(new AnimatorListenerAdapter() {
        @Override
        public void onAnimationEnd(Animator animation) {
            super.onAnimationEnd(animation);
            myView.setVisibility(View.INVISIBLE);

            // Finish the activity after the exit transition completes.
            supportFinishAfterTransition();
        }
    });
```

To enable exit reveal transition on press of `ActionBar` up button or back button, call `exitReveal()` from `onOptionsItemSelected()` and `onBackPressed()` respectively.

```java
@Override
public boolean onOptionsItemSelected(MenuItem item) {
    switch (item.getItemId()) {
        // Respond to the action bar's Up/Home button
        case android.R.id.home:
            exitReveal();
            return true;
    }
    return super.onOptionsItemSelected(item);
}

@Override
public void onBackPressed() {
    exitReveal();
}
```

### Common Gotcha

If you do the reveal animation directly on the view, it might look weird because the stroke part of the shape gets revealed as well. To prevent this, you can embed the view inside a `FrameLayout` which has stroke border as a background. By doing this, the animation reveals the view while the background part remains the same, so it looks like only color part is revealed.

circular_button.xml:

```xml
<?xml version="1.0" encoding="utf-8"?>

<shape xmlns:android="http://schemas.android.com/apk/res/android" android:shape="oval">
    <solid android:color="@android:color/white"/>
    <stroke android:width="1dp" android:color="#AAA"/>
</shape>
```

```xml
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="@dimen/button_size"
    android:layout_height="@dimen/button_size"
    android:background="@drawable/circular_button">
	<Button
		android:id:@"+id/my_view"
		android:layout_width="match_parent"
    android:layout_height="match_parent"
	    ...
	</Button>
</FrameLayout>
```

### Enhancements

The reveal effect can be made to look more natural if it starts where the user tapped. To do this, you could get the x & y co-ordinates from the MotionEvent and use that to start the reveal effect.

```java
@Override
public boolean onTouch(View view, MotionEvent motionEvent) {
    if (motionEvent.getAction() == MotionEvent.ACTION_UP) {
      // get the final radius for the clipping circle
      int finalRadius = Math.max(myView.getWidth(), myView.getHeight()) / 2;

      // create the animator for this view (the start radius is zero)
      Animator anim =
          ViewAnimationUtils.createCircularReveal(myView, (int) motionEvent.getX(), (int) motionEvent.getY(), 0, finalRadius);

      // make the view visible and start the animation
      myView.setVisibility(View.VISIBLE);
      anim.start();
    }
    return false;
}
```

## References

* <https://developer.android.com/training/material/animations.html>
* <http://trickyandroid.com/simple-ripple-reveal-elevation-tutorial/>
