## Overview

Styles in Android allow you to define the look and feel, for example colors and fonts, of Android components in XML resource files. This way you have to set common style attributes only once in one central place.

This is typically used for reducing styling duplication in a way highly analogous to CSS in the web development world. By specifying styles in one central file, we can then apply consistent styling across our application's views.

## Usage

Styles in conjunction with drawables are how more views are kept maintainable in the face of heavy UI customization. Styles work by defining style names associated with a series of properties to apply to a view. Styles can also inherit from other style and compound styles can be created as well.

### Defining and Using Styles

First, you define the XML style in `res/values/styles.xml`:

```xml
<style name="LargeRedFont">
    <item name="android:textColor">#C80000</item>
    <item name="android:textSize">40sp</item>
</style>
```

Now you can use the style within your activities:

```xml
<TextView
  android:id="@+id/tv_text"
  style="@style/LargeRedFont"
  android:layout_width="wrap_content"
  android:layout_height="wrap_content"
  android:text="@string/hello_world" /> 
```

### Inheriting Styles

In many cases, you may want to extend a style and modify certain attributes. The `parent` attribute in the `<style>` element lets you specify a style from which your style should inherit properties. You can use this to inherit properties from an existing style and then define only the properties that you want to change or add.

```xml
<style name="LargeFont">
    <item name="android:textSize">40sp</item>
</style>

<style name="LargeBlueFont" parent="@style/LargeFont">
  <item name="android:textColor">#00007f</item>
</style>
```

If you want to inherit from styles that you've defined yourself, you do not even have to use the parent attribute. Instead, as a shortcut just prefix the name of the style you want to inherit to the name of your new style, separated by a period:

```xml
<style name="LargeFont">
    <item name="android:textSize">40sp</item>
</style>

<style name="LargeFont.Red">
    <item name="android:textColor">#C80000</item>
</style>
```

You can continue to extend styles inheriting from them by using multiple periods:

```xml
<style name="LargeFont.Red.Bold">
    <item name="android:textStyle">bold</item>
</style>

<style name="LargeFont.Red.Big">
    <item name="android:textSize">30sp</item>
</style>
```

You can't inherit Android built-in styles this way. To reference a built-in style you must use the parent attribute:

```xml
<style name="CustomButton" parent="@android:style/Widget.Button">
  <item name="android:gravity">center_vertical|center_horizontal</item>
  <item name="android:textColor">#FFFFFF</item>
</style>
```

### Using Themes

In some cases, we want to apply a consistent theme to all activities within our application. Instead of applying the style to a particular individual view, you can apply a collection of styles as a **Theme** to an Activity or application. When you do so, every View within the Activity or application will apply each property that it supports. 

#### Defining a Theme

```xml
<style name="LightThemeSelector" parent="android:Theme.AppCompat.Light">
    ...
</style>
```

This theme contains `item` nodes that often an reference other styles or colors:

```xml
<style name="LightThemeSelector" parent="android:Theme.AppCompat.Light">
    <item name="android:windowBackground">@color/custom_theme_color</item>
    <item name="android:colorBackground">@color/custom_theme_color</item>
</style>
```

#### Customizing a Theme

In many cases, you will want to **customize the default appearance of views** within your application. For example, you may want to set the `textColor` of a TextView or Button as the default for your application. This can be done by defining styles that inherit from the defaults and then overwriting those properties in `res/values/styles.xml`:

```xml
<resources xmlns:android="http://schemas.android.com/apk/res/android">
    <!-- ...generated stuff here -->
     
    <!-- This is the generated app theme -->
    <style name="AppTheme" parent="AppBaseTheme">
        <!-- These are your custom properties -->
        <item name="android:buttonStyle">@style/Widget.Button.Custom</item>
        <item name="android:textViewStyle">@style/Widget.TextView.Custom</item>
    </style>
    
    <!-- This is the custom button styles for this application -->
    <style name="Widget.Button.Custom" parent="android:Widget.Button">
      <item name="android:textColor">#0000FF</item>
    </style>
    
    <!-- This is the custom textview styles for this application -->
    <style name="Widget.TextView.Custom" parent="android:Widget.TextView">
      <item name="android:textColor">#00FF00</item>
    </style>
</resources>
```

Notice that we use the `AppTheme` generated for us to make modifications to [buttonStyle](http://developer.android.com/reference/android/R.attr.html#buttonStyle) and `textViewStyle` in order to determine the default styles for those controls. Next, we inherit from the default `Widget.Button` or `Widget.TextView` to take the default styles and make our changes. The result of this is the default text color is different for Button and TextView:

![Screen](http://i.imgur.com/fF7UiTo.png)

#### Additional Resources

Having trouble figuring out which style attributes you can use when defining a theme?  Here are a few resources:

 * [themes.xml](http://omapzoom.org/?p=platform/frameworks/base.git;a=blob;f=core/res/res/values/themes.xml;hb=master) for the complete list of the thousands of default styles for an app. 
 * [R.attr](http://developer.android.com/reference/android/R.attr.html) documentation for a full rundown as well. 
 * [Useful holo theme generator tool](http://android-holo-colors.com/) for creating a theme that has a modified base color for certain widgets by default.  
 * [Customizing Action Bar styles guide](http://guides.codepath.com/android/Extended-ActionBar-Guide#custom-actionbar-styles) and [sample code](https://github.com/codepath/android-actionbar-style-demo) for theming the Action Bar.

#### Applying a Theme

To set a theme for all the activities of your application, open the AndroidManifest.xml file and edit the <application> tag to include the android:theme attribute with the style name. For example:

```xml
<application android:theme="@style/CustomTheme">
```

You can also apply to a particular activity in the manifest:

```xml
<activity android:theme="@style/CustomTheme">
```

You can see more about all this in the [official styles guide](http://developer.android.com/guide/topics/ui/themes.html).

## Tools

 * <http://jgilfelt.github.io/android-actionbarstylegenerator>
 * <http://android-holo-colors.com/>
 * <http://android-ui-utils.googlecode.com/hg/asset-studio/dist/index.html>

## References

 * <http://developer.android.com/guide/topics/ui/themes.html>
 * <http://www.vogella.com/articles/AndroidStylesThemes/article.html>
 * <http://mobile.tutsplus.com/tutorials/android/android-sdk-exploring-styles-and-themes/>
 * <http://developer.android.com/guide/topics/resources/style-resource.html>
 * <http://java.dzone.com/articles/creating-custom-android-styles>
 * <http://janrain.com/blog/introduction-to-android-theme-customization/>
 * <http://javatechig.com/android/android-styles-and-themes-tutorial/>