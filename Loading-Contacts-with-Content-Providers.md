## Overview

The following guide walks step by step through loading contacts from the phone using Content Providers. 

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
				RawContacts.CONTACT_ID + "= ?", new String[] { String.valueOf(contact.id) },
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
				RawContacts.CONTACT_ID + "= ?", new String[] { String.valueOf(contact.id) },
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
			view = inflater.inflate(R.layout.adapter_contact_item, null);
		}
		// Populate the data into the template view using the data object
		TextView tvName = (TextView) view.findViewById(R.id.tvName);
		TextView tvEmail = (TextView) view.findViewById(R.id.tvEmail);
		TextView tvPhone = (TextView) view.findViewById(R.id.tvPhone);
		tvName.setText(contact.name);
		if (contact.emails.get(0) != null) {
			tvEmail.setText(contact.emails.get(0).address);
		}
		if (contact.numbers.get(0) != null) {
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

## References

 * <http://www.app-solut.com/blog/2011/03/working-with-the-contactscontract-to-query-contacts-in-android/>