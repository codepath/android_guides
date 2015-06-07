## Overview

The way to access data from content providers is to use the `LoaderManager` to execute query and bind cursor result to list using `SimpleCursorAdapter`.  The loader manager is used to fetch the cursor asynchronously and the cursor is loaded directly into a `SimpleCursorAdapter`. 

## Using CursorLoader and SimpleCursorAdapter

In this example, we load the data directly from the ContentProvider using a [CursorLoader](https://developer.android.com/training/load-data-background/setup-loader.html) and then plugging the resulting dataset directly into a [SimpleCursorAdapter](http://developer.android.com/reference/android/widget/SimpleCursorAdapter.html). 

Let's define a few terms used below so we understand how this example fits together:

 * `CursorLoader` - A loader object that queries a `ContentResolver` for data and returns a `Cursor`.
 * `Cursor` - Provides random read-write access to the result set returned by the `CursorLoader`.
 * `LoaderManager` - Manages background loading operations such as async querying with `CursorLoader`.  
 * `CursorAdapter` - Adapter that exposes data from a Cursor source to a ListView widget.

These four concepts are the underlying pieces to loading data through a content provider.

### Permissions

First, setup permissions in the manifest:

```xml
<uses-permission android:name="android.permission.READ_CONTACTS"/>
```

### Constructing the SimpleCursorAdapter

The [SimpleCursorAdapter](http://developer.android.com/reference/android/widget/SimpleCursorAdapter.html) is an adapter that binds a `ListView` to a `Cursor` dataset displaying the result set as rows in the list. We can create the adapter before receiving the cursor by constructing as follows:

```java
public class SampleActivity extends FragmentActivity {
    // ... existing code ...
    private SimpleCursorAdapter adapter;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
       // ...
       setupCursorAdapter();
    }
    
    // Create simple cursor adapter to connect the cursor dataset we load with a ListView
    private void setupCursorAdapter() {
        // Column data from cursor to bind views from	
      	String[] uiBindFrom = { ContactsContract.Contacts.DISPLAY_NAME,
            ContactsContract.Contacts.PHOTO_URI };
      	// View IDs which will have the respective column data inserted
        int[] uiBindTo = { R.id.tvName, R.id.ivImage };
        // Create the simple cursor adapter to use for our list
        // specifying the template to inflate (item_contact),
      	adapter = new SimpleCursorAdapter(
                  this, R.layout.item_contact,
                  null, uiBindFrom, uiBindTo,
                  0);
    }
}
```

Note that if you want to use a more complex custom layout, you should construct a [[custom CursorAdapter|Populating-a-ListView-with-a-CursorAdapter]] to replace the `SimpleCursorAdapter` shown above.

### Connecting the ListView

Once we've defined our cursor adapter, we can add a `ListView` to our activity called `R.id.lvContacts` which will contain the list of contacts loaded from the content provider. Once we've **defined the ListView in our layout XML**, we can bind the list to our adapter:

```java
public class SampleActivity extends FragmentActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
       // ...
       setupCursorAdapter();
       // Find list and bind to adapter
       ListView lvContacts = (ListView) findViewById(R.id.lvContacts);
       lvContacts.setAdapter(adapter);
    }
}
```

Now, once we've loaded the `Cursor` from the Contacts ContentProvider, we can plug the dataset into the adapter and the list will automatically populate accordingly.

### Using CursorLoader to Query the ContentProvider

Whenever we want to load data from a registered ContactProvider, we want to do the query asynchronously in order to avoid blocking the UI. Easiest way to execute this query is using a [CursorLoader](https://developer.android.com/training/load-data-background/setup-loader.html) which runs an asynchronous query in the background against a `ContentProvider`, and returns the results to our Activity.

To execute the request to our contacts provider, we need to define the callbacks that the loader will use to create the request and handle the results. These callbacks are of type [LoaderCallbacks](http://developer.android.com/reference/android/app/LoaderManager.LoaderCallbacks.html) and can be defined as follows:


```java
public class SampleActivity extends FragmentActivity {
   // ... existing code

    // Defines the asynchronous callback for the contacts data loader
    private LoaderManager.LoaderCallbacks<Cursor> contactsLoader = 
        new LoaderManager.LoaderCallbacks<Cursor>() {
    	// Create and return the actual cursor loader for the contacts data
    	@Override
    	public Loader<Cursor> onCreateLoader(int id, Bundle args) {
    		// Define the columns to retrieve
    		String[] projectionFields =  new String[] { ContactsContract.Contacts._ID, 
    	               ContactsContract.Contacts.DISPLAY_NAME, 
                       ContactsContract.Contacts.PHOTO_URI };
    		// Construct the loader
    		CursorLoader cursorLoader = new CursorLoader(SampleActivity.this,
    				ContactsContract.Contacts.CONTENT_URI, // URI
    				projectionFields,  // projection fields
    				null, // the selection criteria
    				null, // the selection args
    				null // the sort order
    		);
    		// Return the loader for use
    		return cursorLoader;
    	}

    	// When the system finishes retrieving the Cursor through the CursorLoader, 
        // a call to the onLoadFinished() method takes place. 
    	@Override
    	public void onLoadFinished(Loader<Cursor> loader, Cursor cursor) {
    		// The swapCursor() method assigns the new Cursor to the adapter
    		adapter.swapCursor(cursor);  
    	}

    	// This method is triggered when the loader is being reset 
        // and the loader data is no longer available. Called if the data 
        // in the provider changes and the Cursor becomes stale.
    	@Override
    	public void onLoaderReset(Loader<Cursor> loader) {
    		// Clear the Cursor we were using with another call to the swapCursor()
    		adapter.swapCursor(null);
    	}
    };
}
```

Now when a result comes back to the callback defined, the adapter will be bound to the cursor. With the loader callbacks specified, we can now setup our loader and execute the asynchronous request to the content provider:

```java
public class SampleActivity extends FragmentActivity {
    // ... existing code
    // Defines the id of the loader for later reference
    public static final int CONTACT_LOADER_ID = 78; // From docs: A unique identifier for this loader. Can be whatever you want.

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        // Bind adapter to list
        setupCursorAdapter();
        // Initialize the loader with a special ID and the defined callbacks from above
        getSupportLoaderManager().initLoader(CONTACT_LOADER_ID, 
            new Bundle(), contactsLoader);
    }
}
```

Now we have completed the process of loading the contacts into our list using a `CursorLoader` querying the `ContactProvider` and loading the resulting `Cursor` into the `CursorAdapter`. We should now see the list of the names of our contacts.

## Contracts

There is yet to be a standard place to find what data is exposed by various Content Providers.  Here is a list of known contracts defined for a few common ones.

* `Browser.BookmarkColumns` ([Docs](http://developer.android.com/reference/android/provider/Browser.BookmarkColumns.html))
* `CalendarContract` ([Docs](http://developer.android.com/reference/android/provider/CalendarContract.html))
* `ContactsContract` ([Docs](http://developer.android.com/reference/android/provider/ContactsContract.html))
* `UserDictionary` ([Docs](http://developer.android.com/reference/android/provider/UserDictionary.html))
* `MediaStore.Audio`, `MediaStore.Video` ([Docs](http://developer.android.com/reference/android/provider/MediaStore.html))
* `DocumentsContract` ([Docs](https://developer.android.com/reference/android/provider/DocumentsContract.html))
* `Settings` ([Docs](http://developer.android.com/reference/android/provider/Settings.html))

## References

 * <http://www.androiddesignpatterns.com/2012/07/loaders-and-loadermanager-background.html> 
 * <http://code.tutsplus.com/tutorials/android-sdk_loading-data_cursorloader--mobile-5673>
 * <http://www.app-solut.com/blog/2011/03/working-with-the-contactscontract-to-query-contacts-in-android/>
 * <https://developer.android.com/training/load-data-background/setup-loader.html>
 * <http://www.compiletimeerror.com/2013/12/how-to-use-android-cursorloader.html#.VDHm9GRdUt0>
 * <http://developer.android.com/reference/android/widget/SimpleCursorAdapter.html>
 * <http://www.higherpass.com/Android/Tutorials/Working-With-Android-Contacts/>