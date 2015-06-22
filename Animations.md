## Overview

Android supports powerful animations for both views and transitions between activities. There are three animation systems that work differently for different cases but the most important are Property animations. Property animations allow us to animate any property of any object from one value to another over a specified duration. 

These can be applied to anything within the Android application. This is typically used for any dynamic movement for views including position changes, rotations, expansion or coloration changes. 

Animations like many resources for Android can be defined both through XML resources as well as 
dynamically within the Java code.

### Animation Types

There are actually two distinct animation frameworks for Android:

 - [Property Animations](http://developer.android.com/guide/topics/graphics/prop-animation.html) - The most powerful and flexible animation system introduced in Android 3.0.
 - [View Animations](http://developer.android.com/guide/topics/graphics/view-animation.html) - Slower and less flexible; deprecated since property animations were introduced

Powered by these animation frameworks, there are five relevant types of animations:

 * [Property Animations](http://developer.android.com/guide/topics/graphics/prop-animation.html) - This is the animation of any property between two values. Frequently used to animate views on screen such as rotating an image or fading out a button.
 * [Activity Transitions](http://ahdidou.com/blog/customize-android-activities-transition/#.Uli6O2Q6VZ8) - Animates the transition as an Activity enters the screen when an Intent is executed.
 * [Fragment Transitions](http://android-er.blogspot.com/2013/04/implement-animation-in.html) - Animates the transition as a fragment enters or exits the screen when a transaction occurs.
 * [Layout Animation](http://developer.android.com/guide/topics/graphics/prop-animation.html#layout) - This allows us to enable animations on any layout container or other ViewGroup such as a ListView. With layout animations enabled, all changes to views inside the container will be animated.
 * [Drawable Animations](http://developer.android.com/guide/topics/graphics/drawable-animation.html) - Used to animate by displaying drawables in quick succession

### Material Animation Principles

With the release of Android 5.0, Google has released detailed [animation design guidelines](http://www.google.com/design/spec/animation) which documents important principles to consider when integrating animations and motion into your application. There are four core principles outlined:

 * [Authentic motion](http://www.google.com/design/spec/animation/authentic-motion.html) - Motion in the world of material design is not only beautiful, it builds meaning about the spatial relationships, functionality, and intention of the system.
 * [Responsive interaction](http://www.google.com/design/spec/animation/responsive-interaction.html) - When a user interacts with an app and beautiful yet perfectly logical things happen, the user feels satisfied—even delighted.
 * [Meaningful transitions](http://www.google.com/design/spec/animation/meaningful-transitions.html) - Carefully choreographed motion design can effectively guide the user’s attention and focus through multiple steps of a process or procedure and avoid confusion. 
 * [Delightful details](http://www.google.com/design/spec/animation/delightful-details.html) - Animation can exist within all components of an app and at all scales, from finely detailed icons to key transitions and actions. All elements work together to construct a seamless experience and a beautiful, functional app.

Review these principles for a detailed look at how to think about animations in a way that aligns with a material app.

## Property Animations

Property animations were a more recent Android feature introduced in 3.0. The primary motivations for the introduction of the property animations system are outlined in this [post from 2011 introducing property animations](http://android-developers.blogspot.com/2011/02/animation-in-honeycomb.html). 

Common properties commonly animated on views include:

| Property                                                         | Description          | 
|--------------------------------------------------------------    | ------------         |
| `alpha`                                                          | Fade in or out       |
| `rotation`, `rotationX`, `rotationY`                             | Spinning             |
| `scaleX`, `scaleY`                                               | Grow or shrink       |
| `x`, `y`, `z`                                                    | Position             |
| `translationX`, `translationY`, **`translationZ` (API 21+)**     | Offset from Position |

To use animations in a way that is **compatible with pre-3.0 Android versions**, we must use the [NineOldAndroids](http://nineoldandroids.com/) for all our property animations. 

If you are an Android Studio user, add the following dependency to your `app/build.gradle` file.

```gradle
compile 'com.nineoldandroids:library:2.4.0+'
```

One library that simplifies common animations is called [AndroidViewAnimations](https://github.com/daimajia/AndroidViewAnimations) and makes certain common animations on views much easier to achieve. This library is definitely worth a look.

### Using ObjectAnimator in Java

Once we have setup NineOldAndroids, we can use the [ObjectAnimator](http://developer.android.com/reference/android/animation/ObjectAnimator.html) method to execute simple animations for a particular property on a specified object:

```java
ObjectAnimator fadeAnim = ObjectAnimator.ofFloat(tvLabel, "alpha", 0.2f);
fadeAnim.start();
```

This code will execute the animation to fade out the button.   Notice that the "alpha" is designated as a string type.  The ObjectAnimator relies on [reflection](https://github.com/android/platform_frameworks_base/blob/master/core/java/android/animation/PropertyValuesHolder.java#L658-719) and uses the button's [getAlpha()](http://developer.android.com/reference/android/view/View.html#getAlpha()) and [setAlpha()](http://developer.android.com/reference/android/view/View.html#setAlpha(float)) methods to perform the animation.

![Simple Fadeout](http://i.imgur.com/dsyRMsl.gif)

We can also use the [properties system in 4.0](http://android-developers.blogspot.com/2011/11/android-40-graphics-and-animations.html) to execute property animations on common properties using:

```java
ObjectAnimator fadeAltAnim = ObjectAnimator.ofFloat(image, View.ALPHA, 0, 1);
fadeAltAnim.start();
```

This is considerably faster because this property check doesn't require as much [slow runtime reflection](https://github.com/android/platform_frameworks_base/blob/master/core/java/android/view/ViewPropertyAnimator.java#L989-L1022). Properties supported include `ALPHA`, `ROTATION`, `ROTATION_X`, `ROTATION_Y`, `SCALE_X`, `SCALE_Y`, `TRANSLATION_X`, `TRANSLATION_Y`, `TRANSLATION_Z`, `X`, `Y`, `Z` in order to improve performance on these animations. 

#### Setting Duration or Repeat on Property Animation 

This code above will fade the specified view to 20% opacity. We can add additional behavior such as duration or repeating the animation with:

```java
ObjectAnimator scaleAnim = ObjectAnimator.ofFloat(tvLabel, "scaleX", 1.0f, 2.0f);
scaleAnim.setDuration(3000);
scaleAnim.setRepeatCount(ValueAnimator.INFINITE);
scaleAnim.setRepeatMode(ValueAnimator.REVERSE);
scaleAnim.start();
```

#### Setting Interpolation

Whenever we define a property animation, we should also consider the rate of change of that animation. In other words, we don't just want to consider the value to change a property to but also the trajectory of the movement. This is done by specifying a [TimeInterpolator](http://developer.android.com/reference/android/animation/TimeInterpolator.html) on your `ObjectAnimator`:

```java
ObjectAnimator moveAnim = ObjectAnimator.ofFloat(v, "Y", 1000);
moveAnim.setDuration(2000);
moveAnim.setInterpolator(new BounceInterpolator());
moveAnim.start();
```

The result of this is a different rate of motion:

![Bounce](http://i.imgur.com/Wu96uOS.gif)

Common pre-defined interpolators are listed below:

| Name                   | Description |
| ---------------------  | ---------------------------------------------------------  |
| AccelerateInterpolator | Rate of change starts out slowly and and then accelerates  |
| BounceInterpolator     | Change bounces right at the end                            |
| DecelerateInterpolator | Rate of change starts out quickly and and then decelerates |
| LinearInterpolator     | Rate of change is constant throughout                      |

There are several other [time interpolators](http://developer.android.com/reference/android/animation/TimeInterpolator.html) which can be reviewed.

#### Listening to the Animation Lifecycle

We can add an [AnimatorListenerAdapter](http://developer.android.com/reference/android/animation/AnimatorListenerAdapter.html) to manage events during the animation lifecycle such as `onAnimationStart` or `onAnimationEnd`:

```java
ObjectAnimator anim = ObjectAnimator.ofFloat(v, "alpha", 0.2f);
anim.addListener(new AnimatorListenerAdapter() {
    @Override
    public void onAnimationEnd(Animator animation) {
        Toast.makeText(MainActivity.this, "End!", Toast.LENGTH_SHORT).show();
    }
});
anim.start();
```

#### Choreographing Animations

We can play multiple `ObjectAnimator` objects together concurrently with the [AnimatorSet](http://developer.android.com/reference/android/animation/AnimatorSet.html) with:

```java
AnimatorSet set = new AnimatorSet();
set.playTogether(
    ObjectAnimator.ofFloat(tvLabel, "scaleX", 1.0f, 2.0f)
        .setDuration(2000),
    ObjectAnimator.ofFloat(tvLabel, "scaleY", 1.0f, 2.0f)
        .setDuration(2000),
    ObjectAnimator.ofObject(tvLabel, "backgroundColor", new ArgbEvaluator(),
          /*Red*/0xFFFF8080, /*Blue*/0xFF8080FF)
        .setDuration(2000)
);
set.start(); 
```

This results in the following:

![Animation 3](http://i.imgur.com/Q2uORr7.gif)

We can also animate sets of other animator sets:

```java
// Define first set of animations
ObjectAnimator anim1 = ObjectAnimator.ofFloat(tvLabel, "scaleX", 2.0f);
ObjectAnimator anim2 = ObjectAnimator.ofFloat(tvLabel, "scaleY", 2.0f);
AnimatorSet set1 = new AnimatorSet();
set1.playTogether(anim1, anim2);
// Define second set of animations
ObjectAnimator anim3 = ObjectAnimator.ofFloat(v, "X", 300);
ObjectAnimator anim4 = ObjectAnimator.ofFloat(v, "Y", 300);
AnimatorSet set2 = new AnimatorSet();
set2.playTogether(anim3, anim4);
// Play the animation sets one after another
AnimatorSet set3 = new AnimatorSet();
set3.playSequentially(set1, set2);
set3.start();
```

Here's one more example of a choreographed set of animations:

```java
// Create two animations to play together
ObjectAnimator bounceAnim = ...;
ObjectAnimator squashAnim = ...;
// Construct set 1 playing together
AnimatorSet bouncer = new AnimatorSet();
bouncer.play(bounceAnim).with(squashAnim);
// Create second animation to play after
ObjectAnimator fadeAnim = ObjectAnimator.ofFloat(view1, "alpha", 1f, 0f);
fadeAnim.setDuration(250);
// Play bouncer before fade
AnimatorSet animatorSet = new AnimatorSet();
animatorSet.play(bouncer).before(fadeAnim);
animatorSet.start();
``` 

See the [Property Animation](http://developer.android.com/guide/topics/graphics/prop-animation.html) official docs for more detailed information in addition to the [NineOldAndroids](http://nineoldandroids.com/) website.

### Using ViewPropertyAnimator in Java

We can also do property animations in an even simpler way using the `ViewPropertyAnimator` system which is built on top of the `ObjectAnimator`.  It also enables faster performance as described in this [blog post](http://android-developers.blogspot.com/2011/05/introducing-viewpropertyanimator.html) and provides a convenient way of doing animations.

Without NineOldAndroids and therefore incompatible with Android versions of 3.0 or below we can run concurrent animations with:

```java
Button btnExample = (Button) findViewById(R.id.btnExample);
btnExample.animate().alpha(0.2f).xBy(-100).yBy(100);
```

For any activity that uses [NineOldAndroids](https://github.com/JakeWharton/NineOldAndroids), be sure to include a static import to the ViewPropertyAnimator, as shown below:

```java
import static com.nineoldandroids.view.ViewPropertyAnimator.animate;
```

Now we can execute property animations on our views. For example, suppose we want to fade out a button on screen. All we need to do is pass the button view into the `animate` method and then invoke the `alpha` property:

```java
Button btnExample = (Button) findViewById(R.id.btnExample);
//  Note: in order to use the ViewPropertyAnimator like this add the following import:
//  import static com.nineoldandroids.view.ViewPropertyAnimator.animate;
animate(btnExample).alpha(0);
```

This will automatically create and execute the animation to fade out the button: 

![Simple Fadeout](http://i.imgur.com/dsyRMsl.gif)

The animate method has many properties that mirror the methods from the [ViewPropertyAnimator](http://developer.android.com/reference/android/view/ViewPropertyAnimator.html) class including changing many possible properties such as opacity, rotation, scale, x&y positions, and more. For example, here's a more complex animation being executed:

```java
animate(btnExample).alpha(0.5f).rotation(90f).
  scaleX(2).xBy(100).yBy(100).setDuration(1000).setStartDelay(10).
  setListener(new AnimatorListenerAdapter() {
      @Override
      public void onAnimationStart(Animator animation) {
          Toast.makeText(MainActivity.this, "Started...", Toast.LENGTH_SHORT).show();
      };
  });
```

This applies multiple property animations at once including opacity change, rotation, scale and modifying the position of the button. Here we also can modify the duration, introduce a start delay and even execute a listener at the beginning or end of the animation. 

### Using XML

We can also use NineOldAndroids to load property animations from XML. All we have to do is create an XML file that describes the object property animation we want to run. For example, if we wanted to animate a fade out for a button, we could add this file to `res/animator/fade_out.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<objectAnimator xmlns:android="http://schemas.android.com/apk/res/android"
    android:propertyName="alpha"
    android:duration="1000"
    android:valueTo="0" />
```

Now we can load that XML file into our Activity and execute an animation with:

```java
Animator anim = AnimatorInflater.loadAnimator(this, R.animator.fade_out);
anim.setTarget(btnExample);
anim.start();
```

And that's all! We've now executed our predefined XML animation. Here's a more complicated animation in `res/animator/multi.xml` that applies multiple animations to the button in parallel:

```xml
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android"
    android:ordering="together" >
    <objectAnimator
        android:propertyName="alpha"
        android:valueTo="0.5" >
    </objectAnimator>
    <objectAnimator
        android:propertyName="rotation"
        android:valueTo="90.0" >
    </objectAnimator>
    <objectAnimator
        android:propertyName="scaleX"
        android:valueTo="2.0" >
    </objectAnimator>
    <objectAnimator
        android:propertyName="translationX"
        android:valueTo="100.0" >
    </objectAnimator>
    <objectAnimator
        android:propertyName="translationY"
        android:valueTo="100.0" >
    </objectAnimator>
</set>
```

and now let's execute this on the button by inflating the XML animation description:

```java
Animator anim = AnimatorInflater.loadAnimator(this, R.animator.multi_button);
anim.setTarget(tvBlah);
anim.setDuration(1000);
anim.setStartDelay(10);
anim.addListener(new AnimatorListenerAdapter() {
     @Override
     public void onAnimationStart(Animator animation) {
          Toast.makeText(MainActivity.this, "Started...", Toast.LENGTH_SHORT).show();
     };
});
anim.start();
```

This results in the following:

![Complex Animation](http://i.imgur.com/0AB0E2Q.gif) 

See more details in the [Property Animation](http://developer.android.com/guide/topics/graphics/prop-animation.html) topic guide and the [Animation Resource](http://developer.android.com/guide/topics/resources/animation-resource.html#Property) guide.

### Custom Animations with ValueAnimator

In certain cases, instead of animating the property of an object directly (i.e alpha) as shown above, we might need finer-grained control over the animation at each step of execution. In these situations, we can use a [ValueAnimator](http://developer.android.com/reference/android/animation/ValueAnimator.html) and setup a custom listener to adjust the view as the animation executes:

```java
// Construct the value animator and define the range
ValueAnimator valueAnim = ValueAnimator.ofFloat(0, 1);
// Animate over the course of 700 milliseconds
valueAnim.setDuration(700);
// Choose an interpolator
valueAnim.setInterpolator(new DecelerateInterpolator());
// Define how to update the view at each "step" of the animation
valueAnim.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
  @Override
    public void onAnimationUpdate(ValueAnimator animation) {
      float animatedValue = (float) animation.getAnimatedValue();
      // Use the animated value to affect the view
  }
});
// Start the animation if not already running
if (!valueAnim.isStarted()) {
  valueAnim.start();
}
```

See the [ValueAnimator](http://developer.android.com/reference/android/animation/ValueAnimator.html) for more information. Note that in most cases we can use an `ObjectAnimator` as shown above but the value animator is a lower-level construct that can be used when we do not want to animate the property of an object directly. 

## View Animations

View animations is a slower and less flexible system for animation that predates the property animation system that was introduced later. **Property animations are generally preferred** but let's take a look at the older system and how to apply animations using the original XML syntax.

The primary motivations for the introduction of the property animations system are outlined clearly in this [post from 2011 introducing property animations](http://android-developers.blogspot.com/2011/02/animation-in-honeycomb.html).   Some of the major differences include:

  * The old view animation system only supported move, fade, scale, and rotate, whereas the new one provides a more extensible framework (i.e. animating background color, gradients, or even [marker map locations](http://blog.neteril.org/blog/2013/05/26/exploring-android-property-animations/)).  

  * The old system only supported View objects (i.e. Button, TextView, ListView, etc.) but the new Property animations can support any objects such as drawables.

  * The old system would not actually update the location after a move animation, requiring manual code to update to the new position. This issue has been fixed in the new system.

See this [Google I/O talk](https://www.youtube.com/watch?v=sTx-5CGDvM8&t=2126) discussing some of the limitations in the older view animation system.

There have been misconceptions that Android did not use dedicated hardware ([GPU's](http://en.wikipedia.org/wiki/Graphics_processing_unit)) to perform animations in the old view system.  This point has been [debunked by one of Google's engineers](https://plus.google.com/105051985738280261832/posts/2FXDCz8x93s).  When the new Property animations systems was announced along with Android v3.0, [support for hardware acceleration](http://android-developers.blogspot.com/2011/03/android-30-hardware-acceleration.html) for common views and widgets was also added, making the overall experience seem smoother.

### Using XML

We can define our view animations using XML instead of using the Property XML Animations. First, we define
our animations in the `res/anim` folder. You can see **several popular animations** by checking out [this Android Animation XML Pack](https://gist.github.com/nesquena/2dab264ed3bcec9e520a). 

For example, a value animation XML file might look like this for fading in an object in `res/anim/fade_out.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android"
    android:fillAfter="true" >
    <alpha
        android:duration="1000"
        android:fromAlpha="1.0"
        android:interpolator="@android:anim/accelerate_interpolator"
        android:toAlpha="0.0" />
</set>
```

Now, we can load that animation within an activity and apply this to a view:

```java
// Inflate animation from XML
Animation animFadeOut = AnimationUtils.loadAnimation(this, R.anim.fade_out);  
// Setup listeners (optional)
animFadeOut.setAnimationListener(new AnimationListener() {
    @Override
    public void onAnimationStart(Animation animation) {
        // Fires when animation starts
    }

    @Override
    public void onAnimationEnd(Animation animation) {
       // ...			
    }

    @Override
    public void onAnimationRepeat(Animation animation) {
       // ...			
    }
});
// start the animation
btnExample.startAnimation(animFadeOut);
```

This results in the following:

![Simple Fadeout](http://i.imgur.com/dsyRMsl.gif)

See more details in the [View Animation Resource](http://developer.android.com/guide/topics/resources/animation-resource.html#View) guide or this [more detailed tutorial](http://www.androidhive.info/2013/06/android-working-with-xml-animations/#fade_in).

### Activity Transitions

In addition to property animations for views, we can also manage the animations and transitions for Activities as they become visible on screen. We do this by managing the transitions around the time we trigger an intent.
The basic approach is that right after executing an intent with `startActivity`, you call the [overridePendingTransition](http://developer.android.com/reference/android/app/Activity.html#overridePendingTransition\(int, int\)) method such as:

```java
Intent i = new Intent(MainActivity.this, SecondActivity.class);
startActivity(i);
overridePendingTransition(R.anim.right_in, R.anim.left_out);
```

The first parameter is the "enter" animation for the launched activity and the second is the "exit" animation for the current activity. These animations can be any XML view animation files such as the animations in our [XML Pack](https://gist.github.com/nesquena/2dab264ed3bcec9e520a). For example, `right_in` might be defined in `res/anim/right_in.xml` as:

```xml
<?xml version="1.0" encoding="utf-8"?>  
<set xmlns:android="http://schemas.android.com/apk/res/android"
     android:interpolator="@android:anim/linear_interpolator">  
  <translate 
      android:fromXDelta="100%p" 
      android:toXDelta="0" 
      android:duration="500"/>
</set> 
```

This would move the X position of the new window from off the screen sliding in from the right ([`100%p`](http://developer.android.com/guide/topics/resources/animation-resource.html) means the Activity should start at a position equal to the full width of the window). We might then define `left_out` in `res/anim/left_out.xml` with:

```xml
<?xml version="1.0" encoding="utf-8"?>  
<set xmlns:android="http://schemas.android.com/apk/res/android"
     android:interpolator="@android:anim/linear_interpolator">  
  <translate 
      android:fromXDelta="0" 
      android:toXDelta="-100%p" 
      android:duration="500"/>
</set> 
```

This causes the parent activity to slide off screen to left. When these animations run together, it creates the effect that the new activity is pushing the old one off the screen while sliding in. In order to control the "back" animation transition as well, we need to create a `left_in.xml` and `right_out.xml` that reverse the values of `left_out.xml` and `right_in.xml`:

left_in.xml:

```xml
<?xml version="1.0" encoding="utf-8"?>  
<set xmlns:android="http://schemas.android.com/apk/res/android"
     android:interpolator="@android:anim/linear_interpolator">  
  <translate 
      android:fromXDelta="-100%p" 
      android:toXDelta="0" 
      android:duration="500"/>
</set> 
```

right_out.xml
```xml
<?xml version="1.0" encoding="utf-8"?>  
<set xmlns:android="http://schemas.android.com/apk/res/android"
     android:interpolator="@android:anim/linear_interpolator">  
  <translate 
      android:fromXDelta="0" 
      android:toXDelta="100%p" 
      android:duration="500"/>
</set> 
```

We then use `overridePendingTransition` with these two XML-based animations:

```java
public class SecondActivity extends Activity {
    // onBackPressed is what is called when back is hit, call `overridePendingTransition`
    @Override
    public void onBackPressed() {
        finish();
        overridePendingTransition(R.anim.left_in, R.anim.right_out);
    }

    // to handle activity transitions for Up navigation add it to the onOptionsItemSelected
    // as below 
    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        // Handle action bar item clicks here. The action bar will
        // automatically handle clicks on the Home/Up button, so long
        // as you specify a parent activity in AndroidManifest.xml.
        int id = item.getItemId();

        //noinspection SimplifiableIfStatement
        if (id == R.id.action_settings) {
            return true;
        }

        // This refers to the Up navigation button in the action bar
        if (id == android.R.id.home) {
            finish();
            overridePendingTransition(R.anim.left_in, R.anim.right_out);
            return true;
        }

        return super.onOptionsItemSelected(item);
    }
}
```

This results in the following:

![Activity Transition](http://i.imgur.com/lRU3wrn.gif)

You can see several complete examples of activity transitions in the following resources:

 * [Card Flip Animation](http://developer.android.com/training/animation/cardflip.html)
 * [Vine Activity Transition](http://blog.quent.in/index.php/2013/06/activity-transition-animations-like-the-vine-android-application/)
 * [Sliding In From Left Animation](http://android-er.blogspot.com/2013/04/custom-animation-while-switching.html)
 * [Sliding Drawer Animation](http://blog.blundell-apps.com/animate-an-activity/)

Check out these above to get a deeper understanding of how to create custom and unique transitions. In **Android 5.0 and above** the ability for activities to "share elements" was introduced allowing an element in on activity to morph into an element within the child activity. Check out our [[Shared Element Activity Transition]] guide for more details.

### Fragment Transitions

Similar to activities, we can also animate the transition between fragments by invoking the [setCustomAnimations](http://developer.android.com/reference/android/app/FragmentTransaction.html#setCustomAnimations\(int, int\)) method on a `FragmentTransaction`. The animations are simply view animation XML files as used for activity transitions above. For example, suppose we want the fragment to slide in from the left and push the old fragment out. First, let's define the custom view animations starting with `res/anim/slide_in_left.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android">
    <translate android:fromXDelta="-50%p" android:toXDelta="0"
        android:duration="@android:integer/config_mediumAnimTime"/>
    <alpha android:fromAlpha="0.0" android:toAlpha="1.0"
        android:duration="@android:integer/config_mediumAnimTime" />
</set>
```

and then `res/anim/slide_out_right.xml` as well:

```xml
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android">
    <translate android:fromXDelta="0" android:toXDelta="50%p"
     android:duration="@android:integer/config_mediumAnimTime"/>
    <alpha android:fromAlpha="1.0" android:toAlpha="0.0"
      android:duration="@android:integer/config_mediumAnimTime" />
</set>
```

Now we just have to configure the animations of the fragment transaction before we replace the fragment or call commit:

```java
// Create the transaction
FragmentTransaction fts = getSupportFragmentManager().beginTransaction();
// Configure the "in" and "out" animation files
fts.setCustomAnimations(R.anim.slide_in_left, R.anim.slide_out_right);
// Perform the fragment replacement
ft.replace(R.id.fragment_container, newFragment, "fragment");
// Start the animated transition.
ft.commit();
```

This results in the following:

![Fragment Transition](http://i.imgur.com/Y9L82FI.gif)

**Compatibility Note:** The animation files explained above are used in conjunction with support fragments. Keep in mind that if you are not using support fragments, you need to use object animations instead as [explained on StackOverflow](http://stackoverflow.com/a/9856449).

Note that Android has **built-in animations** with [R.anim](http://developer.android.com/reference/android/R.anim.html) that can be used as well by reference `android.R.anim` such as:

```java
FragmentTransaction fts = getSupportFragmentManager().beginTransaction();
fts.setCustomAnimations(android.R.anim.slide_in_left, android.R.anim.slide_out_right);
fts.replace(R.id.fragment_container, newFragment, "fragment");
fts.commit();
```

Read more about Fragment Transitions in this [detailed article](http://android-er.blogspot.com/2013/04/implement-animation-in.html). You can even [check out the source code](http://grepcode.com/file/repository.grepcode.com/java/ext/com.google.android/android/4.4_r1/frameworks/base/core/res/res/anim/slide_in_right.xml?av=f) of those animations. For a step-by-step example on how to implement a "flip" animation, check out the official [Fragment Flip Tutorial](http://developer.android.com/training/animation/cardflip.html). 

**Extended Note:** Check out [this stackoverflow post](http://stackoverflow.com/a/15816189/313399) if you are looking to animate the appearance of a DialogFragment.

## Layout Animations

### Animating on Start

A particular animation can be specified when the layout first appears on screen. This can be done by using the [android:layoutAnimation](http://developer.android.com/reference/android/view/ViewGroup.html#attr_android:layoutAnimation) property to specify an animation to execute.

First, let's define an animation we'd like to use when the views in the layout appear on the screen in `res/anim/slide_right.xml` which defines sliding in right from outside the screen:

```xml
<set  xmlns:android="http://schemas.android.com/apk/res/android"  
      android:interpolator="@android:anim/accelerate_interpolator">
    <translate android:fromXDelta="-100%p" android:toXDelta="0"
            android:duration="1000" /> 
</set>
```

and now we need to create a special `layoutAnimation` which references that animation:

```xml
<?xml version="1.0" encoding="utf-8"?>
<layoutAnimation xmlns:android="http://schemas.android.com/apk/res/android"
        android:delay="30%"
        android:animationOrder="reverse"
        android:animation="@anim/slide_right"
/>
```

and now we need to apply this `layoutAnimation` to our Layout or ViewGroup:

```xml
<LinearLayout 
    ...
    android:layoutAnimation="@anim/layout_bottom_to_top_slide" />
```

and now when you launch the application, the views within this layout will slide in from the right. See more information about layout animation in [this guide by linkyan](http://linkyan.com/2013/06/layoutanimation/) or in this [layout animation tutorial](http://android-er.blogspot.com/2009/10/layout-animation.html) or in the [ui action layout tutorial](http://mobile.dzone.com/articles/android-ui-action-layout).

### Animating Changes

[Layout Change Animations](http://developer.android.com/training/animation/layout.html) allow us to enable animations on any Layout container or other ViewGroup such as a ListView. With layout animations enabled, all changes to views inside the container will be animated automatically. This is especially useful for ListViews which causes items to be animated as they are added or removed.

To enable the default animations, all we have to do is set the `animateLayoutChanges` property on any ViewGroup within the XML:

```xml
<LinearLayout
  ...
  android:animateLayoutChanges="true">
  
  <ListView android:id="@+id/list"
    android:animateLayoutChanges="true"
    ...
  />

</LinearLayout>
```

The [android:animateLayoutChanges](http://developer.android.com/reference/android/view/ViewGroup.html#attr_android:animateLayoutChanges) property enables a default animation if no further customization is made.

## Animated Images

In certain cases, we want to be able to display animated images such as an simple animated GIF. The underlying class for making this happen is called [AnimationDrawable](http://developer.android.com/reference/android/graphics/drawable/AnimationDrawable.html) which is an XML format for describing a sequence of images to display. 

One of the simplest ways to display an animated gif is to use a third-party library. [Glide](https://github.com/bumptech/glide) is an easy way to display a local or remote gif into an `ImageView`. First, let's add Glide to our `app/build.gradle` file:

```gradle
dependencies {
    ...
    compile 'com.github.bumptech.glide:glide:3.6.0'
}
```

Next, setup an `ImageView` in the the activity i.e `activity_main.xml`:

```xml
<ImageView
    android:id="@+id/ivGif"
    android:contentDescription="Gif"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content" />
```

For local GIFs, be sure to put an animated gif into the `res/raw` folder for displaying. Next, we can convert and load the GIF into the `ImageView` within the activity i.e `MainActivity.java` using Glide:

```java
// MainActivity.java
public class MainActivity extends Activity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        // Find the ImageView to display the GIF
        ImageView ivGif = (ImageView) findViewById(R.id.ivGif);
        // Display the GIF (from raw resource) into the ImageView
        Glide.with(this).load(R.raw.my_gif).asGif().into(imageView);
        // OR even download from the network
        // Glide.with(this).load("http://i.imgur.com/l9lffwf.gif").asGif().into(imageView);
    }
}
```

and now we have this:

![GIF](http://i.imgur.com/YEmtaE6.gif)

You can also check out the popular [android-gif-drawable](https://github.com/koral--/android-gif-drawable) library for another solution. An alternative method is simply to [use a WebView](http://droid-blog.net/2011/10/17/tutorial-how-to-play-animated-gifs-in-android-part-3/). 

## Lollipop Animations

In Android 5.0, several new animation features were introduced including:

 * [[Shared Element Activity Transition]] - Transitions that have shared layout elements that transform as one activity is transitioned to the other.
 * [[Ripple Animation]] - Used provide an instantaneous visual confirmation at the point of contact when users interact with UI elements.
 * [[Circular Reveal Animation]] - Reveal is a new animation introduced in Android L that animates the view's clipping boundaries. Often used in conjunction with [[material floating action buttons|Floating Action Buttons]].

Note that these animations are **lollipop only** and do not work on devices with an Android version less than API 21. Since less than 10% of devices have lollipop enabled, use of these animations is often not worth the effort.

## Particle Effects

Use the third-party [Leonids](https://github.com/plattysoft/Leonids) library for simple particle effects. Particle systems are often used in games for a wide range of purposes: explosions, fire, smoke, etc. This effects can also be used on normal apps to add elements of playful design. Here's an example of particle motion:

![Particles](http://i.imgur.com/gTnpLuk.gif)

Precisely because its main use is games, all engines have support for particle systems, but there is no such thing built-in for standard Android UI. Instead we can use Leonids by [reviewing this tutorial](http://www.plattysoft.com/2014/05/30/leonids-particle-system-lib/) for further details.

## Libraries

* [AndroidViewAnimations](https://github.com/daimajia/AndroidViewAnimations) - Common property animations made easy.
* [ListViewAnimations](https://github.com/nhaarman/ListViewAnimations) - List view item animations made simple including insertion and deletion.
* [NineOldAndroids](http://nineoldandroids.com/) - Compatibility library supporting property animations all the way back to Android 1.0.
* [Leonids](https://github.com/plattysoft/Leonids) - Simple particle effects for Android.

## References

 * <http://www.vogella.com/articles/AndroidAnimation/article.html>
 * <http://developer.android.com/guide/topics/graphics/overview.html>
 * <http://developer.android.com/guide/topics/graphics/drawable-animation.html>
 * <http://developer.android.com/guide/topics/resources/animation-resource.html>
 * <http://developer.android.com/reference/android/view/ViewPropertyAnimator.html>
 * <http://www.androidhive.info/2013/06/android-working-with-xml-animations/>
 * <http://karanbalkar.com/2013/05/tutorial-30-create-animation-in-android/>
 * <http://www.javacodegeeks.com/2011/01/android-animations-quick-guide.html>
 * <http://android-developers.blogspot.com/2011/02/animation-in-honeycomb.html>
 * <http://android-developers.blogspot.com/2011/05/introducing-viewpropertyanimator.html>
 * <http://learnandroideasily.blogspot.in/2013/07/imageview-animation-in-android.html>
 * <http://java.dzone.com/articles/using-view-animations-android>
 * <http://mobile.dzone.com/articles/android-ui-action-layout>
 * <http://www.google.com/design/spec/animation/authentic-motion.html> 