This is a tutorial about building a basic app for displaying popular movies currently in theaters using the unauthenticated [RottenTomatoes](http://www.rottentomatoes.com/) API.

## App

Let's build an Android app step by step. The app will display a list of popular movies currently in theaters  which includes the poster, title, and cast of each film. 

<img src="http://i.imgur.com/zQPzAxD.png" alt="Screen Demo" width="350" />

The app will do the following:

1. Fetch the box office movies from the [Box Office Movie API](http://developer.rottentomatoes.com/docs/read/json/v10/Box_Office_Movies) in JSON format
2. Deserialize the JSON data for each of the movies into `BoxOfficeMovie` objects
3. Build an array of `BoxOfficeMovie` objects and create an `ArrayAdapter` for those movies
4. Define `getView` to define how to inflate a layout for each movie row and display each movie's data.
5. Attach the adapter for the movies to a ListView to display the data on screen

To do this, we can organize the different components into the following objects:

1. `RottenTomatoesClient` - Responsible for executing the API requests and retrieving the JSON
2. `BoxOfficeMovie` - Model object responsible for encapsulating the attributes for each individual movie
3. `BoxOfficeMoviesAdapter` - Responsible for mapping each `BoxOfficeMovie` to a particular view layout
4. `BoxOfficeActivity` - Responsible for fetching and deserializing the data and configuring the adapter

The full source code for this app can be [downloaded here](https://github.com/codepath/android-rottentomatoes-demo) for review as well.

## Tutorial

### Setup with the API

RottenTomatoes has an [extensive API](http://developer.rottentomatoes.com/docs/read/json/v10) which includes a [Movie Search](http://developer.rottentomatoes.com/docs/read/json/v10/Movies_Search), [Lists](http://developer.rottentomatoes.com/docs/read/json/v10/Lists_Directory), [Box Office Movies](http://developer.rottentomatoes.com/docs/read/json/v10/Box_Office_Movies) and much more. We can use one of these APIs to create a simple demonstration of networking with Android.

First, we need to ensure we have an API Key which is the mechanism used to authorize our API requests. We cannot make any API calls without first [registering for an account](http://developer.rottentomatoes.com/member/register) (or [signing in](https://secure.mashery.com/login/developer.rottentomatoes.com/)) and then setting up an API key.

**Hint:** If you don't have a key, feel free to use the key **`9htuhtcb4ymusd73d4z6jxcj`** as you follow the demo below.

### Explore the API Endpoints

In particular, this app will use the [Box Office Movies](http://developer.rottentomatoes.com/docs/read/json/v10/Box_Office_Movies) endpoint to retrieve a list of movies currently playing in theaters. The url to retrieve this data is `http://api.rottentomatoes.com/api/public/v1.0/lists/movies/box_office.json?apikey=<api-key-here>`. The [response](http://api.rottentomatoes.com/api/public/v1.0/lists/movies/box_office.json?apikey=9htuhtcb4ymusd73d4z6jxcj) is as follows:

```json
{
  "movies": [{
    "id": "770687943",
    "title": "Harry Potter and the Deathly Hallows - Part 2",
    "year": 2011,
    "mpaa_rating": "PG-13",
    "runtime": 130,
    "critics_consensus": "Thrilling, powerfully acted, and visually dazzling...",
    "release_dates": {"theater": "2011-07-15"},
    "ratings": {
      "critics_rating": "Certified Fresh",
      "critics_score": 97,
      "audience_rating": "Upright",
      "audience_score": 93
    },
    "synopsis": "Harry Potter and the Deathly Hallows, is the final adventure...",
    "posters": {
      "thumbnail": "http://content8.flixster.com/movie/11/15/86/11158674_mob.jpg",
      "profile": "http://content8.flixster.com/movie/11/15/86/11158674_pro.jpg",
      "detailed": "http://content8.flixster.com/movie/11/15/86/11158674_det.jpg",
      "original": "http://content8.flixster.com/movie/11/15/86/11158674_ori.jpg"
    },
    "abridged_cast": [
      {
        "name": "Daniel Radcliffe",
        "characters": ["Harry Potter"]
      },
      {
        "name": "Rupert Grint",
        "characters": [
          "Ron Weasley",
          "Ron Wesley"
        ]
      }
    ]
  }, 
  {
     "id": "770687943",
     ...
  }]
}
```

As you can see the API returns a **dictionary with a single "movies" key** which contains an **array of json dictionaries** which each represent the data for a particular film. This is the response we will parse and use to create our application.

### Install Library Dependencies

Before we continue, we need to setup the [android-async-http-client](http://loopj.com/android-async-http/) for sending asynchronous network requests. 

Let's also install a library for remote image loading called [Picasso](http://square.github.io/picasso/) so we can easily display movie posters. 

_Eclipse_ users:

Drop the library jars into the "libs" folder of our Android app before continuing

_Android Studio_ users:

You can also drop these jars into the "libs" folder too and add each file by right-clicking on the .jar file and choosing the Add as Library option, or you can simply edit the build.gradle file and add them as dependencies that can be downloaded via a remote repository:

```gradle
repositories {
    jcenter()
}
dependencies {
    // ...
    compile 'com.squareup.picasso:picasso:2.4.0'
    compile 'com.loopj.android:android-async-http:1.4.6'
}
```

Then you need to sync with gradle and make sure there are no errors.

### Download Assets

Make sure to download the two placeholder movie posters:

<a href="http://i.imgur.com/o8Y6d6u.png/small_movie_poster.png"><img 
src="http://i.imgur.com/o8Y6d6u.png/small_movie_poster.png" /></a>
<a href="http://i.imgur.com/Z2MYNbj.png/large_movie_poster.png"><img src="http://i.imgur.com/Z2MYNbj.png/large_movie_poster.png" /></a>

Copy and paste both [small_movie_poster.png](http://i.imgur.com/o8Y6d6u.png/small_movie_poster.png) and [large_movie_poster.png](http://i.imgur.com/Z2MYNbj.png/large_movie_poster.png) into `res/drawable` for use later.

### Setup Internet Permission

Add the proper internet permissions to the `AndroidManifest.xml` within the `manifest` node:

```xml
<uses-permission android:name="android.permission.INTERNET" /> 
```

Once you add this, network requests can be executed.  Make sure that the permissions are defined before the &lt;application&gt;...&lt;/application&gt; is declared.  Otherwise, you may notice that your network requests are quietly being ignored.

### Setup Basic Activity Layout

Generate a project with the name "RottenTomatoesDemo", a minimum API of 10 and a first activity called **BoxOfficeActivity**. In the `res/layout/activity_box_office.xml`, let's drop out a ListView with the id `lvMovies`:

```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".BoxOfficeActivity" >

    <ListView
        android:id="@+id/lvMovies"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_alignParentBottom="true"
        android:layout_alignParentLeft="true"
        android:layout_alignParentRight="true"
        android:layout_alignParentTop="true" >

    </ListView>
</RelativeLayout>
```

![Imgur](http://i.imgur.com/ze0XeWe.png)

### Construct API Client

Let's generate a Java class that will act as our RottenTomato API client for sending out the network requests to the specific endpoints. Create a Java class called `RottenTomatoesClient` with the following code:

```java
public class RottenTomatoesClient {
    private final String API_KEY = "...getkey...";
    private final String API_BASE_URL = "http://api.rottentomatoes.com/api/public/v1.0/";
    private AsyncHttpClient client;

    public RottenTomatoesClient() {
        this.client = new AsyncHttpClient();
    }
    
    private String getApiUrl(String relativeUrl) {
        return API_BASE_URL + relativeUrl;
    }
}
```

Next, let's define a method for accessing the Box Office Movies API within our client by executing an asynchronous request to the [appropriate endpoint](http://developer.rottentomatoes.com/docs/read/json/v10/Box_Office_Movies):

```java
public class RottenTomatoesClient {
    // ...

    // http://api.rottentomatoes.com/api/public/v1.0/lists/movies/box_office.json?apikey=<key>
    public void getBoxOfficeMovies(JsonHttpResponseHandler handler) {
        String url = getApiUrl("lists/movies/box_office.json");
        RequestParams params = new RequestParams("apikey", API_KEY);
        client.get(url, params, handler);
    }

    // ...
}
```

Looking at the JSON response format for the box office movies API, we can understand what information we will be provided:

```json
{
  "movies": [{
    "id": "770687943",
    "title": "Harry Potter and the Deathly Hallows - Part 2",
    "year": 2011,
    "mpaa_rating": "PG-13",
    "runtime": 130,
    "critics_consensus": "Thrilling, powerfully acted, and visually dazzling...",
    "release_dates": {"theater": "2011-07-15"},
    "ratings": {
      "critics_rating": "Certified Fresh",
      "critics_score": 97,
      "audience_rating": "Upright",
      "audience_score": 93
    },
    "synopsis": "Harry Potter and the Deathly Hallows, is the final adventure...",
    "posters": {
      "thumbnail": "http://content8.flixster.com/movie/11/15/86/11158674_mob.jpg",
      "profile": "http://content8.flixster.com/movie/11/15/86/11158674_pro.jpg",
      "detailed": "http://content8.flixster.com/movie/11/15/86/11158674_det.jpg",
      "original": "http://content8.flixster.com/movie/11/15/86/11158674_ori.jpg"
    },
    "abridged_cast": [
      {
        "name": "Daniel Radcliffe",
        "characters": ["Harry Potter"]
      },
      {
        "name": "Rupert Grint",
        "characters": [
          "Ron Weasley",
          "Ron Wesley"
        ]
      }
    ]
  }, 
  {
     "id": "770687943",
     ...
  }]
}
```

### Define the Data Model

Next, let's define a model that represents the relevant data for a single box office movie. This movie needs to contain basic info for each `movie` such as `title`, `year`, `synopsis`, `posters.thumbnail`, `ratings.critics_score` and the `abridged_cast`. Create a Java class called `BoxOfficeMovie` with the relevant attributes defined:

```java
public class BoxOfficeMovie {
	private String title;
	private int year;
	private String synopsis;
	private String posterUrl;
	private int criticsScore;
	private ArrayList<String> castList;

	public String getTitle() {
		return title;
	}

	public int getYear() {
		return year;
	}

	public String getSynopsis() {
		return synopsis;
	}

	public String getPosterUrl() {
		return posterUrl;
	}

	public int getCriticsScore() {
		return criticsScore;
	}

	public String getCastList() {
		return TextUtils.join(", ", castList);
	}
}
```

Next, let's add a factory method for deserializing the JSON response in order to construct an instance of the `BoxOfficeMovie` class:

```java
public class BoxOfficeMovie {
    // ...
    
    // Returns a BoxOfficeMovie given the expected JSON
    // BoxOfficeMovie.fromJson(movieJsonDictionary)
    // Stores the `title`, `year`, `synopsis`, `poster` and `criticsScore`
    public static BoxOfficeMovie fromJson(JSONObject jsonObject) {
        BoxOfficeMovie b = new BoxOfficeMovie();
        try {
            // Deserialize json into object fields
            b.title = jsonObject.getString("title");
            b.year = jsonObject.getInt("year");
            b.synopsis = jsonObject.getString("synopsis");
            b.posterUrl = jsonObject.getJSONObject("posters").getString("thumbnail");
            b.criticsScore = jsonObject.getJSONObject("ratings").getInt("critics_score");
            // Construct simple array of cast names
            b.castList = new ArrayList<String>();
            JSONArray abridgedCast = jsonObject.getJSONArray("abridged_cast");
            for (int i = 0; i < abridgedCast.length(); i++) {
                b.castList.add(abridgedCast.getJSONObject(i).getString("name"));
            }
        } catch (JSONException e) {
            e.printStackTrace();
            return null;
        }
        // Return new object
        return b;
    }
  
    // ...
}
```

Let's also add a convenience static method for parsing an array of JSON movie dictionaries into an `ArrayList<BoxOfficeMovie>` which we will use to manage the box office movie API endpoint response:

```java
public class BoxOfficeMovie {
    // ...
    
    // Decodes array of box office movie json results into movie model objects
    // BoxOfficeMovie.fromJson(jsonArrayOfMovies)
    public static ArrayList<BoxOfficeMovie> fromJson(JSONArray jsonArray) {
        ArrayList<BoxOfficeMovie> movies = new ArrayList<BoxOfficeMovie>(jsonArray.length());
        // Process each result in json array, decode and convert to movie
        // object
        for (int i = 0; i < jsonArray.length(); i++) {
            JSONObject moviesJson = null;
            try {
                moviesJson = jsonArray.getJSONObject(i);
            } catch (Exception e) {
                e.printStackTrace();
                continue;
            }

            BoxOfficeMovie movie = BoxOfficeMovie.fromJson(moviesJson);
            if (movie != null) {
                movies.add(movie);
            }
        }

        return movies;
    }
    
    // ...
}
```

### Construct an ArrayAdapter

Now, we have the API Client (`RottenTomatoesClient`) and the data Model object (`BoxOfficeModel`). Final step is to construct an adapter that allows us to translate a `BoxOfficeMovie` into a particular view. Let's generate a Java class called `BoxOfficeMoviesAdapter` which extends from `ArrayAdapter<BoxOfficeMovie>`:

```java
public class BoxOfficeMoviesAdapter extends ArrayAdapter<BoxOfficeMovie> {
	public BoxOfficeMoviesAdapter(Context context, ArrayList<BoxOfficeMovie> aMovies) {
		super(context, 0, aMovies);
	}
	
	@Override
	public View getView(int position, View convertView, ViewGroup parent) {
		// TODO: Complete the definition of the view for each movie
		return null;
	}
}
```

Now, we need to define a layout to use for visualizing a particular movie. Let's create a layout file called `item_box_office_movie.xml` with a `RelativeLayout` root layout for defining the movie row layout:

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="5dp" >

    <ImageView
        android:id="@+id/ivPosterImage"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_centerVertical="true"
        android:maxHeight="70dp"
        android:scaleType="fitXY"
        android:adjustViewBounds="true"
        android:src="@drawable/ic_launcher" />

    <RelativeLayout
        android:layout_width="match_parent"
        android:layout_toRightOf="@+id/ivPosterImage"
        android:layout_centerVertical="true"
        android:gravity="center_vertical"
        android:layout_marginLeft="10dp"
        android:layout_height="match_parent">

        <TextView
            android:id="@+id/tvTitle"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_alignParentTop="true"
            android:textStyle="bold"
            android:hint="The Dark Knight"
            android:textSize="12sp" />
        
        <TextView
            android:id="@+id/tvCriticsScore"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_alignLeft="@+id/tvTitle"
            android:layout_below="@+id/tvTitle"
            android:hint="91%"
            android:textSize="12sp" />

        <TextView
            android:id="@+id/tvCast"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_below="@+id/tvCriticsScore"
            android:maxLines="1"
            android:ellipsize="end"
            android:hint="Christian Bale, Joseph-Gordon Levitt, Heath Ledger, Maggie Gylenhall"
            android:textSize="12sp" />

    </RelativeLayout>

</RelativeLayout>
```

and now we can fill in the `getView` method to finish off the `BoxOfficeMoviesAdapter`:

```java
public class BoxOfficeMoviesAdapter extends ArrayAdapter<BoxOfficeMovie> {
    // ...
    
    // Translates a particular `BoxOfficeMovie` given a position 
    // into a relevant row within an AdapterView
    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
        // Get the data item for this position
        BoxOfficeMovie movie = getItem(position);
        // Check if an existing view is being reused, otherwise inflate the view
        if (convertView == null) {
        	LayoutInflater inflater = LayoutInflater.from(getContext());
        	convertView = inflater.inflate(R.layout.item_box_office_movie, parent, false);
        }
        // Lookup views within item layout
        TextView tvTitle = (TextView) convertView.findViewById(R.id.tvTitle);
        TextView tvCriticsScore = (TextView) convertView.findViewById(R.id.tvCriticsScore);
        TextView tvCast = (TextView) convertView.findViewById(R.id.tvCast);
        ImageView ivPosterImage = (ImageView) convertView.findViewById(R.id.ivPosterImage);
        // Populate the data into the template view using the data object
        tvTitle.setText(movie.getTitle());
        tvCriticsScore.setText("Score: " + movie.getCriticsScore() + "%");
        tvCast.setText(movie.getCastList());
        Picasso.with(getContext()).load(movie.getPosterUrl()).into(ivPosterImage);
        // Return the completed view to render on screen
        return convertView;
    }
    
    // ...
}
```

### Bringing It Together

Now that we have defined the adapter, let's plug all of these components together within the `BoxOfficeActivity` by using the client to send out an API call, deserialize the response into an array of `BoxOfficeMovie` object and then render those into a ListView using the `BoxOfficeMoviesAdapter`. Let's start by initializing the ArrayList, adapter, and the ListView: 

```java
public class BoxOfficeActivity extends Activity {
	private ListView lvMovies;
	private BoxOfficeMoviesAdapter adapterMovies;

	@Override
	protected void onCreate(Bundle savedInstanceState) {
	    super.onCreate(savedInstanceState);
	    setContentView(R.layout.activity_box_office);
	    lvMovies = (ListView) findViewById(R.id.lvMovies);
	    ArrayList<BoxOfficeMovie> aMovies = new ArrayList<BoxOfficeMovie>();
	    adapterMovies = new BoxOfficeMoviesAdapter(this, aMovies);
	    lvMovies.setAdapter(adapterMovies);
	}
	
}
```

Now let's make the call to the box office API from the activity using our client:

```java
public class BoxOfficeActivity extends Activity {
    RottenTomatoesClient client;

    protected void onCreate(Bundle savedInstanceState) {
       // ...
       // Fetch the data remotely
       fetchBoxOfficeMovies();
    }
    
    // Executes an API call to the box office endpoint, parses the results
    // Converts them into an array of movie objects and adds them to the adapter
    private void fetchBoxOfficeMovies() {
        client = new RottenTomatoesClient();
        client.getBoxOfficeMovies(new JsonHttpResponseHandler() {
            @Override
            public void onSuccess(int statusCode, Header[] headers, JSONObject responseBody) {
                JSONArray items = null;
                try {
                    // Get the movies json array
                    items = responseBody.getJSONArray("movies");
                    // Parse json array into array of model objects
                    ArrayList<BoxOfficeMovie> movies = BoxOfficeMovie.fromJson(items);
                    // Load model objects into the adapter
                    for (BoxOfficeMovie movie : movies) {
                       adapterMovies.add(movie); // add movie through the adapter
                    }
                    adapterMovies.notifyDataSetChanged();
                } catch (JSONException e) {
                    e.printStackTrace();
                }
            }
        });
    }
}
```

After running the app, we should see the populated box office movie data:

![Imgur](http://i.imgur.com/zQPzAxD.png)

The full source code for this app can be [downloaded here](https://github.com/codepath/android-rottentomatoes-demo) for review as well.

### Adding a Detail View

Suppose we wanted to add a detail view which would display more details about a movie when the movie was selected within the list. In order to achieve that we would need to:

1. Generate an activity for the details view
2. Define the detail Layout XML with views
3. Setup an item click listener for the list of movies
4. Fire an intent when a user selects a movie from the list
5. Pass the movie through the intent and populate the detail view

First, let's generate the activity that will display all the details by selecting `File => New => Other => Android Activity` and naming our activity "BoxOfficeDetailActivity". This activity should simply display additional information about the film for people to view, let's edit the layout `res/layout/activity_box_office_detail.xml` to include all the movie details and a larger poster:

```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="5dp"
    tools:context=".BoxOfficeDetailActivity" >

    <!-- @drawable/large_movie_poster sourced from 
         http://content8.flixster.com/movie/11/15/86/11158674_pro.jpg -->
    <ImageView
        android:id="@+id/ivPosterImage"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentLeft="true"
        android:layout_alignParentTop="true"
        android:maxHeight="160dp"
        android:scaleType="fitXY"
        android:adjustViewBounds="true"
        android:src="@drawable/ic_launcher" />


    <TextView
        android:id="@+id/tvTitle"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignTop="@+id/ivPosterImage"
        android:textStyle="bold"
        android:layout_marginLeft="8dp"
        android:layout_toRightOf="@+id/ivPosterImage"
        android:text="The Dark Knight"
        android:textSize="18sp" />

    <TextView
        android:id="@+id/tvCriticsScore"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignLeft="@+id/tvTitle"
        android:layout_below="@+id/tvTitle"
        android:layout_marginTop="5dp"
        android:text="Critics Score: 93%"
        android:textSize="14sp" />

    <TextView
        android:id="@+id/tvAudienceScore"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignLeft="@+id/tvCriticsScore"
        android:layout_below="@+id/tvCriticsScore"
        android:layout_marginTop="5dp"
        android:text="Audience Score: 93%"
        android:textSize="14sp" />

    <TextView
        android:id="@+id/tvCast"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignLeft="@+id/tvAudienceScore"
        android:layout_below="@+id/tvAudienceScore"
        android:layout_marginTop="5dp"
        android:text="Christian Bale, Joseph-Gordon Levitt"
        android:textSize="12sp" />

    <ScrollView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_below="@+id/ivPosterImage"
        android:layout_margin="10dp" >

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical" >

            <TextView
                android:id="@+id/tvCriticsConsensus"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="This is an excellent movie that has action and romance" />

            <TextView
                android:id="@+id/tvSynopsis"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_marginTop="10dp"
                android:text="This is a story about a protagonist defeating an antagonist" />
        </LinearLayout>
    </ScrollView>

</RelativeLayout>
```

Next, we want to be able to pass the movie object from the list to this detail view, so let's make our `BoxOfficeMovie` serializable:

```java
public class BoxOfficeMovie implements Serializable {
	private static final long serialVersionUID = -8959832007991513854L;
	// ...
}
```

We also want to be able to access a few key attributes of the API data for the detail page such as a larger picture and critics consensus:

```java
public class BoxOfficeMovie implements Serializable {
    // ...
    private String largePosterUrl;
    private String criticsConsensus;
    private int audienceScore;
	
    public static BoxOfficeMovie fromJson(JSONObject jsonObject) {
        // ...
        b.largePosterUrl = jsonObject.getJSONObject("posters").getString("detailed");
        b.criticsConsensus = jsonObject.getString("critics_consensus"); 
        b.audienceScore = jsonObject.getJSONObject("ratings").getInt("audience_score");
        // ...
    }
  
    // ...
    public String getLargePosterUrl() {
        return largePosterUrl;
    }

    public String getCriticsConsensus() {
        return criticsConsensus;
    }
	
    public int getAudienceScore() {
        return audienceScore;
    }
}
```


Let's now add an item click handler to our `BoxOfficeActivity` which will launch the detail view with an intent:

```java
public class BoxOfficeActivity extends Activity {
   // ...
   public static final String MOVIE_DETAIL_KEY = "movie";
   
   @Override
   protected void onCreate(Bundle savedInstanceState) { 
      // ...
      setupMovieSelectedListener();  
   }
   
   // ...
  
   public void setupMovieSelectedListener() {
       lvMovies.setOnItemClickListener(new OnItemClickListener() {
           @Override
           public void onItemClick(AdapterView<?> adapterView, View item, int position, long rowId) {
               // Launch the detail view passing movie as an extra
               Intent i = new Intent(BoxOfficeActivity.this, BoxOfficeDetailActivity.class);
               i.putExtra(MOVIE_DETAIL_KEY, adapterMovies.getItem(position));
               startActivity(i);
           }
        });
    } 
}
```

Now, let's populate the data from the movie into the details view with `BoxOfficeDetailActivity`:

```java
public class BoxOfficeDetailActivity extends Activity {
	private ImageView ivPosterImage;
	private TextView tvTitle;
	private TextView tvSynopsis;
	private TextView tvCast;
	private TextView tvAudienceScore;
	private TextView tvCriticsScore;
	private TextView tvCriticsConsensus;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_box_office_detail);
        // Fetch views
        ivPosterImage = (ImageView) findViewById(R.id.ivPosterImage);
        tvTitle = (TextView) findViewById(R.id.tvTitle);
        tvSynopsis = (TextView) findViewById(R.id.tvSynopsis);
        tvCast = (TextView) findViewById(R.id.tvCast);
        tvCriticsConsensus = (TextView) findViewById(R.id.tvCriticsConsensus);
        tvAudienceScore =  (TextView) findViewById(R.id.tvAudienceScore);
        tvCriticsScore = (TextView) findViewById(R.id.tvCriticsScore);
        // Use the movie to populate the data into our views
        BoxOfficeMovie movie = (BoxOfficeMovie) 
            getIntent().getSerializableExtra(BoxOfficeActivity.MOVIE_DETAIL_KEY);
        loadMovie(movie);
    }
	
    // Populate the data for the movie
    public void loadMovie(BoxOfficeMovie movie) {
        // Populate data
        tvTitle.setText(movie.getTitle());
        tvCriticsScore.setText(Html.fromHtml("<b>Critics Score:</b> " + movie.getCriticsScore() + "%"));
        tvAudienceScore.setText(Html.fromHtml("<b>Audience Score:</b> " + movie.getAudienceScore() + "%"));
        tvCast.setText(movie.getCastList());
        tvSynopsis.setText(Html.fromHtml("<b>Synopsis:</b> " + movie.getSynopsis()));
        tvCriticsConsensus.setText(Html.fromHtml("<b>Consensus:</b> " + movie.getCriticsConsensus()));
        // R.drawable.large_movie_poster from 
        // http://content8.flixster.com/movie/11/15/86/11158674_pro.jpg -->
        Picasso.with(this).load(movie.getLargePosterUrl()).
          placeholder(R.drawable.large_movie_poster).
          into(ivPosterImage);
    }  

}
```

With this, we should now be able to view the movie detail view:

<img src="http://i.imgur.com/9Uw7qLc.png" width="400" />

The full source code for this app can be [downloaded here](https://github.com/codepath/android-rottentomatoes-demo) for review as well.