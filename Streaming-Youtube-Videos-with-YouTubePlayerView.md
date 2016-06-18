## Overview

We can use the [YouTube Android Player API] (https://developers.google.com/youtube/android/player/) to play YouTube videos within an Android app.  

<img src="http://g.recordit.co/xYGct0S1QE.gif" width="600" />

### Setup

In order to use this API, you will **need to have Google Play installed on your emulator** because the YouTube API interacts with a service that is distributed with the YouTube app.  Either use a physical device to test or make sure you have followed [[this emulator setup guide|Genymotion-2.0-Emulators-with-Google-Play-support#setup-google-play-services]] to install Google Play services. Otherwise, you are likely to see `An error has occurred while initializing the YouTube player`.

To start, you will need to create an API key through [https://console.developers.google.com/](https://console.developers.google.com/).  Make sure to enable the `YouTube Data API v3`.  Go to the `Credentials` section and generate an API key.

### Installation

Next, add the [YouTubeAndroidPlayerApi.jar](https://github.com/google/iosched/raw/master/android/libs/YouTubeAndroidPlayerApi.jar) file into Android Studio by [following this GIF](http://i.imgur.com/k9a6WET.gif) to add this to your `libs` dir. Once you've added the JAR, select `Tools => Android => Sync project files with Gradle` in Android Studio to complete installation.

## Usage

Instead of a `VideoView`, you should add `YouTubePlayerView` to your layout file where you'd like the video to be displayed:

```xml
<com.google.android.youtube.player.YouTubePlayerView
    android:id="@+id/player"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"/>
```

If you intend for your Activity, you will need to extend `YouTubeBaseActivity`.  One current drawback is that this library does not inherit from `AppCompatActivity` so some of your styles may not match those that are defined in `styles.xml`.

```java
public class QuickPlayActivity extends YouTubeBaseActivity {
```

You then should initialize the YouTube Player by calling `initialize()` with your API key on the `YouTubePlayerView`:

```java
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_quick_play);

        YouTubePlayerView youTubePlayerView = 
            (YouTubePlayerView) findViewById(R.id.player);

        youTubePlayerView.initialize("YOUR API KEY",
                new YouTubePlayer.OnInitializedListener() {
                    @Override
                    public void onInitializationSuccess(YouTubePlayer.Provider provider,
                            YouTubePlayer youTubePlayer, boolean b) {

                        // do any work here to cue video, play video, etc.    
                        youTubePlayer.cueVideo("5xVh-7ywKpE");
                    }
                    @Override
                    public void onInitializationFailure(YouTubePlayer.Provider provider,
                            YouTubeInitializationResult youTubeInitializationResult) {

                    }
        });
}
```

Playing videos involves passing along the YouTube video key (do not include the full URL): 

```java
youTubePlayer.loadVideo("5xVh-7ywKpE");
```

If you wish to only load the video but not play, use `cueVideo()` instead of `loadVideo()`.

If you wish to force the video to landscape mode, you can also add the `screenOrientation` flag inside your Activity in the AndroidManifest.xml file:

```xml
<activity
  android:name=".QuickPlayActivity"
  android:screenOrientation="landscape">
</activity>
```