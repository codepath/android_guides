## Overview

[Retrofit](http://square.github.io/retrofit/) is a type-safe REST client for Android built by Square. The library provides a powerful framework for authenticating and interacting with APIs and sending network requests with [OkHttp](http://square.github.io/okhttp/).

This library makes downloading JSON or XML data from a web API fairly straightforward. Once the data is downloaded then it is parsed into a Plain Old Java Object (POJO) which must be defined for each "resource" in the response. 

**Attribution:** This guide has been adapted from [this external guide](http://engineering.meetme.com/2014/03/best-practices-for-consuming-apis-on-android/) authored by [MeetMe](http://www.meetme.com/). 

### Setup

First, we need to add the libraries to our project. Without Gradle, this means:

1. Download the jar files for [Retrofit](https://github.com/square/retrofit), [OkHttp](https://github.com/square/okhttp), and [GSON](https://code.google.com/p/google-gson/).
2. Drag each of the jar files into your libs folder in your project

With Gradle, add the following to your `app/build.gradle` file:

```gradle
dependencies {
  compile 'com.squareup.okhttp:okhttp:2.0.0'
  compile 'com.squareup.retrofit:retrofit:1.7.0'
  compile 'com.google.code.gson:gson:2.3'
}
```

### Create Java Objects for Resources

Capture the actual JSON output for an API endpoint in order to generate the POJO (java objects) that will be used to deserialize the response. Once you have the sample JSON output, you can run the response through [jsonschema2pojo](http://www.jsonschema2pojo.org/) which will auto-generate the necessary objects within the response:

<img src="http://i.imgur.com/DG4TAB2.png" width="400" alt="POJO Generator" />

Paste the generated classes into your project under a `models` sub-package.

### Creating the RestAdapter

To send out network requests to an API, we need to construct a [RestAdapter](http://square.github.io/retrofit/javadoc/retrofit/RestAdapter.html) by specifying the base url for the service:

```java
public static final String BASE_URL = "http://api.myservice.com";
RestAdapter restAdapter = new RestAdapter.Builder()
    .setEndpoint(BASE_URL)
    .build();
```

### Define the Endpoints

With Retrofit, endpoints are defined inside of an interface using special retrofit annotations to encode details about the parameters and request method. The interface defines each endpoint in the following way:

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
