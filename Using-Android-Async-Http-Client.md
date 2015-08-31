## Overview

A popular third-party library called [android-async-http](http://loopj.com/android-async-http/) helps handle the entire process of sending and parsing network requests for us in a more robust and easy-to-use way.

### Setup

We simply need to add the library to our `app/build.gradle` file:

```gradle
dependencies {
  compile 'com.loopj.android:android-async-http:1.4.8'
}
```

#### Resolving Android Marshmallow Compatibility Issues

##### Option 1: rely on the `useLibrary` Gradle command

Apache HTTP client (a dependency of [android-async-http](http://loopj.com/android-async-http/)) has been [removed from Marshmallow](http://developer.android.com/preview/behavior-changes.html#behavior-apache-http-client). If your app targets API 23, you'll need to add the following to your gradle file until the [library is updated](https://github.com/loopj/android-async-http/issues/830):

  ```gradle
  android {
      compileSdkVersion 23
      useLibrary 'org.apache.http.legacy'   // required if compileSdkVersion >= 23
  }
  ```

You may also need to add the import statement manually to your Java file wherever you make network calls with this library:

```java
import org.apache.http.Header;
```

The reason is that is a current bug in Android Studio 1.3.1 where it may not recognized this added library.  You will notice that Android Studio will not recognized the module:

  <img src="https://i.imgur.com/jreDUla.png"/>

Assuming you have included the `useLibrary` statement, your build should however compile successfully.  The Gradle configuration will add this library to the Java classpath, but the IDE currently has a bug where it is not recognized as an added dependency.

##### Option #2: add the Apache Http library manually

You can copy the `org.apache.http.legacy.jar` file to the `libs/` dir of your app.  This JAR is included with Android 23 in the respective locations:

Mac OS users: `/Users/[username]/Library/Android/sdk/platforms/android-23/optional`

PC users: `C:\Documents and Settings\<user>\AppData\Local\Android\sdk\platforms\android-23\optional`

Copy this JAR file to your `app/libs` dir.  Make sure to `Mark as Library` if the file does not get expanded.  You should see something similar to the following screenshot if the library was added correctly:

<img src="http://i.imgur.com/AtvF9Vm.png">

#### Fixing issues with Garbled JSON Response

If you are noticing garbled data in the responses, it's likely that you have encountered a bug in the Android Async HTTP Client library:

<img src="https://cloud.githubusercontent.com/assets/126405/5264140/d878ea00-7a40-11e4-867f-6515814861a0.png"/>

The current workaround is to disable Gzip compression as described in this [GitHub comment](https://github.com/loopj/android-async-http/issues/932#issuecomment-134549073):

```java
httpClient.addHeader("Accept-Encoding", "identity"); // disable gzip
```

You can also downgrade to an earlier version of the library:

```gradle
compile 'com.loopj.android:android-async-http:1.4.4'
```

### Sending a Network Request

Now, we just create an `AsyncHttpClient`, and then execute a request specifying an anonymous class as a callback:

```java
import com.loopj.android.http.*;
import org.apache.http.Header;

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

The request will be sent out with the appropriate parameters passed in the query string and then the response will be parsed as JSON and made available within `onSuccess`. Check the [[Converting JSON to Models]] guide for more details on parsing a JSON response.

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

Note that as shown above you should also **handle failure cases** with [JsonHttpResponseHandler](http://loopj.com/android-async-http/doc/com/loopj/android/http/JsonHttpResponseHandler.html#onFailure\(java.lang.Throwable, org.json.JSONObject\)) using the `onFailure` method so your application is robust to "losing internet" and user doesn't become confused with unexpected results.