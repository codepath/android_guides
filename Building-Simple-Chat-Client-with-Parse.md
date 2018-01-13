The following tutorial explains how to build a very simple chat application in Android using Parse backend-as-a-service. 

>**Note:** This chat application is by no means a fully-featured or production ready chat app. This tutorial is an illustration of how to quickly build an app using [Parse](http://parseplatform.org/).

## 1. Setup Parse server

We can deploy our own Parse data store and push notifications systems to [Heroku](https://www.heroku.com/) leveraging the [server open-sourced by Parse](https://github.com/ParsePlatform/parse-server-example). Parse is built on top of the MongoDB database which can be added to Heroku using MongoLab. 

To follow this guide we need to [[setup our own Parse server|configuring a Parse Server]]. Once the Parse server is configured, we can [initialize Parse within our Android app](http://guides.codepath.com/android/Configuring-a-Parse-Server#enabling-client-sdk-integration) pointing the client to our self-hosted URL. After that, the tutorial works the same as before. 

## 2. Setup Parse client

Let's setup Parse into a brand new Android app following the steps below.

* Generate a new android project in your IDE (minSDK 18) and call it `SimpleChat`.
  * Name the first activity `ChatActivity`.  
  * Add the following to your `app/build.gradle`:
    
    ```gradle
    dependencies {
      compile 'com.parse:parse-android:1.16.3'
      compile 'com.squareup.okhttp3:logging-interceptor:3.8.1' // for logging API calls to LogCat
    }
    ```

    When you sync your project after adding these dependencies then you will likely have a failure that complains about "support-annotations".  If you run into this error then you need to make one more Gradle change.  Open your main project `build.gradle` file (not your app's `build.gradle` but your project's `build.gradle`).  Add the following "maven" part under "allprojects" > "repositories":

    ```gradle
    allprojects {
        repositories {
            jcenter()
            // The next part if what you don't want to add Google Play Service SDK manually
            google()
        }
    }
    ```

  * Make sure you have added these lines before the `<application>` tag in your `AndroidManifest.xml`.

    ```xml
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    ```

* Create [an application class](http://guides.codepath.com/android/Understanding-the-Android-Application-Class) called `ChatApplication` which extends from `android.app.Application`
  * In the [Android application class](http://guides.codepath.com/android/Understanding-the-Android-Application-Class), initialize Parse as shown below:

    ```java
    public class ChatApplication extends Application {
        @Override
        public void onCreate() {
            super.onCreate();

           // Use for monitoring Parse network traffic        
            OkHttpClient.Builder builder = new OkHttpClient.Builder();
            HttpLoggingInterceptor httpLoggingInterceptor = new HttpLoggingInterceptor();
            // Can be Level.BASIC, Level.HEADERS, or Level.BODY
            httpLoggingInterceptor.setLevel(HttpLoggingInterceptor.Level.BODY);
            builder.networkInterceptors().add(httpLoggingInterceptor);

            // set applicationId and server based on the values in the Heroku settings.
            // any network interceptors must be added with the Configuration Builder given this syntax
            Parse.initialize(new Parse.Configuration.Builder(this)
                 .applicationId("YOUR_APPLICATION_ID") // should correspond to APP_ID env variable
                 .clientBuilder(builder)
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

**WARNING:** Be sure to **add the application name** above after creating the custom Application class or the following code won't work!!

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
        android:gravity="center"
        android:paddingRight="10dp"
        android:layout_alignParentRight="true"
        android:text="@string/send"
        android:textSize="18sp" />

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

## 7. Add RecyclerView to Chat Layout

Now that we have verified that messages are successfully being saved to your parse database, lets go ahead and build the UI to retrieve these messages. Open your layout file `activity_chat.xml` and add a `RecyclerView` to display the text messages from parse.

First, add the RecyclerView as a dependency in your `app/build.gradle`:
```gradle
dependencies {
    ...
    compile 'com.android.support:recyclerview-v7:25.3.1'
}
```

Next, add your `RecyclerView` to the layout:
```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android" 
  android:background="@android:color/white"
  android:layout_width="match_parent"
  android:layout_height="match_parent">
    <android.support.v7.widget.RecyclerView
      android:id="@+id/rvChat"
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
          android:gravity="center"
          android:paddingRight="10dp"
          android:layout_alignParentRight="true"
          android:text="@string/send"
          android:textSize="18sp" >
        </Button>
    </RelativeLayout>
</RelativeLayout>
```

We will be showing the logged in user's gravatar and messages on the right and the other gravatars and messages on the left. You can read more about creating gravatars [here](https://en.gravatar.com/site/implement/images/). We need to create another layout file to represent each chat message row in the list view. Put this into `res/layout/item_chat.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
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
        android:lines="1"
        android:ellipsize="end"
        android:layout_marginEnd="64dp"
        android:layout_width="0dp"
        android:layout_height="64dp"
        android:layout_weight="1">
    </TextView>
    <ImageView
        android:id="@+id/ivProfileMe"
        android:contentDescription="@string/profile_me"
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

Now let's create a `Message.java` class which will extend from `ParseObject`. This model class will provide message data for the `RecyclerView` and will be used to retrieve and save messages to Parse.

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
}
```

We also need to make sure to register this class with Parse before we call Parse.initialize within the `ChatApplication.java` file:

```java
public class ChatApplication extends Application {
    // ...
    @Override
    public void onCreate() {
        super.onCreate();
        // ...
        ParseObject.registerSubclass(Message.class);

        // Use for monitoring Parse OkHttp trafic
        // Can be Level.BASIC, Level.HEADERS, or Level.BODY
        // See http://square.github.io/okhttp/3.x/logging-interceptor/ to see the options.
        OkHttpClient.Builder builder = new OkHttpClient.Builder();
        HttpLoggingInterceptor httpLoggingInterceptor = new HttpLoggingInterceptor();
        // ...

    }
    // ...
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

                /*** START OF CHANGE **/

                // Using new `Message` Parse-backed model now
                Message message = new Message();
                message.setBody(data);
                message.setUserId(ParseUser.getCurrentUser().getObjectId());

                /*** END OF CHANGE **/

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

Create a class named `ChatAdapter.java` with below code. This is a custom list adapter class which provides data to list view. In other words it renders the `item_chat.xml `in list by pre-filling appropriate information. We'll be using the open source `Glide` library to load profile images. 

First, add dependency for this library to the `app/build.gradle` file:

```groovy
...
dependencies {
    compile 'com.github.bumptech.glide:glide:3.8.0'
}
```

Next, add the `ChatAdapter`:

```java
public class ChatAdapter extends RecyclerView.Adapter<ChatAdapter.ViewHolder> {
    private List<Message> mMessages;
    private Context mContext;
    private String mUserId;

    public ChatAdapter(Context context, String userId, List<Message> messages) {
        mMessages = messages;
        this.mUserId = userId;
        mContext = context;
    }

    @Override
    public ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        Context context = parent.getContext();
        LayoutInflater inflater = LayoutInflater.from(context);
        View contactView = inflater.inflate(R.layout.item_chat, parent, false);

        ViewHolder viewHolder = new ViewHolder(contactView);
        return viewHolder;
    }

    @Override
    public void onBindViewHolder(ViewHolder holder, int position) {
        Message message = mMessages.get(position);
        final boolean isMe = message.getUserId() != null && message.getUserId().equals(mUserId);

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
        Glide.with(mContext).load(getProfileUrl(message.getUserId())).into(profileView);
        holder.body.setText(message.getBody());
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

    @Override
    public int getItemCount() {
        return mMessages.size();
    }

    public class ViewHolder extends RecyclerView.ViewHolder {
        ImageView imageOther;
        ImageView imageMe;
        TextView body;

        public ViewHolder(View itemView) {
            super(itemView);
            imageOther = (ImageView)itemView.findViewById(R.id.ivProfileOther);
            imageMe = (ImageView)itemView.findViewById(R.id.ivProfileMe);
            body = (TextView)itemView.findViewById(R.id.tvBody);
        }
    }
}
```

## 10. Bind Adapter to the RecyclerView

Next, we will setup the ReyclerView and bind our custom adapter to this ReyclerView within the `ChatActivity.java` source file:

```java
public class ChatActivity extends AppCompatActivity {
...

    RecyclerView rvChat;    
    ArrayList<Message> mMessages;
    ChatAdapter mAdapter;
    // Keep track of initial load to scroll to the bottom of the ListView
    boolean mFirstLoad;
	
    // Setup message field and posting
    void setupMessagePosting() {
        // Find the text field and button
        etMessage = (EditText) findViewById(R.id.etMessage);
        btSend = (Button) findViewById(R.id.btSend);
        rvChat = (RecyclerView) findViewById(R.id.rvChat);
        mMessages = new ArrayList<>();
        mFirstLoad = true;
        final String userId = ParseUser.getCurrentUser().getObjectId();
        mAdapter = new ChatAdapter(ChatActivity.this, userId, mMessages);
        rvChat.setAdapter(mAdapter);

        // associate the LayoutManager with the RecylcerView
        final LinearLayoutManager linearLayoutManager = new LinearLayoutManager(ChatActivity.this);
        rvChat.setLayoutManager(linearLayoutManager);

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

Now we can fetch last 50 messages from parse and bind them to the RecyclerView with the use of our custom messages adapter within `ChatActivity.java`:

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

        // get the latest 50 messages, order will show up newest to oldest of this group
        query.orderByDescending("createdAt");
        // Execute query to fetch all messages from Parse asynchronously
        // This is equivalent to a SELECT query with SQL
        query.findInBackground(new FindCallback<Message>() {
            public void done(List<Message> messages, ParseException e) {
                if (e == null) {
                    mMessages.clear();
                    mMessages.addAll(messages);
                    mAdapter.notifyDataSetChanged(); // update adapter
                    // Scroll to the bottom of the list on initial load
                    if (mFirstLoad) {
                        rvChat.scrollToPosition(0);
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

If you get to this step, you will display the newest posts ordered from newest to oldest.  You can reverse the order without necessarily doing a linear sort by setting `setReverseLayout` to be `true`.
```java
public class ChatActivity extends AppCompatActivity {

    // Get the items in the reverse order:
    void setupMessagePosting() {
        // ...
        final LinearLayoutManager linearLayoutManager = new LinearLayoutManager(ChatActivity.this);
        linearLayoutManager.setReverseLayout(true);
        // ...
    }
    // ...
}
```

Now, we should be able to see the messages in the list after posting but we won't yet see them update on load or as new messages are created on other clients.

## 12. Refreshing Messages

Finally, let's periodically refresh the RecyclerView with latest messages [[using a handler|Repeating-Periodic-Tasks#handler]]. The handler will call a runnable to fetch new messages every 1 second. This is a primitive "polling" rather than the more efficient "push" technique for refreshing new messages - but will work for the purposes of this simple project.

```java
...

// Create a handler which can run code periodically
static final int POLL_INTERVAL = 1000; // milliseconds
Handler myHandler = new Handler();  // android.os.Handler
Runnable mRefreshMessagesRunnable = new Runnable() {
    @Override
    public void run() {
       refreshMessages();
       myHandler.postDelayed(this, POLL_INTERVAL);
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
    myHandler.postDelayed(mRefreshMessagesRunnable, POLL_INTERVAL);
}
```

See the [[repeating periodic tasks|Repeating-Periodic-Tasks#handler]] guide to learn more about the handler.

## 13. Live Queries

Alternatively, assuming the server is configured properly to support it (see [[this guide|Configuring-a-Parse-Server#adding-support-for-live-queries]]), we can also use [[Parse Live Queries|Building-Data-driven-Apps-with-Parse#live-queries]] to listen for new messages.  We can disable the use of the `postDelayed()` runnable that we created in the earlier step:

```java
// myHandler.postDelayed(mRefreshMessagesRunnable, POLL_INTERVAL);
```

First, make sure to add the Parse LiveQuery dependency to your `app/build.gradle` config:

```gradle
dependencies {
      compile 'com.parse:parse-livequery-android:1.0.4' // for Parse Live Queries
}
```

Next, we will configure to listen for any newly created Message object:

```java
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);

   // Parse.initialize(...) should come first

   // Make sure the Parse server is setup to configured for live queries
   // URL for server is determined by Parse.initialize() call.
   ParseLiveQueryClient parseLiveQueryClient = ParseLiveQueryClient.Factory.getClient();

   ParseQuery<Message> parseQuery = ParseQuery.getQuery(Message.class);
   // This query can even be more granular (i.e. only refresh if the entry was added by some other user)
   // parseQuery.whereNotEqualTo(USER_ID_KEY, ParseUser.getCurrentUser().getObjectId());

   // Connect to Parse server
   SubscriptionHandling<Message> subscriptionHandling = parseLiveQueryClient.subscribe(parseQuery);

   // Listen for CREATE events
   subscriptionHandling.handleEvent(SubscriptionHandling.Event.CREATE, new 
   SubscriptionHandling.HandleEventCallback<Message>() {
                      @Override
                      public void onEvent(ParseQuery<Message> query, Message object) {
                          mMessages.add(0, object);
  
                          // RecyclerView updates need to be run on the UI thread
                          runOnUiThread(new Runnable() {
                              @Override
                              public void run() {
                                mAdapter.notifyDataSetChanged();
                                rvChat.scrollToPosition(0);
                              }
                          });
                       }
                  });
```

## 14. Final AndroidManifest.xml

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
    </application>
</manifest>
```

## 15. Final Output

Run your project and test it out with your pair partner. Below is the final output.

![Chat App|250](https://i.imgur.com/3UZ6ZNy.png)

