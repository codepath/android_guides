In this tutorial, we'll make an app that searches the [OpenLibrary API](https://openlibrary.org/developers/api) to search books and display cover images. It also allows you to recommend books to friends.

## Objectives
1. How to access a typical RESTful API?
2. How to parse JSON results and understand JSON format?
3. How to replace ActionBar with ToolBar?
4. How to add SearchView to ToolBar to search books?
5. How to use explicit and implicit intents?
6. How to communicate data between apps using intents?

## Steps to build app
### Create New Project
Create a new project in Android Studio and call it 'BookSearch'. Select the defaults and rename 'MainActivity' to 'BookListActivity'.

### Install Library Dependencies
Let's setup the [android-async-http-client](http://loopj.com/android-async-http/) for sending asynchronous network requests.

Let's also install a library for remote image loading called [Picasso](http://square.github.io/picasso/) so we can easily display cover images.

Open your `build.gradle` file and add these libraries as dependencies that can be downloaded via a remote repository:

```gradle
repositories {
  mavenCentral()
}
dependencies {
  // ...
  compile 'com.loopj.android:android-async-http:1.4.6'
  compile 'com.squareup.picasso:picasso:2.4.0'
}
```

Then you need to sync with gradle and make sure there are no errors.

### Download Assets
We'll be using the following icon for our app.

<a href="http://imgur.com/3odnsIE"><img src="http://i.imgur.com/3odnsIEl.png" title="source: imgur.com" /></a>

We'll use the following placeholder image when the cover image is not available.

<a href="http://imgur.com/DTkTKtI"><img src="http://i.imgur.com/DTkTKtIl.png" title="source: imgur.com" /></a>

