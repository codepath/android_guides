### Overview

Parse provides a cloud-based backend service to build data-driven mobile apps quickly.  Facebook, which acquired the company more than 3 years ago, announced that the service would be shutting down on **January 28, 2017**.   An [open source version](https://github.com/ParsePlatform/parse-server) enables developers to continue using Parse to build apps.

While there are many
[alternate options to Parse](https://github.com/relatedcode/ParseAlternatives), most of them lack either the functionality, documentation, or sample code to enable quick prototyping.  For this reason, the open source Parse version is a good option to use with minimal deployment/configuration needed.

#### Differences with Open Source Parse

You can review this [Wiki](https://github.com/ParsePlatform/parse-server/wiki) to understand the current development progress of open source Parse.  There are a few notable differences in the open source version from the originally hosted version:

* **Authentication**: By default, only an application ID is needed to authenticate with open source Parse.  The [base configuration](https://github.com/ParsePlatform/parse-server-example/blob/master/index.js#L13-L18) that comes with the one-click deploy options does not require authenticating with any other types of keys.   Therefore, specifying client keys on Android or iOS is not needed.

* **Push notifications**: Because of the implicit [security issues](https://github.com/ParsePlatform/parse-server/issues/396#issuecomment-183792657) with allowing push notifications to be sent through Android or iOS directly to other devices, this feature is disabled. Note that for open source Parse, you must implement pre-defined code written in JavaScript that can be called by the clients to execute, otherwise known as [Parse Cloud](http://blog.parse.com/announcements/pushing-from-the-javascript-sdk-and-cloud-code/).

* **Single app aware**: The current version only supports single app instances.  There is ongoing work to make this version multi-app aware.  However, if you intend to run many different apps with different datastores, you currently would need to instantiate separate instances.

* **File upload limitations**: The backend for open source is backed by MongoDB, and the default storage layer relies on Mongo's GridFS layer.  The current limit is set for 20 MB but if you depend on storing large files, you should really configure the server to use Amazon's Simple Storage Service (S3).

Many of the options need to be configured by tweaking your own configuration.  You may wish to fork the [code](https://github.com/ParsePlatform/parse-server-example/) that helps instantiate a Parse server and change them based on your own needs.  The guide below includes instructions on [[how to add push notifications|Configuring-a-Parse-Server#enabling push notifications]] and [[storing files with Parse|Configuring-a-Parse-Server#storing files with parse]].

### Setting up a new Parse Server

The steps described this guide walk through most of the process of setting an open source version with Parse.  There are obviously many other [hosting options](https://github.com/parse-community/parse-server#running-parse-server), but the one-click deploy made available with [back4app](https://www.back4app.com/) is the simplest option. 

#### Deploying with back4app

Use back4app if you have little or no experience with setting up web sites. back4app allows you to manage changes to deploy easily by specifying a GitHub repository to use.  In addition, it comes with a UI data viewer from mLab.

1. Sign Up / Sign in at [back4app](https://www.back4app.com/)

1. Click on the option to 'Build a New App'

    <img src="https://i.imgur.com/MYENcDn.png" width=800>

1. Enter a name for your app and click Create.

    <img src="https://i.imgur.com/4wBs2A0.png" width=300>

1. Once the setup is finished, you should see the dashboard for your app.

    <img src="https://i.imgur.com/pBztIvq.png" width=800>    

Now, we can [[test our deployment|Configuring-a-Parse-Server#testing-deployment]] to verify that the Parse deployment is working as expected!

### Testing Deployment

After deployment, you can click on the API Reference section on the left nav bar to explore different ways to connect with your Parse server.

<img src="https://i.imgur.com/C21YgDn.png" width=800>

For example, you can do a quick test on whether the deployment was successful by going to the 'Initializing a Parse SDK' section for curl and making a curl request to your newly deployed Parse instance.

<img src="https://i.imgur.com/v98plKI.png" width=1000>

If the command works as expected, then your Parse instance is now setup and ready to be used!  

Similarly, you can also see how to connect to different clients (such as Android or iOS) in this API reference section.

### Browsing Parse Data

To browse your Parse Data, you can visit the 'Database Browser' page. 

<img src="https://i.imgur.com/uNpK5in.png" width=800>

### Enabling Client SDK integration

Follow the [[setup guide|Building-Data-driven-Apps-with-Parse#setup]].
Make sure you have the latest Parse-Android SDK <img src="https://jitpack.io/v/parse-community/Parse-SDK-Android.svg" style="max-width:100%;">
 in your `app/build.gradle` file.

### Troubleshooting Parse Server

* If you see `Application Error` or `An error occurred in the application and your page could not be served. Please try again in a few moments.`, double-check that you set a `MASTER_KEY` in the environment settings for that app.

  <img src="https://imgur.com/uMYwPmS.png">

* If you are seeing `Master key is invalid, you should only use master key to send push`, chances are you are trying to send Push notifications without enable client push.  On hosted parse there is an outstanding [issue](https://github.com/ParsePlatform/parse-server/issues/396) that must be resolved to start supporting it.

* If you see the exception `Error Loading Messagescom.parse.ParseException: java.lang.IllegalArgumentException: value == null`, try setting the `clientKey` to a blank string such as `Parse.initialize(...).applicationId(...).clientKey("")` rather than `null`. Review [this issue](https://github.com/ParsePlatform/Parse-SDK-Android/issues/392) and [this issue](https://github.com/ParsePlatform/Parse-SDK-Android/issues/201) for further details.

* You can also use Facebook's [Stetho](http://facebook.github.io/stetho/) interceptor to watch network logs with Chrome:

    ```gradle
    dependencies {
      implementation 'com.facebook.stetho:stetho:1.3.0'
    }
    ```

  And add this network interceptor as well:

  ```java
     Stetho.initializeWithDefaults(this); // init Stetho before Parse

     Parse.initialize(new Parse.Configuration.Builder(this)
       .addNetworkInterceptor(new ParseStethoInterceptor())
  ```

* If you wish to troubleshoot your Parse JavaScript code is to run the Parse server locally (see [instructions](https://github.com/ParsePlatform/parse-server-example#for-local-development)).  You should leverage the `--inspect` command, which allows you to use Chrome or Safari to step through the code yourself:

   ```bash
   node --inspect index.js
   ```

   Open up `chrome://inspect`.  You can use the Chrome debugging tools to set breakpoints in the JavaScript code.

   Point your Android client to this server::

   ```java
   Parse.initialize(new Parse.Configuration.Builder(this)
        .applicationId("myAppId")
        .server("http://192.168.3.116:1337/parse/")  // lookup your IP address of your machine
   ```

* **Running into issues with mLab and MongoDB or `MONGODB_URL`?** Check the [more detailed instructions here](https://gist.github.com/nesquena/7786c5a876e335737fb28d037400fd40) for getting the `MONGODB_URL` setup properly. 

### Enabling Push Notifications

See [[this guide|Push-Notifications-Setup-for-Parse]] for more details about how to setup both the Parse open source version and the Parse Android SDK Client.

### Storing Files with Parse

You can continue to save large files to your Parse backend using `ParseFile` without any major changes:

```java
byte[] data = "Working at Parse is great!".getBytes();
ParseFile file = new ParseFile("resume.txt", data);
file.saveInBackground();
```

By default, the open source version of Parse relies on the MongoDB [GridStore](https://mongodb.github.io/node-mongodb-native/api-generated/gridstore.html) adapter to store large files.  There is an alternate option to leverage Amazon's Simple Storage Service (S3) but still should be considered experimental.

### Adding Support for Live Queries

One of the newer features of Parse is that you can monitor for live changes made to objects in your database (i.e. creation, updates, and deletes)  To get started, make sure you have defined the ParseObjects that you want in your NodeJS server.  Make sure to define a list of all the objects by declaring it in the `liveQuery` and `classNames listing`:

```javascript
let api = new ParseServer({
  ...,
  // Make sure to define liveQuery AND classNames
  liveQuery: {
    // define your ParseObject names here
    classNames: ['Post', 'Comment']
  }
});
```

See [this guide](http://parseplatform.org/docs/parse-server/guide/#live-queries) and [this spec](https://github.com/parse-community/parse-server/wiki/Parse-LiveQuery-Protocol-Specification) for more details.  Parse Live Queries rely on the websocket protocol, which creates a bidirectional channel between the client and server and periodically exchange ping/pong frames to validate the connection is still alive.

Websocket URLs are usually prefixed with ws:// or wss:// (secure) URLs.  If you are deploying to an Amazon server, you may need to make sure that TCP port 80 or TCP port 443 are available.

#### Using Amazon S3

If you wish to store the files in an Amazon S3 bucket, you will need to make sure to setup your Parse server to use the S3 adapter instead of the default GridStore adapter.  See [this Wiki](https://github.com/ParsePlatform/parse-server/wiki/Configuring-File-Adapters#configuring-s3adapter) for how to configure your setup.  The steps basically include:

1. Modify the Parse server to use the S3 adapter.  See [these changes](https://github.com/codepath/parse-server-example/blob/master/index.js#L20-L29) as an example.
2. Create an Amazon S3 bucket.
3. Create an Amazon user with access to this S3 bucket.
4. Generate an authorized key/secret pair to write to this bucket.
5. Set the environment variables:
      * Set `S3_ENABLE` to be 1.
      * Set `AWS_BUCKET_NAME` to be the AWS bucket name.
      * Set `AWS_ACCESS_KEY` and ` AWS_SECRET_ACCESS_KEY` to be the user that has access to read/write to this S3 bucket.

When testing, try to write a file and use the Amazon S3 console to see if the files were created in the right place.
