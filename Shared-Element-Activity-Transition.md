## Overview

Traditionally transitions between different activities or fragments involved enter and exit transitions that animated entire view hierarchies independant to each other. Example of such transitions are a fade transition, slide transition or the newly introduced explode transition.

Default Activity Transition:

![Imgur](http://i.imgur.com/KW88kGk.gif)

However, many times, there are elements common to both activities and providing the ability to transition these shared elements separately emphasizes continuity between transitions and breaks activity boundaries as the user navigates the app.

![SharedElement](http://3.bp.blogspot.com/-DnRuIu0QqgE/VAeEWFgCVdI/AAAAAAAAqak/t5NF8kHVRG8/s1600/heroview.png)

The nature of this transition forces the human eye to focus on the content and its representation in the new activity instead of the actual activity frame sliding or fading which makes the experience a lot more seamless.

![Imgur](http://i.imgur.com/IuYcb05.gif)


### Getting Started

The shared element transitions require Android 5.0 (API level 21) and above.

### 1. Enable Window Content Transitions

Enable Window Content Transitions in your `styles.xml` file.
 ```xml
 <!-- Base application theme. -->
 <style name="AppTheme" parent="Theme.AppCompat.Light.DarkActionBar">
     <!-- Customize your theme here. -->
     <item name="android:windowContentTransitions">true</item>
     ...
 </style>
 ```

### 2. Assign a Common Transition Name

Assign a common transition name to the shared elements in both layouts. Use the   `android:transitionName` attribute.

For e.g.
In `MainActivity.xml`:
```xml
<android.support.v7.widget.CardView
  ...
      <ImageView
          android:id="@+id/ivProfile"
          android:transitionName="profile"
          android:scaleType="centerCrop"
          android:layout_width="match_parent"
          android:layout_height="160dp" />
      ...
</android.support.v7.widget.CardView>
```

In `DetailActivity.xml`:
```xml
<FrameLayout
  ...
      <ImageView
          android:id="@+id/ivProfile"
          android:transitionName="profile"
          android:scaleType="centerCrop"
          android:layout_width="match_parent"
          android:layout_height="380dp" />
      ...
</FrameLayout>
```

Note that it doesn't matter if the `android:id` is different or where in the layout hierarchy the source and target views exist.

### 3. Start Activity

Start the target activity by specifying a bundle of those shared elements and views from the source.

```xml
final Intent intent = new Intent(context, DetailsActivity.class);
// Pass data object in the bundle and populate details activity.
intent.putExtra(DetailsActivity.EXTRA_CONTACT, contact);
final Pair<View, String> p1 = Pair.create((View)ivProfile, "profile");
ActivityOptionsCompat options = ActivityOptionsCompat.makeSceneTransitionAnimation((Activity) context, p1);
context.startActivity(intent, options.toBundle());
```

Thats it! Specifying the source view along with the transition name ensures that even if you have multiple views with the same transition name in the source hierarchy, it will essentially be able to pick the right view to start the animation from.

To reverse the scene transition animation when you finish the second activity, call the `Activity.supportFinishAfterTransition()` method instead of `Activity.finish()`. Also, you will need to override the behavior of the home button in the `ToolBar/ ActionBar` for such cases:

```java
@Override
public boolean onOptionsItemSelected(MenuItem item) {
    switch (item.getItemId()) {
        // Respond to the action bar's Up/Home button
        case android.R.id.home:
            supportFinishAfterTransition();
            return true;
    }
    return super.onOptionsItemSelected(item);
}
```

### 4. Multiple Shared Elements

Sometimes, you might want to animate multiple elements from the source view hierarchy. This can be achieved by using distinct transition names in the source and target layout xml files.

 ```xml
final Intent intent = new Intent(context, DetailsActivity.class);
intent.putExtra(DetailsActivity.EXTRA_CONTACT, contact);
final Pair<View, String> p1 = Pair.create((View)ivProfile, "profile");
final Pair<View, String> p2 = Pair.create(vPalette, "palette");
final Pair<View, String> p3 = Pair.create((View)tvName, "text");
ActivityOptionsCompat options = ActivityOptionsCompat.makeSceneTransitionAnimation(this, p1, p2, p3);
startActivity(intent, options.toBundle());
```

Be careful to not overdo transitions between shared elements. It makes sense to have one cohesive unit animate from one screen to another (which may or may not contain multiple shared elements); having too many shared elements will result in a very distracting animation and make the experience more jarring.

### 5. Customizing Shared Elements Transition

In Android L, shared elements transition defaults to a combination of ChangeBounds, ChangeTransform, ChangeImageTransform, and ChangeClipBounds. This works well for most typical cases. However, you may customize this behavior or even define your own custom transition. See this official guide on [Defining Custom Animations](https://developer.android.com/training/material/animations.html).

## References

* <https://developer.android.com/training/material/animations.html>
* <http://java.dzone.com/articles/material-design-activity>
