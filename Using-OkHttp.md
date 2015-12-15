## Overview

OkHttp is a third-party library developed by Square for sending and receive HTTP-based network requests.  It is built on top of the [Okio](https://corner.squareup.com/2014/04/okio.html) library, which tries to be more efficient about reading and writing data than
the standard Java I/O libraries by creating a [shared memory pool](https://www.youtube.com/watch?v=WvyScM_S88c&feature=youtu.be).  It also is the underlying library for [[Retrofit|Consuming-APIs-with-Retrofit]] library that provides type safety for consuming REST-based APIs.

The OkHttp library actually provides an [implementation](https://github.com/square/okhttp/blob/master/okhttp-urlconnection/src/main/java/com/squareup/okhttp/internal/huc/HttpURLConnectionImpl.java) of the [HttpUrlConnection](http://developer.android.com/reference/java/net/HttpURLConnection.html)
interface, which Android 4.4 and later versions now use.  Therefore, when using the manual approach described in this [[section|Sending-and-Managing-Network-Requests#sending-an-http-request-the-hard-way]] of the guide, the underlying `HttpUrlConnection` class may be leveraging code from the OkHttp library. However, there is a separate API provided by OkHttp that makes it easier to send and receive network requests, which is described in this guide.  

In addition, [OkHttp v2.4](https://corner.squareup.com/2015/05/okhttp-2-4.html) also provides a more updated way of managing URLs internally.  Instead of the `java.net.URL`, `java.net.URI`, or `android.net.Uri` classes, it provides a new `HttpUrl` class that makes it easier to get an HTTP port, parse URLs, and canonicalizing URL strings.

## Setup

Makes sure to enable the use of the Internet permission in your `AndroidManifest.xml` file:

```java
<uses-permission android:name="android.permission.INTERNET"/>
```

Simply add this line to your `app/build.gradle` file:

```gradle
compile 'com.squareup.okhttp:okhttp:2.7.0'
```

## Sending and Receiving Network Requests

First, we must instantiate an OkHttpClient and create a `Request` object:

```java
OkHttpClient client = new OkHttpClient();

Request request = new Request.Builder()
                     .url("http://publicobject.com/helloworld.txt")
                     .build();
```

If there are any query parameters that need to be added, the `HttpUrl` class provided by OkHttp can be leveraged to construct the URL:

```java
HttpUrl.Builder urlBuilder = HttpUrl.parse("https://ajax.googleapis.com/ajax/services/search/images").newBuilder();
urlBuilder.addQueryParameter("v", "1.0");
urlBuilder.addQueryParameter("q", "android");
urlBuilder.addQueryParameter("rsz", "8");
String url = urlBuilder.build().toString();

Request request = new Request.Builder()
                     .url(url)
                     .build();
```

If there are any authenticated query parameters, headers can be added to the request too:

```java
Request request = new Request.Builder()
    .header("Authorization", "token abcd")
    .url("https://api.github.com/users/codepath")
    .build();
```

### Synchronous Network Calls

We can create a `Call` object and dispatch the network request synchronously:

```java
Response response = client.newCall(request).execute();
```

Because Android disallows network calls on the main thread, you can only make synchronous calls if you do so on a separate thread or a background service.  You can use also use [[AsyncTask|Creating-and-Executing-Async-Tasks]] for lightweight network calls.

### Asynchronous Network Calls

We can also make asynchronous network calls too by creating a `Call` object, using the `enqueue()` method, and
passing an anonymous `Callback` object that implements both `onFailure()` and `onResponse()`.  

```java
// Get a handler that can be used to post to the main thread
client.newCall(request).enqueue(new Callback() {
    @Override
    public void onFailure(Request request, IOException e) {
        e.printStackTrace();
    }

    @Override
    public void onResponse(final Response response) throws IOException {
        if (!response.isSuccessful()) {
            throw new IOException("Unexpected code " + response);
        }
    }
}
```

OkHttp normally creates a new worker thread to dispatch the network request and uses the same thread to handle the response.  It is built [primarily as a Java library](http://stackoverflow.com/questions/24246783/okhttp-response-callbacks-on-the-main-thread#comment45410547_24248963) so does not handle the Android framework limitations that only permit views to be updated on the main UI thread.  If you need to update any views, you will need to use `runOnUiThread()` or post the result back on the main thread.  See [[this guide|Managing-Threads-and-Custom-Services#handler-and-loopers]] for more context.

```java
client.newCall(request).enqueue(new Callback() {
    @Override
    public void onResponse(final Response response) throws IOException {
        // ... check for failure using `isSuccessful` before proceeding

        // Read data on the worker thread
        final String responseData = response.body().string();

        // Run view-related code back on the main thread
        MainActivity.this.runOnUiThread(new Runnable() {
            @Override
            public void run() {
                try {
                    TextView myTextView = (TextView) findViewById(R.id.myTextView);
                    myTextView.setText(responseData);
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
       }
    }
});
```

## Processing Network Responses

Assuming the request is not cancelled and there are no connectivity issues, the `onResponse()` method will be fired. It passes a `Response` object that can be used to check the status code, the response body, and any headers that were returned. Calling `isSuccessful()` for instance if the code returned a status code of 2XX (i.e. 200, 201, etc.)

```java
if (!response.isSuccessful()) {
    throw new IOException("Unexpected code " + response);
}
```

The header responses are also provided as a list:

```java
Headers responseHeaders = response.headers();
for (int i = 0; i < responseHeaders.size(); i++) {
  Log.d("DEBUG", responseHeaders.name(i) + ": " + responseHeaders.value(i));
}
```

The headers can also be access directly using `response.header()`:

```java
String header = response.header("Date");
```

We can also get the response data by calling `response.body()` and then calling `string()` to read the entire payload.  Note that `response.body()` can only be run once and should be done on a background thread.

```java
Log.d("DEBUG", response.body().string());
```

### Processing JSON data

Suppose we make a call to the GitHub API, which returns JSON-based data:

```java
Request request = new Request.Builder()
             .url("https://api.github.com/users/codepath")
             .build();
```

We can also decode the data by converting it to a `JSONObject` or `JSONArray`, depending on the response data:

```java
client.newCall(request).enqueue(new Callback() {
    @Override
    public void onResponse(final Response response) throws IOException {  
        try {
            String responseData = response.body().string();
            JSONObject json = new JSONObject(responseData);
            final String owner = json.getString("name");
        } catch (JSONException e) {

        }
    }
});
```

### Processing JSON data with Gson

Note that the `string()` method on the response body will load the entire data into memory.  To make more efficient use of memory, it is recommended that the response be processed as a stream by using `charStream()` instead.  This approach however requires using the [Gson](https://github.com/google/gson) library.  See [[this guide|Leveraging-the-Gson-Library]] for setup instructions.

To use the Gson library, we first must declare a class that maps directly to the JSON response:

```java
static class GitUser {
    String name;
    String url;
    int id;
}
```

We can then use the Gson parser to convert the data directly to a Java model:

```java
// Create new gson object
final Gson gson = new Gson();
// Get a handler that can be used to post to the main thread
client.newCall(request).enqueue(new Callback() {
    // Parse response using gson deserializer
    @Override
    public void onResponse(final Response response) throws IOException {
        // Process the data on the worker thread
        GitUser user = gson.fromJson(response.body().charStream(), GitUser.class);
        // Access deserialized user object here
    }
}
```

### Sending Authenticated Requests

OkHttp has a mechanism to modify outbound requests using [interceptors](https://github.com/square/okhttp/wiki/Interceptors).
A common use case is the OAuth protocol, which requires requests to be signed using a private key.  The [OkHttp signpost library](https://github.com/pakerfeldt/okhttp-signpost) works with the [SignPost library](https://github.com/mttkay/signpost) to use an interceptor to sign each request.  This way, the caller does not need to remember to sign each request:

```java
OkHttpOAuthConsumer consumer = new OkHttpOAuthConsumer(CONSUMER_KEY, CONSUMER_SECRET);
consumer.setTokenWithSecret(token, secret);
okHttpClient.interceptors().add(new SigningInterceptor(consumer));
```

### Recipe guide

Check out Square's official [recipe guide](https://github.com/square/okhttp/wiki/Recipes) for other examples of using OkHttp.

## References

* [Android’s HTTP Clients - Android Developers blog, September 29, 2011](http://android-developers.blogspot.com/2011/09/androids-http-clients.html)
* <https://www.reddit.com/r/androiddev/comments/29p3zz/til_okhttp_engine_is_backing_httpurlconnection_as/>
* <https://github.com/square/okhttp/blob/master/okhttp-urlconnection/src/main/java/com/squareup/okhttp/internal/huc/HttpURLConnectionImpl.java>
* <https://speakerdeck.com/jakewharton/a-few-ok-libraries-droidcon-mtl-2015>
* <https://github.com/square/okio#sources-and-sink>
* <http://stackoverflow.com/questions/24246783/okhttp-response-callbacks-on-the-main-thread>
* <https://github.com/square/okhttp/wiki/Recipes>
* <https://twitter.com/jakewharton/status/482563299511250944>