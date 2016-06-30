## Overview

In Android apps, there are often cases where we need to play audio files. Checking the [Supported Media Formats](http://developer.android.com/guide/appendix/media-formats.html) we can see that several audio formats (MP3, AAC, FLAC, Vorbis) are playable by default. 

In this guide we will take a look at how to play audio content using the [MediaPlayer](http://developer.android.com/reference/android/media/MediaPlayer.html) and capture audio with the [MediaRecorder](http://developer.android.com/reference/android/media/MediaRecorder.html).

### Playing Local Audio

To play local audio in the [supported formats](http://developer.android.com/guide/appendix/media-formats.html), first we should put the local audio file into the `res/raw` folder. For example put this [sample_audio.mp3](https://dl.dropboxusercontent.com/u/10281242/sample_audio.mp3) into `res/raw/sample_audio.mp3`.

Note: If you get the compile error message `INVALID FILE NAME: MUST CONTAIN ONLY [a-z0-9_.]`, this is because your MP3 can only have lowercase letters, numbers, periods, and underscores in its name.

Now we can use the [MediaPlayer](http://developer.android.com/reference/android/media/MediaPlayer.html) in order to playback any local files:

```java
MediaPlayer mediaPlayer = MediaPlayer.create(this, R.raw.sample_audio);
mediaPlayer.start();
```

On call to `start()` method, the music will start playing from the beginning. If this method is called again after the `pause()` method, the music would start playing from where it left off and not from the beginning. To play back audio from a local file path, simply do:

```java
Uri myUri = "...."; // initialize Uri here
MediaPlayer mediaPlayer = new MediaPlayer();
mediaPlayer.setAudioStreamType(AudioManager.STREAM_MUSIC);
mediaPlayer.setDataSource(getApplicationContext(), myUri);
// or just mediaPlayer.setDataSource(mFileName);
mediaPlayer.prepare(); // must call prepare first
mediaPlayer.start(); // then start
```

We can release the resources taken by the player by calling `release` to free up system resources:

```java
mediaPlayer.release();
mediaPlayer = null;
```

MediaPlayer has many key methods to control playback such as:

```java
MediaPlayer mediaPlayer = MediaPlayer.create(this, R.raw.sample_audio);
mediaPlayer.start();              // start the audio from last location
mediaPlayer.pause();              // pause audio
mediaPlayer.reset();              // reset audio to beginning of track
mediaPlayer.isPlaying();          // returns true/false indicating the song is playing
mediaPlayer.seekTo(100);           // move song to that particular millisecond
mediaPlayer.getCurrentDuration(); // current position of song in milliseconds
mediaPlayer.getDuration();        // total time duration of song in milliseconds
```

Check the [MediaPlayer](http://developer.android.com/reference/android/media/MediaPlayer.html) docs for a full list of methods. These are the basics for playing audio but check out [this androidhive](http://www.androidhive.info/2012/03/android-building-audio-player-tutorial/) and [this tutorial](http://www.tutorialspoint.com/android/android_mediaplayer.htm) if you want to understand how to hook up a graphical interface for managing the audio playback.

### Playing Streaming Audio

If you are using MediaPlayer to stream network-based content, your application must request network access in the manifest:

```xml
<uses-permission android:name="android.permission.INTERNET" />
```

Now we can stream remote audio files in the [supported formats](http://developer.android.com/guide/appendix/media-formats.html) with:

```java
String url = "https://dl.dropboxusercontent.com/u/10281242/sample_audio.mp3";
final MediaPlayer mediaPlayer = new MediaPlayer();
// Set type to streaming
mediaPlayer.setAudioStreamType(AudioManager.STREAM_MUSIC);
// Listen for if the audio file can't be prepared
mediaPlayer.setOnErrorListener(new OnErrorListener() {
	@Override
	public boolean onError(MediaPlayer mp, int what, int extra) {
		// ... react appropriately ...
        // The MediaPlayer has moved to the Error state, must be reset!
		return false;
	}
});
// Attach to when audio file is prepared for playing
mediaPlayer.setOnPreparedListener(new OnPreparedListener() {
	@Override
	public void onPrepared(MediaPlayer mp) {
		mediaPlayer.start();
	}
});
// Set the data source to the remote URL
mediaPlayer.setDataSource(url);
// Trigger an async preparation which will file listener when completed
mediaPlayer.prepareAsync();
```

With that, the remote file will start streaming once preparation is completed and loading the audio file won't block the UI thread. For more details, see the official [MediaPlayer](http://developer.android.com/guide/topics/media/mediaplayer.html) playback guide.

### Capturing Audio

You can record audio using the [MediaRecorder APIs](http://developer.android.com/reference/android/media/MediaRecorder.html) if supported by the device hardware.

First, let's add the correct permissions to our `AndroidManifest.xml` (**Note:** The permissions model has changed starting in Marshmallow. If your `targetSdkVersion` >= `23` and you are running on a Marshmallow (or later) device, you may need to [[enable runtime permissions|Managing-Runtime-Permissions-with-PermissionsDispatcher]]. You should also read more about the [[runtime permissions changes|Understanding-App-Permissions#runtime-permissions]]):

```xml
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
<uses-permission android:name="android.permission.RECORD_AUDIO" />
```

Recording audio is as simple as starting and stopping the `MediaRecorder`:

```java
// File path of recorded audio
private String mFileName;
private static final String LOG_TAG = "AudioRecordTest";
// Verify that the device has a mic first
PackageManager pmanager = this.getPackageManager();
if (pmanager.hasSystemFeature(PackageManager.FEATURE_MICROPHONE)) {
    // Set the file location for the audio
    mFileName = Environment.getExternalStorageDirectory().getAbsolutePath();
    mFileName += "/audiorecordtest.3gp";
    // Create the recorder
    MediaRecorder mediaRecorder = new MediaRecorder();
    // Set the audio format and encoder
    mediaRecorder.setAudioSource(MediaRecorder.AudioSource.MIC);
    mediaRecorder.setOutputFormat(MediaRecorder.OutputFormat.THREE_GPP);
    mediaRecorder.setAudioEncoder(MediaRecorder.AudioEncoder.AMR_NB);
    // Setup the output location
    mediaRecorder.setOutputFile(mFileName);
    // Start the recording
    try {
        mediaRecorder.prepare();
        mediaRecorder.start();
    } catch (IOException e) {
        Log.e(LOG_TAG, "prepare() failed");
    }
} else { // no mic on device
    Toast.makeText(this, "This device doesn't have a mic!", Toast.LENGTH_LONG).show();
}
```

When you want to stop capturing audio, simply notify the recorder:

```java
// Stop the recording of the audio
mediaRecorder.stop();
mediaRecorder.reset();
mediaRecorder.release();
```

If you wanted to now playback the recorded audio:

```java
MediaPlayer mediaPlayer = new MediaPlayer();
mediaPlayer.setAudioStreamType(AudioManager.STREAM_MUSIC);
mediaPlayer.setDataSource(mFileName);
mediaPlayer.prepare(); // must call prepare first
mediaPlayer.start(); // then start
```

If you wanted to add this audio to the media library on the phone:

```java
// Setup values for the media file
ContentValues values = new ContentValues(4);
long current = System.currentTimeMillis();
values.put(MediaStore.Audio.Media.TITLE, "audio file");
values.put(MediaStore.Audio.Media.DATE_ADDED, (int) (current / 1000));
values.put(MediaStore.Audio.Media.MIME_TYPE, "audio/3gpp");
values.put(MediaStore.Audio.Media.DATA, audiofile.getAbsolutePath());
ContentResolver contentResolver = getContentResolver();
// Construct uris
Uri base = MediaStore.Audio.Media.EXTERNAL_CONTENT_URI;
Uri newUri = contentResolver.insert(base, values);
// Trigger broadcast to add
sendBroadcast(new Intent(Intent.ACTION_MEDIA_SCANNER_SCAN_FILE, newUri));
Toast.makeText(this, "Added File " + newUri, Toast.LENGTH_LONG).show();
```

For more details, check out this [tutorialspoint article](http://www.tutorialspoint.com/android/android_audio_capture.htm) and the official [Audio Capture](http://developer.android.com/guide/topics/media/audio-capture.html) guide.

## References

* <http://developer.android.com/guide/topics/media/mediaplayer.html>
* <http://developer.android.com/training/managing-audio/index.html>
* <http://developer.android.com/guide/appendix/media-formats.html>
* <http://www.vogella.com/tutorials/AndroidMedia/article.html>
* <http://developer.android.com/guide/topics/media/audio-capture.html>
* <http://www.tutorialspoint.com/android/android_audio_capture.htm>
* <http://www.androidhive.info/2012/03/android-building-audio-player-tutorial/>
* <http://developer.android.com/reference/android/media/MediaPlayer.html>