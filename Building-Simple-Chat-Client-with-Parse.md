The following tutorial explains how to build a very simple chat application in Android using Parse backend-as-a-service. 

## 1. Parse Registration

First, we need to [sign up for a Parse account](https://www.parse.com/#signup) unless we are already registered. 

## 2. Setup Parse

Let's setup Parse into a brand new Android app following the steps below.

* Generate a new android project in your IDE (minSDK 16) and call it `SimpleChat`.
  * Name the first activity `ChatActivity`.
* Next, create an app in Parse and call it `SimpleChat`. Make note of the `Application ID` and `Client Key` values after you have done so.
* Follow the the steps mentioned under the [[setup|Building-Data-driven-Apps-with-Parse#setup]] guide to create and setup your project. 
  * Open the [Parse Quickstart Android Guide](https://www.parse.com/apps/quickstart#parse_data/mobile/android/native/existing) for updated instructions.
  * Download the latest [Parse jar files](https://parse.com/downloads/android/Parse/latest) in a zip file so we can add them to our project
  * Unzip the jars and copy them each into your `app/libs` folder (see [this to find libs folder](http://stackoverflow.com/a/28020621/313399))
  * Add the following to your `app/build.gradle`:
    
    ```gradle
    dependencies {
      compile 'com.parse.bolts:bolts-android:1.+'
      compile fileTree(dir: 'libs', include: 'Parse-*.jar')
    }
    ```
  * Make sure you have added these lines before the `<application>` tag in your `AndroidManifest.xml`.

    ```xml
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    ```

* Create a class called `ChatApplication` which extends from `android.app.Application`
  * In the application, initialize parse with your application id and client key
  * Your application class should look like this after you have performed the above steps:

    ```java
    public class ChatApplication extends Application {
        public static final String YOUR_APPLICATION_ID = "AERqqIXGvzH7Nmg45xa5T8zWRRjqT8UmbFQeeI";
        public static final String YOUR_CLIENT_KEY = "8bXPznF5eSLWq0sY9gTUrEF5BJlia7ltmLQFRh";
        @Override
        public void onCreate() {
            super.onCreate();
            Parse.enableLocalDatastore(this);
            Parse.initialize(this, YOUR_APPLICATION_ID, YOUR_CLIENT_KEY);
        }
    }
    ```

* Add the qualified `android:name` of your `Application` subclass to the `<application>` tag in your `AndroidManifest.xml`.

    ```xml
    <application
      android:name=".ChatApplication"
      android:icon="@drawable/ic_launcher"
      android:label="@string/app_name"
      ...>
          <activity 
             ... 
          />
    />
    ```

## 3. Design Messages Layout

Let's create an XML layout which allows us to post messages by typing into a text field. Open your layout file `activity_chat.xml`, add an `EditText` and a `Button` to compose and send text messages.

```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="${relativePackage}.${activityClass}" >

    <EditText
        android:id="@+id/etMessage"
        android:layout_toLeftOf="@+id/btSend"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_alignBottom="@+id/btSend"
        android:gravity="top"
        android:hint="@string/message_hint"
        android:imeOptions="actionSend"/>
      <Button
        android:id="@+id/btSend"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:gravity="center_vertical|right"
        android:paddingRight="10dp"
        android:layout_alignParentRight="true"
        android:text="@string/send"
        android:textSize="18sp" >
    </Button>

</RelativeLayout>
```

The imeOptions attribute is used to control the icon in the [Soft Keyboard](https://guides.codepath.com/android/Working-with-the-Soft-Keyboard).  The gravity attribute will center the button vertically AND right horizontally.

* Add the following values to `res-->values-->strings.xml` file.

```xml
<string name="message_hint">Say anything</string>
<string name="send">Send</string>
```

## 4. Login With Anonymous ParseUser

For the sake of simplicity, we will use an anonymous user to log into our simple chat app. An anonymous user is a user that can be created without a username and password but still has all of the same capabilities as any other ParseUser. After logging out, an anonymous user is abandoned, and its data is no longer accessible. 

Open your main activity class (`ChatActivity.java`) and make the  following changes:

```java
public class ChatActivity extends Activity {
    private static final String TAG = ChatActivity.class.getName();
    private static String sUserId;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_chat);
        // User login
        if (ParseUser.getCurrentUser() != null) { // start with existing user
            startWithCurrentUser();
        } else { // If not logged in, login as a new anonymous user
            login();
        }
    }
    
    // Get the userId from the cached currentUser object
    private void startWithCurrentUser() {
        sUserId = ParseUser.getCurrentUser().getObjectId();		
    }
    
    // Create an anonymous user using ParseAnonymousUtils and set sUserId 
    private void login() {
        ParseAnonymousUtils.logIn(new LogInCallback() {
	    @Override
	    public void done(ParseUser user, ParseException e) {
                if (e != null) {
                    Log.d(TAG, "Anonymous login failed: " + e.toString());
                } else {
                    startWithCurrentUser();
                }
            }
       });		
    }
}
```

## 5. Save Messages

Next, we will setup UI views in `ChatActivity.java`. On click of 'Send' button, we'll save the message object to Parse. This is done by constructing a new `ParseObject` and then calling `saveInBackground()` to persist data to the database.

```java
...

public static final String USER_ID_KEY = "userId";

private EditText etMessage;
private Button btSend;

...

// Get the userId from the cached currentUser object
private void startWithCurrentUser() {
    sUserId = ParseUser.getCurrentUser().getObjectId();  
    setupMessagePosting();
}

// Setup button event handler which posts the entered message to Parse
private void setupMessagePosting() {
        // Find the text field and button
        etMessage = (EditText) findViewById(R.id.etMessage);
        btSend = (Button) findViewById(R.id.btSend);
        // When send button is clicked, create message object on Parse
        btSend.setOnClickListener(new OnClickListener() {

            @Override
            public void onClick(View v) {
                String data = etMessage.getText().toString();
                ParseObject message = new ParseObject("Message");
                message.put(USER_ID_KEY, sUserId);
                message.put("body", data);
                message.saveInBackground(new SaveCallback() {
                    @Override
                    public void done(ParseException e) {
                    	Toast.makeText(ChatActivity.this, "Successfully created message on Parse",
                             Toast.LENGTH_SHORT).show();
                    }
                });
                etMessage.setText("");
            }
        });
}
```

## 6. Verify Save

At this point, run your application and try to send a text to parse. If the save was successful, you should see 'Successfully sent message to parse.' toast on your screen. To make sure the data was saved, you can look at the `message` class in the Data Browser of your app on Parse.

## 7. Add ListView to Chat Layout

Now that we have verified that messages are successfully being saved to your parse database, lets go ahead and build the UI to retrieve these messages. Open your layout file `activity_chat.xml`and add a `ListView` to display the text messages from parse.

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android" 
  android:orientation="vertical"
  android:background="#ffffff"
  android:layout_width="match_parent"
  android:layout_height="match_parent">
    <ListView
      android:id="@+id/lvChat"
      android:transcriptMode="alwaysScroll"
      android:layout_alignParentTop="true"
      android:layout_alignParentLeft="true"
      android:layout_alignParentRight="true"
      android:layout_above="@+id/llSend"
      android:layout_width="wrap_content" 
      android:layout_height="match_parent" />
    <RelativeLayout 
      android:id="@+id/llSend"
      android:layout_alignParentBottom="true"
      android:layout_width="match_parent"
      android:background="#ffffff"
      android:paddingTop="5dp"
      android:paddingBottom="10dp"
      android:paddingLeft="0dp"
      android:paddingRight="0dp"
      android:layout_height="wrap_content" >
      <EditText
          android:id="@+id/etMessage"
          android:layout_toLeftOf="@+id/btSend"
          android:layout_alignBottom="@+id/btSend"
          android:layout_width="match_parent"
          android:layout_height="wrap_content"
          android:gravity="top"
          android:hint="@string/message_hint"
          android:inputType="textShortMessage"
          android:imeOptions="actionSend"
        />
        <Button
          android:id="@+id/btSend"
          android:layout_width="wrap_content"
          android:layout_height="wrap_content"
          android:gravity="center_vertical|right"
          android:paddingRight="10dp"
          android:layout_alignParentRight="true"
          android:text="@string/send"
          android:textSize="18sp" >
        </Button>
    </RelativeLayout>
</RelativeLayout>
```

We will be showing the logged in user's gravatar and messages on the right and the other gravatars and messages on the left. You can read more about creating gravatars [here](https://en.gravatar.com/site/implement/images/). We need to create another layout file to represent each chat message row in the list view. Put this into `res/layout/chat_item.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="horizontal" >
    <ImageView
        android:id="@+id/ivProfileLeft"
        android:contentDescription="@string/profile_other"
        android:layout_width="64dp"
        android:layout_height="64dp"
        android:src="@drawable/ic_launcher" />
    <TextView
        android:textSize="18sp"
        android:id="@+id/tvBody"
        android:padding="20dp"
        android:lines="3"
        android:layout_marginRight="64dp"
        android:layout_width="match_parent"
        android:layout_height="64dp">
    </TextView>
    <ImageView
        android:id="@+id/ivProfileRight"
        android:contentDescription="@string/profile_me"
        android:layout_marginLeft="-64dp"
        android:layout_width="64dp"
        android:layout_height="64dp"
        android:src="@drawable/ic_launcher" />
</LinearLayout>
```

Add the following values to `res-->values-->strings.xml` file:

```xml
<string name="profile_me">My Profile Pic</string>
<string name="profile_other">Other profile pic</string>
```

## 8. Create Model Class

Now let's create a `Message.java` class which will extend from `ParseObject`. This model class will provide message data for the `ListView` and will be used to retrieve and save messages to Parse.

```java
@ParseClassName("Message")
public class Message extends ParseObject {
    public String getUserId() {
    	return getString("userId");
    }
    
    public String getBody() {
    	return getString("body");
    }
    
    public void setUserId(String userId) {
        put("userId", userId);	
    }
    
    public void setBody(String body) {
    	put("body", body);
    }
}
```

We also need to make sure to register this class with Parse before we call Parse.initialize within the `ChatApplication.java` file:

```java
public class ChatApplication extends Application {
	// ...
	public void onCreate() {
		super.onCreate();
		// Register your parse models here
		ParseObject.registerSubclass(Message.class);
		// Existing initialization happens after all classes are registered
		Parse.initialize(this, YOUR_APPLICATION_ID, YOUR_CLIENT_KEY);
	}
}
```

With our model defined with Parse and properly registered, we can now use this model to store and retrieve message data.

## 9. Create Custom List Adapter

Create a class named `ChatListAdapter.java` with below code. This is a custom list adapter class which provides data to list view. In other words it renders the layout_row.xml in list by pre-filling appropriate information. We'll be using the open source `Picasso`library to load profile images. Add dependency for this library to the `app/build.gradle` file.

```groovy
...
dependencies {
    compile fileTree(dir: 'libs', include: '*.jar')
    compile 'com.android.support:appcompat-v7:22.2.0+'
    compile 'com.squareup.picasso:picasso:2.5.0'
}
```

```java
public class ChatListAdapter extends ArrayAdapter<Message> {	
	private String mUserId;
	
	public ChatListAdapter(Context context, String userId, List<Message> messages) {
	        super(context, 0, messages);
	        this.mUserId = userId;
	}

	@Override
	public View getView(int position, View convertView, ViewGroup parent) {
		if (convertView == null) {
			convertView = LayoutInflater.from(getContext()).
			    inflate(R.layout.chat_item, parent, false);
			final ViewHolder holder = new ViewHolder();
			holder.imageLeft = (ImageView)convertView.findViewById(R.id.ivProfileLeft);
			holder.imageRight = (ImageView)convertView.findViewById(R.id.ivProfileRight);
			holder.body = (TextView)convertView.findViewById(R.id.tvBody);
			convertView.setTag(holder);
		}
		final Message message = (Message)getItem(position);
		final ViewHolder holder = (ViewHolder)convertView.getTag();
		final boolean isMe = message.getUserId().equals(mUserId);
		// Show-hide image based on the logged-in user. 
                // Display the profile image to the right for our user, left for other users.
		if (isMe) {
			holder.imageRight.setVisibility(View.VISIBLE);
			holder.imageLeft.setVisibility(View.GONE);
			holder.body.setGravity(Gravity.CENTER_VERTICAL | Gravity.RIGHT);
		} else {
			holder.imageLeft.setVisibility(View.VISIBLE);
			holder.imageRight.setVisibility(View.GONE);
			holder.body.setGravity(Gravity.CENTER_VERTICAL | Gravity.LEFT);
		}
		final ImageView profileView = isMe ? holder.imageRight : holder.imageLeft;
		Picasso.with(getContext()).load(getProfileUrl(message.getUserId())).into(profileView);
		holder.body.setText(message.getBody());
		return convertView;
	}
	
	// Create a gravatar image based on the hash value obtained from userId
	private static String getProfileUrl(final String userId) {
		String hex = "";
		try {
			final MessageDigest digest = MessageDigest.getInstance("MD5");
			final byte[] hash = digest.digest(userId.getBytes());
			final BigInteger bigInt = new BigInteger(hash);
			hex = bigInt.abs().toString(16);
		} catch (Exception e) {
			e.printStackTrace();
		}
		return "http://www.gravatar.com/avatar/" + hex + "?d=identicon";
	}
	
	final class ViewHolder {
		public ImageView imageLeft;
		public ImageView imageRight;
		public TextView body;
	}

}
```

## 10. Bind Adapter to the ListView

Next, we will setup the ListView and bind our custom adapter to this ListView within the `ChatActivity.java` source file:

```java
...
...

private ListView lvChat;
private ArrayList<Message> mMessages;
private ChatListAdapter mAdapter;

...
	
// Setup message field and posting
private void setupMessagePosting() {
    	etMessage = (EditText) findViewById(R.id.etMessage);
    	btSend = (Button) findViewById(R.id.btSend);
    	lvChat = (ListView) findViewById(R.id.lvChat);
    	mMessages = new ArrayList<Message>();
    	mAdapter = new ChatListAdapter(ChatActivity.this, sUserId, mMessages);
    	lvChat.setAdapter(mAdapter);
    	btSend.setOnClickListener(new OnClickListener() {

			@Override
			public void onClick(View v) {
				String body = etMessage.getText().toString();
				// Use Message model to create new messages now      
				Message message = new Message();
				message.setUserId(sUserId);
				message.setBody(body);
				message.saveInBackground(new SaveCallback() {
					@Override
					public void done(ParseException e) {
						receiveMessage();
					}
				});
				etMessage.setText("");
			}
	});
}
```

## 11. Receive Messages

Now we can fetch last 50 messages from parse and bind them to the ListView with the use of our custom messages adapter within `ChatActivity.java`:

```java
...

private static final int MAX_CHAT_MESSAGES_TO_SHOW = 50;

...

// Query messages from Parse so we can load them into the chat adapter
private void receiveMessage() {
                // Construct query to execute
		ParseQuery<Message> query = ParseQuery.getQuery(Message.class);
                // Configure limit and sort order
		query.setLimit(MAX_CHAT_MESSAGES_TO_SHOW);
		query.orderByAscending("createdAt");
		// Execute query to fetch all messages from Parse asynchronously
                // This is equivalent to a SELECT query with SQL
		query.findInBackground(new FindCallback<Message>() {
			public void done(List<Message> messages, ParseException e) {
				if (e == null) {					
					mMessages.clear();
					mMessages.addAll(messages);
					mAdapter.notifyDataSetChanged(); // update adapter
					lvChat.invalidate(); // redraw listview
				} else {
					Log.d("message", "Error: " + e.getMessage());
				}
			}
		});
}
```

Now, we should be able to see the messages in the list after posting but we won't yet see them update on load or as new messages are created on other clients.

## 12. Refreshing Messages

Finally, let's periodically refresh the ListView with latest messages [using a handler](http://guides.codepath.com/android/Repeating-Periodic-Tasks#handler). The handler will call a runnable to fetch new messages every 100ms. This is a primitive "polling" rather than "push" technique for loading new messages but will work for the purposes of this simple project.

```java
...

// Create a handler which can run code periodically
private Handler handler = new Handler();

...

@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_chat);
    if (ParseUser.getCurrentUser() != null) {
        startWithCurrentUser();
    } else {
    	login();
    }
    // Run the runnable object defined every 100ms
    handler.postDelayed(runnable, 100);
}

// Defines a runnable which is run every 100ms
private Runnable runnable = new Runnable() {
    @Override
    public void run() {
       refreshMessages();
       handler.postDelayed(this, 100);
    }
};

private void refreshMessages() {
    receiveMessage();		
}
```

See the [repeating periodic tasks](http://guides.codepath.com/android/Repeating-Periodic-Tasks#handler) guide to learn more about the handler.

## 13. SimpleChatManifest.xml

The final manifest for this chat application looks like:

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.codepath.simplechat"
    android:versionCode="1"
    android:versionName="1.0" >
    
    <uses-permission android:name="android.permission.INTERNET" />
	<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />

    <application
        android:name="com.codepath.simplechat.ChatApplication"
        android:allowBackup="true"
        android:icon="@drawable/ic_launcher"
        android:label="@string/app_name"
        android:theme="@style/AppTheme" >
        <activity
            android:name=".ChatActivity"
            android:label="@string/app_name" >
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

</manifest>
```

## 14. Final Output

Run your project and test it out with your pair partner. Below is the final output.

![Chat App|250](http://i.imgur.com/3UZ6ZNy.png)