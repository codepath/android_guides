The following tutorial explains how to build a very simple chat application in Android using Parse backend-as-a-service. 

>**Note:** This chat application is by no means a fully-featured or production ready chat function. This tutorial is an illustration of how to quickly build an app using [Parse](https://www.parse.com).

## 1. Setup Parse server

Parse is shutting down in a year on **January 28, 2017**.  New accounts are no longer being accepted, and existing apps are being requested to migrate to their own backend by **April 28, 2016**. 

### Parse on Heroku

We can deploy our own Parse data store and push notifications systems to [Heroku](https://www.heroku.com/) leveraging the [server open-sourced by Parse](https://github.com/ParsePlatform/parse-server-example). Parse is built on top of the MongoDB database which can be added to Heroku using MongoLab. 

To follow this guide we need to [[setup our own Parse server|configuring a Parse Server]]. Once the Parse server is configured, we can [initialize Parse within our Android app](http://guides.codepath.com/android/Configuring-a-Parse-Server#enabling-client-sdk-integration) pointing the client to our self-hosted URL. After that, the tutorial works the same as before. 

## 2. Setup Parse client

Let's setup Parse into a brand new Android app following the steps below.

* Generate a new android project in your IDE (minSDK 18) and call it `SimpleChat`.
  * Name the first activity `ChatActivity`.  
  * Add the following to your `app/build.gradle`:
    
    ```gradle
    dependencies {
      compile 'com.parse:parse-android:1.13.1'
      compile 'com.parse:parseinterceptors:0.0.2' // for logging API calls to LogCat
    }
    ```
  * Make sure you have added these lines before the `<application>` tag in your `AndroidManifest.xml`.

    ```xml
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    ```

* Create a class called `ChatApplication` which extends from `android.app.Application`
  * In the application, initialize parse

    ```java
    public class ChatApplication extends Application {
        @Override
        public void onCreate() {
            super.onCreate();
            // set applicationId and server based on the values in the Heroku settings.
            // any network interceptors must be added with the Configuration Builder given this syntax
            Parse.initialize(new Parse.Configuration.Builder(this)
                 .applicationId("YOUR_APPLICATION_ID") // should correspond to APP_ID env variable
                 .addNetworkInterceptor(new ParseLogInterceptor())
                 .server("https://myparseapp.herokuapp.com/parse/").build());
        }
    }
    ```

* Add the qualified `android:name` of your `Application` subclass to the `<application>` tag in your `AndroidManifest.xml`.

    ```xml
    <application
      android:name=".ChatApplication"
      android:icon="@mipmap/ic_launcher"
      android:label="@string/app_name"
      ...>
          <activity 
             ... 
          />
    />
    ```

* Add your `YOUR_APPLICATION_ID` to the `<meta-data>` tag inside your `<application>` in your `AndroidManifest.xml`:

    ```xml
    <application
        ...>
        <!-- activities and everything here. meta-data last inside application tag -->
        <meta-data
            android:name="com.parse.APPLICATION_ID"
            android:value="{YOUR_APPLICATION_ID_HERE}" />
    />
    ```

**WARNING:** Be sure to **add the application name** above after creating the custom Application class or the following code won't work!! Also be **sure to add the application id** inside the manifest as wel. 

## 3. Design Messages Layout

Let's create an XML layout which allows us to post messages by typing into a text field. Open your layout file `activity_chat.xml`, add an `EditText` and a `Button` to compose and send text messages.

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent" >

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

The imeOptions attribute is used to control the icon in the [[Soft Keyboard|Working-with-the-Soft-Keyboard]].  The gravity attribute will center the button vertically AND right horizontally.

* Add the following values to `res-->values-->strings.xml` file.

```xml
<string name="message_hint">Say anything</string>
<string name="send">Send</string>
```

## 4. Login With Anonymous ParseUser

For the sake of simplicity, we will use an anonymous user to log into our simple chat app. An anonymous user is a user that can be created without a username and password but still has all of the same capabilities as any other ParseUser. After logging out, an anonymous user is abandoned, and its data is no longer accessible. 

Open your main activity class (`ChatActivity.java`) and make the  following changes:

```java
public class ChatActivity extends AppCompatActivity {
    static final String TAG = ChatActivity.class.getSimpleName();

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
    void startWithCurrentUser() {
        // TODO:
    }
    
    // Create an anonymous user using ParseAnonymousUtils and set sUserId 
    void login() {
        ParseAnonymousUtils.logIn(new LogInCallback() {
	    @Override
	    public void done(ParseUser user, ParseException e) {
                if (e != null) {
                    Log.e(TAG, "Anonymous login failed: ", e);
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
public class ChatActivity extends AppCompatActivity {
...
    static final String USER_ID_KEY = "userId";
    static final String BODY_KEY = "body";

    EditText etMessage;
    Button btSend;

...

    // Get the userId from the cached currentUser object
    void startWithCurrentUser() {
        setupMessagePosting();
    }

    // Setup button event handler which posts the entered message to Parse
    void setupMessagePosting() {
        // Find the text field and button
        etMessage = (EditText) findViewById(R.id.etMessage);
        btSend = (Button) findViewById(R.id.btSend);
        // When send button is clicked, create message object on Parse
        btSend.setOnClickListener(new OnClickListener() {
            @Override
            public void onClick(View v) {
                String data = etMessage.getText().toString();
                ParseObject message = ParseObject.create("Message");
                message.put(USER_ID_KEY, ParseUser.getCurrentUser().getObjectId());
                message.put(BODY_KEY, data);
                message.saveInBackground(new SaveCallback() {
                    @Override
                    public void done(ParseException e) {
                        if(e == null) {
                    	    Toast.makeText(ChatActivity.this, "Successfully created message on Parse",
                             Toast.LENGTH_SHORT).show();
                        } else {
                            Log.e(TAG, "Failed to save message", e);
                        }
                    }
                });
                etMessage.setText(null);
            }
        });
    }
}
```

## 6. Verify Save

At this point, run your application and try to send a text to parse. If the save was successful, you should see 'Successfully sent message to parse.' toast on your screen. To make sure the data was saved, you can verify whether the objects were created by clicking on the MongoDB instance in the Heroku panel. Refer [testing parse deployment guide](http://guides.codepath.com/android/Configuring-a-Parse-Server#testing-deployment) for more info.

## 7. Add ListView to Chat Layout

Now that we have verified that messages are successfully being saved to your parse database, lets go ahead and build the UI to retrieve these messages. Open your layout file `activity_chat.xml`and add a `ListView` to display the text messages from parse.

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android" 
  android:orientation="vertical"
  android:background="@android:color/white"
  android:layout_width="match_parent"
  android:layout_height="match_parent">
    <ListView
      android:id="@+id/lvChat"
      android:transcriptMode="alwaysScroll"
      android:layout_alignParentTop="true"
      android:layout_alignParentLeft="true"
      android:layout_alignParentRight="true"
      android:layout_above="@+id/rlSend"
      android:layout_width="wrap_content" 
      android:layout_height="match_parent" />
    <RelativeLayout 
      android:id="@+id/rlSend"
      android:layout_alignParentBottom="true"
      android:layout_width="match_parent"
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
        android:id="@+id/ivProfileOther"
        android:contentDescription="@string/profile_other"
        android:layout_width="64dp"
        android:layout_height="64dp"
        android:src="@mipmap/ic_launcher" />
    <TextView
        android:textSize="18sp"
        android:id="@+id/tvBody"
        android:padding="20dp"
        android:lines="3"
        android:layout_marginEnd="64dp"
        android:layout_width="match_parent"
        android:layout_height="64dp">
    </TextView>
    <ImageView
        android:id="@+id/ivProfileMe"
        android:contentDescription="@string/profile_me"
        android:layout_marginStart="-64dp"
        android:layout_width="64dp"
        android:layout_height="64dp"
        android:src="@mipmap/ic_launcher" />
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
    public static final String USER_ID_KEY = "userId";
    public static final String BODY_KEY = "body";

    public String getUserId() {
        return getString(USER_ID_KEY);
    }

    public String getBody() {
        return getString(BODY_KEY);
    }

    public void setUserId(String userId) {
        put(USER_ID_KEY, userId);
    }

    public void setBody(String body) {
        put(BODY_KEY, body);
    }
}}
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
	
                // For open-source Parse backend
                Parse.initialize(new Parse.Configuration.Builder(this)
                     .applicationId("YOUR_APPLICATION_ID") // should correspond to APP_ID env variable
                     .addNetworkInterceptor(new ParseLogInterceptor())
                     .server("https://myparseapp.herokuapp.com/parse/").build());
	}
}
```

Finally, we refactor `ChatActivity` and rename the references to the model keys

```java
...
void setupMessagePosting() {
        // Find the text field and button
        etMessage = (EditText) findViewById(R.id.etMessage);
        btSend = (Button) findViewById(R.id.btSend);
        // When send button is clicked, create message object on Parse
        btSend.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                String data = etMessage.getText().toString();
                //ParseObject message = ParseObject.create("Message");
                //message.put(Message.USER_ID_KEY, ParseUser.getCurrentUser().getObjectId());
                //message.put(Message.BODY_KEY, data);
                // Using new `Message` Parse-backed model now
                Message message = new Message();
                message.setBody(data);
                message.setUserId(ParseUser.getCurrentUser().getObjectId());
                message.saveInBackground(new SaveCallback() {
                    @Override
                    public void done(ParseException e) {
                        if(e == null) {
                    	    Toast.makeText(ChatActivity.this, "Successfully created message on Parse",
                             Toast.LENGTH_SHORT).show();
                        } else {
                            Log.e(TAG, "Failed to save message", e);
                        }
                    }
                });
                etMessage.setText(null);
            }
        });
    }
