## Overview

Network requests are used to retrieve or modify API data or media from a server. This is a very common task in Android development especially for dynamic data-driven clients.

The underlying Java class used for network connections is [HTTPURLConnection](http://developer.android.com/reference/java/net/HttpURLConnection.html) or [DefaultHTTPClient](http://developer.android.com/reference/org/apache/http/impl/client/DefaultHttpClient.html). Both of these are lower-level and require completely manual management of parsing the data from the input stream and executing the request asynchronously.

For most common cases, we are better off using a popular third-party library called [android-async-http](http://loopj.com/android-async-http/) which will handle the entire process of sending and parsing network requests for us in a more robust and easy-to-use way.

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

### Checking for Network Connectivity

First, make sure to setup the `android.permission.ACCESS_NETWORK_STATE` permission as shown above. To verify network availability you can then define and call this method:

```java
private Boolean isNetworkAvailable() {
    ConnectivityManager connectivityManager 
          = (ConnectivityManager) getSystemService(Context.CONNECTIVITY_SERVICE);
    NetworkInfo activeNetworkInfo = connectivityManager.getActiveNetworkInfo();
    return activeNetworkInfo != null && activeNetworkInfo.isConnectedOrConnecting();
}
```

You can also verify internet access with this method:

```java
public Boolean isOnline() {
    try {
        Process p1 = java.lang.Runtime.getRuntime().exec("ping -c 1 www.google.com");
        int returnVal = p1.waitFor();
        boolean reachable = (returnVal==0);
        return reachable;
    } catch (Exception e) {
        // TODO Auto-generated catch block
        e.printStackTrace();
    }
    return false;
}
```

See [this official connectivity guide](http://developer.android.com/training/monitoring-device-state/connectivity-monitoring.html) for more details.

### Sending an HTTP Request (Third Party)

Using a third-party library such as [android-async-http](http://loopj.com/android-async-http/), sending an HTTP request is quite straightforward. First, we add the library to gradle:

```gradle
dependencies {
  compile 'com.loopj.android:android-async-http:1.4.6'
}
```

Now, we just create an `AsyncHttpClient`, and then execute a request specifying an anonymous class as a callback:

```java
import com.loopj.android.http.*;

AsyncHttpClient client = new AsyncHttpClient();
RequestParams params = new RequestParams();
params.put("key", "value");
params.put("more", "data");
client.get("http://www.google.com", params, new TextHttpResponseHandler() {
        @Override
        public void onSuccess(int statusCode, Header[] headers, String res) {
            // called when response HTTP status is "200 OK"
        }

        @Override
        public void onFailure(int statusCode, Header[] headers, String res, Throwable t) {
            // called when response HTTP status is "4XX" (eg. 401, 403, 404)
        }	
    }
);
```

This will automatically execute the request asynchronously and fire the `onSuccess` when the response returns a success code and `onFailure` if the response does not. 

### Sending a JSON Request

Similar to sending a regular HTTP request, [android-async-http](http://loopj.com/android-async-http/) can also be used for sending JSON API requests:

```java
String url = "https://ajax.googleapis.com/ajax/services/search/images";
AsyncHttpClient client = new AsyncHttpClient();
RequestParams params = new RequestParams();
params.put("q", "android");
params.put("rsz", "8");
client.get(url, params, new JsonHttpResponseHandler() {    	    
    @Override
    public void onSuccess(int statusCode, Header[] headers, JSONObject response) {
       // Root JSON in response is an dictionary i.e { "data : [ ... ] }
       // Handle resulting parsed JSON response here
    }

    @Override
    public void onSuccess(int statusCode, Header[] headers, JSONArray response) {
       // Root JSON in response is an array i.e [ { ... }, { ... } ] 
       // Handle resulting parsed JSON response here
    }
});
```

The request will be sent out with the appropriate parameters passed in the query string and then the response will be parsed as JSON and made available within `onSuccess`.

### Sending an Authenticated API Request

API requests tend to be JSON or XML responses that are sent to a server and then the result needs to be parsed and processed as data models on the client. In addition, many API requests require authentication in order to identify the user making the request. This is typically done with a standard [OAuth](http://oauth.net/2/) process for authentication.

Fortunately, there are several OAuth libraries out there to simplify the process of authentication such as [scribe](https://github.com/fernandezpablo85/scribe-java) and [signpost](https://code.google.com/p/oauth-signpost/). You can explore several examples of [using scribe](https://github.com/fernandezpablo85/scribe-java/tree/master/src/test/java/org/scribe/examples) or [signpost](https://github.com/mttkay/signpost-examples) to authenticate.

We have also created a meta-library to make this process as simple as possible called [android-oauth-handler](https://github.com/codepath/android-oauth-handler) and a skeleton app to act as a template for a simple rest client called [android-rest-client-template](https://github.com/codepath/android-rest-client-template). You can see the details of these libraries by checking out their respective READMEs.

Using these wrappers, you can then send an API request and properly process the response using code like this:

```java
// SomeActivity.java
RestClient client = RestClientApp.getRestClient();
RequestParams params = new RequestParams();
params.put("key", "value");
params.put("more", "data");
client.getHomeTimeline(1, new JsonHttpResponseHandler() {
    public void onSuccess(int statusCode, Header[] headers, JSONObject json) {
        // Response is automatically parsed into a JSONArray
        // json.getJSONObject(0).getLong("id");
        // Here we want to process the json data into Java models.
    }

  public void onFailure(int statusCode, Header[] headers, Throwable t, JSONObject e)  {
    // Handle the failure and alert the user to retry
    Log.e("ERROR", e.toString());
  }
});
```

Note that as shown above you should also **handle failure cases** with [JsonHttpResponseHandler](http://loopj.com/android-async-http/doc/com/loopj/android/http/JsonHttpResponseHandler.html#onFailure\(java.lang.Throwable, org.json.JSONObject\)) using the `onFailure` method so your application is robust to "losing internet" and user doesn't become confused with unexpected results.

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

Displaying images is easiest using a third party library such as [Picasso](http://square.github.io/picasso/) from Square which will download and cache remote images and abstract the complexity behind an easy to use DSL.

After [downloading the Picasso jar](http://repo1.maven.org/maven2/com/squareup/picasso/picasso/2.4.0/picasso-2.4.0.jar), or adding Picasso to our `app/build.gradle` file:

```gradle
dependencies {
    compile 'com.squareup.picasso:picasso:2.4.0'
}
```

We can then load a remote image into any `ImageView` with:

```java
String imageUri = "http://i.imgur.com/tGbaZCY.jpg";
ImageView ivBasicImage = (ImageView) findViewById(R.id.ivBasicImage);
Picasso.with(context).load(imageUri).into(ivBasicImage);
```

We can do more sophisticated work with Picasso configuring placeholders, error handling, adjusting size of the image, and scale type with:

```java
Picasso.with(context).load(imageUri).fit().centerCrop()
    .placeholder(R.drawable.user_placeholder)
    .error(R.drawable.user_placeholder_error)
    .into(imageView);
```

For more details check out the [Picasso](http://square.github.io/picasso/) documentation. 

### Displaying Remote Images (The "Hard" Way)

Suppose we wanted to load an image using only the built-in Android network constructs. In order to download an image from the network, convert the bytes into a bitmap and then insert the bitmap into an imageview, you would use the following pseudo-code:

```java
// 1. Declare a URL Connection
URL url = new URL("http://i.imgur.com/tGbaZCY.jpg");
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
        String url = "http://i.imgur.com/tGbaZCY.jpg";
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
            InputStream in;
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
                 in.close();
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

Of course, doing this the "hard" way is not recommended. In most cases, to avoid having to manually manage caching and download management, we are better off creating your own libraries or in most cases utilizing existing [third-party libraries](http://square.github.io/picasso/). 

**Note:** If you use the approach above to download and display many images within a ListView, you might run into some threading issues that cause buggy loading of images. The blog post [Multithreading for Performance](http://android-developers.blogspot.com/2010/07/multithreading-for-performance.html) offers a solution in which you manage the active remote downloading background tasks to ensure that too many tasks are not being spun up at once. 

## References

 * <http://loopj.com/android-async-http/>
 * <http://developer.android.com/reference/java/net/HttpURLConnection.html>
 * <https://github.com/codepath/android-rest-client-template>
 * <http://developer.android.com/training/displaying-bitmaps/process-bitmap.html#concurrency>