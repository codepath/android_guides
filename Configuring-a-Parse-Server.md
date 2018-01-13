### Overview

Parse provides a cloud-based backend service to build data-driven mobile apps quickly.  Facebook, which acquired the company more than 3 years ago, announced that the service would be shutting down on **January 28, 2017**.   An [open source version](https://github.com/ParsePlatform/parse-server) enables developers to continue using Parse to build apps.

While there are many
[alternate options to Parse](https://github.com/relatedcode/ParseAlternatives), most of them lack either the functionality, documentation, or sample code to enable quick prototyping.  For this reason, the open source Parse version is a good option to use with minimal deployment/configuration needed.

#### Differences with Open Source Parse

You can review this [Wiki](https://github.com/ParsePlatform/parse-server/wiki) to understand the current development progress of this app.  There are a few notable differences in the open source version:

* **Authentication**: By default, only an application ID is needed to authenticate with open source Parse.  The [base configuration](https://github.com/ParsePlatform/parse-server-example/blob/master/index.js#L13-L18) that comes with the one-click deploy options does not require authenticating with any other types of keys.   Therefore, specifying client keys on Android or iOS is not needed.

* **Push notifications**: Because of the implicit [security issues](https://github.com/ParsePlatform/parse-server/issues/396#issuecomment-183792657) with allowing push notifications to be sent through Android or iOS directly to other devices, this feature is disabled. Note that for open source Parse, you must implement pre-defined code written in JavaScript that can be called by the clients to execute, otherwise known as [Parse Cloud](http://blog.parse.com/announcements/pushing-from-the-javascript-sdk-and-cloud-code/).

* **Single app aware**: The current version only supports single app instances.  There is ongoing work to make this version multi-app aware.  However, if you intend to run many different apps with different datastores, you currently would need to instantiate separate instances.

* **File upload limitations**: The backend for open source is backed by MongoDB, and the default storage layer relies on Mongo's GridFS layer.  The current limit is set for 20 MB but if you depend on storing large files, you should really configure the server to use Amazon's Simple Storage Service (S3).

Many of the options need to be configured by tweaking your own configuration.  You may wish to fork the [code](https://github.com/ParsePlatform/parse-server-example/) that helps instantiate a Parse server and change them based on your own needs.  The guide below includes instructions on [[how to add push notifications|Configuring-a-Parse-Server#enabling push notifications]] and [[storing files with Parse|Configuring-a-Parse-Server#storing files with parse]].

### Setting a new Parse Server

The steps described this guide walk through most of the process of setting an open source version with Parse.  There are obviously many other hosting options, but the one-click deploys made available with Heroku as discussed in [this guide](https://github.com/ParsePlatform/parse-server-example) are the simplest.   In both cases, you are likely to need a credit card attached to your account to activate.

#### Signing up with Heroku

Use Heroku if you have little or no experience with setting up web sites. Heroku allows you to manage changes to deploy easily by specifying a GitHub repository to use.  In addition, it comes with a UI data viewer from mLab.

1. Sign Up / Sign In at [Heroku](https://www.heroku.com)

2. Click on the button below to create a Parse App

      <a href="https://dashboard.heroku.com/new?button-url=https%3A%2F%2Fgithub.com%2FParsePlatform%2Fparse-server-example&template=https%3A%2F%2Fgithub.com%2FParsePlatform%2Fparse-server-example"><img src="https://camo.githubusercontent.com/c0824806f5221ebb7d25e559568582dd39dd1170/68747470733a2f2f7777772e6865726f6b7563646e2e636f6d2f6465706c6f792f627574746f6e2e706e67" alt="Deploy" data-canonical-src="https://www.herokucdn.com/deploy/button.png" style="max-width:100%;"></a>

2. Make sure to enter an App Name.  Scroll to the bottom of the page.

      <img src="https://imgur.com/0JcJrn5.png">

3. Make sure to change the config values.
      <img src="https://imgur.com/vQV0X0S.png"/>
      * Leave `PARSE_MOUNT` to be `/parse`.  It does not need to be changed.
      * Set `APP_ID` for the app identifier.  If you do not set one, the default is set as `myAppId`.  You will need this info for the Client SDK setup.
      * Set `MASTER_KEY` to be the master key used to read/write all data.  **Note**: in hosted Parse, client keys are not used by default.
      * Set `SERVER_URL` to be `http://yourappname.herokuapp.com/parse`.  Assuming you have left `PARSE_MOUNT` to be /parse, this will enable the use of Parse Cloud to work correctly.
      * If you intend to use Parse's Facebook authentication, set `FACEBOOK_APP_ID` to be the [FB application ID](https://developers.facebook.com/apps).
      * If you intend to setup push notifications, there are additional environment variables such as `GCM_SENDER_KEY` and `GCM_API_KEY` that will need to be configured.  See [[this section|Configuring-a-Parse-Server#enabling-push-notifications]] for the required steps.

4. Deploy the Heroku app.  The app should be hosted at `https://<app name>.herokuapp.com` where `<app name>` represents your App Name that you provided (or if one was assigned to you if you left this field blank).

If you ever need to change these values later, you can go to (`https://dashboard.heroku.com/apps/<app name>/settings`). Check out [this guide](https://devcenter.heroku.com/articles/deploying-a-parse-server-to-heroku) for a more detailed set of steps for deploying Parse to Heroku.

Now, we can [[test our deployment|Configuring-a-Parse-Server#testing-deployment]] to verify that the Parse deployment is working as expected!

### Testing Deployment

After deployment, try to connect to the site.  You should see `I dream of being a website. Please star the parse-server repo on GitHub!` if the site loaded correctly.   If you try to connect to the `/parse` endpoint, you should see `{error: "unauthorized"}`.  If both tests pass, the basic configuration is successful.

Next, make sure you can create Parse objects.  You do not need a client Key to write new data:

```bash
curl -X POST -H "X-Parse-Application-Id: myAppId" \
-H "Content-Type: application/json" \
-d '{"score":1337,"playerName":"Sean Plott","cheatMode":false}' \
https://yourappname.herokuapp.com/parse/classes/GameScore
```

Be sure to **replace the values** for `myAppId` and the server URL. If you see `Cannot POST` error then be sure both the `X-Parse-Application-Id` and the URL are correct for your application. To read data back, you will need to specify your master key as well:

```bash
curl -X GET -H "X-Parse-Application-Id: myAppId" -H "X-Parse-Master-Key: abc"  \
https://yourappname.herokuapp.com/parse/classes/GameScore
```

Be sure to **replace the values** for `myAppId` and the server URL. If these commands work as expected, then your Parse instance is now setup and ready to be used!

### Browsing Parse Data

There are several options that allow you to view the data.  First, you can use the `mLab` viewer to examine the store data.  Second, you can setup the open source verson of the Parse Dashboard, which gives you a similar UI used in hosted Parse.  Finally, you can use Robomongo.

#### mLab

The hosted Parse instance deployed uses [mLab](https://mlab.com/) (previously called MongoLab) to store all of your data. mLab is a hosted version of [MongoDB](https://www.mongodb.org/) which is a document-store which uses JSON to store your data.

If you are using Heroku, you can verify whether the objects were created by clicking on the MongoDB instance in the Heroku panel:

<img src="https://imgur.com/bbj2e9N.png"/>

<img src="https://imgur.com/snPqYkz.png"/>

#### Parse Dashboard

You can also install Parse's open source dashboard locally. Download [NodeJS v4.3](https://nodejs.org/en/download/) or higher.  Make sure you have at least Parse server v2.1.3 or higher (later versions include a `/parse/serverInfo` that is needed).

```bash
npm install -g parse-dashboard
parse-dashboard --appId myAppId --masterKey myMasterKey --serverURL "https://yourapp.herokuapp.com/parse"
```

Connect to your dashboard at `http://localhost:4040/apps`. Assuming you have specified the correct application ID, master Key, and server URL, as well as installed a Parse open source version v2.1.3 or higher, you should see the app appear correctly:

<img src="https://imgur.com/Z0Rz5Xs.png"/>

#### Robomongo

You can also setup [Robomongo](https://robomongo.org/download) to connect to your remote mongo database hosted on Heroku to get a better data browser and dashboard for your app.

<a href="https://robomongo.org/download"><img src="https://i.imgur.com/9Qtt6Xs.png" width="450" /></a>

To access mLab databases using Robomongo, be sure to go the MongoDB instance in the Heroku panel as shown above. Look for the following URL: `mongodb://<dbuser>:<dbpassword>@ds017212.mlab.com:11218/heroku_2flx41aa`. Use that to identify the login credentials:

```
address: ds017212.mlab.com
port: 11218
db: heroku_2flx41aa
user: dbuser
password: dbpassword
```

Note that the **user and password** provided are for a database user you configure and are **not your mLab login credentials**. Using that cross-platform app to easily access and modify the data for your Parse MongoDB data.

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

Websocket URLs are usually prefixed with ws:// or wss:// (secure) URLs.  Heroku instances already provide websocket support, but if you are deploying to a different server (Amazon), you may need to make sure that TCP port 80 or TCP port 443 are available.

### Enabling Client SDK integration

Follow the [[setup guide|Building-Data-driven-Apps-with-Parse#setup]].
Make sure you have the latest Parse-Android SDK <img src="https://camo.githubusercontent.com/4da68f9e3fc719ae9f0198d780c611d61bb8843b/68747470733a2f2f6d6176656e2d6261646765732e6865726f6b756170702e636f6d2f6d6176656e2d63656e7472616c2f636f6d2e70617273652f70617273652d616e64726f69642f62616467652e7376673f7374796c653d666c6174" alt="Maven Central" data-canonical-src="https://maven-badges.herokuapp.com/maven-central/com.parse/parse-android/badge.svg?style=flat" style="max-width:100%;">
 in your `app/build.gradle` file.

### Troubleshooting Parse Server

* If you see `Application Error` or `An error occurred in the application and your page could not be served. Please try again in a few moments.`, double-check that you set a `MASTER_KEY` in the environment settings for that app.

  <img src="https://imgur.com/uMYwPmS.png">

* If you are using Heroku, download the Heroku Toolbelt app [here](https://toolbelt.heroku.com/) to help view system logs.

  <img src="https://imgur.com/Ch0mZOK.png"/>

  First, you must login with your Heroku login and password:

  ```bash
  heroku login
  ```

   You can then view the system logs by specifying the app name:
   ```bash
   heroku logs --app <app name>
   ```

   The logs should show the response from any types of network requests made to the site.  Check the `status` code.
   ```
   2016-02-07T08:28:14.292475+00:00 heroku[router]: at=info method=POST path="/parse/classes/Message" host=parse-testing-port.herokuapp.com request_id=804c2533-ac56-4107-ad05-962d287537e9 fwd="101.12.34.12" dyno=web.1 connect=1ms service=2ms status=404 bytes=179
   ```

* If you are seeing `Master key is invalid, you should only use master key to send push`, chances are you are trying to send Push notifications without enable client push.  On hosted parse there is an outstanding [issue](https://github.com/ParsePlatform/parse-server/issues/396) that must be resolved to start supporting it.

* If you see the exception `Error Loading Messagescom.parse.ParseException: java.lang.IllegalArgumentException: value == null`, try setting the `clientKey` to a blank string such as `Parse.initialize(...).applicationId(...).clientKey("")` rather than `null`. Review [this issue](https://github.com/ParsePlatform/Parse-SDK-Android/issues/392) and [this issue](https://github.com/ParsePlatform/Parse-SDK-Android/issues/201) for further details.

* You can also use Facebook's [Stetho](http://facebook.github.io/stetho/) interceptor to watch network logs with Chrome:

    ```gradle
    dependencies {
      compile 'com.facebook.stetho:stetho:1.3.0'
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
