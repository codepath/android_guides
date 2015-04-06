With the release of Android 5.0, there are updated guidelines for modern Android UI design. These guidelines are called "material design". This page is dedicated to living in the "material world". 

## Design Guidelines

This new framework encompasses several changes to the interface of Android apps and Google strongly encourages the adoption of these new principles outlined below:

 * [Material Design Guidelines](http://www.google.com/design/spec/material-design/introduction.html) - Principles that guide material design
 * [Material Design Docs](https://developer.android.com/design/material/index.html) - Overview of material design
 * [Material Design for Developers](https://developer.android.com/training/material/index.html) - Developer guides for material 
 * [Material Design Examples](http://www.materialup.com/) - Blog posting beautiful material designs
 * [Material Palette](http://www.materialpalette.com/) - Easy material color selection
 * [Material Design Icons](http://materialdesignicons.com/) - Standard material icons from google and community

Using these materials, you can understand and begin adhering to these guidelines for your own apps.

## Material Design Setup

When switching a fresh material design theme, you can’t leave behind all older devices since the vast majority of current Android devices can't upgrade to lollipop. This is where the [Material Support Library](https://developer.android.com/training/material/compatibility.html#SupportLib) comes in to save the day. 

Google has released a handy [Material Design Checklist](http://android-developers.blogspot.ca/2014/10/material-design-on-android-checklist.html) to help ensure you've applied the necessary steps to adhere to the material look and feel.

### Include Support Library

By default, new generated apps are **automatically material design support enabled** within Android Studio. However, what about our old apps? How do we change them to support material design? First, let's add the support library to the `app/build.gradle`:

```gradle
dependencies {
    compile 'com.android.support:appcompat-v7:21.0.+'
}
```

### Change the Application Theme

Change your application theme to extend from a `Theme.Appcompat` theme in the `values/styles.xml` file:

```xml
<style name="AppTheme" parent="Theme.AppCompat.Light.DarkActionBar">
  <item name="colorPrimary">@color/primary</item>
  <item name="colorPrimaryDark">@color/primary_dark</item>
  <item name="colorAccent">@color/accent</item>
</style>
``` 

The available `Theme.Appcompat` which support material design include the following: `Theme.AppCompat`, `Theme.AppCompat.Light`, `Theme.AppCompat.Light.DarkActionBar`, `Theme.AppCompat.DeviceDefault`, `Theme.AppCompat.DeviceDefault.Light`, `Theme.AppCompat.DeviceDefault.Light.DarkActionBar`.

### Extend Activities from ActionBarActivity

`ActionBarActivity` is a subclass of `FragmentActivity`, so there should be no issues if you were using `FragmentActivity` from the older support library:

```java
public class HomeActivity extends ActionBarActivity {
    // Implementation
}
```

### Use Support ActionBar

See [[our ActionBar Guide|Extended-ActionBar-Guide#setting-up-actionbaractivity]] for more details on how to setup the compatibility ActionBar using `ActionBarActivity` including updating all of your `menu` files to use `app:showAsAction` instead of `android:showAsAction`.

Once you've switched to the `ActionBarActivity`, your code needs to access the ActionBar using `getSupportActionBar` instead of `getActionBar`:

```java
getSupportActionBar.show(); // RIGHT
getActionBar.hide(); // WRONG
```

In addition, styling the ActionBar needs to be done updated to use the [[material support styles|Extended-ActionBar-Guide#custom-actionbar-styles]] rather than the traditional ActionBar styles.

### Material Navigation Drawer

Take a look at the [Material Design Checklist](http://android-developers.blogspot.ca/2014/10/material-design-on-android-checklist.html) and you will notice that you need to change your existing drawer layout to  be over the toolbar and under the status bar. There’s a [handy StackOverflow post](http://stackoverflow.com/a/26440880/313399) explaining how to implement this.

Check out our [[Material Navigation Drawer|Fragment-Navigation-Drawer#usage]] guide for a step-by-step tutorial for setting up the updated material navigation drawer behavior. Note that to do this properly, you'll need to [[replace your ActionBar with a Toolbar|Defining-The-ActionBar#using-toolbar-as-actionbar]].

## Material Views

### RecyclerView

With our material theme, there comes a much more flexible way to display lists of items called the  [[RecyclerView|Using-the-RecyclerView]], the spiritual successor to the `ListView`. 

![RecyclerView](http://i.imgur.com/bjKHkQFm.png)

In theory, we can still use either of these but Google recommends using the `RecyclerView` going forward. Check out the differences in our [[RecyclerView vs. ListView section|Using-the-RecyclerView#compared-to-listview]].

### CardView

Android 5.0 introduces a [[new widget called CardView|Using-the-CardView]] which is an easy way to decorate items in a list with a modern style.

<img src="https://developer.android.com/design/material/images/card_travel.png" width="200" />

`CardView` can be thought of as a FrameLayout with rounded corners and shadow based on its elevation. This is best used as part of a `ListView` or `RecyclerView` when displaying homogeneous content.

### Toolbar

With our material themes, there is now a spiritual successor to the `ActionBar` called the [[Toolbar|Defining-The-ActionBar#toolbar-basics]]. 

![Toolbar](http://i.imgur.com/dGzDoDSm.png)

The toolbar operates as a replacement for the original `ActionBar` but is added directly to the layout XML file and can be placed and styled like any other view in the layout.

### Floating Action Buttons

[[Floating action buttons|Floating-Action-Buttons]] (or FAB) are a special "promoted action" within an activity or fragment that is moved from the `ActionBar` to a round icon floating above the UI in the bottom right corner.

<img src="https://github.com/makovkastar/FloatingActionButton/raw/master/art/demo.gif" width="185" alt="FAB1" />&nbsp;
<img src="http://i.imgur.com/SBbLXo2.png" width="200" alt="FAB2" />

The floating action button should represent **the primary action** within a screen. More info and use cases of the FAB button can be found in Google’s official [design specs found here](http://www.google.com/design/spec/components/buttons.html#buttons-floating-action-button).

## Material Styles

### Dialog Styles

If you want your Dialogs to have a Material look and feel across all Android versions, you’ll want to use a library to achieve modern dialogs such as:

 * [material-dialogs](https://github.com/afollestad/material-dialogs)
 * [AlertDialogPro](https://github.com/fengdai/AlertDialogPro/)
 * [MaterialDialog](https://github.com/drakeet/MaterialDialog)

![Material Dialog](https://raw.githubusercontent.com/afollestad/material-dialogs/master/art/screenshots.png)

### Dynamic Color Palettes

Material Design encourages dynamic use of color, especially when you have rich images to work with. The [[new Palette support library|Dynamic-Color-using-Palettes]] lets you extract a small set of colors from an image to style your UI controls to match; creating an immersive experience. 

![Palette](http://i.imgur.com/nwtguAS.png)

The extracted swatches will include vibrant and muted tones as well as foreground text colors for optimal legibility. This allows us to use these dynamically selected colors to apply better color selections to our layouts.

## Material Animations

### Ripple Animations

Ripple provides an instantaneous visual confirmation at the point of contact when users interact with UI elements. Note that the ripples will only show up on devices running Lollipop, and will fall back to a static highlight on previous versions.

![Ripple](http://i.imgur.com/L9ZnabL.gif)

See our detailed [[ripple animations guide|Ripple-Animation]] for an overview of how to enable this effect on views and buttons.

### Shared Element Transitions

In many situations when navigating from a list of items to a detail view, there are elements common to both activities. With the new shared element transitions system, we can transition these shared elements from one activity or fragment to another creating a much more contiguous navigation effect.

![Shared Element Transition](http://i.imgur.com/1cjqsSA.gif)

See our detailed [[shared element transition guide|Shared-Element-Activity-Transition]] for an overview of how to enable these beautiful transitions. 

### Circular Reveal

[[Circular Reveal|Circular-Reveal-Animation]] is a new animation introduced in Android 5 that animates the view's clipping boundaries. Reveal animations provide users visual continuity when you show or hide a group of UI elements. 

![reveal](http://i.imgur.com/8jzWpX1.gif)

This animation is often times used in conjunction with a floating action button that will grow onto the screen using this transtion.

## References

* <http://code.hootsuite.com/tips-and-tricks-for-android-material-support-library/>
* <http://code.hootsuite.com/tips-and-tricks-for-android-material-support-library-2-electric-boogaloo/>