## Overview

CodePath built a [simple lightweight library](https://github.com/codepath/AsyncHttpClient) to make it easy to send and parse network requests.  The goal of this library is to have an API that clearly and cleanly supports the following features:

 * Asynchronous network requests without any need for manual thread handling
 * Response `onSuccess` callbacks run on the mainthread (by default)
 * Easy way to catch all errors and failures and handle them
 * Easy pluggable Text, JSON, and Bitmap response handlers to parse the response
 
This client tries to follow a similar API inspired by this [older now deprecated android-async-http library](https://github.com/android-async-http/android-async-http).

### Setup

To use this library, add the following dependency into our `app/build.gradle` file:

```gradle
dependencies {
  implementation 'com.codepath.libraries:asynchttpclient:0.0.9'
}
```

### Sending a Network Request

Now, we just create an `AsyncHttpClient`, and then execute a request specifying an anonymous class as a callback:

```java
import com.codepath.asynchttpclient.AsyncHttpClient;

AsyncHttpClient client = new AsyncHttpClient();
RequestParams params = new RequestParams();
params.put("limit", "5");
params.put("page", 0);
client.get("https://api.thecatapi.com/v1/images/search", params, new TextHttpResponseHandler() {
        @Override
        public void onSuccess(int statusCode, Headers headers, String response) {
            // called when response HTTP status is "200 OK"
        }

        @Override
        public void onFailure(int statusCode, Headers headers, String errorResponse, Throwable t) {
            // called when response HTTP status is "4XX" (eg. 401, 403, 404)
        }	
    }
);
```

This will automatically execute the request asynchronously and fire the `onSuccess` when the response returns a success code and `onFailure` if the response does not.

### Sending a JSON Request

Similar to sending a regular HTTP request, this library can also be used for sending JSON API requests:

```java
RequestParams params = new RequestParams();
params.put("limit", "5");
params.put("page", 0);

client.get("https://api.thecatapi.com/v1/images/search", params, new JsonHttpResponseHandler() {
   @Override
   public void onSuccess(int statusCode, Headers headers, JSON json) {
         // Access a JSON array response with `json.jsonArray` 
         Log.d("DEBUG ARRAY", json.jsonArray.toString());
         // Access a JSON object response with `json.jsonObject` 
         Log.d("DEBUG OBJECT", json.jsonObject.toString());
   }

   @Override
   public void onFailure(int statusCode, Headers headers, String response, Throwable throwable) {

   }
});
```

Assuming the callback uses `JsonHttpResponseHandler`, the request will be sent out with the appropriate parameters passed in the query string and then the response can be parsed as JSON and made available within `onSuccess`. 

The `JSON` returned can be either an Array or an Object, and you can access them with `.jsonArray` and `.jsonObject` respectively depending on the root JSON content of the response. Check the [[Converting JSON to Models]] guide for more details on parsing a JSON response manually.

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
    public void onSuccess(int statusCode, Headers headers, JSON json) {
        // Response is automatically parsed into a JSONArray
        // json.jsonObject.getLong("id");
        // Here we want to process the json data into Java models.
    }

  public void onFailure(int statusCode, Headers headers, String responseString, Throwable t)  {
    // Handle the failure and alert the user to retry
    Log.e("ERROR", e.toString());
  }
});
```
