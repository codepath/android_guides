The following tutorial explains how to build a very simple chat application in Android using Parse backend-as-a-service. 

>**Note:** This chat application is by no means a fully-featured or production ready chat app. This tutorial is an illustration of how to quickly build an app using [Parse](http://parseplatform.org/).

## 1. Setup Parse server

We can deploy our own Parse data store and push notifications systems to [Back4App](https://www.back4app.com/) leveraging the [server open-sourced by Parse](https://github.com/ParsePlatform/parse-server-example). Parse is built on top of the MongoDB database technology and is hosted by Back4App.

To follow this guide we need to [[setup our own Parse server|configuring a Parse Server]]. Once the Parse server is configured, we can [initialize Parse within our Android app](http://guides.codepath.com/android/Configuring-a-Parse-Server#enabling-client-sdk-integration) pointing the client to our self-hosted URL. After that, the tutorial works the same as before. 

You will need to save application id & client key that you can find in Back4App Dashboard. Look for "App Settings" section in the right nav bar of Back4App Dashboard, then open Security&Keys.  

## 2. Setup Parse client

Let's setup Parse into a brand new Android app following the steps below.

* Generate a new android project in your IDE
  * Start it with an Empty Activity [**[?]**](https://i.imgur.com/nzD9pA8.png)
  * Call it `SimpleChat`
  * Select API 29 for the Minimum SDK
  * [**Rename**](https://hackmd.io/@tejen/ryejOdba_) the first activity ("MainActivity") to `ChatActivity`
  * Add this to the `allprojects` section of your root `build.gradle:`:

    ```gradle
    allprojects {
        repositories {
            google()
            jcenter()
            maven { url "https://jitpack.io" } // add this line
        }
    }
    ```

  * Add the following to your other `app/build.gradle` file:
    
    ```gradle
    dependencies {
      implementation 'com.github.parse-community.Parse-SDK-Android:parse:1.24.1'
      implementation 'com.squareup.okhttp3:logging-interceptor:3.8.1' // for logging API calls to LogCat
    }
    ```

  * Add these lines before the `<application>` tag in your `AndroidManifest.xml`:

    ```xml
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    ```

* Create [an application class](http://guides.codepath.com/android/Understanding-the-Android-Application-Class) called `ChatApplication` that extends from `android.app.Application`.
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
            // any network interceptors must be added with the Configuration Builder given this syntax
            builder.networkInterceptors().add(httpLoggingInterceptor);

            // Set applicationId and server based on the values in the Back4App settings.
            Parse.initialize(new Parse.Configuration.Builder(this)
                 .applicationId("YOUR_APPLICATION_ID") // ⚠️ TYPE IN A VALID APPLICATION ID HERE
                 .clientKey("YOUR_CLIENT_KEY") // ⚠️ TYPE IN A VALID CLIENT KEY HERE
                 .clientBuilder(builder)
                 .server("https://parseapi.back4app.com").build());
        }
    }
    ```

  * In the above code, replace "YOUR_APPLICATION_ID" and "YOUR_CLIENT_KEY" with a valid Application ID and Client Key.

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

⚠️ **Note:** Be sure to **add the `android:name`** above after creating the custom Application class or the remainder of this guide will not work.

## 3. Design Messages Layout

Let's create an XML layout which allows us to post messages by typing into a text field. Open your layout file `activity_chat.xml`, add an `EditText` and a `Button` to compose and send text messages. 

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:background="@android:color/white"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <!-- Chat messages view will go here -->

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
        <ImageButton
            android:id="@+id/btSend"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:gravity="center"
            android:paddingRight="10dp"
            android:layout_alignParentRight="true"
            android:contentDescription="@string/send"
            android:src="@drawable/ic_baseline_send_24"
            android:textSize="18sp" />
    </RelativeLayout>
</RelativeLayout>
```

The imeOptions attribute is used to control the icon in the [[Soft Keyboard|Working-with-the-Soft-Keyboard]].  The gravity attribute will position the button at the center vertically AND on the right horizontally.

Notice that we are using `ImageButton`. It allows setting a background image using `android:src` attribute. Now, let's add a new icon that represents a send message. Select File -> New -> Vector Asset. In the "Asset Studio" dialog select "Asset Type: Clip Art". Then click on a "Clip Art" button and search for "Send" asset. You can keep defaults for the rest of the settings or experiment changing colors and outline style.

Another thing to notice is a new attribute `android:contentDescription`. This attribute refers to a string that we will define in a second. Although `ImageButton` doesn't display text, this text is used for **accessibility**.

* Add the following values to `res-->values-->strings.xml` file.  

```xml
<string name="message_hint">Say anything</string>
<string name="send">Send</string>
```

## 4. Login With Anonymous Parse user

For the sake of simplicity, we will use an anonymous user to log into our simple chat app. An anonymous user is a user that can be created without a username and password but still has all of the same capabilities as any other `ParseUser`. After logging out, an anonymous user is abandoned, and its data is no longer accessible. 

Open your main activity class (`ChatActivity.java`) and make the following changes:

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
    ImageButton btSend;

...

    // Get the userId from the cached currentUser object
    void startWithCurrentUser() {
        setupMessagePosting();
    }

    // Setup button event handler which posts the entered message to Parse
    void setupMessagePosting() {
        // Find the text field and button
        etMessage = (EditText) findViewById(R.id.etMessage);
        btSend = (ImageButton) findViewById(R.id.btSend);
        
        // When send button is clicked, create message object on Parse
        btSend.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                String data = etMessage.getText().toString();
                ParseObject message = ParseObject.create("Message");
                message.put(USER_ID_KEY, ParseUser.getCurrentUser().getObjectId());
                message.put(BODY_KEY, data);
                message.saveInBackground(new SaveCallback() {
                    @Override
                    public void done(ParseException e) {
                        if (e == null) {
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

At this point, run your application and try to send a text to Parse. If the save was successful, you should see 'Successfully created message on Parse' toast on your screen. 

**WARNING** If you're using a self-deployed Back4App backend, and your own private Application ID and Client Key that you created by yourself in Back4App, then you will also need to create a new Class in your Back4App Dashboard which messages will be saved to. Navigate to your project's `Dashboard --> Core --> Database Browser` and click "Create a class". Select "Message" as a name for your class and add 2 columns:

- `userId` of type `String`
- `body` of type `String` 

Try sending message again from the app. You can then check if the data was saved by verifying whether the objects were created in Back4App Dashboard. Navigate to your project's `Dashboard --> Core --> Database Browser --> Messages` to see if the message appears there. 

Refer to the [Testing Deployment section](http://guides.codepath.com/android/Configuring-a-Parse-Server#testing-deployment) of [Configuring a Parse Server](http://guides.codepath.com/android/Configuring-a-Parse-Server) for more info.

## 7. Add RecyclerView to Chat Layout

Now that we have verified that messages are successfully being saved to your Parse database, let's go ahead and build the UI to send and retrieve these messages. Open your layout file `activity_chat.xml` and add a `RecyclerView` to display the text messages from parse.

First, add the RecyclerView as a dependency in your `app/build.gradle`:
```gradle
dependencies {
    ...
    implementation 'androidx.recyclerview:recyclerview:1.0.0'
}
```

Next, add your `RecyclerView` to the layout (scroll down to see code snippet for `ConstraintLayout`):
```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android" 
  android:background="@android:color/white"
  android:layout_width="match_parent"
  android:layout_height="match_parent">
    <androidx.recyclerview.widget.RecyclerView
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
        <ImageButton
            android:id="@+id/btSend"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:gravity="center"
            android:paddingRight="10dp"
            android:layout_alignParentRight="true"
            android:contentDescription="@string/send"
            android:src="@drawable/ic_baseline_send_24"
            android:textSize="18sp" />
    </RelativeLayout>
</RelativeLayout>
```

`ConstraintLayout` code will look like this:
```xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.constraint.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <EditText
        android:id="@+id/etMessage"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginStart="8dp"
        android:layout_marginTop="8dp"
        android:layout_marginBottom="8dp"
        android:hint="@string/message_hint"
        android:imeOptions="actionSend"
        android:inputType="textShortMessage"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toStartOf="@+id/btSend"
        app:layout_constraintHorizontal_bias="0.5"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/rvChat" />

    <ImageButton
        android:id="@+id/btSend"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:gravity="center"
        android:paddingRight="10dp"
        android:layout_alignParentRight="true"
        android:contentDescription="@string/send"
        android:src="@drawable/ic_baseline_send_24"
        android:textSize="18sp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintHorizontal_bias="0.5"
        app:layout_constraintStart_toEndOf="@+id/etMessage"
        app:layout_constraintTop_toTopOf="@+id/etMessage" />

    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/rvChat"
        android:layout_width="0dp"
        android:layout_height="0dp"
        android:layout_marginStart="8dp"
        android:layout_marginTop="8dp"
        android:layout_marginEnd="8dp"
        app:layout_constraintBottom_toTopOf="@+id/etMessage"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintHorizontal_bias="0.5"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />
</android.support.constraint.ConstraintLayout>
```

### 7.1 Add layouts for incoming and outgoing messages

In this lab we will learn how to use adapter to create different views for incoming and outgoing messages. We will be showing the outgoing messages with our gravatar on the right and all the incoming messages with gravatars and sender names on the left. You can read more about creating gravatars [here](https://en.gravatar.com/site/implement/images/). 

Now we need to create two layout file to represent each chat type of message - incoming & outgoing - in the list view. Put this into `res/layout/message_incoming.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:paddingVertical="10dp"
    android:paddingLeft="15dp"
    android:paddingRight="60dp"
    android:clipToPadding="false">

    <TextView
        android:id="@+id/tvName"
        android:layout_marginLeft="15dp"
        android:layout_toRightOf="@+id/ivProfileOther"
        android:layout_alignTop="@+id/ivProfileOther"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:paddingBottom="4dp"
        android:text="Johny Lindsey" />

    <ImageView
        android:id="@+id/ivProfileOther"
        android:layout_alignParentLeft="true"
        android:contentDescription="@string/profile_other"
        android:layout_width="64dp"
        android:layout_height="64dp"
        android:src="@mipmap/ic_launcher" />
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:id="@+id/tvBody"
        android:layout_below="@+id/tvName"
        android:layout_alignLeft="@+id/tvName"
        android:background="@drawable/message_incoming"
        android:paddingVertical="12dp"
        android:paddingHorizontal="16dp"
        android:elevation="2dp"
        android:textSize="18dp"
        android:text="Someone else's message" />
</RelativeLayout>
```

For outgoing (our) messages create another layout file `messages_outgoing`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:clipToPadding="false"
    android:paddingVertical="10dp"
    android:paddingLeft="60dp"
    android:paddingRight="15dp">

    <TextView
        android:id="@+id/tvBody"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginRight="8dp"
        android:layout_toLeftOf="@+id/ivProfileMe"
        android:background="@drawable/message_outgoing"
        android:elevation="2dp"
        android:padding="8dp"
        android:text="Your outgoing message here"
        android:textColor="@android:color/white"
        android:textSize="18dp" />

    <ImageView
        android:id="@+id/ivProfileMe"
        android:layout_width="64dp"
        android:layout_height="64dp"
        android:layout_alignParentRight="true"
        android:contentDescription="@string/profile_me"
        android:src="@mipmap/ic_launcher" />

</RelativeLayout>
```

Add the following values to `res-->values-->strings.xml` file:

```xml
<string name="profile_me">My Profile Pic</string>
<string name="profile_other">Other profile pic</string>
```

As we learned before, those labels can be picked up by **accessibility** framework in Android and pronounced to a user who relies on [TalkBack](https://support.google.com/accessibility/android/answer/6283677?hl=en)

### 7.2 Creating chat "bubble" backgrounds

In order to re-create popular chat "bubble" UI we need to define proper backgrounds. We referred to them in the section 7.1 as `@drawable/message_outgoing` and `@drawable/message_incoming`. In `res --> drawable` folder create two files:

```xml
<!-- message_outgoing.xml -->
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="rectangle">
    <solid android:color="@color/colorPrimary" />
    <corners android:topRightRadius="5dp" android:radius="15dp" />
</shape>
```

and 

```xml
<!-- message_incoming.xml -->
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="rectangle">
    <solid android:color="#fff" />
    <corners android:topLeftRadius="5dp" android:radius="15dp" />
</shape>
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
        btSend = (ImageButton) findViewById(R.id.btSend);
        // When send button is clicked, create message object on Parse
        btSend.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                String data = etMessage.getText().toString();

                // Delete the following three lines:
                //ParseObject message = ParseObject.create("Message");
                //message.put(Message.USER_ID_KEY, ParseUser.getCurrentUser().getObjectId());
                //message.put(Message.BODY_KEY, data);

                /*** START OF CHANGE **/

                // Using new `Message` Parse-backed model now
                Message message = new Message();
                message.setUserId(ParseUser.getCurrentUser().getObjectId());
                message.setBody(data);

                /*** END OF CHANGE **/

                message.saveInBackground(new SaveCallback() {
                    @Override
                    public void done(ParseException e) {
                        if (e == null) {
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

Create a class named `ChatAdapter.java` with below code. This is a custom list adapter class which provides data to list view. In other words it renders the `message_incoming.xml` (or `message_outgoing.xml`) in list by pre-filling appropriate information. We'll be using the open source `Glide` library to load profile images. 

First, add dependency for this library to the `app/build.gradle` file:

```groovy
...
dependencies {
    implementation 'com.github.bumptech.glide:glide:4.11.0'
}
```

### 9.1 Create a basic adapter

We start by creating a very basic `ChatAdapter` that extends `RecyclerView.Adapter`. We will initialize the adapter instance by passing `context`, `userId` and the list of our chat `messages`:

```java
public class ChatAdapter extends RecyclerView.Adapter<ChatAdapter.MessageViewHolder> {

    private List<Message> mMessages;
    private Context mContext;
    private String mUserId;

    public ChatAdapter(Context context, String userId, List<Message> messages) {
        mMessages = messages;
        this.mUserId = userId;
        mContext = context;
    }

    @Override
    public int getItemCount() {
        return mMessages.size();
    }

    @Override
    public int getItemViewType(int position) {
        // TODO: implement method
    }

    @Override
    public MessageViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        // TODO: implement method
    }

    @Override
    public void onBindViewHolder(MessageViewHolder holder, int position) {
        // TODO: implement method
    }

}
```

We will keep implementing the adapter in the next couple chapters.

### 9.2 Define view types

Now it's time to define the view types and implement the code that will allow adapter to decide which view type to use for every message in our chat.

First, let's define two integer constants types `MESSAGE_INCOMING` and `MESSAGE_OUTGOING`. Their values can be choosen at random as long as they are not the same 😅

Second, we will implement `getItemViewType(int position)` that will return view type (incoming or outgoing) based on the message position. Our adapter should look like this:

```java

public class ChatAdapter extends RecyclerView.Adapter<ChatAdapter.MessageViewHolder> {
...
    private static final int MESSAGE_OUTGOING = 123;
    private static final int MESSAGE_INCOMING = 321;

    @Override
    public int getItemViewType(int position) {
        if (isMe(position)) {
            return MESSAGE_OUTGOING;
        } else {
            return MESSAGE_INCOMING;
        }
    }
}
```

The function `isMe()` is a little helper function that can be implemented like this:

```java
    private boolean isMe(int position) {
        Message message = mMessages.get(position);
        return message.getUserId() != null && message.getUserId().equals(mUserId);
    }
```

### 9.3 Creating custom `ViewHolder`s for view types

Every message type is represented by its own view that we defined in XML layouts. Each of those views can be assigned its own `ViewHolder` type. If you need a refresher on ViewHolder patern, see [this guide](https://guides.codepath.org/android/Using-an-ArrayAdapter-with-ListView#improving-performance-with-the-viewholder-pattern).

Let's begin by defining a common ViewHolder base class - `MessageViewHolder`. This class will be the lowest common denominator between Incoming and Outgoing messages. You can define those classes inside `ChatAdapter` class or create a separate file for ViewHolders. 

```java
public class ChatAdapter extends RecyclerView.Adapter<ChatAdapter.MessageViewHolder> {
...
   public abstract class MessageViewHolder extends RecyclerView.ViewHolder {

        public MessageViewHolder(@NonNull View itemView) {
            super(itemView);
        }

        abstract void bindMessage(Message message);
    }
}
```

In this example we define the class as `abstract` meaning it can't have any instances. Let's define actual implementations for incoming and outgoing message view holders:

```java
...
   public abstract class MessageViewHolder extends RecyclerView.ViewHolder {

        public MessageViewHolder(@NonNull View itemView) {
            super(itemView);
        }

        abstract void bindMessage(Message message);
    }

    public class IncomingMessageViewHolder extends MessageViewHolder {
        ImageView imageOther;
        TextView body;
        TextView name;

        public IncomingMessageViewHolder(View itemView) {
            super(itemView);
            imageOther = (ImageView)itemView.findViewById(R.id.ivProfileOther);
            body = (TextView)itemView.findViewById(R.id.tvBody);
            name = (TextView)itemView.findViewById(R.id.tvName);
        }

        @Override
        public void bindMessage(Message message) {
            // TODO: implement later
        }
    }

    public class OutgoingMessageViewHolder extends MessageViewHolder {
        ImageView imageMe;
        TextView body;

        public OutgoingMessageViewHolder(View itemView) {
            super(itemView);
            imageMe = (ImageView)itemView.findViewById(R.id.ivProfileMe);
            body = (TextView)itemView.findViewById(R.id.tvBody);
        }

        @Override
        public void bindMessage(Message message) {
            // TODO: implement later
        }
    }
```

As you can see those ViewHolders have different view components assigned to them. This is because we previously decided to have different view layouts to easily distinguish the messages in our chat.

Next, we need to assign correct view holders based on a view type:

```java
public class ChatAdapter extends RecyclerView.Adapter<ChatAdapter.MessageViewHolder> {
...
    @Override
    public MessageViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        Context context = parent.getContext();
        LayoutInflater inflater = LayoutInflater.from(context);

        if (viewType == MESSAGE_INCOMING) {
            View contactView = inflater.inflate(R.layout.message_incoming, parent, false);
            return new IncomingMessageViewHolder(contactView);
        } else if (viewType == MESSAGE_OUTGOING) {
            View contactView = inflater.inflate(R.layout.message_outgoing, parent, false);
            return new OutgoingMessageViewHolder(contactView);
        } else {
            throw new IllegalArgumentException("Unknown view type");
        }
    }
}
```

In the example above we create new views from xml layout and assign them to corresponding view holders. In case if the view type is different from the ones we defined we will throw an exception. This is a good choice that follows "fail fast" programming principle. Since the "Unknown view type" is an unlikely scenario it would be better to crash application notifying a developer that something goes wrong.

### 9.4 Binding data to `ViewHolder`s

In this final part of the adapter setup we will bind data from individual messages to corresponding `ViewHolder`s. Let's start by defining how each ViewHolder displays their message:

```java
    public class IncomingMessageViewHolder extends MessageViewHolder {
...
        @Override
        public void bindMessage(Message message) {
            Glide.with(mContext)
                    .load(getProfileUrl(message.getUserId())) 
                    .circleCrop() // create an effect of a round profile picture
                    .into(imageOther);
            body.setText(message.getBody());
            name.setText(message.getUserId()); // in addition to message show user ID
        }
    }

    public class OutgoingMessageViewHolder extends MessageViewHolder {
...

        @Override
        public void bindMessage(Message message) {
            Glide.with(mContext)
                    .load(getProfileUrl(message.getUserId()))
                    .circleCrop() // create an effect of a round profile picture
                    .into(imageMe);
            body.setText(message.getBody());
        }
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
        return "https://www.gravatar.com/avatar/" + hex + "?d=identicon";
    }
```

In the code above we use `Glide` library to load gravatars from a given URL. If you need a refresher on Glide see [Displaying Images with the Glide Library](https://guides.codepath.org/android/Displaying-Images-with-the-Glide-Library). The method `getProfileUrl()` is responsible for decoding user ID and creating an image URL that can be passed into the Glide library. 

Finally, we should tell `ChatAdapter` how (and when) to bind different message types to different view types:

```java
public class ChatAdapter extends RecyclerView.Adapter<ChatAdapter.MessageViewHolder> {
...
    @Override
    public void onBindViewHolder(MessageViewHolder holder, int position) {
        Message message = mMessages.get(position);
        holder.bindMessage(message);
    }

}
```

Here you can see an example of one of the object-oriented programming principles - polymorphism - in action. Since we defined the `MessageViewHolder` as a parent class for both types of the message the runtime will decide what is the right `bindMessage()` call for a given `ViewHolder`. 

## 10. Bind Adapter to the RecyclerView

Next, we will setup the ReyclerView and bind our custom adapter to this ReyclerView within the `ChatActivity.java` source file:

```java
public class ChatActivity extends AppCompatActivity {
    ...

    RecyclerView rvChat;    
    List<Message> mMessages;
    ChatAdapter mAdapter;
    // Keep track of initial load to scroll to the bottom of the ListView
    boolean mFirstLoad;
	
    // Setup message field and posting
    void setupMessagePosting() {
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
                message.setUserId(ParseUser.getCurrentUser().getObjectId());
                message.setBody(data);
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

Finally, let's periodically refresh the RecyclerView with latest messages [[using a handler|Repeating-Periodic-Tasks#handler]]. The handler will call a runnable to fetch new messages every 3 second. This is a primitive "polling" rather than the more efficient "push" technique for refreshing new messages - but will work for the purposes of this simple project.

**WARNING** It's important to remember, that polling often leads to redundancy, e.g. making many unnecessary requests when there was no change in data. In addition to that, frequent requests are costly due to increased battery drain and cost of cell data traffic. We will look at a couple techniques to prevent unnecessary polling when application is inactive. 

```java
...

// Create a handler which can run code periodically
static final long POLL_INTERVAL = TimeUnit.SECONDS.toMillis(3); 
Handler myHandler = new android.os.Handler();
Runnable mRefreshMessagesRunnable = new Runnable() {
    @Override
    public void run() {
       refreshMessages();
       myHandler.postDelayed(this, POLL_INTERVAL);
    }
};

...

    @Override
    protected void onResume() {
        super.onResume();

        // Only start checking for new messages when the app becomes active in foreground
        myHandler.postDelayed(mRefreshMessagesRunnable, POLL_INTERVAL);
    }

    @Override
    protected void onPause() {
        // Stop background task from refreshing messages, to avoid unnecessary traffic & battery drain
        myHandler.removeCallbacksAndMessages(null);
        super.onPause();
    }
}
```

The methods `onResume()` and `onPause()` are sort of mirror lifecycle callbacks. First is getting called when the `Activity` is ready to be resumed and about to be displayed to the user. `onPause()` is the exact opposite and gets called when a current `Activity` is about to go into background.

It's important to only start polling when `onResume()` is getting called and ensuring we stop listening for any callbacks as soon as activity becomes invisible. However, in production chat application you might want to run a background service that will be periodically checking for messages and displaying Message notifications to a user. 

See the [[repeating periodic tasks|Repeating-Periodic-Tasks#handler]] guide to learn more about the handler.

## 13. Live Queries

Alternatively, the Back4App server can be configured properly to listen to the Message object for changes (see [this example](https://github.com/codepath/parse-server-example/blob/master/index.js#L49)).  We need to enable `liveQuery` functionality in our Back4App server installation.  

See following [[these instructions|https://www.back4app.com/docs/platform/parse-server-live-query-example]] to enable Live Queries in your Back4App server, we can now use [[Parse Live Queries|Building-Data-driven-Apps-with-Parse#live-queries]] in our Android app to listen for new messages.  We can disable the use of the `postDelayed()` runnable that we created in the earlier step:

```java
// myHandler.postDelayed(mRefreshMessagesRunnable, POLL_INTERVAL); // comment this out
```

First, make sure to add the Parse LiveQuery dependency to your `app/build.gradle` config:

```gradle
allprojects {
	repositories {
		maven { url "https://jitpack.io" }
	}
}

dependencies {
      implementation 'com.github.parse-community:ParseLiveQuery-Android:1.1.0' // for Parse Live Queries
}
```

Next, we will configure to listen for any newly created Message object:

```java
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);

    // ...

    // Parse.initialize(...) should happen first, preferably in a different file such as your MyApplication.java

    // Load existing messages to begin with
    refreshMessages();

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
}
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


![Chat App|250](https://i.imgur.com/v8R5LyU.png)

