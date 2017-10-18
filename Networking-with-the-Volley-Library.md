## Overview

Volley is a library that makes networking for Android apps easier and most importantly, faster. Volley Library was announced by [Ficus Kirkpatrick](https://plus.google.com/+FicusKirkpatrick) at Google I/O '13.
It was first used by the Play Store team in Play Store Application and then they released it as an open source library.  Although it is part of the Android Open Source Project (AOSP), Google announced in January 2017 that Volley will move to a [standalone library](https://plus.google.com/+IanLake/posts/EvB2BeqDUjC).

## Why Volley?

* Volley can pretty much do everything with that has to do with Networking in Android.
* Volley automatically schedules all network requests such as fetching responses for image from web.
* Volley provides transparent disk and memory caching.
* Volley provides powerful cancellation request API for canceling a single request or you can set blocks of requests to cancel.
* Volley provides powerful customization abilities.
* Volley provides debugging and tracing tools.

## Setup Volley

Adding Volley to our `app/build.gradle` file:

```gradle
dependencies {
    compile 'com.android.volley:volley:1.0.0'
}
```

And add the internet permission in `AndroidManifest.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.simplenetworking"
    android:versionCode="1"
    android:versionName="1.0" >
 
   <!-- Add permissions here -->
   <uses-permission android:name="android.permission.INTERNET" /> 
   <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />

</manifest>
```

## How to use Volley?

Volley has two classes that you will have to deal with:

1. `RequestQueue` - Requests are queued up here to be executed
2. `Request` (and any extension of it) - Constructing an network request

A Request object comes in three major types:

* [JsonObjectRequest](http://afzaln.com/volley/com/android/volley/toolbox/JsonObjectRequest.html) — To send and receive JSON Object from the server
* [JsonArrayRequest](http://afzaln.com/volley/com/android/volley/toolbox/JsonArrayRequest.html) — To receive JSON Array from the server
* [ImageRequest](http://afzaln.com/volley/com/android/volley/toolbox/ImageRequest.html) - To receive an image from the server
* [StringRequest](http://afzaln.com/volley/com/android/volley/toolbox/StringRequest.html) — To retrieve response body as String (ideally if you intend to parse the response by yourself)

### Constructing a RequestQueue

All requests in Volley are placed in a queue first and then processed, here is how you will be creating a request queue:

```java
public MainActivity extends Activity {
	private RequestQueue mRequestQueue;

	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.main_screen_layout);
		// ...
		mRequestQueue = Volley.newRequestQueue(this);
	}
}
```

### Creating a Singleton Queue

See [this guide for creating a singleton to use for sending requests](https://developer.android.com/training/volley/requestqueue.html). 

### Requesting images

Volley provides the ability to make image requests and receive back as bitmap.  You can use this bitmap to set directly onto an ImageView.

```java
ImageRequest imageRequest = new ImageRequest("https://i.imgur.com/Nwk25LA.jpg", 
  new Response.Listener<Bitmap>() {
    @Override
    public void onResponse(Bitmap response) {

    }, 
    // Image width & height equals 0 means to use the actual size
    0, 0, 
    // ImageView scale type
    ImageView.ScaleType.FIT_XY, 
    // 8 bytes per pixel image 
    Bitmap.Config.ARGB_8888, new Response.ErrorListener() {
      @Override
      public void onErrorResponse(VolleyError error) {
        error.printStackTrace();
      }
    });
```

### Accessing JSON Data

After this step you are ready to create your `Request` objects which represents a desired request to be executed. Then we add that request onto the queue. 

```java
public class MainActivity extends Activity {
	private RequestQueue mRequestQueue;

        // ...

	private void fetchJsonResponse() {
		// Pass second argument as "null" for GET requests
		JsonObjectRequest req = new JsonObjectRequest(Request.Method.GET, "http://ip.jsontest.com/", null,
		    new Response.Listener<JSONObject>() {
		        @Override
		        public void onResponse(JSONObject response) {
		            try {
		                String result = "Your IP Address is " + response.getString("ip");
		                Toast.makeText(MainActivity.this, result, Toast.LENGTH_SHORT).show();
		            } catch (JSONException e) {
		                e.printStackTrace();
		            }
		        }
		    }, new Response.ErrorListener() {
		        @Override
		        public void onErrorResponse(VolleyError error) {
		            VolleyLog.e("Error: ", error.getMessage());
		        }
		});

		/* Add your Requests to the RequestQueue to execute */
		mRequestQueue.add(req);
	}
}
```

And that will execute the request to the server and respond back with the result as specified in the `Response.Listener` callback. For a more detailed look at Volley, check out [this volley tutorial](http://arnab.ch/blog/2013/08/asynchronous-http-requests-in-android-using-volley/).

### Canceling Requests

You can tag a request with:

```java
StringRequest stringRequest = ...;
RequestQueue mRequestQueue = ...;

// Set the tag on the request.
stringRequest.setTag(TAG);

// Add the request to the RequestQueue.
mRequestQueue.add(stringRequest);
```

You can now cancel all requests with this tag using the `cancelAll` on the request queue:

```java
@Override
protected void onStop() {
    super.onStop();
    if (mRequestQueue != null) {
        mRequestQueue.cancelAll(TAG);
    }
}
```

### Using with OkHttp

Although Volley uses the `HttpUrlConnection` interface for networking calls, the OkHttp library instead can be used instead since it also provides an implementation of this interface:

```java
public class OkHttpStack extends HurlStack {
  private final OkHttpClient client;

  public OkHttpStack() {
    this(new OkHttpClient());
  }

  public OkHttpStack(OkHttpClient client) {
    if (client == null) {
      throw new NullPointerException("Client must not be null.");
    }
    this.client = client;
  }

  @Override protected HttpURLConnection createConnection(URL url) throws IOException {
    return client.open(url);
  }   
}
```

You can then generate a request queue:

```java
Volley.newRequestQueue(context, new OkHttpStack());
```

Follow this [gist](https://gist.github.com/JakeWharton/5616899) for more information.

### Troubleshooting

You can enable verbose logging by simplify setting `VolleyLog.DEBUG` to be true before issuing network requests:

```java
VolleyLog.DEBUG = true;
```

You can also activate verbose logging after an app has already been running by typing this command using the Android Debug Shell (ADB):

```bash
adb shell setprop log.tag.Volley VERBOSE
```

The output will show cache hits, queue additions, and network latency calls:

```
03-13 23:32:11.382 2565-2565/com.test D/Volley: [1] MarkerLog.finish: (1494 ms) [ ] https://i.imgur.com/Nwk25LA.jpg 0x189700ee LOW 1
03-13 23:32:11.382 2565-2565/com.test D/Volley: [1] MarkerLog.finish: (+0   ) [ 1] add-to-queue
03-13 23:32:11.382 2565-2565/com.test D/Volley: [1] MarkerLog.finish: (+85  ) [191] cache-queue-take
03-13 23:32:11.382 2565-2565/com.test D/Volley: [1] MarkerLog.finish: (+107 ) [191] cache-hit
03-13 23:32:11.383 2565-2565/com.test D/Volley: [1] MarkerLog.finish: (+957 ) [191] cache-hit-parsed
03-13 23:32:11.383 2565-2565/com.test D/Volley: [1] MarkerLog.finish: (+0   ) [191] post-response
03-13 23:32:11.383 2565-2565/com.test D/Volley: [1] MarkerLog.finish: (+345 ) [ 1] done
```

## References

* <https://developer.android.com/training/volley/index.html>
* <https://smaspe.github.io/2013/06/03/volley-part1.html>
* <http://java.dzone.com/articles/android-%E2%80%93-volley-library>
* <http://arnab.ch/blog/2013/08/asynchronous-http-requests-in-android-using-volley/>
* <http://files.evancharlton.com/volley-docs/>
* <https://www.youtube.com/watch?v=yhv8l9F44qo/>
* <https://commondatastorage.googleapis.com/io-2013/presentations/110%20-%20Volley-%20Easy,%20Fast%20Networking%20for%20Android.pdf>
