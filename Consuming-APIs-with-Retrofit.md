## Overview

[Retrofit](http://square.github.io/retrofit/) is a type-safe REST client for Android (or just Java) developed by Square. The library provides a powerful framework for authenticating and interacting with APIs and sending network requests with [OkHttp](http://square.github.io/okhttp/).  See [[this guide|Using-OkHttp]] to understand how OkHttp works.

This library makes downloading JSON or XML data from a web API fairly straightforward. Once the data is downloaded then it is parsed into a Plain Old Java Object (POJO) which must be defined for each "resource" in the response.

### Setup

Make sure to require Internet permissions in your `AndroidManifest.xml` file:

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
    <uses-permission android:name="android.permission.INTERNET" />
</manifest>
```

Add the following to your `app/build.gradle` file:

```gradle
dependencies {
    compile 'com.google.code.gson:gson:2.8.2'
    compile 'com.squareup.retrofit2:retrofit:2.3.0'
    compile 'com.squareup.retrofit2:converter-gson:2.3.0'  

}
```

**Note**: if you are upgrading from Retrofit 2 beta 1 or beta2, your package imports will need to be changed from `import retrofit.XXXX` to `import retrofit2.XXXX`.  You will also need to update your `OkHttp` imports from `import okhttp.XXXX` to `import okhttp3.XXXX` because of the new OkHttp3 dependency:

```bash
find . -name '*.java' -exec gsed -i 's/import retrofit\./import retrofit2./g' \{\} +
find . -name '*.java' -exec gsed -i 's/import com.squareup.okhttp/import okhttp3/g' \{\} +
```

If you intend to use [[RxJava]] with Retrofit 2, you will also need to include the RxJava adapter:

```gradle
dependencies {
  compile 'io.reactivex:rxjava:1.1.6'
  compile 'io.reactivex:rxandroid:1.2.1'
  compile 'com.squareup.retrofit2:adapter-rxjava:2.1.0'
}
```

In the past, Retrofit relied on the [Gson](https://github.com/google/gson) library to serialize and deserialize JSON data. Retrofit 2 now supports many different parsers for processing network response data, including [Moshi](https://github.com/square/moshi), a library build by Square for efficient JSON parsing.  However, there are a few [limitations](https://github.com/square/moshi#borrows-from-gson), so if you are not sure which one to choose, use the Gson converter for now.  

|Converter  | Library             
|-----------|------------------------------------------------
|Gson       | com.squareup.retrofit2:converter-gson:2.1.0          
|Jackson    | com.squareup.retrofit2:converter-jackson:2.1.0         
|Moshi      | com.squareup.retrofit2:converter-moshi:2.1.0           
|Protobuf   | com.squareup.retrofit2:converter-protobuf:2.1.0       
|Wire       | com.squareup.retrofit2:converter-wire:2.1.0           
|Simple XML | com.squareup.retrofit2:converter-simplexml:2.1.0      

### Create Java Classes for Resources

There are two approaches discussed in this guide.  The first way is the manual approach, which requires you to learn how to use the [Gson](https://github.com/google/gson) library.  The second approach is you can also auto-generate the Java classes you need by capturing the JSON output and using [jsonschema2pojo](http://www.jsonschema2pojo.org/).   We encourage you to follow the manual way to best understand how the auto-generated code approach works.

#### Manual approach - Creating Java classes by hand

See [[this guide|Leveraging-the-Gson-Library]]  about leveraging the Gson library for more information about how to create your own Java classes for use with Retrofit.   The guide shows how to use Gson to ingest data from the Rotten Tomatoes API, but it can be used in the same way for any RESTful web service.

#### Automated approach - Auto-generating the Java classes

Assuming you have the JSON response already, go to [jsonschema2pojo](http://www.jsonschema2pojo.org/).
Make sure to select `JSON` as the Source Type:

<img src="https://imgur.com/3DRH984.png"/>

Set the Annotation style to `Gson`.

<img src="https://i.imgur.com/RohCHRK.png" width="200"/>

Next, paste the JSON output into the textbox:

<img src="https://i.imgur.com/vn44UFk.png" width="600" alt="POJO Generator" />

Click the Preview button.  You should see the top section look sort of like the following:

<img src="https://i.imgur.com/Cxxmnu1.png" width="600"/>

Paste the generated class into your project under a `models` sub-package.  Rename the class name `Example` to reflect your model name.  For this example, we will call this file and class the `User` model.

*Note*: Android does not come normally with many of the `javax.annotation` library by default.  If you wish to keep the `@Generated` annotation, you will need to add this dependency.  See this [Stack Overflow discussion](http://stackoverflow.com/questions/9650808/android-javax-annotation-processing-package-missing) for more context.  Otherwise, you can delete that annotation and use the rest of the generated code.

```gradle
dependencies {
  provided 'org.glassfish:javax.annotation:10.0-b28'
}
```

### Creating the Retrofit instance

To send out network requests to an API, we need to use the [Retrofit builder] (http://square.github.io/retrofit/2.x/retrofit/retrofit2/Retrofit.Builder.html) class and specify the base URL for the service.

```java
// Trailing slash is needed
public static final String BASE_URL = "http://api.myservice.com/";
Retrofit retrofit = new Retrofit.Builder()
    .baseUrl(BASE_URL)
    .addConverterFactory(GsonConverterFactory.create())
    .build();
```

Note also that we need to specify a factory for deserializing the response using the Gson library.  The order in which the converters are added will be the sequence in which they are attempted to be processed as discussed in this [video talk](https://youtu.be/KIAoQbAu3eA?t=35m8s).  If we wish to pass in a custom Gson parser instance, it can be specified too:

```java
Gson gson = new GsonBuilder()
        .setDateFormat("yyyy-MM-dd'T'HH:mm:ssZ")
        .create();

Retrofit retrofit = new Retrofit.Builder()
    .baseUrl(BASE_URL)
    .addConverterFactory(GsonConverterFactory.create(gson))
    .build();
```

### Define the Endpoints

With Retrofit 2, endpoints are defined inside of an interface using special retrofit annotations to encode details about the parameters and request method.   In addition, the return value is always a parameterized `Call<T>` object such as `Call<User>`.   If you do not need any type-specific response, you can specify return value as simply `Call<ResponseBody>`.

For instance, the interface defines each endpoint in the following way:

```java
public interface MyApiEndpointInterface {
    // Request method and URL specified in the annotation
    // Callback for the parsed response is the last parameter

    @GET("users/{username}")
    Call<User> getUser(@Path("username") String username);

    @GET("group/{id}/users")
    Call<List<User>> groupList(@Path("id") int groupId, @Query("sort") String sort);

    @POST("users/new")
    Call<User> createUser(@Body User user);
}
```

Notice that each endpoint specifies an annotation of the HTTP method (GET, POST, etc.) and method that will be used to dispatch the network call.  Note that the parameters of this method can also have special annotations:

| Annotation| Description
------------|-------------------
| `@Path`   | variable substitution for the API endpoint (i.e. username will be swapped for `{username}` in the URL endpoint).
| `@Query`  | specifies the query key name with the value of the annotated parameter.
| `@Body`   | payload for the POST call (serialized from a Java object to a JSON string)
| `@Header` | specifies the header with the value of the annotated parameter


#### Changing the base URL

Normally, the base URL is defined when you instantiated an [Retrofit instance](#creating-the-retrofit-instance). Retrofit 2 allows you to override the base URL specified by changing it in the annotation (i.e. if you need to keep one specific endpoint using an older API endpoint)

```java
@POST("https://api.github.com/api/v3")
```

There are also others that allow you to modify the base URL using relative paths (and not the fully qualified URL) as discussed in this [blog article](http://inthecheesefactory.com/blog/retrofit-2.0/en).

#### Adding headers

Notice that there is a `@Headers` and `@Header` annotation.  The `Headers` can be used to provide pre-defined ones:

```java
@Headers({"Cache-Control: max-age=640000", "User-Agent: My-App-Name"})
@GET("/some/endpoint")
```

We can also add headers as a parameter to the endpoint:

```java
@Multipart
@POST("/some/endpoint")
Call<SomeResponse> someEndpoint(@Header("Cache-Control") int maxAge)
```

#### Form data

If we wish to submit form-encoded data, we can use the `FormUrlEncoded` annotation.  The `@Field` annotation will denote what payload will be submitted as form data.

```java
@FormUrlEncoded
@POST("/some/endpoint")
Observable<SomeResponse> someEndpoint(@Field("code") String code);
```

#### Multipart forms

If we need to upload images or files, we need to send by using Multipart forms.  We will to mark the endpoint with `@Multipart`, and label at least one parameter with @Part.

```java
@Multipart
@POST("some/endpoint")
Call<Response> uploadImage(@Part("description") String description, @Part("image") RequestBody image)
```

Assuming we have a reference to the file, we can create a RequestBody object:
```java
MediaType MEDIA_TYPE_PNG = MediaType.parse("image/png");
file = new File("/storage/emulated/0/Pictures/MyApp/test.png");
RequestBody requestBody = RequestBody.create(MEDIA_TYPE_PNG, file);

Call<Response> call = apiService.uploadImage("test", requestBody);
```

If you need to specify a unique filename for your multipart upload, there is currently an issue in Retrofit 2 tracked in this [ticket](https://github.com/square/retrofit/issues/1063).  Alternatively, you can create a multi-part RequestBody according to this [OkHttp recipe guide](https://github.com/square/okhttp/wiki/Recipes#posting-a-multipart-request) and pass it along as one of the `@Part` annotated parameters:

```java
RequestBody requestBody = new MultipartBody.Builder()
        .setType(MultipartBody.FORM)
        .addFormDataPart("title", "Square Logo")
        .addFormDataPart("image", "logo-square.png",
            RequestBody.create(MEDIA_TYPE_PNG, new File("website/static/logo-square.png")))
        .build();
```

#### Form URL encoding

If we wish to POST form-encoded name/value pairs, we can use the @FormUrlEncoded and @FieldMap annotations:

```java
@FormUrlEncoded
@POST("some/endpoint")
Call<SomeResponse> someEndpoint(@FieldMap Map<String, String> names);
```

#### POSTing JSON data

Retrofit 2 will use the converter library chosen to handle the deserialization of data from a Java object.  If you annotate the parameter with a `@Body` parameter, this work will be done automatically.  If you are using the Gson library for instance, any field belonging to the class will be serialized for you.  You can change this name using the `@SerializedName` decorator:

```java
public class User {

  @SerializedName("id")
  int mId;

  @SerializedName("name")
  String mName;

  public User(int id, String name ) {
    this.mId = id;
    this.mName = name;
  }
}
```

Our endpoint would look like the following:

```java
@POST("/users/new")
Call<User> createUser(@Body User user);
```

We could invoke this API call as follows:

```java
User user = new User(123, "John Doe");
Call<User> call = apiService.createuser(user);
call.enqueue(new Callback<User>() {
  @Override
  public void onResponse(Call<User> call, Response<User> response) {

  }

  @Override
  public void onFailure(Call<User> call, Throwable t) {

  }
```

The resulting network call would POST this data:

```javascript
{"name":"John Doe","id":123}
```

#### Upgrading from Retrofit 1

If you are trying to upgrade from Retrofit 1, you may remember that the last parameter had to be a `Callback` type if you wanted to define the API call to run asynchronously instead of synchronously:

```java
public interface MyApiEndpointInterface {
    // Request method and URL specified in the annotation
    // Callback for the parsed response is the last parameter

    @GET("users/{username}")
    void getUser(@Path("username") String username, Callback<User> cb);

    @GET("group/{id}/users")
    void groupList(@Path("id") int groupId, @Query("sort") String sort, Callback<List<User>> cb);

    @POST("users/new")
    void createUser(@Body User user, Callback<User> cb);
}
```

Retrofit 1 relied on this `Callback` type as a last parameter to determine whether the API call would be dispatched asynchronously instead of synchronously.   To avoid having two different calling patterns, this interface was consolidated in Retrofit 2.  You now simply define the return value with a parameterized `Call<T>`, as shown in the previous section.  

#### Accessing the API

We can bring this all together by constructing a service leveraging the `MyApiEndpointInterface` interface with the defined endpoints:

```java
MyApiEndpointInterface apiService =
    retrofit.create(MyApiEndpointInterface.class);
```

If we want to consume the API asynchronously, we call the service as follows:

```java
String username = "sarahjean";
Call<User> call = apiService.getUser(username);
call.enqueue(new Callback<User>() {
    @Override
    public void onResponse(Call<User> call, Response<User> response) {
        int statusCode = response.code();
        User user = response.body();  
    }

    @Override
    public void onFailure(Call<User> call, Throwable t) {
        // Log error here since request failed
    }
});
```

Shown above, Retrofit will download and parse the API data on a background thread, and then deliver the results back to the UI thread via the `onResponse` or `onFailure` method.

Note also that OkHttp, which dispatches the callback on the worker thread, callbacks in Retrofit are dispatched on the [main thread](http://stackoverflow.com/questions/21006807/does-retrofit-make-network-calls-on-main-thread/21010181#21010181).  Because UI updates can only be done on the main thread, the approach used by Retrofit can make it easier to make changes to your views.

If you are using Retrofit in a [[Background Service|Starting-Background-Services]] instead of an Activity or Fragment, you can run the network call synchronously within the same thread by using the `execute()` method.

```java
try {
  Response<User> response = call.execute();
} catch (IOException e ){
   // handle error
}
```

## RxJava

Retrofit 2 also supports [[RxJava]] extensions.  You will need to create an [[RxJava]] Adapter.
By default, all network calls are synchronous:

```java
RxJavaCallAdapterFactory rxAdapter = RxJava2CallAdapterFactory.create();
```

If you wish to default network calls to be asynchronous, you need to use `createWithScheduler()`.  

```java
RxJavaCallAdapterFactory rxAdapter = RxJava2CallAdapterFactory.createWithScheduler(Schedulers.io());
```

You can then instantiate the Retrofit instance:

```java
Retrofit retrofit = new Retrofit.Builder()
                               .baseUrl("https://api.github.com")
                               .addConverterFactory(GsonConverterFactory.create())
                               .addCallAdapterFactory(rxAdapter)
                               .build();
```

Instead of creating `Call` objects, we will use `Observable` types.   

```java
public interface MyApiEndpointInterface {
    // Request method and URL specified in the annotation
    // Callback for the parsed response is the last parameter

    @GET("/users/{username}")
    Observable<User> getUser(@Path("username") String username);

    @GET("/group/{id}/users")
    Observable<List<User>> groupList(@Path("id") int groupId, @Query("sort") String sort);

    @POST("/users/new")
    Observable<User> createUser(@Body User user);
}
```

Consistent with the RxJava framework, we need to create a subscriber to handle the response.  The methods `onCompleted()`, `onError()`, and `onNext()` need to be added.  Using the Android RxJava library, we can also designate that we will handle this event on the UI main thread.  **Note**: If you intend to override the default network call behavior, you can specify `subscribeOn()`.  Otherwise, it can omitted.
**Note**: As the RxJava rewrote their API, the term "Subscription" used here shall be replaced with "Disposable". As the word "Subscription" had conflict intentions.

```java
String username = "sarahjean";
Observable<User> call = apiService.getUser(username);
Disposable disposable = call
.subscribeOn(Schedulers.io()) // optional if you do not wish to override the default behavior
.observeOn(AndroidSchedulers.mainThread()).subscribeWith(new Disposable<User>() {
  @Override
  public void onCompleted() {

  }

  @Override
  public void onError(Throwable e) {
    // cast to retrofit.HttpException to get the response code
    if (e instanceof HttpException) {
       HttpException response = (HttpException)e;
       int code = response.code();
    }
  }

  @Override
  public void onNext(User user) {
  }
});
```

Note that if you are running any API calls in an activity or fragment, you will need to unsubscribe on the `onDestroy()` method.  The reasons are explained in this [Wiki article](https://github.com/ReactiveX/RxJava/wiki/The-RxJava-Android-Module#fragment-and-activity-life-cycle):

```java
// MyActivity
private Disposable disposable;

protected void onCreate(Bundle savedInstanceState) {
    this.disposable = observable.subscribe(this);
}

...

protected void onDestroy() {
    this.disposable.dispose();
    super.onDestroy();
}
```

Note also that subscribing an observer to the observable is what triggers the network request.  For more information about how to attach multiple observers before dispatching the network requests, see [this section](RxJava#hot-vs-cold-observables).

## Retrofit and Authentication

### Using Authentication Headers

Headers can be added to a request using an `Interceptor`. To send requests to an authenticated API, add headers to your requests using an interceptor as outlined below:

```java
// Define the interceptor, add authentication headers
Interceptor interceptor = new Interceptor() {
  @Override
  public okhttp3.Response intercept(Chain chain) throws IOException {
    Request newRequest = chain.request().newBuilder().addHeader("User-Agent", "Retrofit-Sample-App").build();
    return chain.proceed(newRequest);
  }
};

// Add the interceptor to OkHttpClient
OkHttpClient.Builder builder = new OkHttpClient.Builder();
builder.interceptors().add(interceptor);
OkHttpClient client = builder.build();

// Set the custom client when building adapter
Retrofit retrofit = new Retrofit.Builder()
  .baseUrl("https://api.github.com")
  .addConverterFactory(GsonConverterFactory.create())
  .client(client)
  .build();
```

Notice that in Retrofit 2 the interceptor has to be added to a custom `OkHttpClient`.  In Retrofit 1, it could be set directly by the builder class.

### Using OAuth

In order to authenticate with OAuth, we need to sign each network request sent out with a special header that embeds the access token for the user that is obtained during the OAuth process. The actual OAuth process needs to be completed with a third-party library such as [signpost](https://code.google.com/p/oauth-signpost/) and then the access token needs to be added to the header using a [request interceptor](https://github.com/square/retrofit/issues/316). Relevant links for Retrofit and authentication below:

 * [Parameterized Headers in Retrofit](http://stackoverflow.com/a/20493369/313399)
 * [SignPost Retrofit Extension](https://github.com/pakerfeldt/signpost-retrofit)
 * [Demo of Retrofit with Google Auth](https://github.com/bkiers/retrofit-oauth)

Resources for using signpost to authenticate with an OAuth API:

 * [Additional SignPost Code Samples](https://github.com/mttkay/signpost-examples)

Several other Android OAuth libraries can be explored instead of signpost:

 * [Android OAuth Client](https://github.com/wuman/android-oauth-client)
 * [OAuth for Android](https://github.com/novoda/oauth_for_android)


## Issues

There is a known issue currently with Retrofit 2 passing Lint tests tracked in this [ticket](https://github.com/square/okio/issues/58).  In particular, you may see `package reference in library; not included in Android: java.nio.file. Referenced from okio.Okio.` or `Invalid package reference in library; not included in Android: java.lang.invoke. Referenced from retrofit.Platform.Java8.`.  

To bypass this issue, create a `gradle/lint.xml` in your root project:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<lint>
    <issue id="InvalidPackage">
        <ignore regexp=".*okio.*" />
        <ignore regexp=".*retrofit.*" />
    </issue>
</lint>
```

Add this line to your `app/build.gradle` file:

```gradle
lintOptions {
    lintConfig rootProject.file('gradle/lint.xml')
}
```

The lint errors should be suppressed and not trigger any additional errors for now.

## Troubleshooting

Retrofit and OkHttp can be hard to troubleshoot when trying to step through the various layers of abstraction in the libraries.  You can add the `HttpLogInterceptor` that can be added when using the `OkHttp3` library, which will print HTTP requests/responses through LogCat.  You can also leverage Facebook's [Stetho](http://facebook.github.io/stetho/) project to use Chrome to inspect all network traffic. 

### HttpLogInterceptor

To use `HttpLogInterceptor`, add this dependency to your Gradle configuration:

```gradle
compile 'com.squareup.okhttp3:logging-interceptor:3.3.0'
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

You then need to pass this custom `OkHttpClient` into the Retrofit builder:

```java
Retrofit retrofit = new Retrofit.Builder()
                 .client(okHttpClient)
```

### Stetho

Facebook's [Stetho](http://facebook.github.io/stetho/) project enables you to use Chrome debugging tools to troubleshoot network traffic, database files, and view layouts. See this [[guide|Debugging-with-Stetho]] for more details.

## References

 * <http://engineering.meetme.com/2014/03/best-practices-for-consuming-apis-on-android/>
 * <https://github.com/MeetMe/TwitchTvClient>
 * <http://themakeinfo.com/2015/04/android-retrofit-images-tutorial/>
 * <https://github.com/square/retrofit/issues/297>
 * <https://speakerdeck.com/jakewharton/simple-http-with-retrofit-2-droidcon-nyc-2015>
 * <http://inthecheesefactory.com/blog/retrofit-2.0/en>
 * <https://futurestud.io/blog/retrofit-add-custom-request-header/>
