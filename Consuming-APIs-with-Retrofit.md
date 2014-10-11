## Overview

[Retrofit](http://square.github.io/retrofit/) is a type-safe REST client for Android built by Square. The library provides a powerful framework for authenticating and interacting with APIs and sending network requests with [OkHttp](http://square.github.io/okhttp/).

(**Needs Attention**)

Read [this external guide](http://engineering.meetme.com/2014/03/best-practices-for-consuming-apis-on-android/) for an overview of proper usage. 

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