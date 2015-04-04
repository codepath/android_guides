## Overview

Material Design encourages dynamic use of color, especially when you have rich images to work with. The [new Palette support library](https://www.youtube.com/watch?v=97SWYiRtF0Y&t=1903) lets you extract a small set of colors from an image to style your UI controls to match; creating an immersive experience. The extracted palette will include vibrant and muted tones as well as foreground text colors for optimal legibility.

Make sure to add the Palette dependency to your `build.gradle` file if your targeted version of Android is lower than 21.

```gradle
compile 'com.android.support:palette-v7:21.0.+'
```

There are two ways to create a Palette and extract the colors from an image:

* Synchronous
* Asynchronous

### Synchronous Methods

These should be used when you have access to the underlying image loading thread. 

```java
public Palette generate(Bitmap image)
```

Pass only the image you want to extract the colors from as a parameter. This uses a default number of 15 colors, which should be enough to generate colors that correspond to the special set.

```java
public Palette generate(Bitmap image, int numberOfColors)
```

Pass the image and specify the number of colors you want to generate from the image. This allows you to specify the maximum palette size of 24.

### Asynchronous Methods

There is an asynchronous method that uses an `AsyncTask` to gather the Palette swatch information from the bitmap:

```java
// This is the quick and easy integration path. 
// May not be optimal (since you're dipping in and out of threads)
Palette.generateAsync(bitmap, new Palette.PaletteAsyncListener() {
    @Override
    public void onGenerated(Palette palette) {
         // Get the "vibrant" color swatch based on the bitmap
         Palette.Swatch vibrant = palette.getVibrantSwatch();
          if (vibrant != null) {
              // Set the background color of a layout based on the vibrant color
              containerView.setBackgroundColor(vibrant.getRgb());
              // Update the title TextView with the proper text color
              titleView.setTextColor(vibrant.getTitleTextColor());
          }
    }
});
```

This method takes in the image to extract the colors from, and a listener for a callback when the asynchronous task finishes. 

### Palette Properties

When a palette is generated, it tries to pick [six swatches](https://developer.android.com/reference/android/support/v7/graphics/Palette.html) which match certain criteria:

 * Vibrant. `Palette.getVibrantSwatch()`
 * Vibrant dark. `Palette.getDarkVibrantSwatch()`
 * Vibrant light. `Palette.getLightVibrantSwatch()`
 * Muted. `Palette.getMutedSwatch()`
 * Muted dark. `Palette.getDarkMutedSwatch()`
 * Muted light. `Palette.getLightMutedSwatch()`

Which one you choose depends on your use case. Vibrant and Dark Vibrant are the ones that developers will use mostly though.

### Swatch Properties

Each Swatch contains the following methods:

 * `getPopulation()`: the amount of pixels which this swatch represents.
 * `getRgb()`: the RGB value of this color.
 * `getHsl()`: the HSL value of this color.
 * `getBodyTextColor()`: the RGB value of a text color which can be displayed on top of this color.
 * `getTitleTextColor()`: the RGB value of a text color which can be displayed on top of this color.

The different text colors roughly match up to the material design standard styles of the same name. The title text color will be more translucent as the text is larger and thus requires less color contrast. The body text color will be more opaque as text is smaller and thus requires more contrast from color.

### Adjusting the Number of Colors

We can also specify the maximum number of colors in the generated palette (`numColors`) for the given bitmap:

```java
// Allows you to specify the maximum palette size, in this case 24.
// Increasing the number of colors increasing computation time
Palette.generateAsync(bitmap, 24, new Palette.PaletteAsyncListener() {
    // ...
});
```

Good values for `numColors` depend on the source image type. For landscapes, a good values are in the range 12-16. For images which are largely made up of people's faces then this value should be increased to 24-32.

## References

* <https://chris.banes.me/2014/10/20/palette-v21/>
* <http://www.willowtreeapps.com/blog/palette-the-new-api-for-android/>
* <http://android-developers.blogspot.com/2014/10/implementing-material-design-in-your.html>
