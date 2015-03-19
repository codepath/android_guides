## Overview

Ripple touch effect was [introduced with material design in Android 5.0](https://www.youtube.com/watch?v=97SWYiRtF0Y&t=482) (API level 21).

Touch feedback in material design provides an instantaneous visual confirmation at the point of contact when users interact with UI elements. For example, buttons now display a ripple effect when they are touched-this is the default touch feedback animation in Android 5.0. Ripple animation is implemented by the new `RippleDrawable` class. The ripple effect can be configured to end at the bounds of the view or extend beyond the bounds of the view. For example, the following sequence of screenshots illustrates the ripple effect in a button during touch animation:

  ![touch](http://i.imgur.com/hrkinJY.png)

Initial touch contact with the button occurs in the first image on the left, while the remaining sequence (from left to right) illustrates how the ripple effect spreads out to the edge of the button. When the ripple animation ends, the view returns to its original appearance. The default ripple animation takes place in a fraction of a second, but the length of the animation can be customized for longer or shorter lengths of time.

Note that the ripples will only show up on devices running Lollipop, and will fall back to a static highlight on previous versions.

### Clickable Views

In general, ripple effect for regular buttons will work by default in API 21 and for other touchable views, it can be achieved by specifying

```xml
android:foreground="?android:attr/selectableItemBackground">
```

In code:

```java
int[] attrs = new int[]{R.attr.selectableItemBackground};
TypedArray typedArray = getActivity().obtainStyledAttributes(attrs);
int backgroundResource = typedArray.getResourceId(0, 0);
myView.setBackgroundResource(backgroundResource);
```

  ![view](http://i.imgur.com/SkExpyw.gif)


### Buttons

Most buttons are made with several drawables. Usually you’ll have a pressed and normal version of assets like this:

`/drawable/button.xml`:

```xml
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:state_pressed="true" android:drawable="@drawable/button_pressed"/>
    <item android:drawable="@drawable/button_normal"/>
</selector>
```

If you have a custom button with selected state, your text color changes depending on the state, etc. So default button background is not going to work for you here. You can add this feedback for your own drawables and for custom buttons by simply wrapping them in a `ripple` element:

`/drawable-21/button.xml`:

```xml
<ripple xmlns:android="http://schemas.android.com/apk/res/android"
    android:color="?android:colorControlHighlight">
    <item android:drawable="@drawable/button_normal" />
</ripple>
```

Using `?android:colorControlHighlight` will give the ripple the same color as the built-in ripples in your app.

  ![button](http://i.imgur.com/L9ZnabL.gif)

If you don’t like the default grey, you can specify what color you want `android:colorControlHighlight` to be in your theme.

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>

  <style name="AppTheme" parent="android:Theme.Material.Light.DarkActionBar">
    <item name="android:colorControlHighlight">@color/your_custom_color</item>
  </style>

</resources>
```

If you want the ripple to extend past the boundary of the view, then you can instead use `?attr/selectableItemBackgroundBorderless`. This works well with ImageButtons and smaller Buttons that are part of a larger View:

  ![anim](http://i.imgur.com/RYQ6nnB.gif)

## References

* <https://developer.android.com/training/material/animations.html>
* <https://developer.android.com/reference/android/graphics/drawable/RippleDrawable.html>
* <http://appsdevelopment-for-mobiles.blogspot.com/2014/11/android-l-ripple-touch-effect.html>
