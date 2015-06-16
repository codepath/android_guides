## Overview

Material Design encourages dynamic use of color, especially when you have rich images to work with. The [new Palette support library](https://www.youtube.com/watch?v=97SWYiRtF0Y&t=1903) lets you extract a small set of colors from an image to style your UI controls to match; creating an immersive experience. The extracted palette will include vibrant and muted tones as well as foreground text colors for optimal legibility.

Make sure to add the Palette dependency to your `build.gradle` file if your targeted version of Android is lower than 21.

```gradle
compile 'com.android.support:palette-v7:21.0.+'
```

## Generating the Palette

The [Palette.Builder](https://developer.android.com/reference/android/support/v7/graphics/Palette.Builder.html) allows for creation of a [Palette](https://developer.android.com/reference/android/support/v7/graphics/Palette.html). It provides methods to set the maximum number of colors in the generated palette and to generate the palette synchronously or asynchronously.

### Synchronously

Generate the palette synchronously when you have access to the underlying image loading thread. 

```java
Palette.from(bitmap).generate();
```
```java
Palette.from(bitmap).maximumColorCount(numberOfColors).generate();
```

The passed in `bitmap` is the image from which you want to extract the colors. By default, it will use a maximum palette size of 16 colors. This can be overridden using the `maximumColorCount` method.

### Asynchronously

By passing in a [PaletteAsyncListener](https://developer.android.com/reference/android/support/v7/graphics/Palette.PaletteAsyncListener.html) to the `generate` method, it will now generate the palette asynchronously using an `AsyncTask` to gather the Palette swatch information from the bitmap:

```java
// This is the quick and easy integration path. 
// May not be optimal (since you're dipping in and out of threads)
Palette.from(bitmap).maximumColorCount(numberOfColors).generate(new Palette.PaletteAsyncListener() {
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
### Adjusting the Number of Colors

Good values for `numberOfColors` depend on the source image type. For landscapes, good values are in the range 12-16. For images which are largely made up of people's faces then this value should be increased to 24-32. Keep in mind that increasing the number of colors increases the computation time.

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

## References

* <https://chris.banes.me/2014/10/20/palette-v21/>
* <http://www.willowtreeapps.com/blog/palette-the-new-api-for-android/>
* <http://android-developers.blogspot.com/2014/10/implementing-material-design-in-your.html>