## Overview
[[Fresco|http://frescolib.org]] is a powerful library for displaying images in Android, supporting applications all the way back to GingerBread (API 9). It downloads and caches remote images in a memory efficient manner, using a special region of non-garbage collected memory on Android called `ashmem`. 

## Terms
When working with Fresco, it's helpful to be familiar with the following terms:
* ImagePipeline - Responsible for getting you the image. It fetches from the network, a local file, a content provider, or a local resource. It keeps a cache of compressed images in local storage, and a second cache of decompressed images in memory.
* Drawee - Drawees deal with rendering images on screen and are made up of 3 parts.
  * DraweeView - The view that shows the image. It extends from `ImageView`, but only for convenience (see the [[gotchas|https://github.com/codepath/android_guides/wiki/Loading-Images-with-Fresco#Gotachas]] below for more info on this). Most of the time you'll be using a `SimpleDraweeView` in your code.
  * DraweeHierarchy - Fresco provides a lot of customization. You can add a placeholderImage, a retryImage, a failureImage, a backgroundImage, etc. The hierarchy is what keeps track of all these drawables and when they should be shown. 
  * DraweeController - This is the class responsible for dealing with the image loader. Fresco allows you to customize the image loader if you don't want to use the provided `ImagePipeline`.

## Getting Started

First, make sure to add the Fresco dependency in your app/build.gradle file:
```gradle
dependencies {
    compile 'com.facebook.fresco:fresco:0.5.2'
}
```
Then, ensure you have the Internet permission if you plan to fetch any images from the network:
```xml
<uses-permission android:name="android.permission.INTERNET"/>
```

Next, make sure to initialize Fresco. Fresco needs to be initialized before you call `setContentView()` in any `Activity` that uses `Fresco`.

```java
Fresco.initialize(context);
```

And then include it in your layout:
```java
<com.facebook.drawee.view.SimpleDraweeView
    android:id="@+id/sdvImage"
    android:layout_width="130dp"
    android:layout_height="130dp" 
    fresco:placeholderImage="@drawable/myPlaceholderImage" />
```
**Note:** If you want to use any Fresco defined properties, you'll need to add a custom namespace definition:
`xmlns:fresco="http://schemas.android.com/apk/res-auto"`

Finally, set the actual image URI:
```java
Uri imageUri = Uri.parse("http://i.imgur.com/tGbaZCY.jpg");
SimpleDraweeView draweeView = (SimpleDraweeView) findViewById(R.id.sdvImage);
draweeView.setImageURI(imageUri);
```

## Customization
That's all you need to get started using Fresco, but Fresco can do much more than that. Below you can find a majority of the properties that Fresco supports.

```java
<com.facebook.drawee.view.SimpleDraweeView
    android:id="@+id/my_image_view"
    android:layout_width="20dp"
    android:layout_height="20dp"
    fresco:fadeDuration="300"
    fresco:actualImageScaleType="focusCrop"
    fresco:placeholderImage="@color/wait_color"
    fresco:placeholderImageScaleType="fitCenter"
    fresco:failureImage="@drawable/error"
    fresco:failureImageScaleType="centerInside"
    fresco:retryImage="@drawable/retrying"
    fresco:retryImageScaleType="centerCrop"
    fresco:progressBarImage="@drawable/progress_bar"
    fresco:progressBarImageScaleType="centerInside"
    fresco:progressBarAutoRotateInterval="1000"
    fresco:backgroundImage="@color/blue"
    fresco:overlayImage="@drawable/watermark"
    fresco:pressedStateOverlayImage="@color/red"
    fresco:roundAsCircle="false"
    fresco:roundedCornerRadius="1dp"
    fresco:roundTopLeft="true"
    fresco:roundTopRight="false"
    fresco:roundBottomLeft="false"
    fresco:roundBottomRight="true"
    fresco:roundWithOverlayColor="@color/corner_color"
    fresco:roundingBorderWidth="2dp"
    fresco:roundingBorderColor="@color/border_color" />
```

**Note:** DraweeView doesn't support specifying `wrap_content` for the `layout_width` or `layout_height` attributes. This is to prevent situations where your placeholder image might be a different size than your actual image, forcing Android to do another layout pass once the actual image comes in.

There is only one case where DraweeView supports `wrap_content` and this is for the helpful `viewAspectRatio` property, allowing you to configure an aspect ratio.

```java
<com.facebook.drawee.view.SimpleDraweeView
    android:id="@+id/my_image_view"
    android:layout_width="20dp"
    android:layout_height="wrap_content"
    fresco:viewAspectRatio="1.33" />
```

You can read more about Fresco's capabilities in the [[Fresco docs|http://frescolib.org/docs/index.html]].

## Getting the underlying Bitmap out of a DraweeView
Getting the bitmap out of a SimpleDraweeView requires working with the `ImagePipeline` instead of the `DraweeView` itself. The code below shows how to get a bitmap out of the `ImagePipeline`, which comes in useful when you want to share an image with another user.
```java
ImagePipeline imagePipeline = Fresco.getImagePipeline();

ImageRequest imageRequest = ImageRequestBuilder
       .newBuilderWithSource(imageUri)
       .setRequestPriority(Priority.HIGH)
       .setLowestPermittedRequestLevel(ImageRequest.RequestLevel.FULL_FETCH)
       .build();

DataSource<CloseableReference<CloseableImage>> dataSource =
       imagePipeline.fetchDecodedImage(imageRequest, mContext);

try {
   dataSource.subscribe(new BaseBitmapDataSubscriber() {
       @Override
       public void onNewResultImpl(@Nullable Bitmap bitmap) {
           if (bitmap == null) {
               Log.d(TAG, "Bitmap data source returned success, but bitmap null.");
               return;
           }
           // The bitmap provided to this method is only guaranteed to be around 
           // for the lifespan of this method. The image pipeline frees the
           // bitmap's memory after this method has completed.
           // 
           // This is fine when passing the bitmap to a system process as
           // Android automatically creates a copy.
           //
           // If you need to keep the bitmap around, look into using a
           // BaseDataSubscriber instead of a BaseBitmapDataSubscriber.
       }

       @Override
       public void onFailureImpl(DataSource dataSource) {
           // No cleanup required here
       }
   }, CallerThreadExecutor.getInstance());
} finally {
   if (dataSource != null) {
       dataSource.close();
   }
}
```

## Gotchas
Make sure you review the list of [[gotchas|http://frescolib.org/docs/gotchas.html]] in the Fresco documentation. The most notable one is to avoid `ImageView` methods and properties when dealing with a `DraweeView` (even though `DraweeView` extends `ImageView`).
*  Don't call ImageView methods `setImageBitmap`, `setImageDrawable`, etc as this will wipe out the `DraweeHierarchy`
*  Don't set `ImageView` properties like `scaleType`, `src`, etc. Instead use the DraweeView counterparts for these properties.

## References
* http://frescolib.org/index.html
* https://code.facebook.com/posts/366199913563917/introducing-fresco-a-new-image-library-for-android/
* https://github.com/facebook/fresco
