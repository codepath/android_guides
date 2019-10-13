## Overview

[ExoPlayer](https://developer.android.com/guide/topics/media/exoplayer) is a Google own and operated, open-source, application level media player for Android. Built on top of Android's low level media APIs, ExoPlayer offers a more powerful and more robust alternative to [MediaPlayer](https://developer.android.com/reference/android/media/MediaPlayer), with additional features and customization flexibility.  It is usable on API 16 and up.  

There are many steps required to setting up ExoPlayer, so for this walkthrough, we will cover its most key components to better understand how ExoPlayer works, and then look at a third-party solution for quickly adding an ExoPlayer to your app. 

## ExoPlayer vs. MediaPlayer

ExoPlayer has several key advantages over MediaPlayer, including:
* Support for [multiple media formats](https://google.github.io/ExoPlayer/supported-formats.html), including ones not supported by MediaPlayer, such as DASH and SmoothStreaming.
* The ability to merge, concatenate, or loop your media.
* Fewer device and Android version specific issues.
* Advanced HLS features.
* Polymorphism and customization.
* Third-party library extensions.

*Note:* One key disadvantage is that when using **audio only** playback, **some devices** may use more battery than MediaPlayer.  It's also worth considering whether or not any of the above features are needed for your app.  MediaPlayer can still be a valid option for simple use cases.

## Adding ExoPlayer to Your Project

### Adding Gradle Dependencies

Make sure that both the Google and JCenter repositories to the root `build.gradle`.

```
repositories {
    google()
    jcenter()
}
```

In your app level `build.gradle`, also make sure you have Java 8 support.

```java
compileOptions {
    targetCompatibility JavaVersion.VERSION_1_8
}
```

Once these dependencies are added, you can add the version of ExoPlyaer you desire.

```java
implementation 'com.google.android.exoplayer:exoplayer:2.X.X'
```

If you don't believe you need the full library, a list of individual library modules and extensions can be found [here](https://google.github.io/ExoPlayer/guide.html#adding-exoplayer-as-a-dependency).

### Creating the Player 

Google's implemented version of ExoPlayer is the `SimpleExoPlayer`, and can be created using `ExoPlayerFactory`.

```java
private void initializePlayer() {
    SimpleExoPlayer player = ExoPlayerFactory.newSimpleInstance(context);
}
```

There are almost a dozen versions of `ExoPlayer.newSimpleInstance` in the [SimpleExoPlayer](https://google.github.io/ExoPlayer/doc/reference/com/google/android/exoplayer2/ExoPlayerFactory.html) class, with differerent parameters depending on what you need your ExoPlayer to have.

Once our player is ready, we can set it to our view.  It can either be defined as ExoPlayer's `PlayerView` widget, or as a `SurfaceView`, `TexureView`, `SurfaceHolder`, or `Surface`.

```java
private void setPlayerView(View playerView, SimpleExoPlayer player) {
    if (playerView instanceof PlayerView) {
        playerView.setPlayer(player);
    } else if (playerView instanceof SurfaceView) {
        player.setVideoSurfaceView((SurfaceView) playerView);
    } else if (playerView instanceof TextureView) {
        player.setVideoTextureView((TextureView) playerView);
    } else if (playerView instanceof SurfaceHolder) {
        player.setVideoSurfaceHolder((SurfaceHolder) playerView);
    } else if (playerView instanceof Surface) {
        player.setVideoSurface((Surface) playerView);
    }
}

private void initializePlayer() {
    SimpleExoPlayer player = ExoPlayerFactory.newSimpleInstance(context);
    setPlayerView(playerView, player);
}
```

### Creating MediaSource and Preparing the Player

Next we need to create a `MediaSource`, which defines the media to be played, then loads and reads said media.  `MediaSource` is constructed from a `MediaSourceFactory`, creating an ExoPlayer supoorted `MediaSource`.  The formats currently available are DASH, SmoothStreaming, HLS, and regular media files.

In the following code example, based on the ExoPlayer [demo project](https://github.com/google/ExoPlayer/blob/release-v2/demos/main/src/main/java/com/google/android/exoplayer2/demo/PlayerActivity.java#L478), we extract format type from the URI, and create the matching `MediaSoruce`:

```java
private MediaSource buildMediaSource(Uri uri) {
    @ContentType int type = Util.inferContentType(uri);
    DataSourceFactory dataSourceFactory = new DefaultDataSourceFactory(context,
    Util.getUserAgent(context, "yourApplicationName"));
    switch (type) {
      case C.TYPE_DASH:
        return new DashMediaSource.Factory(dataSourceFactory)
            .setManifestParser(
                new FilteringManifestParser<>(new DashManifestParser(), getOfflineStreamKeys(uri)))
            .createMediaSource(uri);
      case C.TYPE_SS:
        return new SsMediaSource.Factory(dataSourceFactory)
            .setManifestParser(
                new FilteringManifestParser<>(new SsManifestParser(), getOfflineStreamKeys(uri)))
            .createMediaSource(uri);
      case C.TYPE_HLS:
        return new HlsMediaSource.Factory(dataSourceFactory)
            .setPlaylistParserFactory(
                new DefaultHlsPlaylistParserFactory(getOfflineStreamKeys(uri)))
            .createMediaSource(uri);
      case C.TYPE_OTHER:
        return new ExtractorMediaSource.Factory(dataSourceFactory).createMediaSource(uri);
      default: {
        throw new IllegalStateException("Unsupported type: " + type);
      }
    }
  }
 
```

Once the `MediaSource` is created, we can call `ExoPlayer.prepare(mediaSource)`.

```java
 private void initializePlayer() {
    SimpleExoPlayer player = ExoPlayerFactory.newSimpleInstance(context);
    setPlayerView(playerView, player);
    MediaSource mediaSource = buildMediaSource(uri);
    player.prepare(mediaSource);
}
 ```
 
### Controlling the player

The key functions for play, pause, and seek, are `setPlayWhenReady(true)`, `setPlayWhenReady(false)`, and `seekTo(positionInMS)` respectively.  
 
When the instance of the app or activity finishes, we also need to release the player with `player.release()`.  Failing to do so will cause the player to continue running in the background and not  video decoders for use by other applications.  Don't forget to release when you're done!

```java
 private voide playPlayer() {
     if (player != null) {
         player.setPlayWhenReady(true);
     }
 }
 
 private voide pausePlayer() {
     if (player != null) {
         player.setPlayWhenReady(false);
     }
 }
 
 private void seekTo(long positionInMS) {
     if (player != null) {
         player.seekTo(positionInMS);
     }
 }
 
 private void releasePlayer() {
     if (player != null) {
         player.release();
     }
 }
```
 
### Additional and Advanced Topics

A more detailed look at the ExoPlayer documentation, including using listeners, DRM, tracking, and modified MediaSources can be found [here](https://google.github.io/ExoPlayer/guide.html#mediasource).  For full code of the ExoPlayer sample project, see [here](https://github.com/google/ExoPlayer/tree/release-v2/demos/main).
 
## TubiPlayer: ExoPlayer Made Easy!

The above is just the high level look at implementing an ExoPlayer in an android app.  There are additional steps still required.  If you're interested, the linked [documentation](https://google.github.io/ExoPlayer/guide.html) and [demo project](https://github.com/google/ExoPlayer/tree/release-v2/demos/main) show the full requirments for a fully functioning ExoPlayer.
 
 But what if you just want to add a player with a few lines of code, and be good to go?  Enter [TubiPlayer](https://github.com/Tubitv/TubiPlayer)!
 
The [Tubi TV](https://play.google.com/store/apps/details?id=com.tubitv) open source player, built for their streaming needs, has already gone through the leg work to set up and initialize the ExoPlayer, through it's `TubiExoPlayer` custom view.
 Currently, you need to add the project as its own module.  A bintray extension is coming soon.  
 
### Run Fullscreen ExoPlayer Activity

If all you need is to run ExoPlayer in fullscreen, call the `DoubleViewTubiPlayerActivity`, passing in the video url, and other optional fields such as video name, artwork, and subtitles.  Pass them into an intent to `DoubleViewTubiPlayerActivity`, using a `MediaModel` extra mapped to `TubiPlayerActivity.TUBI_MEDIA_KEY`, and that's it!

```java
String subs = "http://put_your_own_subtitle.srt";
String artwork = "http://www.put_your_own_art_work.png";
String name = "Example Video Name";
String video_url = "http://put_your_own_video_url.mp4";
Intent intent = new Intent(SelectionActivity.this, 
                                DoubleViewTubiPlayerActivity.class);
intent.putExtra(TubiPlayerActivity.TUBI_MEDIA_KEY,
                    MediaModel.video(name, video_url, artwork, null));
startActivity(intent);
```

### Customize the TubiPlayer Experience
 
 What if you need an ExoPlayer that's not fullscreen?  Then simply create a xml view that includes the `TubiExoPlayer` widget, design the layout as desired, and create an activity that extends from `DoubleViewTubiPlayerActivity`.  Override the `initLayout` function and set the `mTubiPlayerView` to the xml's `TubiExoPlayer`, and it will be fully functional:

```java
 @Override
 protected void initPlayer() {
     super.initPlayer();
     mTubiPlayerView = findViewById(R.id.TUBI_PLAYER);
 }
```

Also, don't forget to pass in the same `MediaModel` extra mapped to `TubiPlayerActivity.TUBI_MEDIA_KEY` for the new activity.  After that, you should be good to go with having ExoPlayer running in your app.
 
 If you wish to change any of the player business logic, look through the `PlayerModuleDefault` class.  This is where nearly every business related dependency is managed, and where you can override the dependencies with your own logic.
 
### Ad Support

TubiPlayer also offers the use of pre-roll and middle-row video ads.  To read up on this additional feature, see [here](https://github.com/Tubitv/TubiPlayer/blob/master/README.md).
 
## References

*  https://developer.android.com/guide/topics/media/exoplayer
*  https://google.github.io/ExoPlayer/
*  https://github.com/google/ExoPlayer/tree/release-v2/demos/main
*  https://github.com/Tubitv/TubiPlayer
*  https://code.tubitv.com/how-exoplayer-reads-hls-files-fcd82b1c8c6b
 
 
 
 