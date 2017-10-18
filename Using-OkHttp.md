## Overview

OkHttp is a third-party library developed by Square for sending and receive HTTP-based network requests.  It is built on top of the [Okio](https://corner.squareup.com/2014/04/okio.html) library, which tries to be more efficient about reading and writing data than
the standard Java I/O libraries by creating a [shared memory pool](https://www.youtube.com/watch?v=WvyScM_S88c&feature=youtu.be).  It is also the underlying library for [[Retrofit|Consuming-APIs-with-Retrofit]] library that provides type safety for consuming REST-based APIs.

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
compile 'com.squareup.okhttp3:okhttp:3.9.0'
```

**Note**: If you are upgrading from an older version of OkHttp, your imports will also need to be changed from `import com.squareup.okhttp.XXXX` to `import okhttp3.XXXX`.

**Note**: If you are intending to use Picasso with OkHttp3, you can either [[upgrade Picasso|Displaying-Images-with-the-Picasso-Library#setup-picasso]] to the latest snapshot version or use this custom downloader.  This change is necessary until the next release of Picasso as described [here](https://github.com/square/picasso/issues/1256).  

```gradle
dependencies {
  compile 'com.jakewharton.picasso:picasso2-okhttp3-downloader:1.0.2'
}
```

You will then wrap the `OkHttpClient` with this `OkHttp3Downloader`.  **Note**: as of `OkHttp3`, it is recommended you declare this object as a singleton because changes in OkHttp3 no longer require a global connection pool.  See [this changelog](see https://github.com/square/okhttp/blob/master/CHANGELOG.md#version-300-rc1) for more details.

```java
// Use OkHttpClient singleton
OkHttpClient client = new OkHttpClient();
Picasso picasso = new Picasso.Builder(context).downloader(new OkHttp3Downloader(client)).build();
```

## Sending and Receiving Network Requests

First, we must instantiate an OkHttpClient and create a `Request` object.  

```java
// should be a singleton
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
    public void onFailure(Call call, IOException e) {
        e.printStackTrace();
    }

    @Override
    public void onResponse(Call call, final Response response) throws IOException {
        if (!response.isSuccessful()) {
            throw new IOException("Unexpected code " + response);
        }
    }
}
```

#### Updating Views on UIThread

OkHttp normally creates a new worker thread to dispatch the network request and uses the same thread to handle the response.  It is built [primarily as a Java library](http://stackoverflow.com/questions/24246783/okhttp-response-callbacks-on-the-main-thread#comment45410547_24248963) so it does not handle the Android framework limitations that only permit views to be updated on the main UI thread.

For this reason, if you try to **access or update views from outside the main thread** in the `Callback`, you will probably receive an exception: `android.view.ViewRootImpl$CalledFromWrongThreadException: Only the original thread that created a view hierarchy can touch its views`. Read more about the [relationship between the main thread and views here](http://guides.codepath.com/android/Managing-Threads-and-Custom-Services#understanding-the-main-thread).

If you need to update any views from within a response, you will need to use `runOnUiThread()` or post the result back on the main thread:  

```java
client.newCall(request).enqueue(new Callback() {
    @Override
    public void onResponse(Call call, final Response response) throws IOException {
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

See [[this guide|Managing-Threads-and-Custom-Services#handler-and-loopers]] for more context. Alternatively, you could use an extension of `Callback` such as [MainThreadCallback](https://gist.github.com/petitviolet/b27b61972a6a18ca7b9e) which wraps up this sort of behavior neatly and places you within the main thread from the response callback by default. 

## Processing Network Responses

Assuming the request is not canceled and there are no connectivity issues, the `onResponse()` method will be fired. It passes a `Response` object that can be used to check the status code, the response body, and any headers that were returned. Calling `isSuccessful()` for instance if the code returned a status code of 2XX (i.e. 200, 201, etc.)

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
    public void onResponse(Call call, final Response response) throws IOException {  
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

Note that the `string()` method on the response body will load the entire data into memory.  To make more efficient use of memory, it is recommended that the response is processed as a stream by using `charStream()` instead.  This approach, however, requires using the [Gson](https://github.com/google/gson) library.  See [[this guide|Leveraging-the-Gson-Library]] for setup instructions.

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
    public void onResponse(Call call, final Response response) throws IOException {
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

### Caching Network Responses

We can setup network caching by passing in a cache when building the `OkHttpClient`:

```java
int cacheSize = 10 * 1024 * 1024; // 10 MiB
Cache cache = new Cache(new File(getApplication().getCacheDir(),"cacheFileName"), cacheSize);
OkHttpClient client = new OkHttpClient.Builder().cache(cache).build();
```

We can control whether to retrieve a cached response by setting the `cacheControl` property on the request.  For instance, if we wish to only retrieve the request if data is cached, we could construct the `Request` object as follows:

```java
Request request = new Request.Builder()
                .url("http://publicobject.com/helloworld.txt")
                .cacheControl(new CacheControl.Builder().onlyIfCached().build())
                .build();
```

We can also force a network response by using `noCache()` for the request:

```java
  .cacheControl(new CacheControl.Builder().noCache().build())        
```

We can also specify a maximum staleness age for the cached response:

```java
   .cacheControl(new CacheControl.Builder().maxStale(365, TimeUnit.DAYS).build())
```

To retrieve the cached response, we can simply call `cacheResponse()` on the Response object:

```java
Call call = client.newCall(request);
call.enqueue(new Callback() {
  @Override
  public void onFailure(Call call, IOException e) {

  }

  @Override
  public void onResponse(Call call, final Response response) throws IOException
  {
     final Response text = response.cacheResponse();
     // if no cached object, result will be null
     if (text != null) {
        Log.d("here", text.toString());
     }
  }
});
```

## Troubleshooting

OkHttp can be hard to troubleshoot when trying to step through the various layers of abstraction in the libraries.  You can add the `HttpLogInterceptor` that can be added when using the `OkHttp3` library, which will print HTTP requests/responses through LogCat.  You can also leverage Facebook's [Stetho](http://facebook.github.io/stetho/) project to use Chrome to inspect all network traffic.

### HttpLogInterceptor

To use `HttpLogInterceptor`, add this dependency to your Gradle configuration:

```gradle
compile 'com.squareup.okhttp3:logging-interceptor:3.6.0'
```

You will need to add a network interceptor for HttpLogInterceptor.  See [this doc](http://square.github.io/okhttp/3.x/logging-interceptor/) for the different options that can be used.

```java
OkHttpClient.Builder builder = new OkHttpClient.Builder();

HttpLoggingInterceptor httpLoggingInterceptor = new HttpLoggingInterceptor();

// Can be Level.BASIC, Level.HEADERS, or Level.BODY
// See http://square.github.io/okhttp/3.x/logging-interceptor/ to see the options.
httpLoggingInterceptor.setLevel(HttpLoggingInterceptor.Level.BODY);
builder.networkInterceptors().add(httpLoggingInterceptor);
builder.build();
```

### Stetho

Use Facebook's [Stetho](http://facebook.github.io/stetho/) plugin to monitor network calls with Chrome:

<img src="http://facebook.github.io/stetho/static/images/inspector-network.png"/>

Add this line to your Gradle configuration:

```gradle
dependencies { 
    compile 'com.facebook.stetho:stetho-okhttp3:1.3.0' 
} 
```

When instantiating `OkHttp`, make sure to pass in the `StethoInterceptor`.

```java
OkHttpClient client = new OkHttpClient.Builder()
    .addNetworkInterceptor(new StethoInterceptor())
    .build();
```

Finally, make sure to initialize Stetho in your Application:

```java
public class MyApplication extends Application {
  public void onCreate() {
    super.onCreate();
    Stetho.initializeWithDefaults(this);
  }
}
```

### Using with Websockets

OkHttp v3.5 now includes [support for bidirectional web sockets](https://medium.com/square-corner-blog/web-sockets-now-shipping-in-okhttp-3-5-463a9eec82d1).   The URL that should be used should be prefixed with `ws://` or `wss://` for the secure version.  Although the connection ports are the same as HTTP (port 80 and port 443), the server still needs to be configured to support WebSockets since they are a completely different protocol.

```java
// URL should be ws:// or wss:// (secure)
Request request = new Request.Builder().url(url).build();
Websocket webSocket = client.newWebSocket(request,  = new WebSocketListener() {

  @Override
  public void onOpen(WebSocket webSocket, Response response) {
     // connection succeeded
  }

  @Override
  public void onMessage(WebSocket webSocket, String text) {
     // text message received
  }

  @Override
  public void onMessage(WebSocket webSocket, ByteString bytes) {
     // binary message received
  }

  @Override
  public void onClosed(WebSocket webSocket, int code, String reason) {
     // no more messages and the connection should be released
  }

  @Override
  public void onFailure(WebSocket webSocket, Throwable t, Response response) {
     // unexpected error 
  });
```

To send a message, simply use the `send()` function:

```java
websocket.send("hello");
```

Note that OkHttp handles all the work on a separate thread, so you don't have to worry about making Websocket calls on the main thread.

If you need to close the connection properly, make sure to use a status code of 1000.  See [this link](https://developer.mozilla.org/en-US/docs/Web/API/CloseEvent) for the different status codes.

```java
webSocket.close(1000, "closing);
```

### Enabling TLS V1.2 on older devices

If you are seeing `SSL handshake terminated` errors and using Android 4.0 devices, you will need to enable TLS v1.2 explicitly.  Android has supported TLS 1.2 since API 16 (Android 4.1).  You also should ensure you are using the latest OpenSSL by using the `ProviderInstaller` as described [here](https://developer.android.com/training/articles/security-gms-provider.html).
 
This should be in the first line of your Application (or at least before OkHttp initializes):

```java
public class MyApplication extends Application {
    @Override
    public void onCreate() {
        super.onCreate();
        try {
          // Google Play will install latest OpenSSL 
          ProviderInstaller.installIfNeeded(getApplicationContext());
          SSLContext sslContext;
          sslContext = SSLContext.getInstance("TLSv1.2");
          sslContext.init(null, null, null);
          sslContext.createSSLEngine();
        } catch (GooglePlayServicesRepairableException | GooglePlayServicesNotAvailableException
            | NoSuchAlgorithmException | KeyManagementException e) {
            e.printStackTrace();
        }
    }
}
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
* [History of OkHttp podcast](https://simplecast.com/s/2e201f87)