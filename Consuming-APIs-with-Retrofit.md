## Overview

[Retrofit](http://square.github.io/retrofit/) is a type-safe REST client for Android built by Square. The library provides a powerful framework for authenticating and interacting with APIs and sending network requests with [OkHttp](http://square.github.io/okhttp/).  See [[this guide|Using-OkHttp]] to understand how OkHttp works.

This library makes downloading JSON or XML data from a web API fairly straightforward. Once the data is downloaded then it is parsed into a Plain Old Java Object (POJO) which must be defined for each "resource" in the response.

### Setup

Make sure to require Internet permissions in your `AndroidManifest.xml` file:

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    <uses-permission android:name="android.permission.INTERNET" />
</manifest>
```

Add the following to your `app/build.gradle` file:

```gradle
dependencies {
    compile 'com.google.code.gson:gson:2.4'
    compile 'com.squareup.retrofit:retrofit:2.0.0-beta1'
    compile 'com.squareup.retrofit:converter-gson:2.0.0-beta1'  
    compile 'com.squareup.okhttp:okhttp:2.4.0'
}
```

In the past, Retrofit relied on the [Gson](https://github.com/google/gson) library to parse JSON data. Retrofit 2 now supports many different parsers for processing network response data, including [Moshi](https://github.com/square/moshi), a library build by Square for efficient JSON parsing.  However, there are a few [limitations](https://github.com/square/moshi#borrows-from-gson), so if you are not sure which one to choose, use the Gson converter for now.  
 
|Converter  | Library             
|-----------|------------------------------------------------
|Gson       | com.squareup.retrofit:converter-gson:2.0.0-beta2           
|Jackson    | com.squareup.retrofit:converter-jackson:2.0.0-beta1          
|Moshi      | com.squareup.retrofit:converter-moshi:2.0.0-beta1            
|Protobuf   | com.squareup.retrofit:converter-protobuf:2.0.0-beta1         
|Wire       | com.squareup.retrofit:converter-wire:2.0.0-beta1             
|Simple XML | com.squareup.retrofit:converter-simplexml:2.0.0-beta1        

### Create Java Objects for Resources

There are two approaches discussed in this guide.  The first way is the manual approach, which requires you to learn how to use the [Gson](https://github.com/google/gson) library.  The second approach is you can also auto-generate the Java objects you need by capturing the JSON output and using [jsonschema2pojo](http://www.jsonschema2pojo.org/).   We encourage you to follow the manual way to best understand how the auto-generated code approach works.

#### Manual approach - Creating Java objects by hand

See [[this guide|Leveraging-the-Gson-Library]]  about leveraging the Gson library for more information about how to create your own Java objects for use with Retrofit.   The guide shows how to use Gson to ingest data from the Rotten Tomatoes API, but it can be used in the same way for any RESTful web service.

#### Automated approach - Auto-generating the Java objects

Assuming you have the JSON response already, go to [jsonschema2pojo](http://www.jsonschema2pojo.org/).
Make sure to select `JSON` as the Source Type:

<img src="http://imgur.com/3DRH984.png"/>

You can select the Annotation style to `Gson`, but it's not entirely necessary since the Java file created will work without it.  For now, select `None` as the Annotation type.

<img src="http://imgur.com/aTrRfpu.png"/>

Next, paste the JSON output into the textbox:

<img src="https://i.imgur.com/DG4TAB2.png" width="400" alt="POJO Generator" />

Click the Preview button.  You should see the top section look sort of like the following:

<img src="http://imgur.com/qoMywKe.png"/>

Paste the generated class into your project under a `models` sub-package.  Rename the class name `Example` to reflect your model name.  For this example, we will call this file and class the `User` model.

*Note*: Android does not come normally with many of the `javax.annotation` library by default.  If you wish to keep the `@Generated` annotation, you will need to add this dependency.  See this [Stack Overflow discussion](http://stackoverflow.com/questions/9650808/android-javax-annotation-processing-package-missing) for more context.  Otherwise, you can delete that annotation and use the rest of the generated code.

```gradle
dependencies {
  provided 'org.glassfish:javax.annotation:10.0-b28'
}
```

### Creating the Retrofit instance

To send out network requests to an API, we need to use the [Retrofit builder] (http://square.github.io/retrofit/javadoc/retrofit/Retrofit.Builder.html) class and specify the base URL for the service. 

```java
public static final String BASE_URL = "http://api.myservice.com";
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

With Retrofit 2, endpoints are defined inside of an interface using special retrofit annotations to encode details about the parameters and request method.   In addition, the return value is always a parameterized `Call<T>` object such as `Call<User>`.   If you do not need any type-specific response, you can specify return value as simply `Call<Response>`.

For instance, the interface defines each endpoint in the following way:

```java
public interface MyApiEndpointInterface {
    // Request method and URL specified in the annotation
    // Callback for the parsed response is the last parameter

    @GET("/users/{username}")
    Call<User> getUser(@Path("username") String username);

    @GET("/group/{id}/users")
    Call<List<User>> groupList(@Path("id") int groupId, @Query("sort") String sort);

    @POST("/users/new")
    Call<User> createUser(@Body User user);
}
```

Notice that each endpoint specifies an annotation of the HTTP method (GET, POST, etc.) and method that will be used to dispatch the network call.  Note that the parameters of this method can also have special annotations:

| Annotation| Description
------------|-------------------
| `@Path`   | variable substitution for the API endpoint (i.e. username will be swapped for `{username}` in the URL endpoint). 
| `@Query`  | specifies the query key name with the value corresponding to the value of that annotated parameter.
| `@Body`   | payload for the POST call (serialized from a Java object to a JSON string)


#### Changing the base URL

Normally, the base URL is defined when you instantiated an [Retrofit instance](#creating-the-retrofit-instance). Retrofit 2 allows you to override the base URL specified by changing it in the annotation (i.e. if you need to keep one specific endpoint using an older API endpoint)

```java
@POST("https://api.github.com/api/v3")
```

There are also others that allow you to modify the base URL using relative paths (and not the fully qualified URL) as discussed in this [blog article](http://inthecheesefactory.com/blog/retrofit-2.0/en).

#### Multipart forms
 
If we wish to POST multi-part form data, we can use the `@Multipart` and `@Part` annotations:

```java
@Multipart
@POST("/some/endpoint")
Call<SomeResponse> someEndpoint(@Part("name1") String name1, @Part("name2") String name2)
```

#### Form URL encoding

If we wish to POST form-encoded name/value pairs, we can use the @FormUrlEncoded and @FieldMap annotations:

```java
@FormUrlEncoded
@POST("/some/endpoint")
Call<SomeResponse> someEndpoint(@FieldMap Map<String, String> names);
```

#### Upgrading from Retrofit 1

If you are trying to upgrade from Retrofit 1, you may remember that the last parameter had to be a `Callback` type if you wanted to define the API call to run asynchronously instead of synchronously:

```java
public interface MyApiEndpointInterface {
    // Request method and URL specified in the annotation
    // Callback for the parsed response is the last parameter

    @GET("/users/{username}")
    void getUser(@Path("username") String username, Callback<User> cb);

    @GET("/group/{id}/users")
    void groupList(@Path("id") int groupId, @Query("sort") String sort, Callback<List<User>> cb);

    @POST("/users/new")
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
    public void onResponse(Response<User> response) {
        int statusCode = response.code();
        User user = response.body();  
    }

    @Override
    public void onFailure(Throwable t) {
        // Log error here since request failed
    }
});
```

Shown above, Retrofit will download and parse the API data on a background thread, and then deliver the results back to the UI thread via the `onResponse` or `onFailure` method.

Note also that OkHttp, which dispatches the callback on the worker thread, callbacks in Retrofit are dispatched on the [main thread](http://stackoverflow.com/questions/21006807/does-retrofit-make-network-calls-on-main-thread/21010181#21010181).  Because UI updates can only be done on the main thread, the approach used by Retrofit can make it easier to make changes to your views.

## Retrofit and Authentication

### Using Authentication Headers

Headers can be added to a request using an `Interceptor`. To send requests to an authenticated API, add headers to your requests using an interceptor as outlined below:

```java
// Define the interceptor, add authentication headers
Interceptor interceptor = new Interceptor() {
  @Override
  public Response intercept(Chain chain) throws IOException {
    Request newRequest = chain.request().newBuilder().addHeader("User-Agent", "Retrofit-Sample-App").build();
    return chain.proceed(newRequest);
  }
};

// Add the interceptor to OkHttpClient 
OkHttpClient client = new OkHttpClient();
client.interceptors().add(interceptor);

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

 * [Authenticating with Twitter using SignPost](http://blog.doityourselfandroid.com/2011/02/13/guide-to-integrating-twitter-android-application/)
 * [Guide for using SignPost for Authentication](http://nilvec.com/implementing-client-side-oauth-on-android.html)
 * [Additional SignPost Code Samples](https://github.com/mttkay/signpost-examples)

Several other Android OAuth libraries can be explored instead of signpost:

 * [Android OAuth Client](https://github.com/wuman/android-oauth-client)
 * [OAuth for Android](https://github.com/novoda/oauth_for_android)

## References

 * <http://engineering.meetme.com/2014/03/best-practices-for-consuming-apis-on-android/>
 * <https://github.com/MeetMe/TwitchTvClient>
 * <http://themakeinfo.com/2015/04/android-retrofit-images-tutorial/>
 * <https://github.com/square/retrofit/issues/297>
 * <https://speakerdeck.com/jakewharton/simple-http-with-retrofit-2-droidcon-nyc-2015>
 * <http://inthecheesefactory.com/blog/retrofit-2.0/en>