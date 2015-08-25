## Overview

A popular third-party library called [android-async-http](http://loopj.com/android-async-http/) helps handle the entire process of sending and parsing network requests for us in a more robust and easy-to-use way.

### Setup

We simply need to add the library to our `app/build.gradle` file:

```gradle
dependencies {
  compile 'com.loopj.android:android-async-http:1.4.8'
}
```

#### Android Marshmallow Compatibility Issues

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