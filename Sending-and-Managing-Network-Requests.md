## Overview

Network requests are used to retrieve or modify API data or media from a server. This is a very common task in Android development especially for dynamic data-driven clients.

The underlying Java class used for network connections is [URLConnection](http://developer.android.com/reference/java/net/HttpURLConnection.html) or [DefaultHTTPClient](http://developer.android.com/reference/org/apache/http/impl/client/DefaultHttpClient.html). Both of these are lower-level and require completely manual management of parsing the data from the input stream and executing the request asynchronously.

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
</manifest>
```

### Sending an HTTP Request

Using [android-async-http](http://loopj.com/android-async-http/), sending an HTTP request becomes quite easy. Simply create an http client, and then execute a request specifying an anonymous class as a callback:

```java
import com.loopj.android.http.*;

AsyncHttpClient client = new AsyncHttpClient();
client.get("http://www.google.com", new
    AsyncHttpResponseHandler() {
        @Override
        public void onSuccess(String response) {
            System.out.println(response);
        }
    }
);
```

### Sending an API Request

API requests tend to be JSON or XML responses that are sent to a server and then the result needs to be parsed and processed as data models on the client.

In addition, many API requests require authentication in order to identify the user making the request. This is typically done with a standard [OAuth]() process for authentication.

We have created a library to make this as simple as possible called [android-oauth-handler](https://github.com/thecodepath/android-oauth-handler) and a skeleton app to act as a template for a simple rest client called [android-rest-client-template](https://github.com/thecodepath/android-rest-client-template). You can see the details of these libraries by checking out their READMEs.

Using these wrappers, you can then send an API request and properly process the response using code like this:

```java
// SomeActivity.java
RestClient client = RestClientApp.getRestClient();
client.getHomeTimeline(1, new JsonHttpResponseHandler() {
  public void onSuccess(JSONArray json) {
    // Response is automatically parsed into a JSONArray
    // json.getJSONObject(0).getLong("id");
    // Here we want to process the json data into Java models.
  }
});
```

You should also **handle failure cases** with [JsonHttpResponseHandler](http://loopj.com/android-async-http/doc/com/loopj/android/http/JsonHttpResponseHandler.html#onFailure(java.lang.Throwable, org.json.JSONObject\)) using the `onFailure` method:

```java
client.getHomeTimeline(1, new JsonHttpResponseHandler() {
  public void onFailure(Throwable e, JSONObject error) {
    // Handle the failure and alert the user to retry
    Log.e("ERROR", e.toString());
  }
});
```

Always handle failure cases so your application is robust to "losing internet" and user doesn't become confused.

### Displaying Remote Images (The "Easy" Way)

In contrast, using the third party library [android-universal-image-loader](https://github.com/nostra13/Android-Universal-Image-Loader) after [proper setup](https://github.com/nostra13/Android-Universal-Image-Loader#quick-setup), we can download an image like this:

```java
String imageUri = "http://2.gravatar.com/avatar/858dfac47ab8176458c005414d3f0c36?s=128&d=&r=G";
ImageView ivBasicImage = (ImageView) findViewById(R.id.ivBasicImage);
imageLoader.displayImage(imageUri, ivBasicImage);
```

The important thing here is to notice just how much cleaner and clearer app develop can be when using third-party high level libraries. 

Note that a popular alternative to the universal-image-loader library is called [Picasso](http://square.github.io/picasso/) which serves a similar purpose. After [downloading the Picasso jar](http://repo1.maven.org/maven2/com/squareup/picasso/picasso/2.1.1/picasso-2.1.1.jar), we can load an image with:

```java
Picasso.with(context).load(imageUri).into(ivBasicImage);
```

For more details check out the [Picasso](http://square.github.io/picasso/) documentation.

### Displaying Remote Images (The "Hard" Way)

Here's the code to construct an AsyncTask to download a remote image and display the image in an ImageView using just the official Google Android SDK. 

```java
package com.example.simplenetworking;

public class MainActivity extends Activity {
	ImageView ivBasicImage;

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		ivBasicImage = (ImageView) findViewById(R.id.ivBasicImage);
		String imageUrl = "http://2.gravatar.com/avatar/858dfac47ab8176458c005414d3f0c36?s=128&d=&r=G";
		new ImageDownloadTask().execute(imageUrl);
	}

	private class ImageDownloadTask extends AsyncTask<String, Void, Bitmap> {
		protected Bitmap doInBackground(String... addresses) {
			// Convert string to URL
			URL url = getUrlFromString(addresses[0]);
			// Get input stream
			InputStream in = getInputStream(url);
			// Decode bitmap
			Bitmap bitmap = decodeBitmap(in);
			// Return bitmap result
			return bitmap;
		}

		private URL getUrlFromString(String address) {
			URL url;
			try {
				url = new URL(address);
			} catch (MalformedURLException e1) {
				url = null;
			}
			return url;
		}

		private InputStream getInputStream(URL url) {
			InputStream in;
			// Open connection
			URLConnection conn;
			try {
				conn = url.openConnection();
				conn.connect();
				in = conn.getInputStream();
			} catch (IOException e) {
				in = null;
			}
			return in;
		}

		private Bitmap decodeBitmap(InputStream in) {
			Bitmap bitmap;
			try {
				// Turn response into Bitmap
				bitmap = BitmapFactory.decodeStream(in);
				// Close the input stream
				in.close();
			} catch (IOException e) {
				in = null;
				bitmap = null;
			}
			return bitmap;
		}

		@Override
		protected void onPostExecute(Bitmap result) {
			// Set bitmap image for the result
			ivBasicImage.setImageBitmap(result);
		}
	}
}
```

Of course, doing this the "hard" way is not recommended. In most cases, to avoid having to manually manage caching and download management, we are better off creating your own libraries or in most cases utilizing existing [third-party libraries](https://github.com/nostra13/Android-Universal-Image-Loader). 

**Note:** If you use to above approach to download and display many images within a ListView, you might run into some threading issues that cause buggy loading of images. The blog post [Multithreading for Performance](http://android-developers.blogspot.com/2010/07/multithreading-for-performance.html) offers a solution in which you manage the active remote downloading background tasks to ensure that too many tasks are not being spun up at once. 

## References

 * <http://loopj.com/android-async-http/>
 * <http://developer.android.com/reference/java/net/HttpURLConnection.html>
 * <https://github.com/thecodepath/android-rest-client-template>
 * <http://developer.android.com/training/displaying-bitmaps/process-bitmap.html#concurrency>