...
```

With our model defined with Parse and properly registered, we can now use this model to store and retrieve message data.

## 9. Create Custom List Adapter

Create a class named `ChatListAdapter.java` with below code. This is a custom list adapter class which provides data to list view. In other words it renders the layout_row.xml in list by pre-filling appropriate information. We'll be using the open source `Picasso`library to load profile images. Add dependency for this library to the `app/build.gradle` file.

```groovy
...
dependencies {
    compile fileTree(dir: 'libs', include: '*.jar')
    compile 'com.android.support:appcompat-v7:23.4.0'
    compile 'com.squareup.picasso:picasso:2.5.2'
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
            holder.imageOther = (ImageView)convertView.findViewById(R.id.ivProfileOther);
            holder.imageMe = (ImageView)convertView.findViewById(R.id.ivProfileMe);
            holder.body = (TextView)convertView.findViewById(R.id.tvBody);
            convertView.setTag(holder);
        }
        final Message message = getItem(position);
        final ViewHolder holder = (ViewHolder)convertView.getTag();
        final boolean isMe = message.getUserId() != null && message.getUserId().equals(mUserId);
        // Show-hide image based on the logged-in user.
        // Display the profile image to the right for our user, left for other users.
        if (isMe) {
            holder.imageMe.setVisibility(View.VISIBLE);
            holder.imageOther.setVisibility(View.GONE);
            holder.body.setGravity(Gravity.CENTER_VERTICAL | Gravity.RIGHT);
        } else {
            holder.imageOther.setVisibility(View.VISIBLE);
            holder.imageMe.setVisibility(View.GONE);
            holder.body.setGravity(Gravity.CENTER_VERTICAL | Gravity.LEFT);
        }
        final ImageView profileView = isMe ? holder.imageMe : holder.imageOther;
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
        public ImageView imageOther;
        public ImageView imageMe;
        public TextView body;
    }
}
```

## 10. Bind Adapter to the ListView

Next, we will setup the ListView and bind our custom adapter to this ListView within the `ChatActivity.java` source file:

```java
public class ChatActivity extends AppCompatActivity {
...
    ListView lvChat;
    ArrayList<Message> mMessages;
    ChatListAdapter mAdapter;
    // Keep track of initial load to scroll to the bottom of the ListView
    boolean mFirstLoad;
	
