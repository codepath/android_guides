## Overview

We can use the [YouTube Android Player API](https://developers.google.com/youtube/android/player/) to play YouTube videos within an Android app.  

<img src="https://i.imgur.com/xeVSOyL.gif" width="600" />

### Setup

In order to use this API, you will **need to have Google Play installed on your emulator** and **the Youtube App installed** because the YouTube API interacts with a service that is distributed with the YouTube app.  Either use a physical device to test or make sure you have followed [[this emulator setup guide|Genymotion-2.0-Emulators-with-Google-Play-support#setup-google-play-services]] to install Google Play services and then added the Youtube App. Otherwise, you are likely to see `An error has occurred while initializing the YouTube player`.

To start, you will need to create an API key through [https://console.developers.google.com/](https://console.developers.google.com/).  Make sure to enable the `YouTube Data API v3`.  Go to the `Credentials` section and generate an API key.

### Installation

Next, add the [YouTubeAndroidPlayerApi.jar](https://developers.google.com/youtube/android/player/downloads/) file into Android Studio by [following this GIF](https://i.imgur.com/k9a6WET.gif) to add this to your `libs` dir. Once you've added the JAR, select `Tools => Android => Sync project files with Gradle` in Android Studio to complete installation.

### Usage

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

If you wish to only load the video but not play, use `cueVideo()` instead of `loadVideo()`. Playing videos involves passing along the YouTube video key (do not include the full URL): 

```java
youTubePlayer.loadVideo("5xVh-7ywKpE");
```

You can review the [working sample code here](https://github.com/codepath/AndroidYoutubeVideoDemo).

#### Forcing Landscape Mode

If you wish to force the video to landscape mode, you can also add the `screenOrientation` flag inside your Activity in the AndroidManifest.xml file:

```xml
<activity
  android:name=".QuickPlayActivity"
  android:screenOrientation="landscape">
</activity>
```

## Using the YouTubePlayerFragment Approach

Alternatively instead of the approach outlined above, rather than `extending YouTubeBaseActivity`, we can use the [YouTubePlayerFragment](https://developers.google.com/youtube/android/player/reference/com/google/android/youtube/player/YouTubePlayerFragment) instead. First, put the player into your activity using this fragment:

```xml
<fragment
    android:id="@+id/youtubeFragment"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:name="com.google.android.youtube.player.YouTubePlayerFragment">
</fragment>
```

Next, we lookup the fragment by ID and then initialize the video player:

```java
    @Override
    protected void onCreate(Bundle savedInstanceState) {

        YouTubePlayerFragment youtubeFragment = (YouTubePlayerFragment)
            getFragmentManager().findFragmentById(R.id.youtubeFragment);
        youtubeFragment.initialize("YOUR API KEY",
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

        // ...
    }
```

Note that in this case, the **activity containing the fragment does not need to extend any special base class**. Otherwise the steps are identical regardless of using the fragment or activity approach. 

## Troubleshooting

Common issues with the `YouTubePlayerView` are listed below:

- **Setup Youtube App** - In regards to setup, if you are using an emulator, make sure you have followed [[this emulator setup guide|Genymotion-2.0-Emulators-with-Google-Play-support#setup-google-play-services]] to install Google Play services and then **also added the Youtube app**. Otherwise, you are likely to see `An error has occurred while initializing the YouTube player`.

- **Verify Video is Valid** - When anything is going wrong, first thing to check that the value passed into the `cueVideo` or `loadVideo` methods is a valid Youtube video. Be sure **not to pass a null value** into those methods. Investigate to make sure that if you take the value given i.e `5xVh-7ywKpE` and then visit this [on youtube](https://www.youtube.com/watch?v=5xVh-7ywKpE) that the video is valid.

- **NullPointerException During Init** - Make sure that your activity `extends YouTubeBaseActivity` or you are using the `YouTubePlayerFragment`. If the activity does not extend that base class then the activity will throw this exception. 

- **Error DeadObjectException:** If you receive the `java.lang.IllegalStateException: android.os.DeadObjectException` exception, you need to open up the "Play Store" and update the Youtube app on the Android device.
  
- **Error Leaked ServiceConnection:** If you are using an emulator, upgrade to Lollipop (API 21) and open up the "Play Store" to check for updates to the Youtube app.

## References

* <https://www.sitepoint.com/using-the-youtube-api-to-embed-video-in-an-android-app/>
