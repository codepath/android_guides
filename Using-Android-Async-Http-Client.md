## Overview

A popular third-party library called [android-async-http](http://loopj.com/android-async-http/) helps handle the entire process of sending and parsing network requests for us in a more robust and easy-to-use way.

### Setup

Add this library to our `app/build.gradle` file:

```gradle
dependencies {
  compile 'com.loopj.android:android-async-http:1.4.9'
}
```

If you are upgrading from previous versions, you need to change these import statements for the `Header` class from:

```java
import org.apache.http.Header;
```

Replaced with this line:

```java
import cz.msebera.android.httpclient.Header;
```

If you have any other import statements that start with `org.apache.http`, you also need to change them to `cz.msebera.android.httpclient`.

Mac users can use the following shortcut:
```bash
brew install gnu-sed
find . -name '*.java' -exec gsed -i 's/org.apache.http/cz.msebera.android.httpclient/g' \{\} +
```

### Sending a Network Request

Now, we just create an `AsyncHttpClient`, and then execute a request specifying an anonymous class as a callback:

```java
import com.loopj.android.http.*;
import cz.msebera.android.httpclient.Header;

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
    public void onFailure(int statusCode, Header[] headers, String res, Throwable t) {
       // called when response HTTP status is "4XX" (eg. 401, 403, 404)
    }
});
```

Assuming the callback uses `JsonHttpResponseHandler`, the request will be sent out with the appropriate parameters passed in the query string and then the response can be parsed as JSON and made available within `onSuccess`. Check the [[Converting JSON to Models]] guide for more details on parsing a JSON response manually.

#### Decoding with GSON library

If you wish to convert the JSON model directly to a Java object that corresponds directly to the field names, you can use a `TextHttpResponseHandler` instead of a `JsonhttpResponseHandler` and follow the [[Leveraging the Gson Library]] guide.  This approach works also for nested JSON objects too.

```java
client.get(url, params, new TextHttpResponseHandler() {

    @Override
    public void onSuccess(int statusCode, Header[] headers, String r
        Gson gson = new GsonBuilder().create();
        // Define Response class to correspond to the JSON response returned
        gson.fromJson(responseString, Response.class);
    }
});
```

#### Sending an Authenticated API Request

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

Note that as shown above you should also **handle failure cases** with 
[JsonHttpResultHandler](http://loopj.com/android-async-http/doc/com/loopj/android/http/JsonHttpResponseHandler.html#onFailure(java.lang.Throwable,%20org.json.JSONObject)) using the `onFailure` method so your application is robust to "losing internet" and user doesn't become confused with unexpected results.