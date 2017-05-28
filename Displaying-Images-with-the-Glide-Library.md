## Overview

[Glide](https://github.com/bumptech/glide) is an Image Loader Library for Android developed by bumptech and is a library that is recommended by Google. It has been used in many Google open source projects including Google I/O 2014 official application.  It provides animated GIF support and handles image loading/caching.  

### Setup

Add to your `app/build.gradle` file:

```gradle
repositories {
   jcenter()
}

dependencies {
  compile 'com.github.bumptech.glide:glide:3.8.0'
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

### Using with OkHttp

By default, Glide uses the [[Volley|Networking with the Volley Library]] networking library.  There is a way to use Glide to use OkHttp instead, which may be useful if you need to do authenticated requests.  First, add the `okhttp3-integration` library as a dependency:

```gradle
dependencies {
    compile 'com.github.bumptech.glide:okhttp3-integration:1.4.0@aar'
}
```

Next, you can configure Glide to use OkHttp in XML or through Java.  The Java approach is useful especially if you already have a shared instance of OkHttpClient:

```java
OkHttpClient okHttpClient = new OkHttpClient();
Glide.get(application).register(GlideUrl.class, InputStream.class, new OkHttpUrlLoader.Factory(okHttpClient));
```

Alternatively, you can declare Glide to use OkHttp by declaring it in your `AndroidManifest.xml` file:

```xml
<meta-data
    android:name="com.bumptech.glide.integration.okhttp3.OkHttpGlideModule"
    android:value="GlideModule" />
```

Review [this section](https://github.com/bumptech/glide#proguard) if are configuring Glide for use with ProGuard.

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

