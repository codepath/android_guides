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

### Showing ProgressBar with Picasso

We can add a progress bar or otherwise handle callbacks while an image is loading with:

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