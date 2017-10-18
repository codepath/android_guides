## Overview

In Android apps, there are often cases where we need to play video files. Checking the [Supported Media Formats](http://developer.android.com/guide/appendix/media-formats.html) we can see that several video formats (H.264, MPEG-4) are playable by default. 

In this guide we will take a look at how to play video content using the [VideoView](http://developer.android.com/reference/android/widget/VideoView.html) and capture video with the [MediaRecorder](http://developer.android.com/reference/android/media/MediaRecorder.html).

## Video Playback

### Playing Local Video

Playing local video in a [supported format](http://developer.android.com/guide/appendix/media-formats.html) can be done using the `VideoView`. First, setup the `VideoView` in your layout:

```xml
<VideoView
    android:id="@+id/video_view"
    android:layout_width="320px"
    android:layout_height="240px" />
```

Next, we can store local files such as [small_video.mp4](http://techslides.com/demos/sample-videos/small.mp4) in `res/raw/small_video.mp4` and than play the video in the view with:

```java
VideoView mVideoView = (VideoView) findViewById(R.id.video_view);
mVideoView.setVideoURI(Uri.parse("android.resource://" + getPackageName() +"/"+R.raw.small_video));
mVideoView.setMediaController(new MediaController(this));
mVideoView.requestFocus();
mVideoView.start();
```

See [this tutorial](https://mobiarch.wordpress.com/2014/02/18/showing-fullscreen-video-in-android/) for playing a video full-screen with a `VideoView`. See [this other edumobile tutorial](http://www.edumobile.org/android/android-beginner-tutorials/how-to-play-a-video-file/) for a more detailed look at using [VideoView](http://developer.android.com/reference/android/widget/VideoView.html).

### Playing Streaming Video

To play back remote video in a [supported format](http://developer.android.com/guide/appendix/media-formats.html), we can still use the `VideoView`. First, setup the correct permissions in the `Android Manifest.xml`:

```xml
<uses-permission android:name="android.permission.INTERNET" />
```

Now, we can play remote video with:

```java
final VideoView mVideoView = (VideoView) findViewById(R.id.video_view);
mVideoView.setVideoPath("http://techslides.com/demos/sample-videos/small.mp4");
MediaController mediaController = new MediaController(this);
mediaController.setAnchorView(mVideoView);
mVideoView.setMediaController(mediaController);
mVideoView.requestFocus();
mVideoView.setOnPreparedListener(new OnPreparedListener() {
    // Close the progress bar and play the video
    public void onPrepared(MediaPlayer mp) {
        mVideoView.start();
    }
});
```

You can see a more complete example of remote streaming with [this androidbegin tutorial](http://www.androidbegin.com/tutorial/android-video-streaming-videoview-tutorial/).

### VideoView Controls

We can remove the `VideoView` media controls with:

```java
mVideoView.setMediaController(null)
```

We can hide or show the media UI controls at runtime with:

```java
// Get instance to media controller
MediaController controller = new MediaController(this);
videoHolder.setMediaController(controller);
// Hide the controller
controller.setVisibility(View.GONE);
// Show the controller 
controller.setVisibility(View.VISIBLE);
```

### VideoView Limitations and Improved Libraries

`VideoView` should not be embedded in a `ListView` or any scrolling view due to a [known bug with Android](https://code.google.com/p/android/issues/detail?id=37229). Instead of using a `VideoView` which extends `SurfaceView`, in order to enable scrolling we need to use a [TextureView instead](https://github.com/dmytrodanylyk/dmytrodanylyk/blob/gh-pages/articles/surface-view-play-video.md). The easiest workaround is to use a library such as [Android-ScalableVideoView](https://github.com/yqritc/Android-ScalableVideoView), [VideoPlayerManager](https://github.com/danylovolokh/VideoPlayerManager), or [fenster](https://github.com/malmstein/fenster). 

<a href="https://github.com/yqritc/Android-ScalableVideoView"><img src="https://i.imgur.com/EFVEg0V.gif" width="250" /></a>

You can read about [how fenster was developed](http://www.malmstein.com/blog/2014/08/09/how-to-use-a-textureview-to-display-a-video-with-custom-media-player-controls/) as well. If you are using fenster and running into a permissions error related to `write_settings` on an Android >= 6.0 device, check out [this snippet as a workaround](https://gist.github.com/shrikant0013/fc3e67b4b898294a03e4eba1b527f898). 

### Streaming from YouTube

We can use the [YouTube Android Player API] (https://developers.google.com/youtube/android/player/) to play YouTube videos within an Android app.  

<img src="https://i.imgur.com/xeVSOyL.gif" width="600" />

Refer to the [[Streaming Youtube Videos with YouTubePlayerView]] guide for detailed instructions.

## Capturing Video

### Using Intents

Capturing video can be done using intents to capture video using the camera. First, let's setup the necessary permissions in `AndroidManifest.xml` (**Note:** The permissions model has changed starting in Marshmallow. If your `targetSdkVersion` >= `23` and you are running on a Marshmallow (or later) device, you may need to [[enable runtime permissions|Managing-Runtime-Permissions-with-PermissionsDispatcher]]. You should also read more about the [[runtime permissions changes|Understanding-App-Permissions#runtime-permissions]]):

```xml
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
```

Next, we can trigger recording video by starting an intent triggering a video capture action:

```java
private static final int VIDEO_CAPTURE = 101;
Uri videoUri;
public void startRecordingVideo() {
    if (getPackageManager().hasSystemFeature(PackageManager.FEATURE_CAMERA_FRONT)) {
        Intent intent = new Intent(MediaStore.ACTION_VIDEO_CAPTURE);
        File mediaFile = new File(
           Environment.getExternalStorageDirectory().getAbsolutePath() + "/myvideo.mp4");
        videoUri = Uri.fromFile(mediaFile);
        intent.putExtra(MediaStore.EXTRA_OUTPUT, videoUri);
        startActivityForResult(intent, VIDEO_CAPTURE);
    } else {
        Toast.makeText(this, "No camera on device", Toast.LENGTH_LONG).show();
    }
}
```

and then we need to manage the `onActivityResult` for when the video has been captured:

```java
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    if (requestCode == VIDEO_CAPTURE) {
        if (resultCode == RESULT_OK) {
            Toast.makeText(this, "Video has been saved to:\n" + data.getData(), Toast.LENGTH_LONG).show();
            playbackRecordedVideo();
        } else if (resultCode == RESULT_CANCELED) {
            Toast.makeText(this, "Video recording cancelled.",  Toast.LENGTH_LONG).show();
        } else {
            Toast.makeText(this, "Failed to record video",  Toast.LENGTH_LONG).show();
        }
    }
}

public void playbackRecordedVideo() {
    VideoView mVideoView = (VideoView) findViewById(R.id.video_view);
    mVideoView.setVideoURI(videoUri);
    mVideoView.setMediaController(new MediaController(this));
    mVideoView.requestFocus();
    mVideoView.start();
}
```

For a more detailed look, check out the [techtopia tutorial](http://www.techotopia.com/index.php/Video_Recording_and_Image_Capture_on_Android_using_Camera_Intents#Calling_the_Video_Capture_Intent) on video recording.

### Using MediaRecorder

If we want to record video from within an app directly, we'll want to use the [MediaRecorder](http://developer.android.com/reference/android/media/MediaRecorder.html). We can find good references for using the recorder in the following resources below:

 * [Grabbing a video using MediaRecorder](https://androidcookbook.com/Recipe.seam;jsessionid=9ACF6480A5FD9DDFEDE7ED6CD35C97EF?recipeId=1375)
 * [Recording and Playing Video on Android](http://www.codeproject.com/Articles/107270/Recording-and-Playing-Video-on-Android)
 * [Google Video Recorder Sample](https://github.com/googlesamples/android-MediaRecorder)

For more advanced usage, see [this video recording and processing guide](http://andrey.chernih.me/2014/06/28/video-recording-and-processing-in-android/).

#### Configuring Encoding

The `MediaRecorder` can be configured to select the encoding, quality and compression settings:

```java
// Create MediaRecorder
recorder.setVideoSize(640, 480);
recorder.setVideoFrameRate(16); //might be auto-determined due to lighting
recorder.setVideoEncodingBitRate(3000000);
recorder.setVideoEncoder(MediaRecorder.VideoEncoder.H264);// MPEG_4_SP
recorder.setAudioEncoder(MediaRecorder.AudioEncoder.AMR_NB);
```

You can also configure using preset qualities with `CamcorderProfile`:

```java
CamcorderProfile cpHigh = CamcorderProfile.get(CamcorderProfile.QUALITY_HIGH);
recorder.setProfile(cpHigh);
```

See the [available options here](http://developer.android.com/reference/android/media/CamcorderProfile.html#constants). You can also review [this stackoverflow post](http://stackoverflow.com/a/14581125) for more details.

## References

* <http://mrbool.com/how-to-play-video-formats-in-android-using-videoview/28299>
* <http://developer.android.com/guide/topics/media/mediaplayer.html>
* <http://developer.android.com/guide/appendix/media-formats.html>
* <https://developers.google.com/youtube/android/player/>
* <http://www.vogella.com/tutorials/AndroidMedia/article.html>
* <http://www.edumobile.org/android/android-beginner-tutorials/how-to-play-a-video-file/>
* <http://www.androidbegin.com/tutorial/android-video-streaming-videoview-tutorial/>
* <http://developer.android.com/reference/android/media/MediaPlayer.html>
