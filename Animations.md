## Overview

Android supports powerful animations for both views and transitions between activities. There are three animation systems that work differently for different cases but the most important are Property animations. 

Property animations allow us to animate any property of any object from one value to another over a specified duration. These can be applied to anything within the Android application. 

This is typically used for any dynamic movement for views including position changes, rotations, expansion or coloration changes. In some cases animation is used for adding visual flair and polish to an application while in others, it could be essential to the operation of the app.

Animations like many resources for Android can be defined both through XML resources as well as 
dynamically within the Java code.

## Usage

There are four relevant animation types for us to understand:

 * [Property Animation](http://developer.android.com/guide/topics/graphics/prop-animation.html) - This is the animation of any property between two values. Frequently used to animate views on screen such as rotating an image or fading out a button.
 * [Layout Animation](http://developer.android.com/guide/topics/graphics/prop-animation.html#layout) - This allows us to enable animations on any layout container or other ViewGroup such as a ListView. With layout animations enabled, all changes to views inside the container will be animated.
 * [Activity Transitions](http://ahdidou.com/blog/customize-android-activities-transition/#.Uli6O2Q6VZ8) - Animates the transition as an Activity enters the screen when an Intent is executed.
 * [Fragment Transitions](http://android-er.blogspot.com/2013/04/implement-animation-in.html) - Animates the transition as a fragment enters or exits the screen when a transaction occurs.

### Property Animations

Property animations were a more recent Android feature [introduced in 3.0](http://android-developers.blogspot.com/2011/05/introducing-viewpropertyanimator.html). To use animations in a way that is **compatible with pre-3.0 Android versions**, we must use the [NineOldAndroids](http://nineoldandroids.com/) for all our property animations. 

The first thing we should do is [download NineOldAndroids](https://github.com/JakeWharton/NineOldAndroids) and install this as a [library project](http://imgur.com/a/N8baF) in Eclipse for use with your apps. For any activity that uses NineOldAndroids, include a static import to the ViewPropertyAnimator, as below.

```java
import static com.nineoldandroids.view.ViewPropertyAnimator.animate;
```

#### Using Java

Once we have setup NineOldAndroids, we can do property animations in a simple and highly compatible way. All we have to do is import the `animate` method and then we can compose and execute property animations on views. For example, suppose we want to fade out a button on screen. All we need to do is pass the button view into the `animate` method and then invoke the `alpha` property:

```java
Button btnExample = (Button) findViewById(R.id.btnExample);
//  Note: in order to use the ViewPropertyAnimator like this add the following import:
//  import static com.nineoldandroids.view.ViewPropertyAnimator.animate;
animate(btnExample).alpha(0);
```

This will automatically create and execute the animation to fade out the button. The animate method has many properties that mirror the methods from the [ViewPropertyAnimator](http://developer.android.com/reference/android/view/ViewPropertyAnimator.html) class including changing many possible properties such as opacity, rotation, scale, x&y positions, and more. For example, here's a more complex animation being executed:

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

This applies multiple property animations at once including opacity change, rotation, scale and modifying the position of the button. Here we also can modify the duration, introduce a start delay and even execute a listener at the beginning or end of the animation. You can also use the [ObjectAnimator](http://developer.android.com/reference/android/animation/ObjectAnimator.html) method to add additional behavior such as repeating the animation with:

```java
ObjectAnimator scaleAnim = ObjectAnimator.ofFloat(tvLabel, "scaleX", 1.0f, 2.0f);
scaleAnim.setDuration(3000);
scaleAnim.setRepeatCount(ValueAnimator.INFINITE);
scaleAnim.setRepeatMode(ValueAnimator.REVERSE);
scaleAnim.start();
```

You can also play multiple `ObjectAnimator` objects together with the [AnimatorSet](http://developer.android.com/reference/android/animation/AnimatorSet.html) with:

```java
AnimatorSet set = new AnimatorSet();
set.playTogether(
    ObjectAnimator.ofFloat(tvLabel, "scaleX", 1.0f, 2.0f)
	.setDuration(2000),
    ObjectAnimator.ofFloat(tvLabel, "scaleX", 1.0f, 2.0f)
        .setDuration(2000),
    ObjectAnimator.ofObject(tvLabel, "backgroundColor", new ArgbEvaluator(),
          /*Red*/0xFFFF8080, /*Blue*/0xFF8080FF)
        .setDuration(2000)
);
set.start(); 
```

See the [Property Animation](http://developer.android.com/guide/topics/graphics/prop-animation.html) official docs for more detailed information in addition to the [NineOldAndroids](http://nineoldandroids.com/) website.

#### Using XML

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

See more details in the [Property Animation Resource](http://developer.android.com/guide/topics/resources/animation-resource.html#Property) guide.

### View Animations

View animations is a slower and less flexible system for animation that predates the property animation system that was introduced later. **Property animations are generally preferred** but let's take a look at the older system and how to apply animations using the original XML syntax.

#### Using XML

We can define our view animations using XML instead of using the Property XML Animations. First, we define
our animations in the `res/anim` folder. You can see **several popular animations** by checking out [this Android Animation XML Pack](https://gist.github.com/nesquena/2dab264ed3bcec9e520a). 

For example, a value animation XML file might look like this for fading in an object:

```xml
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android"
    android:fillAfter="true" >
    <alpha
        android:duration="1000"
        android:fromAlpha="0.0"
        android:interpolator="@android:anim/accelerate_interpolator"
        android:toAlpha="1.0" />
</set>
```

Now, we can load that animation in an activity with:

```java
// Inflate animation from XML
Animation animFadein = AnimationUtils.loadAnimation(this, R.anim.fade_in);  
// Setup listeners (optional)
animFadein.setAnimationListener(new AnimationListener() {
    @Override
    public void onAnimationStart(Animation animation) {
        // Fires when animation starts
    }
});
// start the animation
btnExample.startAnimation(animFadein);
```

See more details in the [View Animation Resource](http://developer.android.com/guide/topics/resources/animation-resource.html#View) guide or this [more detailed tutorial](http://www.androidhive.info/2013/06/android-working-with-xml-animations/#fade_in).

### Layout Animations

#### Animating on Start

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

#### Animating Changes

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

### Activity Transitions

In addition to property animations for views, we can also manage the animations and transitions for Activities as they become visible on screen. We do this by managing the transitions around the time we trigger an intent.
The basic approach is that right after executing an intent with `startActivity`, you call the [overridePendingTransition](http://developer.android.com/reference/android/app/Activity.html#overridePendingTransition\(int, int\)) method such as:

```java
Intent i = new Intent(MainActivity.this, SecondActivity.class);
startActivity(i);
overridePendingTransition(R.anim.right_in, R.anim.left_out);
```

The first parameter is the "enter" animation for the launched activity and the second is the "exit" animation for the current activity. These animations can be any XML view animation files. For example, `right_in` might be defined in `res/anim/right_in.xml` as:

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

This would move the X position of the new window from off the screen sliding in from the right. We might then define `left_out` in `res/anim/left_out.xml` with:

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

This causes the parent activity to slide off screen to left. When these animations run together, it creates the effect that the new activity is pushing the old one off the screen while sliding in. You can also control the "back" animation transition as well using `overridePendingTransition` with:  

```java
public class SecondActivity extends Activity {
    // onBackPressed is what is called when back is hit, call `overridePendingTransition`
    @Override
    public void onBackPressed() {
	finish();
	overridePendingTransition(R.anim.left_in, R.anim.right_out);
    }
}
```

You can see a more complete example on the [customizing activity transition tutorial](http://ahdidou.com/blog/customize-android-activities-transition/) as well as the [custom animation while switching](http://android-er.blogspot.com/2013/04/custom-animation-while-switching.html) tutorial. Yet another example can be found in the [blundell apps animate an activity](http://blog.blundell-apps.com/animate-an-activity/) tutorial.

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

**Compatibility Note:** The animation files explained above are used in conjunction with support fragments. Keep in mind that if you are not using support fragments, you need to use object animations instead as [explained on StackOverflow](http://stackoverflow.com/a/9856449).

Note that Android has built-in animations with [R.anim](http://developer.android.com/reference/android/R.anim.html) that can be used as well by reference `android.R.anim` such as:

```java
FragmentTransaction fts = getSupportFragmentManager().beginTransaction();
fts.setCustomAnimations(android.R.anim.slide_in_left, android.R.anim.slide_out_right);
ft.replace(R.id.fragment_container, newFragment, "fragment");
ft.commit();
```

Read more about Fragment Transitions in this [detailed article](http://android-er.blogspot.com/2013/04/implement-animation-in.html). You can even [check out the source code](http://grepcode.com/file/repository.grepcode.com/java/ext/com.google.android/android/4.4_r1/frameworks/base/core/res/res/anim/slide_in_right.xml?av=f) of those animations. For a step-by-step example on how to implement a "flip" animation, check out the official [Fragment Flip Tutorial](http://developer.android.com/training/animation/cardflip.html). 

**Extended Note:** Check out [this stackoverflow post](http://stackoverflow.com/a/15816189/313399) if you are looking to animate the appearance of a DialogFragment.

### Animated Images

In certain cases, we want to be able to display animated images such as an simple animated GIF. The underlying class for making this happen is called [AnimationDrawable](http://developer.android.com/reference/android/graphics/drawable/AnimationDrawable.html) which is an XML format for describing a sequence of images to display. 

One of the simplest ways to display an animated gif is to use a third-party library. [gifanimateddrawable](https://github.com/Hipmob/gifanimateddrawable) is an easy way to convert a gif into an `AnimationDrawable` which can be displayed in an `ImageView` (see [blog post](http://engineering.hipmob.com/2013/12/02/Showing-animated-gifs-on-Android/)). First, let's [download](https://github.com/Hipmob/gifanimateddrawable/archive/master.zip) the library and import as a library project and then we can display local gifs. First, setup an `ImageView` in the the activity i.e `activity_main.xml`:

```xml
<ImageView
    android:id="@+id/ivGif"
    android:contentDescription="Gif"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content" />
```

Be sure to put an animated gif into the `res/raw` folder for displaying. Next, we can convert the GIF into an `AnimationDrawable` and load the GIF into the `ImageView` within the activity i.e `MainActivity.java`:

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
        loadGifIntoImageView(ivGif, R.raw.my_gif);
    }

    // Loads a GIF from the specified raw resource folder into an ImageView
    protected void loadGifIntoImageView(ImageView ivImage, int rawId) {
        try {
            GifAnimationDrawable anim = new GifAnimationDrawable(getResources().openRawResource(rawId));
            ivImage.setImageDrawable(anim);
            ((GifAnimationDrawable) ivImage.getDrawable()).setVisible(true, true);
        } catch (NotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

You can also check out the popular [ImageViewEx](https://github.com/frapontillo/ImageViewEx) library for another solution. An alternative method is simply to [use a WebView](http://droid-blog.net/2011/10/17/tutorial-how-to-play-animated-gifs-in-android-part-3/). Any of these approaches will make displaying animated gifs fairly straightforward.

## Things To Note

* There are actually three types of animation systems on Android:
    * [Property Animations](http://developer.android.com/guide/topics/graphics/prop-animation.html) - The most powerful and flexible animation system introduced in Android 3.0.
    * [View Animations](http://developer.android.com/guide/topics/graphics/view-animation.html) - Slower and less flexible; deprecated since property animations were introduced
    * [Drawable Animations](http://developer.android.com/guide/topics/graphics/drawable-animation.html) - Used to animate by displaying drawables in quick succession
* Property animations typically take place on properties of objects by using the `ObjectAnimator`
  which is an extension of the `ValueAnimator` which is the core timing engine for animations.
* `AnimatorSet` is the mechanism for logically grouping animations together that run in relation
   to each other and can run them either parallel or sequentially.

## Libraries

* [NineOldAndroids](http://nineoldandroids.com/) - Compatibility library supporting property animations all the way back to Android 1.0.

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