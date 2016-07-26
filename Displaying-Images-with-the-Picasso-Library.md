## Usage

Displaying images is easiest using a third party library such as [Picasso](http://square.github.io/picasso/) from Square which will download and cache remote images and abstract the complexity behind an easy to use DSL.

### Setup Picasso 

Adding Picasso to our `app/build.gradle` file:

```gradle
dependencies {
    compile 'com.squareup.picasso:picasso:2.5.2'
}
```

**Note**: there is a bug with the current version of Picasso that prevents large images (i.e. 10MB) from being loaded, especially with newer camera phones that have larger resolutions.   If you are experiencing [this issue](https://github.com/square/picasso/issues/364), you may need to upgrade to the Picasso 2.6.0 snapshot.  See the [[troubleshooting|Displaying Images with the Picasso Library#troubleshooting]] guide to confirm.

To use this snapshot version, you need to add a custom separate Maven repo first:

```gradle
repositories {
    maven { url "https://oss.sonatype.org/content/repositories/snapshots" }
}

// add directly below repositories section
dependencies {
    compile 'com.squareup.picasso:picasso:2.6.0-SNAPSHOT'
}
```

### Loading an Image from Url

We can then load a remote image into any `ImageView` with:

```java
String imageUri = "https://i.imgur.com/tGbaZCY.jpg";
ImageView ivBasicImage = (ImageView) findViewById(R.id.ivBasicImage);
Picasso.with(context).load(imageUri).into(ivBasicImage);
```

### Configuring Picasso

We can do more sophisticated work with Picasso configuring placeholders, error handling, adjusting size of the image, and scale type with:

```java
Picasso.with(context).load(imageUri).fit().centerCrop()
    .placeholder(R.drawable.user_placeholder)
    .error(R.drawable.user_placeholder_error)
    .into(imageView);
```

Be sure to use `fit()` to resize the image before loading into the ImageView.  Otherwise, you will consume extra memory, experience sluggish scrolling, or encounter out of memory issues if you render a lot of pictures. In addition to `placeholder` and `error`, there is also [other configuration options](https://futurestud.io/blog/picasso-placeholders-errors-and-fading) such as `noFade()` and `noPlaceholder()`.

**Placeholder Note:** Placeholders and error images are **not resized** and must be fairly small images. Open up your static placeholder or error images in your drawable folders and make sure that the dimensions of the images are relatively small (i.e < 500px width). If not, resize those static images first and save them back to the project.

### Resizing with Picasso

We can resize an image with respect to the aspect ratio using `resize` and specifying 0 for the other dimension as [outlined here](https://github.com/square/picasso/pull/663):

```java
// Resize to the width specified maintaining aspect ratio
Picasso.with(this).load(imageUrl).
  resize(someWidth, 0).into(imageView);
```

We can combine resizing with certain transforms to make the image appear differently. For example, we can do a center cropping with:

```java
Picasso.with(context).load(url).resize(50, 50).
  centerCrop().into(imageView);
```

Transform options include `centerCrop()` (Crops an image inside of the bounds), `centerInside()` (Centers an image inside of the bounds), `fit()` (Attempt to resize the image to fit exactly into the target). See [this post for more details](https://futurestud.io/blog/picasso-image-resizing-scaling-and-fit).

## Troubleshooting

### OutOfMemoryError Loading Errors

If an image or set of images aren't loading, make sure to check the Android monitor log in Android Studio. There's a good chance you might see an `java.lang.OutOfMemoryError "Failed to allocate a [...] byte allocation with [...] free bytes"` or a `Out of memory on a 51121168-byte allocation.`. This is quite common and means that **you are loading one or more large images** that have not been properly resized.

First, you have to find which image(s) being loaded are likely causing this error. For any given `Picasso` call, we can fix this by **one or more of the following approaches**:

1. Add an explicit width or height to the `ImageView` by setting `layout_width=500dp` in the layout file and then be sure to call `fit()` during your load: `Picasso.with(...).load(imageUri).fit().into(...)`
1. Call `.resize(width, height)` during the Picasso load and explicitly set a width or height for the image such as: `Picasso.with(...).load(imageUri).resize(500, 0).into(...)`. By passing 0, the correct height is automatically calculated.
1. Try removing `android:adjustViewBounds="true"` from your `ImageView` if present if you are calling `.fit()` rather than using `.resize(width, 0)`
1. Open up your static placeholder or error images and make sure their dimensions are relatively small (< 500px width). If not, resize those static images and save them back to your project.

Applying these tips to all of your Picasso image loads should resolve any out of memory issues. As a fallback, you might want to open up your `AndroidManifest.xml` and then add `android:largeHeap` to your manifest:

```xml
<application
        android:name=".MyApplication"
        ...
        android:largeHeap="true"
        ...
```

Note that this is not generally a good idea, but can be used temporarily to trigger less out of memory errors.

### Loading Errors

If you experience errors loading images, you can attach a listener to the `Builder` object to print the stack trace.

```java
Picasso.Builder builder = new Picasso.Builder(getApplicationContext());
builder.listener(new Picasso.Listener() {
     @Override
     public void onImageLoadFailed(Picasso picasso, Uri uri, Exception exception) {
         exception.printStackTrace();
    });
```

## Advanced Usages

### Showing ProgressBar with Picasso

We can add a progress bar or otherwise handle callbacks for an image that is loading with:

```java
// Show progress bar
progressBar.setVisibility(View.VISIBLE);
// Hide progress bar on successful load
Picasso.with(this).load(imageUrl)
  .into(imageView, new com.squareup.picasso.Callback() {
      @Override
      public void onSuccess() {
          if (progressBar != null) {
              progressBar.setVisibility(View.GONE);
          }
      }

      @Override
      public void onError() {

      }
});
```

### Adjusting Image Size Dynamically

If we wish to readjust the ImageView size after the image has been retrieved, we first define a `Target` object that governs how the Bitmap is handled:

```java
private Target target = new Target() {
    @Override
    public void onBitmapLoaded(Bitmap bitmap, Picasso.LoadedFrom from) {  
       // Bitmap is loaded, use image here
       imageView.setImageBitmap(bitmap);
    }

    @Override
    public void onBitmapFailed() {
        // Fires if bitmap couldn't be loaded.
    }
}
```

Next, we can use the `Target` with a Picasso call with:

```java
Picasso.with(this).load("url").into(target);
```

You can still use all normal Picasso options like `resize`, `fit`, etc.

**Note:** The `Target` object must be **stored as a member field or method** and cannot be an anonymous class otherwise this won't work as expected.  The reason is that Picasso accepts this parameter as a weak memory reference.  Because anonymous classes are eligible for garbage collection when there are no more references, the network request to fetch the image may finish after this anonymous class has already been reclaimed.  See this [Stack Overflow](http://stackoverflow.com/questions/24180805/onbitmaploaded-of-target-object-not-called-on-first-load#answers) discussion for more details.

In other words, you are not allowed to do `Picasso.with(this).load("url").into(new Target() { ... })`.   

#### Creating Staggered Grid Images with RecyclerView

We can use this custom Target approach to create a staggered image view using `RecyclerView`.

<img src="https://i.imgur.com/gsp1prk.png" width="300"/>

We first need to replace the `ImageView` with the [DynamicHeightImageView.java](https://github.com/etsy/AndroidStaggeredGrid/blob/master/library/src/main/java/com/etsy/android/grid/util/DynamicHeightImageView.java) that enables us to update the `ImageView` width and height while still preserving the aspect ratio when new images are replaced with old recycled views. We can then set the ratio before the image has loaded if we already know the height:width ratio using `onBindViewHolder` as shown below:

```java
public class PhotosAdapter extends RecyclerView.Adapter<PhotoViewHolder> {
    // implement other methods here

    @Override
    public void onBindViewHolder(PhotoViewHolder holder, int position) {
        Photo photo = mPhotos.get(position);
        // `holder.ivPhoto` should be of type `DynamicHeightImageView`
        // Set the height ratio before loading in image into Picasso
        holder.ivPhoto.setHeightRatio(((double)photo.getHeight())/photo.getWidth());
        // Load the image into the view using Picasso
        Picasso.with(mContext).load(photo.getUrl()).into(holder.ivPhoto);
    }

}
```

Alternatively, we can set the ratio after the bitmap has loaded if we don't know that ratio ahead of time. To avoid using an anonymous class, we will implement the `Target` interface on the ViewHolder class itself for RecyclerView. When the callback is fired, we will calculate and update the image aspect ratio:

```java
public class ViewHolder extends RecyclerView.ViewHolder implements View.OnClickListener, Target {
    DynamicHeightImageView ivImage;

    // implement ViewHolder methods here

    @Override
    public void onBitmapLoaded(Bitmap bitmap, Picasso.LoadedFrom from) {
        // Calculate the image ratio of the loaded bitmap
        float ratio = (float) bitmap.getHeight() / (float) bitmap.getWidth();
        // Set the ratio for the image 
        ivImage.setHeightRatio(ratio);
        // Load the image into the view
        ivImage.setImageBitmap(bitmap);
    }
}
```

With either of these approaches the staggered grid of images should now render as expected.

### Other Transformations

You can also use this [third-party library](https://github.com/wasabeef/picasso-transformations) for other transformations, such as blur, crop, color, and mask.  
```gradle
dependencies {
    compile 'jp.wasabeef:picasso-transformations:2.1.0'
    // If you want to use the GPU Filters
    compile 'jp.co.cyberagent.android.gpuimage:gpuimage-library:1.4.1'
}
```

To do a rounded corner transformation, you would do the following:

```java
Picasso.with(mContext).load(R.drawable.demo)
        .transform(new RoundedCornersTransformation(10, 10)).into((ImageView) findViewById(R.id.image));
```

## References

* <http://square.github.io/picasso/>