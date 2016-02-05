## Usage

Displaying images is easiest using a third party library such as [Picasso](http://square.github.io/picasso/) from Square which will download and cache remote images and abstract the complexity behind an easy to use DSL.

### Setup Picasso 

Adding Picasso to our `app/build.gradle` file:

```gradle
dependencies {
    compile 'com.squareup.picasso:picasso:2.5.2'
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

Be sure to use `fit()` to resize the image before loading into the ImageView.  Otherwise, you will consume extra memory, experience sluggish scrolling, or encounter out of memory issues if you render a lot of pictures.

### Resizing with Picasso

We can resize an image with respect to the aspect ratio using `resize` and specifying 0 for the other dimension as [outlined here](https://github.com/square/picasso/pull/663):

```java
// Resize to the width specified maintaining aspect ratio
Picasso.with(this).load(imageUrl).
  resize(someWidth, 0).into(imageView);
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

We first need to use [DynamicHeightImageView.java](https://github.com/etsy/AndroidStaggeredGrid/blob/master/library/src/main/java/com/etsy/android/grid/util/DynamicHeightImageView.java) that enables us to update the ImageView width and height while still preserving the aspect ratio when new images are replaced with old recycled views. We can set the ratio before the image has loaded if we already know the height:width ratio using `onBindViewHolder` as shown below:

```java
public class PhotosAdapter extends RecyclerView.Adapter<PhotoViewHolder> {
    // implement other methods here

    @Override
    public void onBindViewHolder(PhotoViewHolder holder, int position) {
        Photo photo = mPhotos.get(position);
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

Now the staggered grid of images should render as expected.

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

For more details check out the [Picasso](http://square.github.io/picasso/) documentation. 

## References

* <http://square.github.io/picasso/>