## Overview

Android L introduces a new widget called `CardView` which essentially can be thought of as a `FrameLayout` with rounded corners and shadow based on its elevation.

`CardView` should be used when displaying heterogenous content. Prefer using a list or a grid of tiles (non-cards) for homogenous content as the boundaries in a card can distract the user from quickly scanning a list.

### Using CardView

Lets assume your layout looks something like this:

```xml
<FrameLayout
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content">

    <!-- Main Content View -->
    <RelativeLayout>
        ...
    </RelativeLayout>
</FrameLayout>
```

To create a card using this layout, first you need to import the support library in your `build.gradle` file.

```groovy
dependencies {
    ...
    compile 'com.android.support:cardview-v7:21.0.2'
}
```

Now replace the `FrameLayout` with `CardView`.

```xml
<android.support.v7.widget.CardView
    xmlns:card_view="http://schemas.android.com/apk/res-auto"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content">

    <!-- Main Content View -->
    <RelativeLayout>
        ...
    </RelativeLayout>
</android.support.v7.widget.CardView>
```

Thats it! Using the support library ensures backward compatibility as well; although the visual treatment for cards are slightly different when running on versions prior to Android L. See [Support on Older Platforms](#Support-on-Older-Platforms) section for more details.

### Customizing CardView

`CardView` provides a default elevation and corner radius so that cards have a consistent appearance across the platforms. However, you may choose to customize these values if you desire to do so.

```xml
<android.support.v7.widget.CardView
    xmlns:card_view="http://schemas.android.com/apk/res-auto"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    card_view:cardCornerRadius="8dp"
    card_view:cardElevation="8dp">

    <!-- Main Content View -->
    <RelativeLayout>
        ...
    </RelativeLayout>
</android.support.v7.widget.CardView>
```

Note that the `card_view:cardElevation` is used to determine the size and softness of the shadow so as to realistically depict the depth.

### Adding Ripple Effect

By default, a `CardView` is not tappable and it doesn't show any touch feedback. Touch feedback animations provide users with visual feedback when a `CardView` is touched. To enable this behavior,add the following attributes to your `CardView`.

```xml
android:clickable="true"
android:foreground="?android:attr/selectableItemBackground"
```

Using the android:foreground="?android:attr/selectableItemBackground" property on a `CardView` enables the ripple effect to originate from the touch origin.

### <a name="Support-on-Older-Platforms"></a>Support on Older Platforms

On platforms before Android L, `CardView` adds padding to support corner radius, since rounded corner clipping can be an expensive operation. Similarly, for shadows, before L, CardView adds content padding and draws shadows to that area. This content padding is based on the elevation, and as per the docs:

> This padding amount is equal to maxCardElevation + (1 - cos45) * cornerRadius on the sides and maxCardElevation * 1.5 + (1 - cos45) * cornerRadius on top and bottom.

Thus, if you would like to specify custom padding on the content, you need to use a new attribute
`card_view:contentPadding`.

Similarly, to change the background color of a CardView, you need to use a new attribute
`card_view:cardBackgroundColor`.

## References

* <https://developer.android.com/reference/android/support/v7/widget/CardView.html>
* <http://www.google.com/design/spec/components/cards.html#cards-usage>
