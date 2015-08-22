## Overview

Google's [Gson](https://github.com/google/gson) library provides a powerful framework for converting between JSON strings and Java objects.  This library helps to avoid needing to write boilerplate code to parse JSON responses yourself.   It can be used with any networking library, including the [Android Async HTTP Client](https://github.com/loopj/android-async-http) and [Retrofit](http://square.github.io/retrofit/).

### Setup

Add the following line to your Gradle configuration:

```gradle
dependencies {
  compile 'com.google.code.gson:gson:2.3'
}
```

### Usage

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

By default, the Gson library will map the fields defined in the class to the JSON keys defined in the response.  In this specific case, the Movie class will correspond to each individual movie element:


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
public class Response {

    List<Movie> movies;

    // public constructor is necessary for collections
    public Movies() {
        movies = new ArrayList<Movie>();
    }
```

Note below that a public constructor is needed to initialize the list.  Without this constructor, the GSON library will be unable to parse the response correctly for collection types.

### Parsing the response

Assuming we have the JSON response in string form, we simply need to use the Gson library and using the  `fromJson` method.  The first parameter for this method is the JSON response in text form, and the second parameter is the class that should be mapped.  We can create a static method that returns back a Response class as shown below:

```java
public class Response {
.
.
.
    public static Response parseJSON(String response) {
        Gson gson = new GsonBuilder().create();
        Response movies = gson.fromJson(response, Response.class);
        return response;
    }
}
```

### Custom options

The [Gson Builder](https://google-gson.googlecode.com/svn/trunk/gson/docs/javadocs/com/google/gson/GsonBuilder.html) class enables a variety of different options that help provide more flexibility for the JSON parsing.  Before we instantiate a Gson parser, it's important to know what options are available using the Builder class.  For instance, if our Java variable names do not match that of a corresponding JSON key, we can annotate it to specify the mapping explicitly.  In addition, we can also specify how a default way in which fields from Java variables should be mapped to JSON keys, as well as creating custom deserializers.

```java
GsonBuilder gsonBuilder = new GsonBuilder();
// register type adapters here, specify field naming policy, etc.
Gson Gson = gsonBuilder.create();
```

##### Matching variable names to JSON keys

For instance, if our property name matches that of the JSON key, then we do not to annotate the attributes.  However, if we have a different name we wish to
use, we can simply annotate the declaration with `@SerializedName`:

```java
public class Response {

    @SerializedName("movies")
    List<Movie> movieList;
```

#### Mapping camel case field names

There is also the option to specify how Java field names should map to JSON keys by default.   For instance, the Rotten Tomatoes API response includes an `mpaa_rating` key in the JSON response.  If we followed Java conventions and named this variable as `mpaaRating`, then we could need to add a `SerializedName` decorator:

```java
class Response {

    @SerializedName("mpaa_rating") 
    String mpaaRating;
}
```

An alternate way, especially if we have many instances, is to set the field naming policy in the Gson library.  We can specify that all field names should be converted to lower cases and separated with underscores, which would caused camel case field names to be converted from `mpaaRating` to `mpaa_rating`:

```java
GsonBuilder gsonBuilder = new GsonBuilder();
gsonBuilder.setFieldNamingPolicy(FieldNamingPolicy.LOWER_CASE_WITH_UNDERSCORES);
Gson gson = gsonBuilder.create();
```

#### Custom Java types

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

### HTTP Client Libraries

We can use any type of HTTP client library with GSON, such as Android Async HTTP Client or Square's Retrofit library.

#### Android Async HTTP Client

```java
AsyncHttpClient client = new AsyncHttpClient();
 client.get(
         "http://api.rottentomatoes.com/api/public/v1.0/lists/movies/box_office.json?apikey=[APIKEY]j",
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