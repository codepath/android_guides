## Overview

Google's [Gson](https://github.com/google/gson) library provides a powerful framework for converting between JSON strings and Java objects.  This library helps to avoid needing to write boilerplate code to parse JSON responses yourself.   It can be used with any networking library, including the [Android Async HTTP Client](https://github.com/loopj/android-async-http) and [OkHttp](http://square.github.io/okhttp/).

### Setup

Add the following line to your Gradle configuration:

```gradle
dependencies {
  compile 'com.google.code.gson:gson:2.8.0'
}
```
[![Javadoc](https://javadoc-emblem.rhcloud.com/doc/com.google.code.gson/gson/badge.svg)](http://www.javadoc.io/doc/com.google.code.gson/gson)

### Auto-generating Model Classes

You can [auto-generate much of the model code](http://www.jsonschema2pojo.org/) as described in [[this guide|https://github.com/codepath/android_guides/wiki/Consuming-APIs-with-Retrofit#automated-approach---auto-generating-the-java-classes]].   The generator will generated a large number of files that may be a bit overwhelming but the key is to parse the JSON to elements that you need.  You can also see how a class is created for each nested layer in order to understand the manual process.

### Generating Models Manually

First, we need to know what type of JSON response we will be receiving.    The following example uses the [Rotten Tomatoes API](http://developer.rottentomatoes.com/docs) as an example and show how to create Java objects that will be able to parse the latest [box office movies](http://developer.rottentomatoes.com/docs/read/json/v10/Box_Office_Movies).  Based on the JSON response returned for this API call, let's first define how a basic movie representation should look like:

```java

class Movie {
    String id;
    String title;
    int year;
    Production production;

    public String getId() {
        return id;
    }

    public String getTitle() {
        return title;
    }

    public int getYear() {
        return year;
    }

    public Production getProduction() {
        return production;
    }
};
```

```java
class Production {
    String director;
    String screenplay;

    public String getDirector() {
        return director;
    }

    public String getScreenplay() {
        return screenplay;
    }
};
```


By default, the Gson library will map the fields defined in the class to the JSON keys defined in the response.  For instance, the fields `id`, `title`, and `year` will be mapped automatically.   We do not need any special annotations unless the field names and JSON keys are different.

In this specific case, the Movie class will correspond to each individual movie element and the Production class corresponds to the nested JSON object under movie's production element: 


```javascript
movies: [
  {
    id: "771305050",
    title: "Straight Outta Compton",
    production: {
      director: "F. Gary Gray"
      screenplay: "Jonathan Herman"
    },
    year: 2015,
  },
  { 
    id: "771357161",
    title: "Mission: Impossible Rogue Nation",
    production: {
      director: "Christopher McQuarrie",
      screenplay: "Christopher McQuarrie"
    },
    year: 2015
  }
]
```

### Initializing collections

Because the API returns a list of movies and not just an individual one, we also need to create a class that will map the `movies` key to a list of Movie objects.

```java
public class BoxOfficeMovieResponse {

    List<Movie> movies;

    // public constructor is necessary for collections
    public BoxOfficeMovieResponse() {
        movies = new ArrayList<Movie>();
    }
```

Note below that a public constructor may be needed to initialize the list.  To avoid null pointer exceptions that may result from trying to get back the movie lists, it is highly recommended to initialize these objects in the empty constructor.

### Parsing the response

Assuming we have the JSON response in string form, we simply need to use the Gson library and using the  `fromJson` method.  The first parameter for this method is the JSON response in text form, and the second parameter is the class that should be mapped.  We can create a static method that returns back a `BoxOfficeMovieResponse` class as shown below:

```java
public class BoxOfficeMovieResponse {
.
.
.
    public static BoxOfficeMovieResponse parseJSON(String response) {
        Gson gson = new GsonBuilder().create();
        BoxOfficeMovieResponse boxOfficeMovieResponse = gson.fromJson(response, BoxOfficeMovieResponse.class);
        return boxOfficeMovieResponse;
    }
}
```

### Custom options

The [Gson Builder](https://github.com/google/gson/blob/master/gson/src/main/java/com/google/gson/GsonBuilder.java) class enables a variety of different options that help provide more flexibility for the JSON parsing.  Before we instantiate a Gson parser, it's important to know what options are available using the Builder class.  

```java
GsonBuilder gsonBuilder = new GsonBuilder();
// register type adapters here, specify field naming policy, etc.
Gson Gson = gsonBuilder.create();
```

#### Matching variable names to JSON keys

For instance, if our property name matches that of the JSON key, then we do not need to annotate the attributes.  However, if we have a different name we wish to
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

In the event that Date fields need to mapped depending on the format, you are likely to need to need to use the custom deserializer approach described in the next section.

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

#### Mapping multiple Date formats

If an API uses different Date formats, then a custom type adapter can be used to make parsing more robust.  To use this approach, a default date policy should not be set by calling the method `setDateFormat()` as described in the [earlier section](#mapping-java-date-objects).  In addition, a custom deserializer should be registered and created:

```java
GsonBuilder gsonBuilder = new GsonBuilder();
gsonBuilder.registerTypeAdapter(Date.class, new DateDeserializer());
Gson Gson = gsonBuilder.create();
```

Leveraging the `DateUtils` class in the Apache Commons Language library, we can try multiple formats when attempting to parse a value to a Date field:

Add this line to your Gradle file:

```gradle
    compile 'org.apache.commons:commons-lang3:3.3.2'
```

We can then use the [parseDate()](https://commons.apache.org/proper/commons-lang/javadocs/api-3.1/org/apache/commons/lang3/time/DateUtils.html#parseDate) method to try multiple date formats:

```java
final class DateAdapter implements JsonDeserializer<Date> {

    DateAdapter() {
    }

    public static Date convertISO8601ToDate(String date) throws ParseException {
        return DateUtils.parseDate(date, Locale.getDefault(),
                DateFormatUtils.ISO_DATETIME_FORMAT.getPattern(),
                DateFormatUtils.ISO_DATETIME_TIME_ZONE_FORMAT.getPattern());
    }

    private Date deserializeToDate(JsonElement json) {
        try {
            return convertISO8601ToDate(json.getAsString());
        } catch (ParseException e) {
            throw new JsonSyntaxException(json.getAsString(), e);
        }
    }
}
```

#### Decoding collections of items

Sometimes our JSON response will be a list of items.  We may also have declared a Java object and want to map the JSON response to a collection of these objects.  

Since the Gson library needs to know what type should be used for decoding a string, we want to pass a type that defines a list of these objects.  Because Java normally doesn't retain generic types because of [type erasure](https://docs.oracle.com/javase/tutorial/java/generics/erasure.html), we need to implement a workaround.   This workaround involves  subclassing `TypeToken` with a parameterized type and creating an anonymous class:

```java
Type collectionType = new TypeToken<List<Multimedia>>(){}.getType();
GsonBuilder gsonBuilder = new GsonBuilder();
gsonBuilder.registerTypeAdapter(collectionType, new MultimediaDeserializer());
Gson gson = gsonBuilder.create();
```

If are deserializing the string directly, we can use this custom type using the `fromJson()` method:

```java
Type collectionType = new TypeToken<List<ImageResult>>(){}.getType();
Gson gson = gsonBuilder.create();
List<ImageResult> imageResults = gson.fromJson(jsonObject, collectionType);
```

This approach essentially creates a custom type for a list of objects for the Gson library.  See this Stack Overflow [discussion](http://stackoverflow.com/questions/15479724/why-the-typetoken-construction-in-gson-is-so-weird) for more details.

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
             public void onSuccess(int statusCode, Header[] headers, String response) {
                 Gson gson = new GsonBuilder().create();
                 Movie movie = gson.fromJson(response, Movie.class);
             }
         });
}
```

Be sure to use the `TextHttpResponseHandler` rather than `JsonHttpResponseHandler`. Then you can access the response model to access the parsed data model. 

#### Retrofit 2

Retrofit uses [OkHttp](http://square.github.io/okhttp/) for the underlying networking library and can use the Gson library to decode API-based responses.  To use it, we first need to define an interface file called `RottenTomatoesService.java`.  To ensure that the API call will be made asynchronously, we also define a callback interface.  Note that if this API call required other parameters, we should always make sure that the `Callback` declaration is last.
    
```java
 public interface RottenTomatoesService {
        @GET("/lists/movies/box_office.json")
        public Call<BoxOfficeMovieResponse> listMovies();
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

Next, we need to create a `Retrofit` instance with a Gson converter and make sure to associate the adapter to this `RequestInterceptor`:

```java
// Add the interceptor to OkHttpClient 
OkHttpClient client = new OkHttpClient();
client.interceptors().add(requestInterceptor);

Retrofit retrofit = new Retrofit.Builder()
               .client(client)
               .baseUrl("http://api.rottentomatoes.com/api/public/v1.0")
               .addConverterFactory(GsonConverterFactory.create())
               .build();
```

We then simply need to create a service class that will enable us to make API calls:

```java
RottenTomatoesService service = retrofit.create(RottenTomatoesService.class);

Call<BoxOfficeMovieResponse> call = service.listMovies();
call.enqueue(new Callback<BoxOfficeMovieResponse>() {
            @Override
            public void onResponse(Call<BoxOfficeMovieresponse> call, Response response) {
                // handle response here
                BoxOfficeMovieResponse boxOfficeMovieResponse = response.body();
            }

            @Override
            public void onFailure(Throwable t) {

            }
        });
```

### References

* <https://github.com/google/gson/blob/master/UserGuide.md/>
* <http://www.javacreed.com/gson-deserialiser-example//>