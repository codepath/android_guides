## Overview

Material Design encourages dynamic use of color, especially when you have rich images to work with. The new Palette support library lets you extract a small set of colors from an image to style your UI controls to match; creating an immersive experience. The extracted palette will include vibrant and muted tones as well as foreground text colors for optimal legibility. For example:

```java
// This is the quick and easy integration path. Internally uses an AsyncTask so 
// this may not be optimal (since you're dipping in and out of threads)
Palette.generateAsync(bitmap, new Palette.PaletteAsyncListener() {
    @Override
    public void onGenerated(Palette palette) {
         Palette.Swatch vibrant = palette.getVibrantSwatch();
          if (swatch != null) {
              // If we have a vibrant color, update the title TextView
              titleView.setBackgroundColor(vibrant.getRgb());
              titleView.setTextColor(vibrant.getTitleTextColor());
          }
    }
});
```

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
* <http://android-developers.blogspot.com/2014/10/implementing-material-design-in-your.html>