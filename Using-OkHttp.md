## Overview

OkHttp is a third-party library developed by Square for sending and receive HTTP-based network requests.  It is built on top of the [Okio](https://corner.squareup.com/2014/04/okio.html) library, which tries to be more efficient about reading and writing data than
the standard Java I/O libraries by creating a [shared memory pool](https://www.youtube.com/watch?v=WvyScM_S88c&feature=youtu.be).

The OkHttp library actually provides an implementation of the [[HttpUrlConnection|Sending-and-Managing-Network-Requests#sending-an-http-request-the-hard-way]]
interface. Android 4.4 and later versions now use this [implementation](https://github.com/square/okhttp/blob/master/okhttp-urlconnection/src/main/java/com/squareup/okhttp/internal/huc/HttpURLConnectionImpl.java), so the manual approach described in this [[section|Sending-and-Managing-Network-Requests#sending-an-http-request-the-hard-way]] actually uses the OkHttp library.
However, there is a separate API provided by OkHttp that makes it easier to send and receive network requests, which is described in this guide.  

In addition, [OkHttp v2.4](https://corner.squareup.com/2015/05/okhttp-2-4.html) also provides a more updated way of managing URLs internally.  Instead of the `java.net.URL`, `java.net.URI`, or `android.net.Uri` classes, it provides a new `HttpUrl` class that makes it easier to get an HTTP port, parse URLs, and canonicalizing URL strings.

## Setup

Makes sure to enable the use of the Internet permission in your `AndroidManifest.xml` file:

```java
<uses-permission android:name="android.permission.INTERNET"/>
```

Simply add this line to your `app/build.gradle` file:

```gradle
compile 'com.squareup.okhttp:okhttp:2.5.0'
```

## Sending and Receiving Network Requests

First, we must instantiate an OkHttpClient and create a `Request` object:

```java
OkHttpClient client = new OkHttpClient();

Request request = new Request.Builder()
                     .url("http://publicobject.com/helloworld.txt")
                     .build();
```

### Synchronous network calls

We can create a `Call` object and dispatch the network request synchronously:

```java
Response response = client.newCall(request).execute();
```

Because Android disallows network calls on the main thread, you can only make synchronous calls if you do so on a separate thread or a background service.  You can use also use [[AsyncTask|Creating-and-Executing-Async-Tasks]] for lightweight network calls.

Assuming the response returns, we can check whether it was successful and parse the response headers and body. Calling `string()` on the response will retrieve the entire data into memory, so only make this call for small payloads.

```java
if (!response.isSuccessful()) {
  throw new IOException("Unexpected code " + response);
} 

Headers responseHeaders = response.headers();
for (int i = 0; i < responseHeaders.size(); i++) {
  Log.d("DEBUG", responseHeaders.name(i) + ": " + responseHeaders.value(i));
}

Log.d("DEBUG", response.body().string());
}
```

### Asynchronous network calls

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
```

OkHttp normally creates a new worker thread to dispatch the network request and uses the same thread
to handle the response.  Note that if you need to update any views, you can only make these updates
on the main UI thread.  Therefore, you will need to use `runOnUiThread()` or post the result back on the main thread:

```java
@Override
public void onResponse(final Response response) throws IOException {
if (!response.isSuccessful()) {
  throw new IOException("Unexpected code " + response);
}

MainActivity.this.runOnUiThread(new Runnable() {
  @Override
  public void run() {
    try {
       TextView myTextView = (TextView) findViewById(R.id.myTextView);
       myTextView.setText(response.body().string());
    } catch (IOException e) {
      e.printStackTrace();
    }
  }
});
```

Check out Square's official [recipe guide](https://github.com/square/okhttp/wiki/Recipes) for other examples.

## References

* [Android’s HTTP Clients - Android Developers blog, September 29, 2011](http://android-developers.blogspot.com/2011/09/androids-http-clients.html)
* <https://www.reddit.com/r/androiddev/comments/29p3zz/til_okhttp_engine_is_backing_httpurlconnection_as/>
* <https://github.com/square/okhttp/blob/master/okhttp-urlconnection/src/main/java/com/squareup/okhttp/internal/huc/HttpURLConnectionImpl.java>
* <https://speakerdeck.com/jakewharton/a-few-ok-libraries-droidcon-mtl-2015>
* <https://github.com/square/okio#sources-and-sink>
* <http://stackoverflow.com/questions/24246783/okhttp-response-callbacks-on-the-main-thread>
* <https://github.com/square/okhttp/wiki/Recipes>
* <https://twitter.com/jakewharton/status/482563299511250944>