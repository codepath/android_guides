## Overview

Network requests are used to retrieve or modify API or media data from a server. This is extremely common in Android development especially for dynamic data-driven clients.

The underlying class used for network connections is [URLConnection](http://developer.android.com/reference/java/net/HttpURLConnection.html) or [DefaultHTTPClient](http://developer.android.com/reference/org/apache/http/impl/client/DefaultHttpClient.html). Both of these are lower-level and require completely manual management of parsing the data from the input stream and executing the request asynchronously.

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

### Network Requests (The Hard Way)

Here's the code to construct an AsyncTask to download a remote image and display the image in an ImageView using just the official Google Android SDK.

```java
public class MainActivity extends Activity {
	
    private ImageView ivBasicImage;

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
           return downloadImageFromUri(addresses[0]);
        }
        
        private Bitmap downloadImageFromUri(String address) {
        	URL url;
    		try {
    			url = new URL(address);
    		} catch (MalformedURLException e1) {
    			url = null;
    		}

    	    URLConnection conn;
    	    // Define an InputStream
    	    InputStream in;
    	    Bitmap bitmap;
    		try {
    			// Open connection
    			conn = url.openConnection();
    			conn.connect();
    			in = conn.getInputStream();
    			// Decode bitmap
    			bitmap = BitmapFactory.decodeStream(in);
    			// Close the input stream
    			in.close();
    		} catch (IOException e) {
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

## References

 * <http://loopj.com/android-async-http/>
 * <http://developer.android.com/reference/java/net/HttpURLConnection.html>
 * <https://github.com/thecodepath/android-rest-client-template>