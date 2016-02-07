### Overview

Parse provides a cloud-based backend service to build data-driven mobile apps quickly.  Facebook, which acquired the company a year ago, announced that the service would be shutting down on **January 28, 2017**.   An [open source version](https://github.com/ParsePlatform/parse-server) enables developers to continue using their apps was published, along with a [migration guide](https://parse.com/docs/server/guide#migrating).  

While there are many 
[alternate options to Parse](https://github.com/relatedcode/ParseAlternatives), most of them lack either the functionality, documentation, or sample code to enable quick prototyping.  For this reason, the open source Parse version is a good option to use with minimal deployment/configuration needed.

### Setting a new Parse Server

The steps described [this guide](https://devcenter.heroku.com/articles/deploying-a-parse-server-to-heroku) walk through most of the process of setting an open source version with Parse.  There are obviously many other hosting options, but deploying with Heroku is probably the simplest.  The instructions basically entail the following steps:

1. [Register](https://id.heroku.com/signup/login) for a free Heroku account.
2. Establish a credit card through [Account Settings](https://dashboard.heroku.com/account) in order to add a free MongoDB sandbox instance.
3. Fork a copy of the [Parse server example](https://github.com/ParsePlatform/parse-server-example). 
4. Create a new Heroku app, pointing to this repository that was created.
5. Add the MongoDB instance as an Add-On.
6. Setup environment variables in the app settings: `https://dashboard.heroku.com/apps/<app name>/settings`.
      a. Set a `MASTER_KEY`.  Otherwise, the server will not load properly.  
      b. Set a `APP_ID`.  If you do not set one, the default is set as `myAppId` (see [the source code]
(https://github.com/ParsePlatform/parse-server-example/blob/master/index.js#L14-L18).
      c. Set `PARSE_MOUNT` to '/`.  This setting is needed to enable Client SDK's to talk to your Parse instance.
7. Deploy the Heroku app.  The app should be hosted at `https://<app name>.herokuapp.com`.

### Testing Deployment

After deployment, try to connect to the site.  If you see `unauthorized`, the deployment is successful.   If you see `I dream of being a web site.`, you need to set the `PARSE_MOUNT` in order for work with the client SDK's.  

Next, make sure you can create Parse objects.  You do not need a client Key to write new data:

```bash
curl -X POST   -H "X-Parse-Application-Id: myAppId"   -H "Content-Type: application/json"   -d '{"score":1337,"playerName":"Sean Plott","cheatMode":false}'   https://yourappname.herokuapp.com/parse/classes/GameScore
```

To read data back, you will need to specify the master key:

```bash
curl -X GET  -H "X-Parse-Application-Id: myAppId"   -H "X-Parse-Master-Key: abc"    https://yourappname.herokuapp.com/parse/classes/GameScore
```

You can also verify whether the objects were created by clicking on the MongoDB instance in the Heroku panel:

<img src="http://imgur.com/bbj2e9N.png"/>

<img src="http://imgur.com/snPqYkz.png"/>

### Enabling Client SDK integration

Make sure you have the latest Parse-Android SDK in your `app/build.gradle` file.  It should be at least `1.13.0`:

```gradle
dependencies {
    compile 'com.parse:parse-android:1.13.0'
}
```java

Modify your `Parse.initialize()` command to point to this Heroku server.  You must be on the latest Parse Android SDK to have these options:

public class ChatApplication extends Application {
    @Override
    public void onCreate() {
        super.onCreate();

        // set applicationId, clientKey, and server based on the values in the Heroku settings.
        Parse.initialize(new Parse.Configuration.Builder(this)
                .applicationId("myAppId")
                .clientKey("clientKey")
                .server("https://parse-testing-port.herokuapp.com").build());
    }
}
```
### Troubleshooting

#### Heroku Toolbelt

Download the Heroku Toolbelt app [here](https://toolbelt.heroku.com/). 

<img src="http://imgur.com/Ch0mZOK.png"/>

You can use this tool to view the system logs.  First, you must login with your Heroku login and password:

```bash
heroku login
```

You can then view the system logs by specifying the app name:
```bash
heroku logs -app <app name>
```


