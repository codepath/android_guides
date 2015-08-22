## Overview

Google's [GSON](https://github.com/google/gson) library provides a powerful framework for converting between JSON strings and Java objects.
You can take advantage of it to avoid needing to write repetitive code to parse JSON responses yourself.  In many ways, the approach models the way
database ORMs work in that the library helps map the JSON representation to their respective models. 

### Setup

Add the following line to your Gradle configuration:

```gradle
dependencies {
  compile 'com.google.code.gson:gson:2.3'
}
```

### Usage

First, we need to know what type of JSON response we will be receiving.    Let's use the Rotten Tomatoes API as an example and show how to create structures
to parse the latest [box office movies](http://developer.rottentomatoes.com/docs/read/json/v10/Box_Office_Movies).  Let's first define how a basic movie
representation should look like:

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

Because the API returns a list of movies and not just an individual one, we also need to create a class that will handle the response.
Note below that a public constructor is needed to initialize the list.  Without this constructor, the GSON library will be unable to parse the
response correctly for collection types.

```java
public class Response {

    List<Movie> movies;

    // public constructor is necessary for collections
    public Movies() {
        movies = new ArrayList<Movie>();
    }
```

### Parsing the response

Assuming we have the JSON response in string form, we simply need to implement a static method to parse this information.
We simply just need to create a Gson parser instance that will use a method called `fromJson`.  The first parameter for
this method is the JSON response in text form, and the second parameter is the class that should be mapped.

```java
public class Response {
.
.
.
    public static Response fromJson(String response) {
        Gson gson = new GsonBuilder().create();
        Response movies = gson.fromJson(response, Response.class);
        return response;
    }
}
```

#### GSON Builder

The GSON Builder enables a variety of different options that help provide more flexibility for JSON parsing.  Before we instantiate a GSON parser,
it's important to know what options are available using the Builder class.  For instance, if our Java variable names do not match that of a corresponding JSON key, we
can annotate it to specify the mapping explicitly.  In addition, we can also specify how a default way in which fields from Java variables should be mapped to JSON keys,
as well as creating custom deserializers.

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

#### Mapping camel case names

There is also the option to specify how Java variable names should map to JSON keys by default.   For instance, if we wanted to declare a Java variable as 'mpaaRating' and
map it to the JSON 'mpaa_rating' key, we can take advantage of the GSON field naming policy:

```java
GsonBuilder gsonBuilder = new GsonBuilder();
gsonBuilder.setFieldNamingPolicy(FieldNamingPolicy.LOWER_CASE_WITH_UNDERSCORES);
Gson gson = gsonBuilder.create();
```

In this way, we do not need to specify a `@SerializedName` decorator:

```java
class Response {

    // @SerializedName("mpaa_rating") is redundant
    String mpaaRating;
}
```

#### Mapping Java types

By default, the GSON library is not aware of many Java types such as Timestamps.  If we wish to convert these types, we can also create a custom deserializer class that
will handle this work for us.

Here is an example of a Timestamp deserializer:

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

Then we simply need to register this new type adapter and this type adapter would convert the value automatically declared as a Timestamp
value.

```java
GsonBuilder gsonBuilder = new GsonBuilder();
gsonBuilder.registerTypeAdapter(Timestamp.class, new TimestampDeserializer());
Gson Gson = gsonBuilder.create();
```

#### Mapping Java Date objects

If we know what date format is used in the response by default, we also specify this date format:

```java
public String ISO_FORMAT = "yyyy-MM-dd'T'HH:mm:ssZ";

GsonBuilder gsonBuilder = new GsonBuilder();
gsonBuilder.setDateFormat(ISO_FORMAT);
Gson gson = gsonBuilder.create();
```

### Parsing the response

```java
public static Response fromJson(String jsonList) {
    Gson gson = new GsonBuilder().create();
    Response movies = gson.fromJson(jsonList, Response.class);
    return movies;
}
```

We can use any type of HTTP client library to parse this response, such as Android Async HTTP Client or Square's Retrofit library.

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
