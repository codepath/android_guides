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

## Material Styles

### Dialog Styles

If you want your Dialogs to have a Material look and feel across all Android versions, you’ll want to use a library to achieve modern dialogs such as:

 * [material-dialogs](https://github.com/afollestad/material-dialogs)
 * [AlertDialogPro](https://github.com/fengdai/AlertDialogPro/)
 * [MaterialDialog](https://github.com/drakeet/MaterialDialog)

![Material Dialog](https://raw.githubusercontent.com/afollestad/material-dialogs/master/art/screenshots.png)

## Material Animations

### Ripple Animations

Ripple provides an instantaneous visual confirmation at the point of contact when users interact with UI elements. Note that the ripples will only show up on devices running Lollipop, and will fall back to a static highlight on previous versions.

![Ripple](http://i.imgur.com/L9ZnabL.gif)

See our detailed [[ripple animations guide|Ripple-Animation]] for an overview of how to enable this effect on views and buttons.

## References

* <http://code.hootsuite.com/tips-and-tricks-for-android-material-support-library/>
* <http://code.hootsuite.com/tips-and-tricks-for-android-material-support-library-2-electric-boogaloo/>