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

The opacity (transparency) of any view can be set in the XML Layout in two ways. First, the `android:background` property of any view supports alpha channels when specifying a color hex in the format of "#AARRGGBB" for example "#80a4113b" would set the alpha channel to 50% for the color "#a4113b". For other alpha values check out [this values list](http://stackoverflow.com/a/11019879):

```xml
<RelativeLayout
  android:background="#80a4113b"
  android:layout_width="wrap_content"
  android:layout_height="wrap_content">
</RelativeLayout>
```

Alternatively, you can use the [android:alpha](http://developer.android.com/reference/android/view/View.html#attr_android:alpha) property which must be a floating point from 0 to 1. Note though that `alpha` sets **both the view and all of it's children** to this opacity which is often not desired.

#### How do I align the position of my view or its inner contents?

To align the **position of your view itself** in the layout, you need to use a different method based on the parent layout type. If the view is contained within a `RelativeLayout`, use the [`android:layout_centerInParent`](http://developer.android.com/reference/android/widget/RelativeLayout.LayoutParams.html#attr_android:layout_centerInParent) property to center the view both horizontally and vertically or `android:layout_centerHorizontal` to set just one or the other. 

If your view is contained within a `LinearLayout`, then you can use the [`android:layout_gravity`](http://developer.android.com/reference/android/widget/LinearLayout.LayoutParams.html#attr_android:layout_gravity) to determine the alignment of the view.

To align the contents within a view, you can use the [[android:gravity|Defining-Views-and-their-Attributes#view-gravity]] property. This property can also be used to set the alignment of text in a `TextView` to the `left`, `right`, or `center`. Note that `android:textAlignment` is **essentially useless** replaced by gravity.

#### How do I specify percentage based width, height and margins for my views?

Simply wrap your views inside a [[PercentRelativeLayout|Constructing-View-Layouts#percentrelativelayout]] or `PercentFrameLayout` and specify attributes with "Percent" suffix. Alternatively, you can also use [[android:layout_weight|Constructing-View-Layouts#linearlayout]] attribute if you are using a `LinearLayout`.

#### How do I add tabs to my activity to switch between views?

Tabs used to swap the content displayed on screen can be added to an Activity using a number of methods. The [[recommended method using a TabLayout|Google-Play-Style-Tabs-using-TabLayout]] involves splitting up the contents of each tab into a "Fragment" (modular piece of an activity) and then using a "ViewPager" which enables us to easily swap the contents of different fragments onto the screen. 

#### How do I control the order of view layers within a layout?

The easiest way to layer is to pay close attention to the order in which the Views are added to your XML file within their container. Lower down in the file means higher up in the Z-axis. Be sure to check out this [[detailed section on view drawing order|Constructing-View-Layouts#understanding-view-layers]] for the details. 

#### How do I overlap two views on top of each other?

The easiest way to overlap two views on top of each other is to use a [[RelativeLayout|Constructing-View-Layouts#relativelayout]] or [[FrameLayout|Constructing-View-Layouts#framelayout]] to wrap the views and then to order them such that views that should be drawn on top are lower down in the file. Be sure to check out this [[detailed section on view drawing order|Constructing-View-Layouts#understanding-view-layers]] for the details.

#### Why is my layout overlapping the status bar?

Use `android:fitsSystemWindows="true"` in the root view of your layout. This is an internal attribute to adjust view layout based on system windows such as the status bar. If true, adjusts the padding of this view to leave space for the system windows. Read more about [caveats here](http://stackoverflow.com/a/29738556).

### Images

#### What basics should I know about loading images?

Be sure to review the following key tips about images:

 * **Icons vs Images.** Don't use the "Image Asset" dialog in Android Studio unless **you want to generate small icons**.
 * **Image Densities**. Instead use [Final Android Resizer](http://guides.codepath.com/android/Working-with-the-ImageView#final-android-resizer) to create appropriate image sizes.
 * **Memory Errors.** Image files larger than `1776 x 1080px` in dimensions will cause Android apps to crash.
 * **Resource Names.** Filenames only include **lowercase letters, numbers and underscores** (i.e `image_1.png`)
 * **Scaling Images.** Understand and adjust the [scaleType](http://guides.codepath.com/android/Working-with-the-ImageView#scale-types) of your `ImageView` to control how the image is displayed.
 * **Aspect Ratio.** Add `android:adjustViewBounds="true"` to your `ImageView` to adjust the size to image aspect ratio.

Refer to the questions below for more detail.

#### How do I load images into an Android app for display?

If you simply want the image to be loaded in the easiest way possible then **just copy and paste** the image from your finder into the Android Studio `res/drawable` folder and select `xxhdpi` as the resolution. Keep in mind that images at `xxhdpi` density (1dp = 3px) should be about **3x the size desired on screen** to appear clear. For example, if you want an image to display as `32x32px` on screen, the image should be `96x96px` in the `drawable-xxhdpi` folder. See [[this page about densities|Working-with-the-ImageView#supporting-multiple-densities]] for more details. 

To generate images that work well at all densities, check out [[this image guide|Working-with-the-ImageView#supporting-multiple-densities]] which allows us to select our resources directory, choose an extra high density image and the tool will automatically generate the corresponding lower density image sizes.

#### How do I control the way images are displayed or scaled within an ImageView?

See the [[Working with ImageView|Working-with-the-ImageView]] guide in particular the [[sizing section|Working-with-the-ImageView#sizing-imageview-controls]]: 

```xml
<ImageView
    android:maxHeight="150dp"
    android:maxWidth="150dp"
    android:scaleType="fitXY"
    android:adjustViewBounds="true"
    ...
/>
```

There are many different `android:scaleType` values which you can read about in [[this scale types guide|Working-with-the-ImageView#scale-types]].

#### How would I create images that look good at any resolution?

You have probably noticed that there are multiple drawable folders (i.e drawable-hdpi, drawable-xhdpi) which allow us to provide multiple resolutions for different density screens. An easy guide for which sizes to create can be found in [[this ImageView guide|Working-with-the-ImageView#supporting-multiple-densities]].

In certain cases a button has an image background that needs to stretch to support different text content inside. In this case you could [draw a 9-patch](http://developer.android.com/tools/help/draw9patch.html) stretchable image. Check out the [Button Custom Background Official Guide](http://developer.android.com/guide/topics/ui/controls/button.html#CustomBackground) for specific details. You can also check out [these nice looking 9-patch buttons](http://www.dibbus.com/2011/03/9patch-images-in-android/).

#### How specifically should I generate images and icons for all density sizes?

**For generating square icons at standard sizes**, use this [handy online resizer tool](http://romannurik.github.io/AndroidAssetStudio/index.html) or the Android Studio `New => Image Asset` generator. Launcher icons are for the app icon displayed on the main screen. "Action Bar icons" can be used for anything displayed on the ActionBar. For any non-icon images, see below.

**For generating arbitrary images** at multiple densities, use the [Final Android Resizer](https://github.com/asystat/Final-Android-Resizer) by [downloading and running this JAR](https://github.com/asystat/Final-Android-Resizer/blob/master/Executable%20Jar/Final%20Android%20Resizer.jar?raw=true). Do **not use the Android Studio Image Asset generator** in this case.

#### How do I add a background image to a view?

Simply drag the image to the `res/drawable-mdpi` folder and then apply the image to any view by setting `android:background="@drawable/my_image_name"`. See [this stackoverflow post](http://stackoverflow.com/a/8177819) for more details. 

Note that the **aspect ratio cannot be changed when a drawable is applied as a background** of a view. If you need more control of how the image is scaled, see [this post](http://stackoverflow.com/a/16139059) using a FrameLayout. In short, if you want to control the scale or stretch type of an image, you must **use an ImageView** that is "match_parent" for both width and height of the parent container.

#### My image isn't loading and I am seeing a memory error instead in the logs

This probably means that the drawable image being used is a large resolution. The easiest fix is to simply re-copy the image as `xxhdpi` density when you copy and paste the image into Android Studio. Note that when adding an image you are prompted to select the density. If you select `xxhdpi` the image will likely be able to be loaded. In the event that you still see this error, **resize the image to a maximum of 900 x 1600px**.

#### How would I create a toggle button that alternates between two images?

For this, we'd use a custom [ToggleButton as described here](http://mirhoseini.com/how-to-create-a-toggle-button-with-custom-image-and-no-text-in-android/) which has a different image applied for the checked and unchecked states.

#### How would I display an animated GIF image?

To display animated GIFs, we can't use Picasso. Instead, we can use the alternative image loading library called [[Glide from Bumptech|http://guides.codepath.com/android/Animations#animated-images]].

### Views

#### How do I display a "toast" to display text on an activity?

Toast is a quick alert that can be displayed with:

```java
Toast.makeText(MyActivity.this, "some message", Toast.LENGTH_SHORT).show();
```

Review the [[Displaying Toasts]] guide for more details on how to display a toast. 

#### How do I setup click handlers for my views or buttons?

Any view can have a click handler setup by [[following this event handler guide|Basic-Event-Listeners]] which triggers an action when the click occurs. 

#### How do I control the capitalization of text in views?

For views that display text, if the view such as a button displays text in all caps and you want to disable that set the `android:textAllCaps="false"` property to change that behavior. This behavior is the default starting with API 21. 

For **EditText** or other views that accept input, use the [android:capitalize](http://developer.android.com/reference/android/widget/TextView.html#attr_android:capitalize) attribute (deprecated from API 3, using `android:inputType` instead) to change the capitalization of text entered by the user.

#### How can I add an image on the left of the text in my button?

First create your [image icons with this generator](http://romannurik.github.io/AndroidAssetStudio/icons-launcher.html) or in Android Studio with `New => Image Asset` and then you can add the image to the left or right with `drawableLeft` or `drawableRight`:

```xml
<TextView
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:layout_centerHorizontal="true"
    android:drawableLeft="@drawable/icon"
    android:drawablePadding="-20sp"
    android:text="blah blah blah" />
```

Note the use of padding to remove the extra spacing added to the text when the drawable is inserted. See the [[textview guide|Working-with-the-TextView#displaying-images-within-a-textview]] for more details.

#### How can I add padding between my `drawableStart` or `drawableLeft` and the text in my EditText?

You can add the `android:drawablePadding` attribute to your EditText. Example layout with 10dp drawable padding:

```xml
<EditText
    android:id="@+id/emailEditText"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:drawableLeft="@drawable/email_drawable"
    android:drawablePadding="10dp"
    android:singleLine="true" />
```

Read more [details on this stackoverflow post](http://stackoverflow.com/a/23608015).

#### How do I support text with rich formatting (bold words, links) in a TextView?

The `TextView` has [[basic support for HTML text formatting|Working-with-the-TextView#inserting-html-formatting]]:

```java
TextView view = (TextView)findViewById(R.id.sampleText);
view.setText(Html.fromHtml("This <i>is</i> a <b>test</b>"));
```

and can even [[autolink URLs contained within the text|Working-with-the-TextView#autolinking-urls]]:

```xml
<TextView
     android:id="@+id/custom_font"
     android:layout_width="match_parent"
     android:layout_height="wrap_content"
     android:autoLink="all"
     android:linksClickable="true"
/>
```

Refer to the [official TextView docs](http://developer.android.com/reference/android/widget/TextView.html) for other details.

#### How do I support different fonts in my app?

Fonts can be customized fairly easily using this [[custom fonts|Working-with-the-TextView#using-custom-fonts]] guide. Be aware that custom fonts can cause performance issues if used too much.

#### How do I change the color of the bottom line indicator for an `EditText` text field?

The easiest way to change the bottom line indicator is to [[follow this EditText guide|Working-with-the-EditText#changing-the-bottom-line-color]] to modify the `colorControl` properties. 

Alternatively, we can use this [holo theme generator](http://android-holo-colors.com/) to change the color. Choose the desired color and select "YES" for EditText and then drag the generated resources into your 'res' folder. See [this stackoverflow post](http://stackoverflow.com/questions/12470487/how-to-change-the-color-of-the-line-in-ics-styled-edittext) for more details.

#### How can I configure the properties of an `EditText` text field?

See this [[guide for working with the EditText|Working-with-the-EditText]] for more information on configuring and accepting text input from a user.

#### How do I customize the border or outline of a text field or another view?

You can add a border to any view by creating a "drawable shape xml" and applying that as the `android:background` of the view. Create an XML file in `res/drawable` called `shape_view_border.xml`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android">
    <solid android:color="#FFFFFF" />
    <stroke android:width="1dp" android:color="#000000" />
</shape>
```

The `stroke` is the border properties and `solid` is the background color of the rest of the view. You can apply this border and background to any view with:

```xml
<EditText
    android:background="@drawable/shape_view_border"  
    ...
/>
```

See [this stackoverflow post about setting borders](http://stackoverflow.com/a/3264140) for more details.  If you want to have a border on just one edge of a view, this is unfortunately more difficult to do but can be achieved with layer lists as [described in this post about borders on one edge](http://stackoverflow.com/a/4313329).

#### How can I create a radial (circular) gradient background for a view?

Simple radial gradients can be applied as a background to any view using a [shape drawable](http://guides.codepath.com/android/Drawables#gradient-colored-shapes). You should create a drawable in `res/drawable/bg_gradient_1.xml` for example with the following parameters:

```xml
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="rectangle">
    <gradient android:type="radial" android:gradientRadius="250"
        android:startColor="#E9E9E9" android:endColor="#D4D4D4" />
</shape> 
```

We can then apply that shape drawable to the background of any view or layout with:

```xml
<SomeView
    android:background="@drawable/bg_gradient_1"  
    ...
/>
```

Refer to the [gradient drawable section](http://guides.codepath.com/android/Drawables#gradient-colored-shapes) for more details on drawing shapes and applying them to views. 

#### How do I remove the grey border from an ImageButton?

You can remove the border from a `ImageButton` by either setting `android:background` to "@android:color/transparent":

```xml
<ImageButton
     android:background="@android:color/transparent"
     android:src="@drawable/fancy_button"
     ...
/>
```

Using this code the border on the button will be removed.

#### How do I customize the style of a button or any view?

Styling a button requires the use of either image assets (see the [ImageButton](http://developer.android.com/reference/android/widget/ImageButton.html)) or alternatively applying the concept of [[custom drawables|Drawables#customizing-a-button]]. For example, to style a button with drawables, you could create a shape at `res/drawable/shape_fancy_button.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android" android:shape="rectangle">
    <gradient android:startColor="#0078a5"  android:endColor="#00adee"  android:angle="90"/>
    <stroke android:width="1dp" android:color="#0076a3" />
    <corners android:radius="8dp" />
</shape>
```

This would create a button with a background gradient, 1dp border (stroke) and rounded corners. You could then apply this to the button with:

```xml
<Button
    android:background="@drawable/shape_fancy_button"  
    ...
/>
```

For readymade solutions, check out [these nice looking pre-built buttons](http://www.dibbus.com/2011/03/9patch-images-in-android/) or [this handy button generator](http://angrytools.com/android/button/).

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

#### How would I toggle the text color of a button or view in different states?

You can use a color selector drawable stored in the `res/color/states_button_color.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
 <selector xmlns:android="http://schemas.android.com/apk/res/android">
     <item android:state_pressed="true" android:color="#000000" /> <!-- pressed -->
     <item android:color="#FFFFFF" /> <!-- default -->
 </selector>
```

and then applied to a button with:

```xml
<Button
       android:layout_width="wrap_content"
       android:layout_height="wrap_content"
       android:textColor="@color/states_button_color" />
```

See [this stackoverflow post](http://stackoverflow.com/a/3565624/313399) for more details.

#### How do I hide a view at runtime or make the view collapse within the layout?

Any view can be hidden by calling the `setVisibility` property and setting the `Visibility` flag:

```java
// This view is invisible, but it still takes up space for layout purposes.
view.setVisibility(View.INVISIBLE);
// This view is invisible, and it doesn't take any space for layout purposes.
view.setVisibility(View.GONE);
// This view is visible and takes up space within the layout
view.setVisibility(View.VISIBLE);
```

You can use `View.INVISIBLE` to hide the element or `View.GONE` to collapse the element entirely.

#### How would I dynamically color text or a view based on the color palette of an image?

We can use the new [[Palette system|Dynamic-Color-using-Palettes]] to easily color any text or backgrounds based on the color scheme of an image at runtime.

#### How would I create responsive views with different spacing or heights based on device?

Rather than hardcoding units such as `10dp` or `21sp` into the XML layouts, we can use a `dimens.xml` file to declare our dimensions instead. In `res/values/dimens.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <!-- Default screen margins, per the Android Design guidelines. -->
    <dimen name="activity_horizontal_margin">16dp</dimen>
    <dimen name="activity_vertical_margin">16dp</dimen>
    <dimen name="navigation_drawer_width">360dp</dimen>
</resources>
```

and then we can use these in our layout files with `@dimen/activity_horizontal_margin` such as:

```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:paddingLeft="@dimen/activity_horizontal_margin">
</RelativeLayout>
```

Using the powerful [[alternative resources feature|Understanding-App-Resources#providing-alternate-resources]], we can easily create alternate `dimens.xml` files for different device types. 

#### What is the difference between `sp` and `dp` units?

`dp` are density-independent pixels and they correspond to pixels depending on the physical density. `sp` is a scale-independent pixel which is scaled when the `Large Text` option is turned on in the device's settings. In short: Use `sp` for Text sizes. Use `dp` for padding, heights, and everything else.

### ActionBar

#### How would I hide the top ActionBar?

You can hide the top bar in any Activity by modifying the "theme" of an Activity in the `AndroidManifest.xml` such as:

```xml
<activity android:name=".Activity"
    android:label="@string/app_name"
    android:theme="@style/Theme.AppCompat.Light.NoActionBar">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>
```

You can also hide the ActionBar programmatically at runtime in Java with `getSupportActionBar().hide()` within the `onCreate` method after `setContentView` in an Activity.

#### How do I change the background color of the ActionBar or Toolbar?

Customize the theme of the ActionBar in your app using the `res/values/styles.xml` as detailed in the [[Advanced ActionBar|Extended-ActionBar-Guide#custom-actionbar-styles]] to adjust the color of the ActionBar. If your app is using the `Toolbar` instead, then customize that using the [[styling the Toolbar guide|Using-the-App-ToolBar#styling-the-toolbar]].

#### How do I change the text color of the ActionBar or Toolbar?

Customize the theme of the ActionBar in your app using the `res/values/styles.xml` as detailed in the [[Advanced ActionBar|Extended-ActionBar-Guide#custom-actionbar-styles]] to adjust the color of the ActionBar. If your app is using the `Toolbar` instead, then customize that using the [[styling the Toolbar guide|Using-the-App-ToolBar#styling-the-toolbar]].

#### How do I add the icon to my ActionBar in API 21 or above?

In the new Android 5.0 material design guidelines, the style guidelines have changed to **discourage the use of the icon** in the ActionBar. Although the icon can be added back with:

```java
getSupportActionBar().setDisplayShowHomeEnabled(true);
getSupportActionBar().setLogo(R.drawable.ic_launcher);
getSupportActionBar().setDisplayUseLogoEnabled(true);
```

You can read more about this on the [material design guidelines](https://developer.android.com/reference/android/support/v7/widget/Toolbar.html) which state: "The use of application icon plus title as a standard layout is discouraged on API 21 devices and newer." 

#### How do I style the title in the ActionBar?

ActionBar title can be styled or centered only if you opt to customize the XML view being inflated to display in the ActionBar. Instead of using the default ActionBar text you can specify your own to use instead that is styled as desired. Check out the [[Advanced ActionBar|Extended-ActionBar-Guide#custom-actionbar-layout]] cliffnotes for details.

#### How can I add additional spacing between action items in the ActionBar?

One way is to use the XML drawable system to create a drawable that combines the icon with extra spacing using the "layer-list". See [the full instructions here](https://futurestud.io/tutorials/android-quick-tips-9-how-to-add-padding-to-actionbar-menu-icons).