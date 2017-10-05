## Overview

Network requests are used to retrieve or modify API data or media from a server. This is a very common task in Android development especially for dynamic data-driven clients.

The underlying Java class used for network connections is [DefaultHTTPClient](http://developer.android.com/reference/org/apache/http/impl/client/DefaultHttpClient.html) or [HttpUrlConnection](http://developer.android.com/reference/java/net/HttpURLConnection.html).  Both of these are lower-level and require completely manual management of parsing the data from the input stream and executing the request asynchronously.  DefaultHTTPClient, otherwise known as the Apache HTTP Client, has been [deprecated] (https://developer.android.com/about/versions/marshmallow/android-6.0-changes.html#behavior-apache-http-client) since Android 6.0.  The reason for two different HTTP clients is described in this [blog article](http://android-developers.blogspot.com/2011/09/androids-http-clients.html).  A historical perspective is also discussed in [this podcast](http://fragmentedpodcast.com/2016/06/).

For most common cases, we are better off using a popular third-party library called [android-async-http](http://loopj.com/android-async-http/) or [OkHttp](http://square.github.io/okhttp/) which will handle the entire process of sending and parsing network requests for us in a more robust and easy-to-use way.

### Permissions

In order to access the internet, be sure to specify the following permissions in `AndroidManifest.xml`:

```java
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.simplenetworking"
    android:versionCode="1"
    android:versionName="1.0" >
 
   <uses-permission android:name="android.permission.INTERNET" /> 
   <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />

</manifest>
```

### Sending an HTTP Request (Third Party)

There are at least three major third-party networking libraries you should consider using.  

* See the [[Android Async Http Client guide|Using-Android-Async-Http-Client]] for making basic network calls.  It is the library often used for learning Android but would not be used in a production application.

* See the [[OkHttp guide|Using-OkHttp]] for making synchronous and asynchronous calls.  
    - See also the [[Retrofit guide|Consuming-APIs-with-Retrofit]], which uses OkHttp and makes it easier to make more RESTful API calls.  Read through [[this guide|Leveraging-the-Gson-Library]] to understand how the Gson library works with Retrofit.

* Check out the [[Volley guide|Networking-with-the-Volley-Library]], a library built by Google that has fallen out of favor for OkHttp.   It was one of the first networking libraries released for Android and provides a more convenient way to make networking requests than using `AsyncTask`.

There can be a bit of a learning curve when using these libraries, so your best bet when first learning is to use Android Async Http Client or Volley.  With OkHttp you also have to deal with the complexity of whether your callbacks need to be run on the main thread to update the UI, as explained in the guide.

Here is a comparison of the different aspects of the libraries.  

|                 |Android Async Http | OkHttp | Volley |
| ----------------     |--------- |:------:| ------:|
| Debugging            | Requires Proxy server | Use LogInterceptor | Use verbose mode |
| Disk Caching              | Yes       | Yes  | Yes  |
| Request Queueing        | No | No | Included |
| Remote Image Fetching| Manual   | Requires Picasso or Glide | Included  |
| Animated GIF Support | No | Requires Glide | Requires Glide |
| Release Cadence  | Infrequent | Monthly | Infrequent | 
| Transport Layer | Apache HTTP Client | OkHttp | HttpUrlConnection (or OkHttp)|
| Synchronous Calls    | use SyncHttpClient |  execute() instead of enqueue() | use RequestFuture |
| HTTP/2 | No | Yes | Works with OkHttp |
| Automatic Gzip processing | Yes | Yes | No (unless using OkHttp) |
| Author     | James Smith | Square | Google | 

One issue with Android Async Http Client is that the library has very limited ways to observe network traces that are useful for debugging.   Volley provides remote fetching images out of the box, while Android Async Http client requires more manual work and OkHttp needs the [[Picasso|Displaying-Images-with-the-Picasso-Library]] or [[Glide|Displaying-Images-with-the-Picasso-Library]] library in order to do so.

Another important point is that OkHttp is not only a standalone networking library but also can be used for the [underlying implementation for HttpUrlConnection](https://android.googlesource.com/platform/external/okhttp/+/master/okhttp-urlconnection/src/main/java/com/squareup/okhttp/internal/huc/HttpURLConnectionImpl.java).  For this reason, Volley can also leverage OkHttp to support automatic Gzip and HTTP/2 processing.

### Sending an HTTP Request (The "Hard" Way)

Sending an HTTP Request involves the following conceptual steps:

1. Declare a URL Connection
2. Open InputStream to connection
3. Download and decode based on data type
4. Wrap in AsyncTask and execute in background thread

This would translate to the following networking code to send a simple request (with try-catch structured exceptions not shown here for brevity):

```java
// 1. Declare a URL Connection
URL url = new URL("http://www.google.com");
HttpURLConnection conn = (HttpURLConnection) url.openConnection();
// 2. Open InputStream to connection
conn.connect();
InputStream in = conn.getInputStream();
// 3. Download and decode the string response using builder
StringBuilder stringBuilder = new StringBuilder();
BufferedReader reader = new BufferedReader(new InputStreamReader(in));
String line;
while ((line = reader.readLine()) != null) {
    stringBuilder.append(line);
}
```

The fourth step requires this networking request to be executed in a background task using [[AsyncTasks|Creating-and-Executing-Async-Tasks]] such as shown below:

```java
// The types specified here are the input data type, the progress type, and the result type
// Subclass AsyncTask to execute the network request
// String == URL, Void == Progress Tracking, String == Response Received
private class NetworkAsyncTask extends AsyncTask<String, Void, Bitmap> {
     protected String doInBackground(String... strings) {
         // Some long-running task like downloading an image.
         // ... code shown above to send request and retrieve string builder
         return stringBuilder.toString();
     }

     protected void onPostExecute(String result) {
         // This method is executed in the UIThread
         // with access to the result of the long running task
         // DO SOMETHING WITH STRING RESPONSE
     }
}

private void downloadResponseFromNetwork() {
    // 4. Wrap in AsyncTask and execute in background thread
    new NetworkAsyncTask().execute("http://google.com");
}
```

### Displaying Remote Images (The "Easy" Way)

Displaying images is easiest using a third party library such as [[Picasso|Displaying-Images-with-the-Picasso-Library]] from Square which will download and cache remote images and abstract the complexity behind an easy to use DSL:

```java
String imageUri = "https://i.imgur.com/tGbaZCY.jpg";
ImageView ivBasicImage = (ImageView) findViewById(R.id.ivBasicImage);
Picasso.with(context).load(imageUri).into(ivBasicImage);
```

Refer to our [[Picasso Guide|Displaying-Images-with-the-Picasso-Library]] for more detailed usage information and configuration.

### Displaying Remote Images (The "Hard" Way)

Suppose we wanted to load an image using only the built-in Android network constructs. In order to download an image from the network, convert the bytes into a bitmap and then insert the bitmap into an imageview, you would use the following pseudo-code:

```java
// 1. Declare a URL Connection
URL url = new URL("https://i.imgur.com/tGbaZCY.jpg");
HttpURLConnection conn = (HttpURLConnection) url.openConnection();
// 2. Open InputStream to connection
conn.connect();
InputStream in = conn.getInputStream();
// 3. Download and decode the bitmap using BitmapFactory
Bitmap bitmap = BitmapFactory.decodeStream(in);
in.close();
// 4. Insert into an ImageView
ImageView imageView = (ImageView) findViewById(R.id.imageView);
imageView.setImageBitmap(bitmap);
```

Here's the complete code needed to construct an `AsyncTask` that downloads a remote image and displays the image in an `ImageView` using just the official Google Android SDK. See the [[Creating and Executing Async Tasks]] for more information about executing asynchronous background tasks: 

```java
public class MainActivity extends Activity {
    private ImageView ivBasicImage;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        ivBasicImage = (ImageView) findViewById(R.id.ivBasicImage);
        String url = "https://i.imgur.com/tGbaZCY.jpg";
        // Download image from URL and display within ImageView
        new ImageDownloadTask(ivBasicImage).execute(url);
    }

    // Defines the background task to download and then load the image within the ImageView
    private class ImageDownloadTask extends AsyncTask<String, Void, Bitmap> {
        ImageView imageView;

        public ImageDownloadTask(ImageView imageView) {
            this.imageView = imageView;
        }

        protected Bitmap doInBackground(String... addresses) {
            Bitmap bitmap = null;
            InputStream in = null;
            try {
                // 1. Declare a URL Connection
                URL url = new URL(addresses[0]);
                HttpURLConnection conn = (HttpURLConnection) url.openConnection();
                // 2. Open InputStream to connection
                conn.connect();
                in = conn.getInputStream();
                // 3. Download and decode the bitmap using BitmapFactory
                bitmap = BitmapFactory.decodeStream(in);
            } catch (IOException e) {
                e.printStackTrace();
            } finally {
              if(in != null)
                 try {
                   in.close();
                 } catch (IOException e) {
                   Log.e(TAG, "Exception while closing inputstream"+ e);
                 }
               }
            }
            return bitmap;
        }

        // Fires after the task is completed, displaying the bitmap into the ImageView
        @Override
        protected void onPostExecute(Bitmap result) {
            // Set bitmap image for the result
            imageView.setImageBitmap(result);
        }
    }
}
```

We could even create our own basic version of a library for loading images by wrapping up this logic into an object as [outlined here](https://gist.github.com/nesquena/d84f30b14288c786772b). 

Of course, doing this the "hard" way is not recommended. In most cases, to avoid having to manually manage caching and download management, we are better off creating your own libraries or in most cases utilizing existing [third-party libraries](http://square.github.io/picasso/). 

**Note:** If you use the approach above to download and display many images within a ListView, you might run into some threading issues that cause buggy loading of images. The blog post [Multithreading for Performance](http://android-developers.blogspot.com/2010/07/multithreading-for-performance.html) offers a solution in which you manage the active remote downloading background tasks to ensure that too many tasks are not being spun up at once. 

### Checking for Network Connectivity

#### Checking Network is Connected

First, make sure to setup the `android.permission.ACCESS_NETWORK_STATE` permission as shown above. To verify network availability you can then define and call this method:

```java
private Boolean isNetworkAvailable() {
    ConnectivityManager connectivityManager 
          = (ConnectivityManager) getSystemService(Context.CONNECTIVITY_SERVICE);
    NetworkInfo activeNetworkInfo = connectivityManager.getActiveNetworkInfo();
    return activeNetworkInfo != null && activeNetworkInfo.isConnectedOrConnecting();
}
```

Note that having an active network interface doesn't guarantee that a particular networked service is available or that **the internet is actually connected**. Network issues, server downtime, low signal, captive portals, content filters and the like can all prevent your app from reaching a server. For instance you can't tell for sure if your app can reach Twitter until you receive a valid response from the Twitter service.

See [this official connectivity guide](http://developer.android.com/training/monitoring-device-state/connectivity-monitoring.html) for more details.

#### Checking the Internet is Connected

To verify if the device is actually connected to the internet, we can use the following method of pinging the Google DNS servers to check for the expected exit value:

```java
public boolean isOnline() {
    Runtime runtime = Runtime.getRuntime();
    try {
        Process ipProcess = runtime.exec("/system/bin/ping -c 1 8.8.8.8");
        int     exitValue = ipProcess.waitFor();
        return (exitValue == 0);
    } catch (IOException e)          { e.printStackTrace(); } 
      catch (InterruptedException e) { e.printStackTrace(); }
    return false;
}
```

Note that this does not need to be run in background and does not require special privileges. See [this stackoverflow post](http://stackoverflow.com/a/27312494) for the source of this solution.

## Troubleshooting

Take a look at [[Troubleshooting API Calls]] to understand how to gain better visibility about what your network calls are doing.

## References

 * <http://loopj.com/android-async-http/>
 * <http://developer.android.com/reference/java/net/HttpURLConnection.html>
 * <https://github.com/codepath/android-rest-client-template>
 * <http://developer.android.com/training/displaying-bitmaps/process-bitmap.html#concurrency>