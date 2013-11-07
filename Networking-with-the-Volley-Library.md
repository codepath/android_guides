## Overview

Volley is a library that makes networking for Android apps easier and most importantly, faster. Volley Library was announced by [Ficus Kirkpatrick](https://plus.google.com/+FicusKirkpatrick) at Google I/O '13!
It was first used by the Play Store team in Play Store Application and then they released it as an Open Source Library.

## Why Volley?

* Volley can pretty much do everything with that has to do with Networking in Android.
* Volley automatically schedules all network requests such as fetching responses for image from web.
* Volley provides transparent disk and memory caching.
* Volley provides powerful cancellation request API for canceling a single request or you can set blocks of requests to cancel.
* Volley provides powerful customization abilities.
* Volley provides debugging and tracing tools.

## Getting started

We need to install volley as a [library project](http://imgur.com/a/N8baF). First, download the volley source code:

```bash
git clone https://android.googlesource.com/platform/frameworks/volley
```

and then we need to import the source code into our workspace and then mark it as a library. Now we can add this library as a dependency of any of our apps.

## How to use Volley?

Volley has two classes that you will have to deal with:

1. `RequestQueue` - Requests are queued up here to be executed
2. `Request` (and any extension of it) - Constructing an network request

A Request object comes in three major types:

* [JsonObjectRequest](http://files.evancharlton.com/volley-docs/com/android/volley/toolbox/JsonObjectRequest.html) — To send and receive JSON Object from the server
* [JsonArrayRequest](http://files.evancharlton.com/volley-docs/com/android/volley/toolbox/JsonArrayRequest.html) — To receive JSON Array from the server
* [ImageRequest](http://files.evancharlton.com/volley-docs/com/android/volley/toolbox/ImageRequest.html) - To receive an image from the server
* [StringRequest](http://files.evancharlton.com/volley-docs/com/android/volley/toolbox/StringRequest.html) — To retrieve response body as String (ideally if you intend to parse the response by yourself)

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

### Accessing JSON Data

After this step you are ready to create your `Request` objects which represents a desired request to be executed. Then we add that request onto the queue. 

```java
public class MainActivity extends Activity {
	private RequestQueue mRequestQueue;

        // ...

	private void fetchJsonResponse() {
		// Pass second argument as "null" for GET requests
		JsonObjectRequest req = new JsonObjectRequest("http://ip.jsontest.com/", null,
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

## References

* <http://njzk2.wordpress.com/2013/06/03/a-volley-introduction-part-1/>
* <http://java.dzone.com/articles/android-%E2%80%93-volley-library>
* <http://arnab.ch/blog/2013/08/asynchronous-http-requests-in-android-using-volley/>
* <http://files.evancharlton.com/volley-docs/>