    // Setup message field and posting
    void setupMessagePosting() {
        // Find the text field and button
        etMessage = (EditText) findViewById(R.id.etMessage);
        btSend = (Button) findViewById(R.id.btSend);
        lvChat = (ListView) findViewById(R.id.lvChat);
        mMessages = new ArrayList<>();
        // Automatically scroll to the bottom when a data set change notification is received and only if the last item is already visible on screen. Don't scroll to the bottom otherwise.
        lvChat.setTranscriptMode(1);
        mFirstLoad = true;
        final String userId = ParseUser.getCurrentUser().getObjectId();
        mAdapter = new ChatListAdapter(ChatActivity.this, userId, mMessages);
        lvChat.setAdapter(mAdapter);
        // When send button is clicked, create message object on Parse
        btSend.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                String data = etMessage.getText().toString();
                //ParseObject message = ParseObject.create("Message");
                //message.put(Message.USER_ID_KEY, userId);
                //message.put(Message.BODY_KEY, data);
                // Using new `Message` Parse-backed model now
                Message message = new Message();
                message.setBody(data);
                message.setUserId(ParseUser.getCurrentUser().getObjectId());
                message.saveInBackground(new SaveCallback() {
                    @Override
                    public void done(ParseException e) {
                        Toast.makeText(ChatActivity.this, "Successfully created message on Parse",
                                Toast.LENGTH_SHORT).show();
                        refreshMessages();
                    }
                });
                etMessage.setText(null);
            }
        });
    }

    // Query messages from Parse so we can load them into the chat adapter
    void refreshMessages() {
        // TODO:
    }
    ...
}
```

## 11. Receive Messages

Now we can fetch last 500 messages from parse and bind them to the ListView with the use of our custom messages adapter within `ChatActivity.java`:

```java
public class ChatActivity extends AppCompatActivity {
...
    static final int MAX_CHAT_MESSAGES_TO_SHOW = 50;
...
    // Query messages from Parse so we can load them into the chat adapter
    void refreshMessages() {
        // Construct query to execute
        ParseQuery<Message> query = ParseQuery.getQuery(Message.class);
        // Configure limit and sort order
        query.setLimit(MAX_CHAT_MESSAGES_TO_SHOW);
        query.orderByDescending("createdAt");
        // Execute query to fetch all messages from Parse asynchronously
        // This is equivalent to a SELECT query with SQL
        query.findInBackground(new FindCallback<Message>() {
            public void done(List<Message> messages, ParseException e) {
                if (e == null) {
                    mMessages.clear();
                    Collections.reverse(messages);
                    mMessages.addAll(messages);
                    mAdapter.notifyDataSetChanged(); // update adapter
                    // Scroll to the bottom of the list on initial load
                    if (mFirstLoad) {
                        lvChat.setSelection(mAdapter.getCount() - 1);
                        mFirstLoad = false;
                    }
                } else {
                    Log.e("message", "Error Loading Messages" + e);
                }
            }
        });
    }
}
```

Now, we should be able to see the messages in the list after posting but we won't yet see them update on load or as new messages are created on other clients.

## 12. Refreshing Messages

Finally, let's periodically refresh the ListView with latest messages [[using a handler|Repeating-Periodic-Tasks#handler]]. The handler will call a runnable to fetch new messages every 1 second. This is a primitive "polling" rather than the more efficient "push" technique for refreshing new messages - but will work for the purposes of this simple project.

```java
...

// Create a handler which can run code periodically
static final int POLL_INTERVAL = 1000; // milliseconds
Handler mHandler = new Handler();  // android.os.Handler
Runnable mRefreshMessagesRunnable = new Runnable() {
    @Override
    public void run() {
       refreshMessages();
       mHandler.postDelayed(this, POLL_INTERVAL);
    }
};

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
    mHandler.postDelayed(mRefreshMessagesRunnable, POLL_INTERVAL);
}
```

See the [[repeating periodic tasks|Repeating-Periodic-Tasks#handler]] guide to learn more about the handler.

## 13. Final AndroidManifest.xml

The final manifest for this chat application looks like:

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.codepath.android.simplechat">

    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />

    <application
        android:name="com.codepath.android.simplechat.ChatApplication"
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
        <activity android:name="com.codepath.android.simplechat.ChatActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
        <meta-data
            android:name="com.parse.APPLICATION_ID"
            android:value="{APPLICATION_ID}" />
    </application>
</manifest>
```

## 14. Final Output

Run your project and test it out with your pair partner. Below is the final output.

![Chat App|250](https://i.imgur.com/3UZ6ZNy.png)