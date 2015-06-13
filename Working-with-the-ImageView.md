## Overview

Typically, images are displayed using the built-in image view. This view takes care of the loading and optimizing of the image, freeing you to focus on app-specific details like the layout and content. 

In this guide, we will take a look at how to use an ImageView, how to manipulate bitmaps, learn about the different density folders and more.

## Usage

At the simplest level, an [ImageView](http://developer.android.com/reference/android/widget/ImageView.html) is simply a view you embed within an XML layout that is used to display an image (or any drawable) on the screen. The ImageView looks like this in `res/layout/activity_main.xml`:

```xml
<ImageView
    android:id="@+id/image"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:scaleType="center"
    android:src="@drawable/my_image" />
```

The ImageView handles all the loading and scaling of the image for you. Note the [scaleType attribute](http://developer.android.com/reference/android/widget/ImageView.ScaleType.html) which defines how the images will be scaled to fit in your layout. In the example, using scaleType "center", the image will be displayed at its native resolution and centered in the view, regardless of how much space the view consumes.

### Sizing ImageView Controls

By default, contents of an ImageView control are of a certain size -- usually the size of the image dimensions. They can also be bounded by their layout_width and layout_height attributes: 

```xml
<ImageView
    android:layout_width="50dp"
    android:layout_height="50dp"
    android:scaleType="fitXY"
    android:adjustViewBounds="true"
    ...
/>
```

In the above we are also  specifying that the ImageView preserve its aspect ratio using the [adjustViewBounds](http://developer.android.com/reference/android/widget/ImageView.html#attr_android:adjustViewBounds) attribute. The `scaleType` above has been set to `fitXY` which scales both the height and the width up or down until it fits within the maximum dimensions specified. By combining these properties together we can control the rough size of the image and still adjust the image according to the proper aspect ratio.

We can also size an `ImageView` at runtime within our Java source code by modifying the `width` or `height` inside `getLayoutParams()` for the view: 

```java
imageView.getLayoutParams().height = 100; // OR
imageView.getLayoutParams().width = 100;
```

In certain cases, the image needs to be scaled to fit the parent view's width and the height should be adjusted proportionally. We can achieve this using an extended [ResizableImageView](http://stackoverflow.com/a/12283909/313399) class as described in the post.

### Scale Types

An ImageView can display an image differently based on the `scaleType` provided. Above we discussed the `fitXY` type along with `adjustViewBounds`. The following is a list of all the most common types:

| Scale Type | Description |
| ---------- | ----------- |
| center     | Displays the image centered in the view with no scaling. |
| centerCrop | Scales the image such that both the x and y dimensions are greater than or equal to the view, while maintaining the image aspect ratio; centers the image in the view. |
| centerInside | Scales the image to fit inside the view, while maintaining the image aspect ratio. If the image is already smaller than the view, then this is the same as center. |
| fitCenter | Scales the image to fit inside the view, while maintaining the image aspect ratio. At least one axis will exactly match the view, and the result is centered inside the view. |
| fitStart | Same as fitCenter but aligned to the top left of the view. |
| fitEnd   | Same as fitCenter but aligned to the bottom right of the view. |
| fitXY | Scales the x and y dimensions to exactly match the view size; does not maintain the image aspect ratio. |
| matrix | Scales the image using a supplied Matrix class. The matrix can be supplied using the setImageMatrix method. A Matrix class can be used to apply transformations such as rotations to an image. |

**Note:** The `fitXY` scale type allows you to set the exact size of the image in your layout. However, be mindful of potential distortions of the image due to scaling. If you’re creating a photo-viewing application, you will probably want to use the `center` or `fitCenter` scale types.

![Screens](http://i.imgur.com/vZHC82o.jpg)

**Pictured**: Examples of `android:scaleType` attribute.<br/>
Top row (l-r) center, centerCrop, centerInside.<br/>
Bottom row (l-r): fitCenter, fitStart, fitEnd, fitXY.

### Supporting Multiple Densities

Since Android has so many different screen sizes, resolutions and densities, there is a system for selecting the correct image asset for the correct device. There are specific drawable folders for each device density category including: ldpi (low), mdpi (medium), hdpi (high), and xhdpi (extra high). Notice that every app has folders for image drawables such as `drawable-mdpi` which is for "medium dots per inch". 

To create alternative bitmap drawables for different densities, you should follow the 3:4:6:8 scaling ratio between the four generalized densities. Refer to the chart below:

| Density  | DPI  | Example Device  | Scale  | Pixels        | 
|--------- |----  | ----------------| ------ | ----------   | 
| ldpi     | 120  | Galaxy Y        | 0.75x  | 1dp = 0.75px |
| mdpi     | 160  | Galaxy Tab      | 1.0x   | 1dp = 1px    |
| hdpi     | 240  | Galaxy S II	    | 1.5x   | 1dp = 1.5px  |
| xhdpi    | 320  | Nexus 4         | 2.0x   | 1dp = 2px    |
| xxhdpi   | 480  | Nexus 5         | 3.0x   | 1dp = 3px    |
| xxxhdpi  | 640  | Nexus 6         | 4.0x   | 1dp = 4px    |

This means that if you generate a 100x100 for mdpi (1x baseline), then you should generate the same resource in 150x150 for hdpi (1.5x), 200x200 image for xhdpi devices (2.0x), 300x300 image for xxhdpi (3.0x) and a 75x75 image for ldpi devices (0.75x). See [these density guidelines](http://iconhandbook.co.uk/reference/chart/android/) for additional details. 

![Densities](http://developer.android.com/images/screens_support/screens-densities.png)

To resize images more easily, check out the [Final Android Resizer](https://github.com/asystat/Final-Android-Resizer) by [downloading and running this JAR](https://github.com/asystat/Final-Android-Resizer/blob/master/Executable%20Jar/Final%20Android%20Resizer.jar?raw=true). 

<a href="https://github.com/asystat/Final-Android-Resizer/blob/master/Executable%20Jar/Final%20Android%20Resizer.jar?raw=true"><img src="http://i.imgur.com/Usx4iXH.png" alt="final resizer" width="350" /></a>

This handy utility allows us to select a resources directory, choose an extra high density image and the tool will automatically generate the corresponding lower size images for us and place them in the correct folders.

See the [screens support](http://developer.android.com/guide/practices/screens_support.html) reference for a more detailed look at supporting a wide range of devices. Also check out the [iconography](http://developer.android.com/design/style/iconography.html) guide for more details.

### Mipmaps and Drawables

Starting with Android 4.3, there is now an option to use the `res/mipmap` folder to store "mipmap" images. Mipmaps are most **commonly used for application icons** such as the launcher icon. To learn more about the benefits of mipmaps be sure to check out the [mipmapping for drawables post](https://programmium.wordpress.com/2014/03/20/mipmapping-for-drawables-in-android-4-3/). 

Mipmap image resources can then be accessed using the `@mipmap/ic_launcher` notation in place of `@drawable`. Placing icons in mipmap folders (rather than drawable) is considered a best practice because they can often be used at resolutions different from the device’s current density. For example, an `xxxhdpi` app icon might be used on the launcher for an `xxhdpi` device. Review this [post about preparing for the Nexus 6](http://android-developers.blogspot.com/2014/10/getting-your-apps-ready-for-nexus-6-and.html) which explains in more detail.

### Working with Bitmaps

We can change the bitmap displayed in an ImageView to a drawable resource with:

```java
ImageView image = (ImageView) findViewById(R.id.test_image);
image.setImageResource(R.drawable.test2);
```

or to any arbitrary bitmap with:

```java
ImageView image = (ImageView) findViewById(R.id.test_image);
Bitmap bMap = BitmapFactory.decodeFile("/sdcard/test2.png");
image.setImageBitmap(bMap);
```

### Scaling a Bitmap

If we need to resize a Bitmap, we can call the [createScaledBitmap](http://developer.android.com/reference/android/graphics/Bitmap.html#createScaledBitmap\(android.graphics.Bitmap, int, int, boolean\)) method to resize any bitmap to our desired width and height:

```java
// Load a bitmap from the drawable folder
Bitmap bMap = BitmapFactory.decodeResource(getResources(), R.drawable.my_image);
// Resize the bitmap to 150x100 (width x height)
Bitmap bMapScaled = Bitmap.createScaledBitmap(bMap, 150, 100, true);
// Loads the resized Bitmap into an ImageView
ImageView image = (ImageView) findViewById(R.id.test_image);
image.setImageBitmap(bMapScaled);
```

You often want to resize a bitmap but preserve the aspect ratio [using a BitmapScaler utility class](https://gist.github.com/nesquena/3885707fd3773c09f1bb) with code like this:

```java
public class BitmapScaler
{
	// Scale and maintain aspect ratio given a desired width
	// BitmapScaler.scaleToFitWidth(bitmap, 100);
	public static Bitmap scaleToFitWidth(Bitmap b, int width)
	{
		float factor = width / (float) b.getWidth();
		return Bitmap.createScaledBitmap(b, width, (int) (b.getHeight() * factor), true);
	}


	// Scale and maintain aspect ratio given a desired height
	// BitmapScaler.scaleToFitHeight(bitmap, 100);
	public static Bitmap scaleToFitHeight(Bitmap b, int height)
	{
		float factor = height / (float) b.getHeight();
		return Bitmap.createScaledBitmap(b, (int) (b.getWidth() * factor), height, true);
	}

	// ...
}
```

In other cases, you may want to determine the device height or width in order to resize the image accordingly. Copy this [DeviceDimensionsHelper.java](https://gist.github.com/nesquena/318b6930aac3a56f96a4) utility class to `DeviceDimensionsHelper.java` in your project and use anywhere that you have a context to determine the screen dimensions:

```java
// Get height or width of screen at runtime
int screenWidth = DeviceDimensionsHelper.getDisplayWidth(this);
// Resize a Bitmap maintaining aspect ratio based on screen width
BitmapScaler.scaleToFitWidth(bitmap, screenWidth);
```

Check out [this source](http://androidsnippets.wordpress.com/2012/10/25/how-to-scale-a-bitmap-as-per-device-width-and-height/) for more information on how to scale a bitmap based instead on relative device width and height.

### Displaying SVG Images

Using a third party library called [svg-android](https://code.google.com/p/svg-android/wiki/Tutorial) we can actually display resolution and density independent SVG images. 

```java
// Parse an SVG resource from the "raw" resource folder
SVG svg = SVGParser.getSVGFromResource(getResources(), R.raw.android);
// Fix issue with renderer on certain devices
myImageView.setLayerType(View.LAYER_TYPE_SOFTWARE, null);
// Get a drawable from the parsed SVG and set it as the drawable for the ImageView
myImageView.setImageDrawable(svg.createPictureDrawable());
```

See the [official tutorial](https://code.google.com/p/svg-android/wiki/Tutorial) for more details.

## References

* <http://www.peachpit.com/articles/article.aspx?p=1846580&seqNum=2>
* <http://code.tutsplus.com/tutorials/android-user-interface-design-basic-image-controls--mobile-6846>
* <http://developer.android.com/reference/android/widget/ImageView.html>
* <http://developer.android.com/guide/practices/screens_support.html>
* <https://code.google.com/p/svg-android/wiki/Tutorial>