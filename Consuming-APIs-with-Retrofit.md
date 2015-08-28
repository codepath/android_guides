## Overview

[Retrofit](http://square.github.io/retrofit/) is a type-safe REST client for Android built by Square. The library provides a powerful framework for authenticating and interacting with APIs and sending network requests with [OkHttp](http://square.github.io/okhttp/).

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
  compile 'com.squareup.okhttp:okhttp:2.4.0'
  compile 'com.squareup.retrofit:retrofit:1.9.0'
  compile 'com.google.code.gson:gson:2.3'
}
```

### Create Java Objects for Resources

You can also create them manually, but you will need to create your own Java classes and learn how to use the [Gson](https://github.com/google/gson) library that is used by default with Retrofit.  You can also auto-generate the Java objects you need by capturing the JSON output and using [jsonschema2pojo](http://www.jsonschema2pojo.org/).   We encourage you to follow the first way to best understand how the auto-generated code approach works.

#### Creating Java objects manually

Retrofit by default relies on the Gson library to parse JSON.  See [[this guide|Leveraging-the-Gson-Library]] for more information about how to create your own Java objects for use with Retrofit.

#### Auto-generating the Java objects

Make sure to select `JSON` as the Source Type:

<img src="http://imgur.com/3DRH984.png"/>

If you happen to be creating Java models using the previous step, you therefore won't need to do anything else and select `None` as the Annotation type.

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
```

Note: if you are trying to use Java models that you may have already created, see [this section](Consuming-APIs-with-Retrofit#migrating-previously-defined-java-objects).

### Creating the RestAdapter

To send out network requests to an API, we need to construct a [RestAdapter](http://square.github.io/retrofit/javadoc/retrofit/RestAdapter.html) by specifying the base url for the service:

```java
public static final String BASE_URL = "http://api.myservice.com";
RestAdapter restAdapter = new RestAdapter.Builder()
    .setEndpoint(BASE_URL)
    .build();
```

### Define the Endpoints

With Retrofit, endpoints are defined inside of an interface using special retrofit annotations to encode details about the parameters and request method. 

The interface defines each endpoint in the following way:

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

Notice that each endpoint specifies an annotation of the HTTP method (GET, POST, etc.) and method that will be used to dispatch the network call.  Note that the parameters of this method can contain either `@Path` or `@Query` annotations.  `@Path` is essentially used to do variable substitution for the API endpoint (i.e. username will be swapped for `{username}` in the URL endpoint).  `@Query` specifies the query key name with the value corresponding to the value of that annotated parameter.

Note also that the last parameter is a `Callback` type.  When this type is specified, Retrofit will dispatch the task asynchronously.  Unlike OkHttp, which dispatches the callback on the worker thread, callbacks in Retrofit are dispatched on the [main thread](http://stackoverflow.com/questions/21006807/does-retrofit-make-network-calls-on-main-thread/21010181#21010181).  Because UI updates can only be done on the main thread, the approach used by Retrofit can make it easier to make changes to your views.

Check out the [retrofit docs](http://square.github.io/retrofit/) for additional features available while defining the endpoints.

### Accessing the API

We can bring this all together by constructing a service through the `RestAdapter` leveraging the `MyApiEndpointInterface` interface with the defined endpoints:

```java
MyApiEndpointInterface apiService =
    restAdapter.create(MyApiEndpointInterface.class);
```

We can now consume our API using the service created above:

```java
String username = "sarahjean";
apiService.getUser(username, new Callback<User>() {
    @Override
    public void success(User user, Response response) {
        // Access user here after response is parsed
    }

    @Override
    public void failure(RetrofitError retrofitError) {
        // Log error here since request failed
    }
});
```

Shown above, Retrofit will download and parse the API data on a background thread, and then deliver the results back to the UI thread via the success or failure method.

Be sure to check out a [sample TwitchTvClient](https://github.com/MeetMe/TwitchTvClient) for a working sample and the [retrofit docs](http://square.github.io/retrofit/) for additional details about how to use the library.


## Retrofit and Authentication

### Using Authentication Headers

Headers can be added to a request using a `RequestInterceptor`. To send requests to an authenticated API, add headers to your requests using an interceptor as outlined below:

```java
// Define the interceptor, add authentication headers
RequestInterceptor requestInterceptor = new RequestInterceptor() {
  @Override
  public void intercept(RequestFacade request) {
    request.addHeader("User-Agent", "Retrofit-Sample-App");
  }
};

// Add interceptor when building adapter
RestAdapter restAdapter = new RestAdapter.Builder()
  .setEndpoint("https://api.github.com")
  .setRequestInterceptor(requestInterceptor)
  .build();
```

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

**Attribution:** This guide has been adapted from [this external guide](http://engineering.meetme.com/2014/03/best-practices-for-consuming-apis-on-android/) authored by [MeetMe](http://www.meetme.com/).