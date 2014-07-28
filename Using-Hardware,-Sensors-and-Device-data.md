## Overview

With Android, there is a lot of access to hardware features using built-in Android libraries and the Google Play SDK. A few examples of things you might need to access:

 * Taking a photo/video with the camera.
 * Accessing and playing video and audio.
 * Accessing hardware sensors like accelerometers, light sensors, etc.
 * Using the Locations API.
 * Using the Maps API.



## Playing Videos

[VideoView](http://developer.android.com/reference/android/widget/VideoView.html) is the default wrapper for MediaPlayer for playing videos:

```java
public void playUrl(String url) {
    VideoView videoView = (VideoView) findViewById(R.id.video_view);
    videoView.setVideoPath(url);
    videoView.setMediaController(new MediaController(this));       
    videoView.requestFocus();   
    videoView.start();
}
```

See our [[Video and Audio Playback and Recording]] cliffnotes for a more detailed look.

## References
 
 * <http://developer.android.com/reference/android/widget/VideoView.html>
 