Copy and paste both [ic_launcher.png](http://i.imgur.com/3odnsIEl.png/ic_launcher.png) and [ic_nocover.png](http://i.imgur.com/DTkTKtIl.png/ic_nocover.png) into `res/drawable` for use later.

### Setup Internet Permission

Add the proper internet permissions to the `AndroidManifest.xml` within the `manifest` node:

```xml
<uses-permission android:name="android.permission.INTERNET" />
```

Once you add this, network requests can be executed.

### Setup BookListActivity Layout
In the `res/layout/activity_book_list.xml`, lets drop out a ListView with the id `lvBooks`:
```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
  xmlns:tools="http://schemas.android.com/tools"
  android:layout_width="match_parent"
  android:layout_height="match_parent"
  tools:context=".BookSearchActivity">

  <ListView
    android:id="@+id/lvBooks"
    android:layout_width="match_parent"
    android:layout_height="match_parent" />

  </RelativeLayout>
```
![Imgur](http://i.imgur.com/4e1PlU4l.png)


### Construct API Client
Let's generate a Java class that will act as our OpenLibrary API client for sending out network requests to specific endpoints. Create a Java class called `BookClient` with the following code:

```java
public class BookClient {
  private static final String API_BASE_URL = "http://openlibrary.org/";
  private AsyncHttpClient client;

  public BookClient() {
    this.client = new AsyncHttpClient();
  }

  private String getApiUrl(String relativeUrl) {
    return API_BASE_URL + relativeUrl;
  }

  // Method for accessing the search API
  public void getBooks(final String query, JsonHttpResponseHandler handler) {
    try {
      String url = getApiUrl("search.json?q=");
      client.get(url + URLEncoder.encode(query, "utf-8"), handler);
    } catch (UnsupportedEncodingException e) {
      e.printStackTrace();
    }
  }
}
```
Looking at the JSON response format for the OpenLibrary search API, we can understand what information we will be provided:

```json
[  
  {  
    "title_suggest":"Lord of the Rings",
    "edition_key":[  
      "OL9409698M",
      "OL10957365M",
      "OL10957364M",
      "OL9314653M",
      "OL9701406M"
    ],
    "cover_i":1454705,
    "isbn":[  
      "0768325293",
      "9780768374698",
      "9780768325782",
      "9780768325249",
      "9780768325294",
      "0768325242",
      "0768325234",
      "0768374693",
      "0768325781",
      "9780768325232"
    ],
    "has_fulltext":false,
    "text":[  
      "OL9409698M",
      "OL10957365M",
      "OL10957364M",
      "OL9314653M",
      "OL9701406M",
      "0768325293",
      "9780768374698",
      "9780768325782",
      "9780768325249",
      "9780768325294",
      "0768325242",
      "0768325234",
      "0768374693",
      "0768325781",
      "9780768325232",
      "Cedco Publishing",
      "OL2853379A",
      "Journal # 4 (Lord of the Rings)",
      "Trilogy Edition 2006 Calendar (Cedco Mini Daily)",
      "Journal # 3 (Lord of the Rings)",
      "Lord of the Rings",
      "\/works\/OL8527426W",
      "Cedco Publishing Company",
      "The Lord of the Rings"
    ],
    "author_name":[  
    "Cedco Publishing"
    ],
    "seed":[  
    "\/books\/OL9409698M",
    "\/books\/OL10957365M",
    "\/books\/OL10957364M",
    "\/books\/OL9314653M",
    "\/books\/OL9701406M",
    "\/works\/OL8527426W",
    "\/authors\/OL2853379A"
    ],
    "author_key":[  
    "OL2853379A"
    ],
    "title":"Lord of the Rings",
    "publish_date":[  
    "August 30, 2005",
    "September 2002",
    "October 2001"
    ],
    "type":"work",
    "ebook_count_i":0,
    "edition_count":5,
    "key":"\/works\/OL8527426W",
    "id_goodreads":[  
    "130022",
    "5984308",
    "59318"
    ],
    "publisher":[  
    "Cedco Publishing Company"
    ],
    "language":[  
    "eng"
    ],
    "last_modified_i":1282021198,
    "id_librarything":[  
    "4607237"
    ],
    "cover_edition_key":"OL9701406M",
    "publish_year":[  
    2002,
    2001,
    2005
    ],
    "first_publish_year":2001
    },
    {  
      "title_suggest":"Ring out, wild bells",
      ...
    }
  ]
```

### Define the Data Model
Next, let's define a model that represents the relevant data for a single book. This object needs to contain basic info for each book such as title, author, cover image and an openLibraryId. Create a Java class called `Book` with the relevant attributes defined.

```java
public class Book {
  private String openLibraryId;
  private String author;
  private String title;

  public String getOpenLibraryId() {
    return openLibraryId;
  }

  public String getTitle() {
    return title;
  }

  public String getAuthor() {
    return author;
  }

  // Get medium sized book cover from covers API
  public String getCoverUrl() {
    return "http://covers.openlibrary.org/b/olid/" + openLibraryId + "-M.jpg?default=false";
  }

  // Get large sized book cover from covers API
  public String getLargeCoverUrl() {
    return "http://covers.openlibrary.org/b/olid/" + openLibraryId + "-L.jpg?default=false";
  }
}
```

Note that by default [Covers API](https://openlibrary.org/dev/docs/api/covers) returns a blank image if the cover cannot be found. If you append ?default=false to the end of the URL, then it returns a 404 instead. This is important because we want to display a placeholder image in case the cover cannot be found.

Next, let's add a factory method for deserializing the JSON response in order to construct an instance of the `Book` class:

```java
public class Book {
  ...

  // Returns a Book given the expected JSON
  public static Book fromJson(JSONObject jsonObject) {
    Book book = new Book();
    try {
      // Deserialize json into object fields
      // Check if a cover edition is available
      if (jsonObject.has("cover_edition_key"))  {
        book.openLibraryId = jsonObject.getString("cover_edition_key");
      } else {
        final JSONArray ids = jsonObject.getJSONArray("edition_key");
        book.openLibraryId = ids.getString(0);
      }
      book.title = jsonObject.has("title_suggest") ? jsonObject.getString("title_suggest") : "";
      book.author = getAuthor(jsonObject);
    } catch (JSONException e) {
      e.printStackTrace();
      return null;
    }
    // Return new object
    return book;
  }

  // Return comma separated author list when there is more than one author
  private static String getAuthor(final JSONObject jsonObject) {
    try {
      final JSONArray authors = jsonObject.getJSONArray("author_name");
      int numAuthors = authors.length();
      final String[] authorStrings = new String[numAuthors];
      for (int i = 0; i < numAuthors; ++i) {
        authorStrings[i] = authors.getString(i);
      }
      return TextUtils.join(", ", authorStrings);
    } catch (JSONException e) {
      return "";
    }
  }
}
```

Let's also add a convenience static method for parsing an array of JSON book dictionaries into an ArrayList<Book> which we will use to manage the OpenLibrary search API endpoint response:
```java
public class Book {
  ...

  // Decodes array of book json results into business model objects
  public static ArrayList<Book> fromJson(JSONArray jsonArray) {
    ArrayList<Book> books = new ArrayList<Book>(jsonArray.length());
    // Process each result in json array, decode and convert to business
    // object
    for (int i = 0; i < jsonArray.length(); i++) {
      JSONObject bookJson = null;
      try {
        bookJson = jsonArray.getJSONObject(i);
      } catch (Exception e) {
        e.printStackTrace();
        continue;
      }
      Book book = Book.fromJson(bookJson);
      if (book != null) {
        books.add(book);
      }
    }
    return books;
  }
}
```

### Construct an ArrayAdapter
Now, we have the API Client (`BookClient`) and the data Model object (`Book`). Final step is to construct an adapter that allows us to translate a `Book` into a particular view. Let's generate a Java class called `BookAdapter` which extends from `ArrayAdapter<Book>`:

```java
public class BookAdapter extends ArrayAdapter<Book> {

  public BookAdapter(Context context, ArrayList<Book> aBooks) {
    super(context, 0, aBooks);
  }

  @Override
  public View getView(int position, View convertView, ViewGroup parent) {
    // TODO: Complete the definition of the view for each book
    return null;
  }
}
```

Now, we need to define a layout to use for visualizing a particular book. Let's create a layout file called `item_book.xml` with a `RelativeLayout` root layout for defining the book row layout:

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
  android:layout_width="match_parent"
  android:layout_height="wrap_content"
  android:orientation="vertical">

  <ImageView
    android:id="@+id/ivBookCover"
    android:layout_width="56dp"
    android:layout_height="56dp"
    android:layout_margin="8dp" />

  <TextView
    android:id="@+id/tvTitle"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:layout_margin="8dp"
    android:layout_toRightOf="@+id/ivBookCover"
    android:text="Title"
    android:textSize="24dp" />

  <TextView
    android:id="@+id/tvAuthor"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:layout_marginLeft="8dp"
    android:layout_toRightOf="@+id/ivBookCover"
    android:layout_below="@+id/tvTitle"
    android:text="Author"
    android:textSize="12dp" />
</RelativeLayout>
```

and now we can fill in the `getView` method to finish off the `BookAdapter`:

```java
public class BookAdapter extends ArrayAdapter<Book> {
  // View lookup cache
  private static class ViewHolder {
    public ImageView ivCover;
    public TextView tvTitle;
    public TextView tvAuthor;
  }

  public BookAdapter(Context context, ArrayList<Book> aBooks) {
    super(context, 0, aBooks);
  }

  // Translates a particular `Book` given a position
  // into a relevant row within an AdapterView
  @Override
  public View getView(int position, View convertView, ViewGroup parent) {
    // Get the data item for this position
    final Book book = getItem(position);
    // Check if an existing view is being reused, otherwise inflate the view
    ViewHolder viewHolder; // view lookup cache stored in tag
    if (convertView == null) {
      viewHolder = new ViewHolder();
      LayoutInflater inflater = (LayoutInflater)getContext().getSystemService(Context.LAYOUT_INFLATER_SERVICE);
      convertView = inflater.inflate(R.layout.item_book, parent, false);
      viewHolder.ivCover = (ImageView)convertView.findViewById(R.id.ivBookCover);
      viewHolder.tvTitle = (TextView)convertView.findViewById(R.id.tvTitle);
      viewHolder.tvAuthor = (TextView)convertView.findViewById(R.id.tvAuthor);
      convertView.setTag(viewHolder);
    } else {
      viewHolder = (ViewHolder) convertView.getTag();
    }
    // Populate the data into the template view using the data object
    viewHolder.tvTitle.setText(book.getTitle());
    viewHolder.tvAuthor.setText(book.getAuthor());
    Picasso.with(getContext()).load(Uri.parse(book.getCoverUrl())).placeholder(R.drawable.ic_nocover).into(viewHolder.ivCover);
    // Return the completed view to render on screen
    return convertView;
  }
}
```

### Bringing It Together
Now that we have defined the adapter, let's plug all of these components together within the `BookListActivity` by using the client to send out an API call, deserialize the response into an array of `Book` object and then render those into a ListView using the `BookAdapter`. Let's start by initializing the ArrayList, adapter, and the ListView:

```java
public class BookListActivity extends ActionBarActivity {
  private ListView lvBooks;
  private BookAdapter bookAdapter;

  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_book_list);
    lvBooks = (ListView) findViewById(R.id.lvBooks);
    ArrayList<Book> aBooks = new ArrayList<Book>();
    bookAdapter = new BookAdapter(this, aBooks);
    lvBooks.setAdapter(bookAdapter);
  }
}
```

Now let's make the call to the book search API from the activity using our client:

```java
public class BookListActivity extends ActionBarActivity {
  ...
  private BookClient client;

  @Override
  protected void onCreate(Bundle savedInstanceState) {
    ...
    // Fetch the data remotely
    fetchBooks();
  }

  // Executes an API call to the OpenLibrary search endpoint, parses the results
  // Converts them into an array of book objects and adds them to the adapter
  private void fetchBooks() {
    client = new BookClient();
    client.getBooks("oscar Wilde", new JsonHttpResponseHandler() {
      @Override
      public void onSuccess(int statusCode, Header[] headers, JSONObject response) {
        try {
          JSONArray docs = null;
          if(response != null) {
            // Get the docs json array
            docs = response.getJSONArray("docs");
            // Parse json array into array of model objects
            final ArrayList<Book> books = Book.fromJson(docs);
            // Load model objects into the adapter
            for (Book book : books) {
              bookAdapter.add(book); // add book through the adapter
            }
            bookAdapter.notifyDataSetChanged();
          }
        } catch (JSONException e) {
          // Invalid JSON format, show appropriate error.
          e.printStackTrace();
        }
      }
    });
  }
}
```

After running the app, we should see the following output:

![Imgur](http://i.imgur.com/NJmF42Yl.png)

Note that the search query is hardcoded in `fetchBooks()` method. We will fix this shortly. In the next step, we will replace the ActionBar with a ToolBar, add SearchView to ToolBar and pass the query entered in the SearchView to retrieve the books from OpenLibrary search API.
