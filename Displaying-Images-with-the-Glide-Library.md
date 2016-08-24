## Overview

[Glide](https://github.com/bumptech/glide) is an Image Loader Library for Android developed by bumptech and is a library that is recommended by Google. It has been used in many Google open source projects including Google I/O 2014 official application.

**Needs attention**

### Setup

Add to your `app/build.gradle` file:

```gradle
repositories {
  mavenCentral() // jcenter() works as well because it pulls from Maven Central
}

dependencies {
  compile 'com.github.bumptech.glide:glide:3.7.0'
  compile 'com.android.support:support-v4:19.1.0'
}
```

### Basic Usage

```java
Glide.with(context)
    .load("http://pathtoimage.com/graphic.png")
    .into(ivImg);
```

### Advanced Usage

Resizing images with:

```java
Glide.with(context)
    .load("http://pathtoimage.com/graphic.png")
    .override(300, 200)
    .into(ivImg);
```

Placeholder and error images:

```java
Glide.with(context)
    .load("http://pathtoimage.com/graphic.png")
    .placeholder(R.drawable.placeholder)
    .error(R.drawable.imagenotfound)
    .into(ivImg);
```

Cropping images with:

```java
Glide.with(context)
    .load("http://pathtoimage.com/graphic.png")
    .centerCrop()
    .into(ivImg);
```

Transforming images with:

```java
Glide.with(context)
    .load("http://pathtoimage.com/graphic.png")
    .transform(new CircleTransform(context))
    .into(ivImg);
```

### Configuration

You can configure Glide by creating a `GlideConfiguration.java` file:

```java
public class GlideConfiguration implements GlideModule {

    @Override
    public void applyOptions(Context context, GlideBuilder builder) {
        // Apply options to the builder here.
        // Glide default Bitmap Format is set to RGB_565 since it 
        // consumed just 50% memory footprint compared to ARGB_8888.
        // Increase memory usage for quality with:
        builder.setDecodeFormat(DecodeFormat.PREFER_ARGB_8888);
    }

    @Override
    public void registerComponents(Context context, Glide glide) {
        // register ModelLoaders here.
    }
}
```

And then define it as `meta-data` inside `AndroidManifest.xml`:

<meta-data android:name="my.app.namespace.utils.GlideConfiguration"
            android:value="GlideModule"/>

## References

* <https://github.com/bumptech/glide>
* <https://inthecheesefactory.com/blog/get-to-know-glide-recommended-by-google/en>
* <http://www.androidtutorialshub.com/glide-image-loading-library-for-android-tutorial/>
* <https://futurestud.io/blog/glide-getting-started>
* <https://github.com/wasabeef/glide-transformations>
* <http://google-opensource.blogspot.com/2014/09/glide-30-media-management-library-for.html>
* <http://tutorialwing.com/android-glide-library-tutorial-example/>
* <http://vardhan-justlikethat.blogspot.com/2014/09/android-image-loading-libraries-picasso.html>
* <http://www.appance.com/glide-image-loading-and-caching-library-for-android/>
* <http://www.androidhive.info/2016/04/android-glide-image-library-building-image-gallery-app/>

