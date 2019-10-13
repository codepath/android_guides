## Overview

[Exoplayer](https://github.com/google/ExoPlayer) is an Advanced Video Library by Google that is aimed to be more customizable and flexible than the traditional `video_view` and `MediaPlayer` approach. Read more about it [here](https://exoplayer.dev).  

### Adding it to a project

In your gradle files, add these lines:  

```groovy
//project's build.gradle
repositories {
    google()
    jcenter()
}

//app/build.gradle
android {
    ...
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
}

dependencies {
    ...
    implementation 'com.google.android.exoplayer:exoplayer:2.10.5' //current version as of oct,2019
    //there is also an option to import only specific modules
}
``` 

## Streaming a video via Exoplayer  

The basic steps for playing a video(that is available on some server on the internet) via Exoplayer are:
  
1. add the `<player_view>`  in xml and create its instance: This will be the view on which the video will be rendered.
2. create a `Player` instance : This will be the main Handler behind all the streaming , decoding and rendering of bitstreams
3. create a `MediaSource` Instance: This is the Class associated with the Uri of the content you intend to play
4. attach playerView to player
5. start video playback via `absPlayerInternal.setPlayWhenReady(true);` :this will automatically start playing video as soon as the video is available.( To control playback manually, simply pass `false`, the playback will then only start when user presses the `play` button .


In your activity or framgment's xml file, add these lines:  

```xml
 <com.google.android.exoplayer2.ui.PlayerView
        android:id="@+id/pv_main"
        android:layout_width="match_parent"
        android:layout_height="300dp"
        />
<!-- there are many customization attributes available like app:auto_show="true" or app:ad_marker_color="@color/colorAccent", etc which can be either used via xml or from java -->
```

In your activity or fragment's java file, add these lines:  

```java
     SimpleExoPlayer absPlayerInternal;PlayerView pvMain;

     protected void onCreate(Bundle savedInstanceState) {
        ...
        
        String CONTENT_URL = "https://www.radiantmediaplayer.com/media/bbb-360p.mp4";
        
        int appNameStringRes = R.string.app_name;

        pvMain = findViewById(R.id.pv_main); // creating player view
        
        TrackSelector trackSelectorDef = new DefaultTrackSelector();
        absPlayerInternal = ExoPlayerFactory.newSimpleInstance(this, trackSelectorDef); //creating a player instance
        
        String userAgent = Util.getUserAgent(this, this.getString(appNameRes));
        DefaultDataSourceFactory defdataSourceFactory = new DefaultDataSourceFactory(this,userAgent);
        Uri uriOfContentUrl = Uri.parse(CONTENT_URL);
        MediaSource mediaSource = new ProgressiveMediaSource.Factory(defdataSourceFactory).createMediaSource(uriOfContentUrl);  // creating a media source

        absPlayerInternal.prepare(mediaSource);
        absPlayerInternal.setPlayWhenReady(true); // start loading video and play it at the moment a chunk of it is available offline

        pvMain.setPlayer(absPlayerInternal); // attach surface to the view
    }
```

### For Pausing Video  

```java
    private void pausePlayer(SimpleExoPlayer player) {
        if (player != null) {
            player.setPlayWhenReady(false);
        }
    }
```

For resuming the video:

```java
    private void playPlayer(SimpleExoPlayer player) {
        if (player != null) {
            player.setPlayWhenReady(true);
        }
    }
```

For stopping the video:  

```java

    private void stopPlayer(){
        pv.setPlayer(null);
        absPlayer.release();
        absPlayer = null;
    }
```

For seeking the video to some point:  

```java
    private void seekTo(SimpleExoPlayer player, long positionInMS) {
        if (player != null) {
            player.seekTo(positionInMS);
        }
    }
```

**Extras**  

- To silence video's volume, simply do : `absPlayerInternal.setVolume(0f)`
- The `<PlayerView>` has a built in Video controller that can be toggled using `pvMain.setUseController(/*true or false*/)` and other functions. But if you want, you can also add an external exoplayer's controller view called `<PlayerControlView>`.
![exoplayer image](https://i.imgur.com/0s9fRcZ.png)  

- Exoplayer has many useful extensions, some of which are mentioned [here](https://github.com/google/ExoPlayer/tree/release-v2/extensions/) .

