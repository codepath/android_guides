## Overview

Google's [Gson](https://github.com/google/gson) library provides a powerful framework for converting between JSON strings and Java objects.  This library helps to avoid needing to write boilerplate code to parse JSON responses yourself.   It can be used with any networking library, including the [Android Async HTTP Client](https://github.com/loopj/android-async-http) and [OkHttp](http://square.github.io/okhttp/).

### Setup

Add the following line to your Gradle configuration:

```gradle
dependencies {
  compile 'com.google.code.gson:gson:2.3'
}
```

### Usage

You can auto-generate many of your code as described in [[this guide|Consuming-APIs-with-Retrofit#auto-generating-the-java-objects]], but for now, we'll walk through how you can do so manually.
First, we need to know what type of JSON response we will be receiving.    Let's use the [Rotten Tomatoes API](http://developer.rottentomatoes.com/docs) as an example and show how to create Java objects that will be able to parse the latest [box office movies](http://developer.rottentomatoes.com/docs/read/json/v10/Box_Office_Movies).  Based on the JSON response returned for this API call, let's first define how a basic movie representation should look like:

```java

class Movie {
    String id;
    String title;
    int year;

    public String getId() {
        return id;
    }

    public String getTitle() {
        return title;
    }

    public int getYear() {
        return year;
    }
};
```

By default, the Gson library will map the fields defined in the class to the JSON keys defined in the response.  For instance, the fields `id`, `title`, and `year` will be mapped automatically.   We do not need any special annotations unless the field names and JSON keys are different.

In this specific case, the Movie class will correspond to each individual movie element:


```javascript
movies: [
  {
    id: "771305050",
    title: "Straight Outta Compton",
    year: 2015,
  },
  { 
    id: "771357161",
    title: "Mission: Impossible Rogue Nation",
    year: 2015
  }
]
```

Because the API returns a list of movies and not just an individual one, we also need to create a class that will map the `movies` key to a list of Movie objects.

```java
public class BoxOfficeMovieResponse {

    List<Movie> movies;

    // public constructor is necessary for collections
    public Movies() {
        movies = new ArrayList<Movie>();
    }
```

Note below that a public constructor is needed to initialize the list.  Without this constructor, the Gson library will be unable to parse the response correctly for collection types.

### Parsing the response

Assuming we have the JSON response in string form, we simply need to use the Gson library and using the  `fromJson` method.  The first parameter for this method is the JSON response in text form, and the second parameter is the class that should be mapped.  We can create a static method that returns back a `BoxOfficeMovieResponse` class as shown below:

```java
public class BoxOfficeMovieResponse {
.
.
.
    public static BoxOfficeMovieResponse parseJSON(String response) {
        Gson gson = new GsonBuilder().create();
        BoxOfficeMovieResponse boxOfficeMovieResponse = gson.fromJson(response, Response.class);
        return boxOfficeMovieResponse;
    }
}
```

### Custom options

The [Gson Builder](https://google-gson.googlecode.com/svn/trunk/gson/docs/javadocs/com/google/gson/GsonBuilder.html) class enables a variety of different options that help provide more flexibility for the JSON parsing.  Before we instantiate a Gson parser, it's important to know what options are available using the Builder class.  

```java
GsonBuilder gsonBuilder = new GsonBuilder();
// register type adapters here, specify field naming policy, etc.
Gson Gson = gsonBuilder.create();
```

#### Matching variable names to JSON keys

For instance, if our property name matches that of the JSON key, then we do not to annotate the attributes.  However, if we have a different name we wish to
use, we can simply annotate the declaration with `@SerializedName`:

```java
public class BoxOfficeMovieResponse {

    @SerializedName("movies")
    List<Movie> movieList;
```

#### Mapping Enums

Enums are not necessarily recommended by Google as described in this [section](https://developer.android.com/training/articles/memory.html#Overhead).  However, if you need to use them for decoding JSON responses, you can map them from string names.   Suppose we had an enum of different colors:

```java
    public ColorTypes colorType;

    public enum ColorTypes { RED, WHITE, BLUE };
```

We can annotate these attributes with `@SerializedName` too:

```java
    @SerializedName("color")
    public ColorTypes colorType;

    public enum ColorTypes {
        @SerializedName("red")
        RED,

        @SerializedName("white")
        WHITE,

        @SerializedName("blue")
        BLUE
    };
```

#### Mapping camel case field names

There is also the option to specify how Java field names should map to JSON keys by default.   For instance, the Rotten Tomatoes API response includes an `mpaa_rating` key in the JSON response.  If we followed Java conventions and named this variable as `mpaaRating`, then we would have to add a `SerializedName` decorator to map them correctly:

```java
public class BoxOfficeMovieResponse {

    @SerializedName("mpaa_rating") 
    String mpaaRating;
}
```

An alternate way, especially if we have many cases similar to this one, is to set the field naming policy in the Gson library.  We can specify that all field names should be converted to lower cases and separated with underscores, which would caused camel case field names to be converted from `mpaaRating` to `mpaa_rating`:

```java
GsonBuilder gsonBuilder = new GsonBuilder();
gsonBuilder.setFieldNamingPolicy(FieldNamingPolicy.LOWER_CASE_WITH_UNDERSCORES);
Gson gson = gsonBuilder.create();
```
 
#### Decoding collections of items

Sometimes our JSON response will be a list of items.  We may also have declared a Java object and want to map the JSON response to a collection of these objects.  

Since the Gson library needs to know what type should be used for decoding a string, we want to pass a type that defines a list of these objects.  Because Java normally doesn't retain generic types because of [type erasure](https://docs.oracle.com/javase/tutorial/java/generics/erasure.html), we need to implement a workaround.   This workaround involves  subclassing `TypeToken` with a parameterized type and creating an anonymous class:

```
Type collectionType = new TypeToken<List<ImageResult>>(){}.getType();
Gson gson = gsonBuilder.create();
List<ImageResult> imageResults = gson.fromJson(jsonObject, collectionType);
```

This approach essentially creates a custom type for a list of objects for the Gson library.  See this Stack Overflow [discussion](http://stackoverflow.com/questions/15479724/why-the-typetoken-construction-in-gson-is-so-weird) for more details.

#### Mapping Java Date objects

If we know what date format is used in the response by default, we also specify this date format.  The Rotten Tomatoes API for instance returns a release date for theaters (i.e. "2015-08-14").    If we wanted to map the data directly from a String to a Date object, we could specify the date format:

```java
public String DATE_FORMAT = "yyyy-MM-dd";

GsonBuilder gsonBuilder = new GsonBuilder();
gsonBuilder.setDateFormat(DATE_FORMAT);
Gson gson = gsonBuilder.create();
```

Similarly, the date format could also be used for parsing standard ISO format time directly into a Date object:
```java
public String ISO_FORMAT = "yyyy-MM-dd'T'HH:mm:ssZ";
GsonBuilder gsonBuilder = new GsonBuilder();
gsonBuilder.setDateFormat(ISO_FORMAT);
Gson gson = gsonBuilder.create();
```

To enable the library to build a map what keys in the JSON data should match your internal variables, you can add annotations to provide the actual corresponding JSON name.  An example of a previously defined Java model using the based on GitHub's [User](https://developer.github.com/v3/users/) model is shown below.  Note how the `@SerializedName` annotation is used.

```java
@SerializedName("login")
private String mLogin;

@SerializedName("id")
private Integer mId;

@SerializedName("avatarUrl")
private String mAvatarUrl;
```

#### Mapping custom Java types

By default, the Gson library is not aware of many Java types such as Timestamps.  If we wish to convert these types, we can also create a custom deserializer class that will handle this work for us.

Here is an example of a deserializer that will convert any JSON data that needs to be converted to a  Java field declared as a Timestamp:

```java
import com.google.gson.JsonDeserializationContext;
import com.google.gson.JsonDeserializer;
import com.google.gson.JsonElement;
import com.google.gson.JsonParseException;

import java.lang.reflect.Type;
import java.sql.Timestamp;

public class TimestampDeserializer implements JsonDeserializer<Timestamp> {
    public Timestamp deserialize(JsonElement json, Type typeOfT, JsonDeserializationContext context)
            throws JsonParseException {
        return new Timestamp(json.getAsJsonPrimitive().getAsLong());
    }
}
```

Then we simply need to register this new type adapter and enable the Gson library to map any JSON value that needs to be converted into a Java Timestamp.

```java
GsonBuilder gsonBuilder = new GsonBuilder();
gsonBuilder.registerTypeAdapter(Timestamp.class, new TimestampDeserializer());
Gson Gson = gsonBuilder.create();
```

### HTTP Client Libraries

We can use any type of HTTP client library with Gson, such as Android Async HTTP Client or Square's OkHttp library.

#### Android Async HTTP Client

```java
String apiKey = "YOUR-API-KEY-HERE";

AsyncHttpClient client = new AsyncHttpClient();
 client.get(
         "http://api.rottentomatoes.com/api/public/v1.0/lists/movies/box_office.json?apikey=" + apiKey,
         new TextHttpResponseHandler() {
             @Override
             public void onFailure(int i, org.apache.http.Header[] headers, String s,
                     Throwable throwable) {

             }

             @Override
             public void onSuccess(int i, org.apache.http.Header[] headers, String s) {
                 Response response = Response.fromJson(s);
             }
         });

    public static Response fromJson(String response) {
        Response movies = gson.fromJson(response, Response.class);
        return movies;
    }
}
```

#### Retrofit

Retrofit uses [OkHttp](http://square.github.io/okhttp/) for the underlying networking library and relies on the Gson library by default for decoding on API-based responses.  To use it, we first need to define an interface file called `RottenTomatoesService.java`.  To ensure that the API call will be made asynchronously, we also define a callback interface.  Note that if this API call required other parameters, we should always make sure that the `Callback` declaration is last.
    
```java
 public interface RottenTomatoesService {
        @GET("/lists/movies/box_office.json")
        public void listRepos(Callback<BoxOfficeMovieResponse> responseCallback);
    }
```

Then we can make sure to always inject the API key for each request by defining a `RequestInterceptor`.  class.  In this way, we can avoid needing to have it be defined for each API call.

```java
String apiKey = "YOUR-API-KEY-HERE";

RequestInterceptor requestInterceptor = new RequestInterceptor() {
   @Override
   public void intercept(RequestFacade request) {
       request.addQueryParam("apikey", apiKey);
   }
};
```

Next, we need to create a `RestAdapter` and making sure to associate the adapter to this `RequestInterceptor`:

```java 
RestAdapter restAdapter = new RestAdapter.Builder()
               .setRequestInterceptor(requestInterceptor)
               .setEndpoint("http://api.rottentomatoes.com/api/public/v1.0")
               .build();
```

We then simply need to create a service class that will enable us to make API calls:

```java
RottenTomatoesService service = restAdapter.create(RottenTomatoesService.class);

service.listRepos(new Callback<BoxOfficeMovies>() {
            @Override
            public void success(BoxOfficeMovies boxOfficeMovies, Response response) {
                // handle response here
            }

            @Override
            public void failure(RetrofitError error) {

            }
        });
```