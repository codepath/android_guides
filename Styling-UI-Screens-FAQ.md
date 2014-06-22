## Overview

This is an FAQ that addresses common questions that arise when building UI screens. For inspiration, see the [android app patterns](http://www.android-app-patterns.com/) site for screens you might try to build. There are several questions that come up commonly addressed below:

### Layouts

#### How do I properly layout a complex screen with many views?

The recommended way to layout most screens is often by using the [[powerful RelativeLayout|Constructing-View-Layouts#relativelayout]] system. Note that you can also **embed layouts within each other**. LinearLayouts positioned within a root RelativeLayout can be a powerful way to achieve complex layouts as well.

#### How do I get rid of the padding around the edges of the activity?

You need to go to the XML Layout and check the root `RelativeLayout` and remove each of the `android:padding` properties that are set by default. After removing these properties from the root layout, that padding around the edges of your layout will disappear. 

#### How do I set an entire region to have a particular background color or image?

Simply assign the [android:background](http://developer.android.com/reference/android/view/View.html#attr_android:background) property to any view or layout to change the background color or image. The `android:background` for any view can be either a color hex value `#000040` or a drawable image `@drawable/some_image`. 

```xml
<RelativeLayout
  android:background="@drawable/some_image"
  android:layout_width="wrap_content"
  android:layout_height="wrap_content">
</RelativeLayout>
```

Use padding around a layout to add extra content spacing. Remember that you can **embed layouts within each other** if needed to achieve the desired effect.

#### How do I set the opacity of a layout or view?

The opacity of any view can be set in the XML Layout in two ways. First, the `android:background` property of any view supports alpha channels when specifying a color hex in the format of "#AARRGGBB" for example "#80a4113b" would set the alpha channel to 50% for the color "#a4113b". For other alpha values check out [this values list](http://stackoverflow.com/a/11019879):

```xml
<RelativeLayout
  android:background="#80a4113b"
  android:layout_width="wrap_content"
  android:layout_height="wrap_content">
</RelativeLayout>
```

Alternatively, you can use the [android:alpha](http://developer.android.com/reference/android/view/View.html#attr_android:alpha) property which must be a floating point from 0 to 1. Note though that `alpha` sets **both the view and all of it's children** to this opacity which is often not desired.

### Views

#### How do I control the way images are displayed or scaled within an ImageView?

See the [[Working with ImageView|Working-with-the-ImageView]] guide in particular the [[sizing section|Working-with-the-ImageView#sizing-imageview-controls]]: 

```xml
<ImageView
    ...
    android:maxHeight="150dp"
    android:maxWidth="150dp"
    android:scaleType="fitXY"
    android:adjustViewBounds="true"
/>
```

There are many different `android:scaleType` values which you can read about in [[this scale types guide|Working-with-the-ImageView#scale-types]].

#### How do I support text with rich formatting (bold words, links) in a TextView?

The `TextView` has [[basic support for HTML text formatting|Working-with-the-TextView#inserting-html-formatting]]:

```
TextView view = (TextView)findViewById(R.id.sampleText);
view.setText(Html.fromHtml("This <i>is</i> a <b>test</b>"));
```

and can even [[autolink URLs contained within the text|Working-with-the-TextView#autolinking-urls]]:

```xml
<TextView
     android:id="@+id/custom_font"
     android:layout_width="fill_parent"
     android:layout_height="wrap_content"
     android:autoLink="all"
     android:linksClickable="true"
/>
```

Refer to the [official TextView docs](http://developer.android.com/reference/android/widget/TextView.html) for other details.

#### How do I support different fonts in my app?

Fonts can be customized fairly easily using this [[custom fonts|Working-with-the-TextView#using-custom-fonts]] guide. Be aware that custom fonts can cause performance issues if used too much.

#### How do I customize the style of a button?

Styling a button requires the use of either image assets (see the [ImageButton](http://developer.android.com/reference/android/widget/ImageButton.html)) or alternatively applying the concept of [[custom drawables|Drawables#customizing-a-button]]. For readymade solutions, check out [these nice looking pre-built buttons](http://www.dibbus.com/2011/03/9patch-images-in-android/) or [this handy button generator](http://angrytools.com/android/button/).

#### How do I control the pressed state of the button?

Button states are created by using a "drawable" xml resource called a [State List](http://developer.android.com/guide/topics/resources/drawable-resource.html#StateList). The core is you define each state with it's own drawable by assigning the `android:background` property of the button as the state list. This involves creating an XML file within a `res/drawable` folder called `states_my_image_button.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android" >
  <item android:drawable="@drawable/btn_pressed" 
        android:state_pressed="true"/> 

  <item android:drawable="@drawable/btn_normal" />
</selector>
```

and then setting this as the background of the `ImageButton` within the layout:

```xml
<Button
    android:background="@drawable/states_my_image_button"  
    ...
/>
```

Check out the [Button Custom Background Official Guide](http://developer.android.com/guide/topics/ui/controls/button.html#CustomBackground) for specific details.

#### How would I create images that look good at any resolution?

You have probably noticed that there are multiple drawable folders (i.e drawable-hdpi, drawable-xhdpi) which allow us to provide [multiple resolutions for different density screens](http://developer.android.com/training/basics/supporting-devices/screens.html#create-bitmaps). An easy guide for which sizes to create can be found in [[this ImageView guide|Working-with-the-ImageView#supporting-multiple-densities]].

Also, there are cases where a button has an image background that needs to stretch to support different text content. In this case you might need to [draw a 9-patch](http://developer.android.com/tools/help/draw9patch.html) stretchable button. Check out the [Button Custom Background Official Guide](http://developer.android.com/guide/topics/ui/controls/button.html#CustomBackground) for specific details. You can also check out [these nice looking 9-patch buttons](http://www.dibbus.com/2011/03/9patch-images-in-android/) for use too.

#### How do I add a border to an image or another view?

You can add a border to any view by creating a "drawable shape xml" and applying that as the `android:background` of the view. Create an XML file in `res/drawable` called `shape_view_border.xml`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android">
    <solid android:color="#FFFFFF" />
    <stroke android:width="1dp" android:color="#000000" />
</shape>
```

The `stroke` is the border properties and `solid` is the background color of the rest of the view. See [this stackoverflow post](http://stackoverflow.com/a/3264140) for more details.

#### How do I add a background image to a view?

Simply drag the image to the `res/drawable-xhdpi` folder and then apply the image to any view by setting `android:background="@drawable/my_image_name"`. See [this stackoverflow post](http://stackoverflow.com/a/8177819) for more details. If you need more control of how the image is scaled, see [this post](http://stackoverflow.com/a/16139059) using a FrameLayout.

#### How do I remove the grey border from an ImageButton?

You can remove the border by either setting `android:background` to "@null" or setting `style` to "android:attr/borderlessButtonStyle":

```xml
<ImageButton
     ...
     style="?android:attr/borderlessButtonStyle"
     android:src="@drawable/image_button_graphic" />
```

Using this code the border on the imagebutton will be removed.

### ActionBar

#### How do I style the title in the ActionBar?

ActionBar title can be styled or centered only if you opt to customize the XML view being inflated to display in the ActionBar. Instead of using the default ActionBar text you can specify your own to use instead that is styled as desired. Check out the [[Advanced ActionBar|Extended-ActionBar-Guide#custom-actionbar-layout]] cliffnotes for details. This [related stackoverflow post](http://stackoverflow.com/questions/12387345/how-to-align-actionbar-title-center-in-android/12388200#12388200) also explains the steps in detail. 

#### How do I change the background color of the ActionBar?

Customize the theme of the ActionBar in the `styles.xml` as explained in [this stackoverflow post](http://stackoverflow.com/a/9249702) to adjust the color of the ActionBar. Also, check out the [[Advanced Actionbar|Extended-ActionBar-Guide#custom-actionbar-styles]] cliffnotes for more details. 

#### How would I hide the top ActionBar?

You can hide the ActionBar by modifying the "theme" of the Activity in the `AndroidManifest.xml` as described in this [stackoverflow post](http://stackoverflow.com/a/19545450) or programmatically at runtime in Java with `getActionBar().hide()` within the `onCreate` method after `setContentView` in an Activity.