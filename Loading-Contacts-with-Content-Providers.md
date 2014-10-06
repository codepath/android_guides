## Overview

The following guide walks step by step through loading contacts from the phone using Content Providers. See [the full sample here](https://github.com/thecodepath/contacts-loader-example) for the source code.

### Permissions

First, setup permissions in the manifest:

```xml
<uses-permission android:name="android.permission.READ_CONTACTS"/>
```

### Data Models

Next, you should create classes that will represent our Contact and his information. Start with `Contact.java`:

```java
public class Contact {
	public String id;
	public String name;
	public ArrayList<ContactEmail> emails;
	public ArrayList<ContactPhone> numbers;

	public Contact(String id, String name) {
		this.id = id;
		this.name = name;
		this.emails = new ArrayList<ContactEmail>();
		this.numbers = new ArrayList<ContactPhone>();
	}

	public void addEmail(String address, String type) {
		emails.add(new ContactEmail(address, type));
	}

	public void addNumber(String number, String type) {
		numbers.add(new ContactPhone(number, type));
	}
}

```

and then the `ContactPhone.java` for the numbers:

```java
public class ContactPhone {
	public String number;
	public String type;

	public ContactPhone(String number, String type) {
		this.number = number;
		this.type = type;
	}
}
```

and then the `ContactEmail.java` for the emails:

```java
public class ContactEmail {
	public String address;
	public String type;

	public ContactEmail(String address, String type) {
		this.address = address;
		this.type = type;
	}
}
```

### Fetcher

Now we need to write the code that actually queries the content provider and builds our Contact models up based on the queries to the provider:

```java
// new ContactFetcher(this).fetchAll();
public class ContactFetcher {
	private Context context;

	public ContactFetcher(Context c) {
		this.context = c;
	}

	public ArrayList<Contact> fetchAll() {
		ArrayList<Contact> listContacts = new ArrayList<Contact>();
		CursorLoader cursorLoader = new CursorLoader(context, RawContacts.CONTENT_URI, 
				null, // the columns to retrieve (all)
				null, // the selection criteria (none)
				null, // the selection args (none)
				null // the sort order (default)
		);

		Cursor c = cursorLoader.loadInBackground();
		if (c.moveToFirst()) {
			do {
			    Contact contact = loadContactData(c);
			    listContacts.add(contact);
			} while (c.moveToNext());
		}
		c.close();
		return listContacts;
	}

	private Contact loadContactData(Cursor c) {
		// Get Contact ID
		int idIndex = c.getColumnIndex(ContactsContract.Contacts._ID);
		String contactId = c.getString(idIndex);
		// Get Contact Name
		int nameIndex = c.getColumnIndex(ContactsContract.Contacts.DISPLAY_NAME);
		String contactDisplayName = c.getString(nameIndex);
		Contact contact = new Contact(contactId, contactDisplayName);
		fetchContactNumbers(c, contact);
		fetchContactEmails(c, contact);
		return contact;
	}

        
	public void fetchContactNumbers(Cursor cursor, Contact contact) {
		// Get numbers
		final String[] numberProjection = new String[] { Phone.NUMBER, Phone.TYPE, };
		Cursor phone = new CursorLoader(context, Phone.CONTENT_URI, numberProjection,
				RawContacts.CONTACT_ID + "= ?", 
                                new String[] { String.valueOf(contact.id) },
                                null).loadInBackground();

		if (phone.moveToFirst()) {
			final int contactNumberColumnIndex = phone.getColumnIndex(Phone.NUMBER);
			final int contactTypeColumnIndex = phone.getColumnIndex(Phone.TYPE);

			while (!phone.isAfterLast()) {
				final String number = phone.getString(contactNumberColumnIndex);
				final int type = phone.getInt(contactTypeColumnIndex);
				String customLabel = "Custom";
				CharSequence phoneType = 
                                   ContactsContract.CommonDataKinds.Phone.getTypeLabel(
						context.getResources(), type, customLabel);
				contact.addNumber(number, phoneType.toString());
				phone.moveToNext();
			}

		}
		phone.close();
	}

	public void fetchContactEmails(Cursor cursor, Contact contact) {
		// Get email
		final String[] emailProjection = new String[] { Email.DATA, Email.TYPE };

		Cursor email = new CursorLoader(context, Email.CONTENT_URI, emailProjection,
				RawContacts.CONTACT_ID + "= ?", 
                                new String[] { String.valueOf(contact.id) },
                                null).loadInBackground();

		if (email.moveToFirst()) {
			final int contactEmailColumnIndex = email.getColumnIndex(Email.DATA);
			final int contactTypeColumnIndex = email.getColumnIndex(Email.TYPE);

			while (!email.isAfterLast()) {
				final String address = email.getString(contactEmailColumnIndex);
				final int type = email.getInt(contactTypeColumnIndex);
				String customLabel = "Custom";
				CharSequence emailType = 
                                    ContactsContract.CommonDataKinds.Email.getTypeLabel(
						context.getResources(), type, customLabel);
				contact.addEmail(address, emailType.toString());
				email.moveToNext();
			}

		}

		email.close();
	}
}
```

### Activity

Now we want to populate the data into a ListView. First, the custom adapter:

```java
public class ContactsAdapter extends ArrayAdapter<Contact> {

	public ContactsAdapter(Context context, ArrayList<Contact> contacts) {
		super(context, 0, contacts);
	}

	@Override
	public View getView(int position, View convertView, ViewGroup parent) {
		// Get the data item
		Contact contact = getItem(position);
		// Check if an existing view is being reused, otherwise inflate the view
		View view = convertView;
		if (view == null) {
			LayoutInflater inflater = LayoutInflater.from(getContext());
			view = inflater.inflate(R.layout.adapter_contact_item, parent, false);
		}
		// Populate the data into the template view using the data object
		TextView tvName = (TextView) view.findViewById(R.id.tvName);
		TextView tvEmail = (TextView) view.findViewById(R.id.tvEmail);
		TextView tvPhone = (TextView) view.findViewById(R.id.tvPhone);
		tvName.setText(contact.name);
		if (contact.emails.size() > 0 && contact.emails.get(0) != null) {
			tvEmail.setText(contact.emails.get(0).address);
		}
		if (contact.numbers.size() > 0 && contact.numbers.get(0) != null) {
			tvPhone.setText(contact.numbers.get(0).number);
		}
		return view;
	}

}
```

and creating the adapter item xml:

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:padding="3dp"
    android:layout_height="match_parent" >

    <TextView
        android:id="@+id/tvName"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentLeft="true"
        android:layout_alignParentTop="true"
        android:text="Bob Marley"
        android:textAppearance="?android:attr/textAppearanceLarge"
        android:textStyle="bold" />

    <TextView
        android:id="@+id/tvPhone"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentLeft="true"
        android:layout_below="@+id/tvName"
        android:text="(567) 789-5889"
        android:textAppearance="?android:attr/textAppearanceMedium" />

    <TextView
        android:id="@+id/tvEmail"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textColor="#575757"
        android:layout_alignBaseline="@+id/tvPhone"
        android:layout_alignBottom="@+id/tvPhone"
        android:layout_alignParentRight="true"
        android:text="bob@fake.com"
        android:textAppearance="?android:attr/textAppearanceMedium" />

</RelativeLayout>
```

and then hooking up the ListView in the activity:

```java
public class MainActivity extends Activity {
	private ArrayList<Contact> listContacts;
	private ListView lvContacts;

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		listContacts = new ContactFetcher(this).fetchAll();
		lvContacts = (ListView) findViewById(R.id.lvContacts);
		ContactsAdapter adapterContacts = new ContactsAdapter(this, listContacts);
		lvContacts.setAdapter(adapterContacts);
	}
}
```

And now we should see the list of phone contacts!

![Screen](http://i.imgur.com/4AHa6KN.png)

## Using CursorLoader and SimpleCursorAdapter

Rather than loading the content using a model and `ArrayList` which we manually construct, we can also load the data directly from the ContentProvider using a [CursorLoader](https://developer.android.com/training/load-data-background/setup-loader.html) and then plugging the result directly into a [SimpleCursorAdapter](http://developer.android.com/reference/android/widget/SimpleCursorAdapter.html). 

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
    
    // Create simple cursor adapter to connect the cursor dataset we load with a listview
    private void setupCursorAdapter() {
        // Column data from cursor to bind views from	
      	String[] uiBindFrom = { ContactsContract.Contacts.DISPLAY_NAME };
      	// View IDs which will have the respective column data inserted
        int[] uiBindTo = { R.id.tvName };
        // Create the simple cursor adapter to use for our list
        // specifying the template to inflate (item_contact),
        // Fields to bind from and to and mark the adapter as observing for changes
      	adapter = new SimpleCursorAdapter(
                  this, R.layout.item_contact,
                  null, uiBindFrom, uiBindTo,
                  CursorAdapter.FLAG_REGISTER_CONTENT_OBSERVER);
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
    	               ContactsContract.Contacts.DISPLAY_NAME };
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

    	// When the system finishes retrieving the Cursor, 
        // a call to the onLoadFinished() method takes place. 
    	@Override
    	public void onLoadFinished(Loader<Cursor> loader, Cursor cursor) {
    		// The swapCursor() method assigns the new Cursor to the adapter
    		adapter.swapCursor(cursor);  
    	}

    	// This method is triggered when the loader is being reset 
        // and the loader data is no longer available.
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
    public static final int CONTACT_LOADER_ID = 78;

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

Now we have completed the process of loading the contacts into our list using a `CursorLoader` querying the `ContactProvider` and loading the resulting `Cursor` into the `CursorAdapter`. 

## References

 * <http://code.tutsplus.com/tutorials/android-sdk_loading-data_cursorloader--mobile-5673>
 * <http://www.app-solut.com/blog/2011/03/working-with-the-contactscontract-to-query-contacts-in-android/>
 * <https://developer.android.com/training/load-data-background/setup-loader.html>
 * <http://www.compiletimeerror.com/2013/12/how-to-use-android-cursorloader.html#.VDHm9GRdUt0>
 * <http://developer.android.com/reference/android/widget/SimpleCursorAdapter.html>