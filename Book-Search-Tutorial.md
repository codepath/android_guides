In this tutorial, we'll make an app that searches the [OpenLibrary API](https://openlibrary.org/developers/api) to search books and display cover images. It also allows you to recommend books to friends.

## The App
The app is composed of two screens. The first screen displays a list of books, in which, each book is described by its title, author and cover photo. After a user selects a book from the list, a second screen appears displaying additional details about the book, including the publisher and no. of pages.

**Book List**

![Imgur](http://i.imgur.com/sSINs2zl.png)

**Book Details**

![Imgur](http://i.imgur.com/y9a4AtQl.png)

The full source code for this app can be [downloaded here](https://github.com/codepath/android-booksearch-demo) for review as well.

## Objectives
1. How to access a typical RESTful API?
2. How to parse JSON results and understand JSON format?
3. How to add SearchView to ActionBar to search books?
4. How to use ProgressBar?
5. How to use explicit and implicit intents?
6. How to communicate data between apps using intents?
7. How to share remote images?

## Steps to build app
### Create New Project
Create a new project in Android Studio and call it 'BookSearch'. Select the defaults and rename 'MainActivity' to 'BookListActivity'.

### Install Library Dependencies
Let's setup the [android-async-http-client](http://loopj.com/android-async-http/) for sending asynchronous network requests.

Let's also install a library for remote image loading called [Picasso](http://square.github.io/picasso/) so we can easily display cover images.

Open your `build.gradle` file and add these libraries as dependencies that can be downloaded via a remote repository:

```gradle
repositories {
  jcenter()
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

We'll use the following image when the cover image is not available.

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
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
  xmlns:tools="http://schemas.android.com/tools"
  android:layout_width="match_parent"
  android:layout_height="match_parent"
  android:orientation="vertical"
  tools:context=".BookListActivity">

  <ListView
    android:id="@+id/lvBooks"
    android:layout_width="match_parent"
    android:layout_height="match_parent" />
</LinearLayout>
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

Note that by default [Covers API](https://openlibrary.org/dev/docs/api/covers) returns a blank image if the cover cannot be found. If you append ?default=false to the end of the URL, then it returns a 404 instead. This is important because we want to display an error drawable in case the cover cannot be found.

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
      } else if(jsonObject.has("edition_key")) {
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
    Picasso.with(getContext()).load(Uri.parse(book.getCoverUrl())).error(R.drawable.ic_nocover).into(viewHolder.ivCover);
    // Return the completed view to render on screen
    return convertView;
  }
}
```

### Bringing It Together
Now that we have defined the adapter, let's plug all of these components together within the `BookListActivity` by using the client to send out an API call, deserialize the response into an array of `Book` object and then render those into a ListView using the `BookAdapter`. Let's start by initializing the ArrayList, adapter, and the ListView:

```java
public class BookListActivity extends AppCompatActivity {
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
public class BookListActivity extends AppCompatActivity {
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
            // Remove all books from the adapter
            bookAdapter.clear();
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

Note that the search query is hardcoded in `fetchBooks()` method. We'll fix this shortly. In the next step, we'll add SearchView to ActionBar and pass the query entered in the SearchView to retrieve the books from OpenLibrary search API.

### Add SearchView to ActionBar
We can use a SearchView to search for a book by its author or title. Using the SearchView widget as an item in the ActionBar is the preferred way to provide search in Android. To add SearchView to your ActionBar, add the SearchView action item in `menu_book_list.xml` file.

```xml
<menu xmlns:android="http://schemas.android.com/apk/res/android"
  xmlns:app="http://schemas.android.com/apk/res-auto"
  xmlns:tools="http://schemas.android.com/tools"
  tools:context=".BookListActivity">
  <item
    android:id="@+id/action_search"
    android:title="Search"
    android:icon="@android:drawable/ic_menu_search"
    app:showAsAction="always"
    app:actionViewClass="android.support.v7.widget.SearchView" />
</menu>
```

If you run your app now, the SearchView appears in the app's action bar, but it isn't functional.

![Imgur](http://i.imgur.com/TtRNzjml.png)

Now we need to hook up a listener for when a search is performed in `BookListActivity.java`. Also note that we modified `fetchBooks()` method to accept the search query and removed the call to fetch books from `onCreate()`.

```java
public class BookListActivity extends AppCompatActivity {
  ...

  @Override
  protected void onCreate(Bundle savedInstanceState) {
    ...
    // Fetch the data remotely
    // fetchBooks();
  }

  // Executes an API call to the OpenLibrary search endpoint, parses the results
  // Converts them into an array of book objects and adds them to the adapter
  private void fetchBooks(String query) {
    client = new BookClient();
    client.getBooks(query, new JsonHttpResponseHandler() {
      ...
    });
  }

  @Override
  public boolean onCreateOptionsMenu(Menu menu) {
    // Inflate the menu; this adds items to the action bar if it is present.
    getMenuInflater().inflate(R.menu.menu_book_list, menu);
    final MenuItem searchItem = menu.findItem(R.id.action_search);
    final SearchView searchView = (SearchView) MenuItemCompat.getActionView(searchItem);
    searchView.setOnQueryTextListener(new SearchView.OnQueryTextListener() {
      @Override
      public boolean onQueryTextSubmit(String query) {
        // Fetch the data remotely
        fetchBooks(query);
        // Reset SearchView
        searchView.clearFocus();
        searchView.setQuery("", false);
        searchView.setIconified(true);
        searchItem.collapseActionView();
        // Set activity title to search query
        BookListActivity.this.setTitle(query);
        return true;
      }

      @Override
      public boolean onQueryTextChange(String s) {
        return false;
      }
    });
    return true;
  }
}
```

You should now be able to search for books from the SearchView in your ActionBar.
![Imgur](http://i.imgur.com/sSINs2zl.png)

### Show Progress Bar
A ProgressBar should be used when we want the user to wait till a task completes. It's a good practice to always show a ProgressBar before making a network call.

To display an indeterminate ProgressBar, add a ProgressBar to `activity_book_list.xml` file.

```xml
<LinearLayout
...>

  <ProgressBar
  android:id="@+id/progress"
  android:layout_width="match_parent"
  android:layout_height="wrap_content"
  style="?android:attr/progressBarStyleHorizontal"
  android:max="100"
  android:indeterminate="true"
  android:visibility="gone" />

  <ListView
  ... />

</LinearLayout>
```

In `BookListActivity.java`, show ProgressBar before calling the search API and hide it when you receive the results back from the server.

```java
public class BookListActivity extends AppCompatActivity {
  ...
  private ProgressBar progress;

  @Override
  protected void onCreate(Bundle savedInstanceState) {
    ...
    progress = (ProgressBar) findViewById(R.id.progress);
  }

  // Executes an API call to the OpenLibrary search endpoint, parses the results
  // Converts them into an array of book objects and adds them to the adapter
  private void fetchBooks(String query) {
    // Show progress bar before making network request
    progress.setVisibility(ProgressBar.VISIBLE);
    ...

    client.getBooks(query, new JsonHttpResponseHandler() {
      @Override
      public void onSuccess(int statusCode, Header[] headers, JSONObject response) {
        try {
          // hide progress bar
          progress.setVisibility(ProgressBar.GONE);
          ...
        } catch (JSONException e) {
          // Invalid JSON format, show appropriate error.
          e.printStackTrace();
        }
      }

      @Override
      public void onFailure(int statusCode, Header[] headers, String responseString, Throwable throwable) {
        progress.setVisibility(ProgressBar.GONE);
      }
    });
  }
}
```

### Adding a Detail View
Let's add a detail view which would display more details about a book when the book was selected within the list. In order to achieve that we would need to:

1. Generate an activity for the details view
2. Define the detail Layout XML with views
3. Setup an item click listener for the list of books
4. Fire an intent when a user selects a book from the list
5. Pass the book through the intent and populate the detail view

First, let's generate an activity to display book details and name it `BookDetailActivity.java`. Edit the layout file `activity_book_detail.xml` to include book details and a cover photo.

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
  xmlns:tools="http://schemas.android.com/tools"
  android:layout_width="match_parent"
  android:layout_height="match_parent"
  android:orientation="vertical"
  tools:context=".BookListActivity">

  <ScrollView
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <LinearLayout
      android:gravity="center_horizontal"
      android:layout_width="match_parent"
      android:layout_height="wrap_content"
      android:orientation="vertical">

      <ImageView
        android:id="@+id/ivBookCover"
        android:layout_marginTop="8dp"
        android:layout_width="280dp"
        android:layout_height="280dp" />

      <TextView
        android:id="@+id/tvTitle"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textSize="32sp"
        android:layout_marginTop="8dp"
        android:paddingLeft="10dp"
        android:paddingRight="10dp" />

      <TextView
        android:id="@+id/tvAuthor"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textSize="22sp"
        android:layout_marginTop="8dp" />

      <View
        android:background="@android:color/darker_gray"
        android:layout_width="280dp"
        android:layout_height="1px"
        android:layout_marginTop="8dp" />

      <TextView
      android:layout_width="wrap_content"
      android:layout_height="wrap_content"
      android:layout_marginTop="8dp"
      android:text="Published By"
      android:textSize="14sp" />

      <TextView
      android:id="@+id/tvPublisher"
      android:layout_width="wrap_content"
      android:layout_height="wrap_content"
      android:textSize="14sp" />

      <TextView
      android:id="@+id/tvPageCount"
      android:layout_width="wrap_content"
      android:layout_height="wrap_content"
      android:layout_marginTop="8dp"
      android:textSize="14sp" />
    </LinearLayout>
  </ScrollView>
</LinearLayout>
```

Next, we want to be able to pass the book object from the list to this detail view, so let's make our `Book` class serializable:

```java
public class Book implements Serializable {
  ...
}
```

Let's now add an item click handler to our `BookListActivity.java` which will launch the detail view with an intent:

```java
public class BookListActivity extends AppCompatActivity {
  public static final String BOOK_DETAIL_KEY = "book";
  ...

  @Override
  protected void onCreate(Bundle savedInstanceState) {
    ...
    setupBookSelectedListener();
  }

  public void setupBookSelectedListener() {
    lvBooks.setOnItemClickListener(new AdapterView.OnItemClickListener() {
      @Override
      public void onItemClick(AdapterView<?> parent, View view, int position, long id) {
        // Launch the detail view passing book as an extra
        Intent intent = new Intent(BookListActivity.this, BookDetailActivity.class);
        intent.putExtra(BOOK_DETAIL_KEY, bookAdapter.getItem(position));
        startActivity(intent);
      }
    });
  }

  ...
}
```

In addition to title, author and cover photo, let's also display the no. of pages in a book and its publisher. To fetch this data, we need to call the OpenLibrary [Books API](https://openlibrary.org/dev/docs/api/books). Add a method to `BookClient.java` to fetch extra details of a book.

```java
public class BookClient {
  ...

  // Method for accessing books API to get publisher and no. of pages in a book.
  public void getExtraBookDetails(String openLibraryId, JsonHttpResponseHandler handler) {
    String url = getApiUrl("books/");
    client.get(url + openLibraryId + ".json", handler);
  }
}
```

Now, let's populate the data from the book into the details view with `BookDetailActivity.java`:

```java
public class BookDetailActivity extends AppCompatActivity {
  private ImageView ivBookCover;
  private TextView tvTitle;
  private TextView tvAuthor;
  private TextView tvPublisher;
  private TextView tvPageCount;
  private BookClient client;

  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_book_detail);
    // Fetch views
    ivBookCover = (ImageView) findViewById(R.id.ivBookCover);
    tvTitle = (TextView) findViewById(R.id.tvTitle);
    tvAuthor = (TextView) findViewById(R.id.tvAuthor);
    tvPublisher = (TextView) findViewById(R.id.tvPublisher);
    tvPageCount = (TextView) findViewById(R.id.tvPageCount);
    // Use the book to populate the data into our views
    Book book = (Book) getIntent().getSerializableExtra(BookListActivity.BOOK_DETAIL_KEY);
    loadBook(book);
  }

  // Populate data for the book
  private void loadBook(Book book) {
    //change activity title
    this.setTitle(book.getTitle());
    // Populate data
    Picasso.with(this).load(Uri.parse(book.getLargeCoverUrl())).error(R.drawable.ic_nocover).into(ivBookCover);
    tvTitle.setText(book.getTitle());
    tvAuthor.setText(book.getAuthor());
    // fetch extra book data from books API
    client = new BookClient();
    client.getExtraBookDetails(book.getOpenLibraryId(), new JsonHttpResponseHandler() {
      @Override
      public void onSuccess(int statusCode, Header[] headers, JSONObject response) {
        try {
          if (response.has("publishers")) {
            // display comma separated list of publishers
            final JSONArray publisher = response.getJSONArray("publishers");
            final int numPublishers = publisher.length();
            final String[] publishers = new String[numPublishers];
            for (int i = 0; i < numPublishers; ++i) {
              publishers[i] = publisher.getString(i);
            }
            tvPublisher.setText(TextUtils.join(", ", publishers));
          }
          if (response.has("number_of_pages")) {
            tvPageCount.setText(Integer.toString(response.getInt("number_of_pages")) + " pages");
          }
        } catch (JSONException e) {
          e.printStackTrace();
        }
      }
      });
    }


    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
      // Inflate the menu; this adds items to the action bar if it is present.
      getMenuInflater().inflate(R.menu.menu_book_detail, menu);
      return true;
    }

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
      // Handle action bar item clicks here. The action bar will
      // automatically handle clicks on the Home/Up button, so long
      // as you specify a parent activity in AndroidManifest.xml.
      int id = item.getItemId();

      //noinspection SimplifiableIfStatement
      if (id == R.id.action_settings) {
        return true;
      }

      return super.onOptionsItemSelected(item);
    }
}
```

If you run your app now, your detail screen should look similar to this:

![Imgur](http://i.imgur.com/SJSEsVml.png)


### Add Share Intent

Let's use an implicit intent `ACTION_SEND` to share a book with your friends. We'll share book title and cover photo.

Start by adding a menu item in `menu_book_detail.xml`.

```xml
<menu xmlns:android="http://schemas.android.com/apk/res/android"
  xmlns:tools="http://schemas.android.com/tools"
  xmlns:app="http://schemas.android.com/apk/res-auto"
  tools:context=".BookListActivity">
  <item
    android:id="@+id/action_share"
    app:showAsAction="ifRoom"
    android:icon="@android:drawable/ic_menu_share"
    android:title="Share" />
</menu>  
```

Trigger a share intent and bring up the share menu on click of share menu item. Also note how remote images are shared using shared intent.

```java
public class BookDetailActivity extends AppCompatActivity {
  ...

  @Override
  public boolean onOptionsItemSelected(MenuItem item) {
    // Handle action bar item clicks here. The action bar will
    // automatically handle clicks on the Home/Up button, so long
    // as you specify a parent activity in AndroidManifest.xml.
    int id = item.getItemId();

    if (id == R.id.action_share) {
      setShareIntent();
      return true;
    }
    return super.onOptionsItemSelected(item);
  }

  private void setShareIntent() {
    ImageView ivImage = (ImageView) findViewById(R.id.ivBookCover);
    final TextView tvTitle = (TextView)findViewById(R.id.tvTitle);
    // Get access to the URI for the bitmap
    Uri bmpUri = getLocalBitmapUri(ivImage);
    // Construct a ShareIntent with link to image
    Intent shareIntent = new Intent();
    // Construct a ShareIntent with link to image
    shareIntent.setAction(Intent.ACTION_SEND);
    shareIntent.setType("*/*");
    shareIntent.putExtra(Intent.EXTRA_TEXT, (String)tvTitle.getText());
    shareIntent.putExtra(Intent.EXTRA_STREAM, bmpUri);
    // Launch share menu
    startActivity(Intent.createChooser(shareIntent, "Share Image"));
  }

  // Returns the URI path to the Bitmap displayed in cover imageview
  public Uri getLocalBitmapUri(ImageView imageView) {
    // Extract Bitmap from ImageView drawable
    Drawable drawable = imageView.getDrawable();
    Bitmap bmp = null;
    if (drawable instanceof BitmapDrawable){
      bmp = ((BitmapDrawable) imageView.getDrawable()).getBitmap();
    } else {
      return null;
    }
    // Store image to default external storage directory
    Uri bmpUri = null;
    try {
      File file =  new File(Environment.getExternalStoragePublicDirectory(
      Environment.DIRECTORY_DOWNLOADS), "share_image_" + System.currentTimeMillis() + ".png");
      file.getParentFile().mkdirs();
      FileOutputStream out = new FileOutputStream(file);
      bmp.compress(Bitmap.CompressFormat.PNG, 90, out);
      out.close();
      bmpUri = Uri.fromFile(file);
    } catch (IOException e) {
      e.printStackTrace();
    }
    return bmpUri;
  }
}
```

Here, `getLocalBitmapUri()` extracts the bitmap from the ImageView and loads it into a file in the default extranl storage directory. To do this, the app will need certain permissions.

Make sure to add the appropriate permissions to your `AndroidManifest.xml`:

```xml
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
```

Here's the final detail view with share icon:

![Imgur](http://i.imgur.com/y9a4AtQl.png)

Tap the share icon and the share menu appears:

![Imgur](http://i.imgur.com/w0ZQQyKl.png)
