## Overview

When building Android apps, your app is bound to crash from time to time or exhibit strange unexpected behavior. You know you have experienced a runtime exception when you see this in your emulator or device:

<img src="https://i.imgur.com/lMTPBfk.png" />

Don't worry though! This is totally normal and there's a specific set of steps you can take to solve these. Refer to our guide below and/or [these debugging slides](https://docs.google.com/presentation/d/1DUigTm6Uh43vatHkB4rFkVVIt1zf7zB7Z5tpGTy2oFY/edit#slide=id.g38d66bc5a_0175) for more a detailed look at debugging crashes and investigating unexpected problems with your app. 

### Debugging Mindset

As an Android developer, you'll need to cultivate a "debugging mindset" as well as build up defensive programming practices that make writing error-prone code less likely. In addition, you'll often find yourself in the role of a coding investigator in order to understand where and why an app is crashing or not working as expected. A few key principles about debugging are captured below:

* Just because a program runs, **doesn’t mean it’s going to work as you expected**.  This class of issues are known as **runtime errors**. This is contrast to compile-time errors which prevent an app from running and are often easier to catch.  
* Think of debugging as an **opportunity to fill gaps in knowledge**. Debugging is an opportunity to understand your app better than you did before and hopefully sharpen your ability to write correct code in the future by programming defensively. 
* Debugging is a vital part of the software development process. Often you may find yourself on some days spending **more time debugging crashes or unexpected behavior then writing new code**. This is entirely normal as a mobile engineer.

### Debugging Principles

The following high-level principles should be applied when faced with an unexpected app behavior during investigation:

* **Replicate**. Convince yourself that there is an issue in the code that can be repeatedly reproduced by following the same steps. Before assuming the code is broken, try restarting the emulator, trying the app on a device and/or fully re-building the app.
* **Reduce**. Try to isolate or reduce the code surrounding the issue and figure out the simplest way to reproduce what’s occurring. Comment out or remove extraneous code that could be complicating the issue.
* **Research**. If you are running into a major unexpected issue, you are probably not alone. Search Google for the behavior using any descriptive identifiers. Visit the issue tracker for the component you are seeing issues with. Search [stackoverflow](http://stackoverflow.com/) for posts about the same issue.

Armed with this mindset and the above principles, let's take a look at how to debug and investigate issues that arise within our apps.

## Debugging Procedure

When you see your app crash and close, the basic steps for diagnosing and resolving this are outlined below:

1. Find the final exception stack trace within the Android Monitor (logcat)
2. Identify the exception type, message, and file with line number
3. Open the file within your app and find the line number
4. Look at the exception type and message to diagnose the problem
5. If the problem is not familiar, google around searching for answers
6. Make fixes based on the proposed solutions and re-run the app
7. Repeat until the crash no longer occurs

This process nearly always starts with an unexpected app crash as we'll see below.

### Witnessing the Crash

Suppose we were building a simple movie trailer app called Flixster that lets the users browse new movies and watch trailers from Youtube. Imagine we ran the app, and we wanted to play the trailer and we saw this crash instead:

<img src="https://i.imgur.com/Z22GWZe.gif" width="300" />

First off though when you see the crash dialog, **don't press OK** on that dialog until **after you've already went through these steps below to identify the stacktrace details**.

### Setting Up Error Filter

First, within Android Studio, be sure to setup your Android Monitor to filter for "Errors" only to reduce noise:

<img src="https://i.imgur.com/fbUq9ok.gif" width="900" />

1. Select "Error" as the log level to display
2. Select "Show only selected application" to filter messages

This will set you up to see only serious issues as they come up.

**Note:** See [this blog post](http://andressjsu.blogspot.com/2016/07/android-debugging.html) for improving the coloring of errors or logs in `LogCat` and for other related tools. 

### Find the Stack Trace

Now let's go into Android Studio and select open up the "Android Monitor". Expand the monitor so you can read the log messages easily. 

<img src="https://i.imgur.com/1atGM14.gif" width="900" />

1. Scroll to **the bottom of the error** looking for a line that says `Caused by` all the way at the bottom of the stack trace block. The "original cause" towards the bottom of the block is the important part of the error, ignore most of the rest of the stack trace above that.
2. Locate that bottom-most `Caused by` line as well as the line that has the blue link with the name of your activity i.e `VideoActivity.java:13`. Copy them onto your clipboard and paste them both into a separate text file for easy review.

In this case the bottom-most "Caused by" line and the adjacent blue file link copied together looks like:

```
Caused by: java.lang.NullPointerException: Attempt to invoke virtual method
 'java.lang.String android.content.Intent.getStringExtra(java.lang.String)' 
 on a null object reference 
      at com.codepath.flixster.VideoActivity.<init>(VideoActivity.java:13)
```

Note that the top part of the stack trace block above that line noting `FATAL EXCEPTION` and the first portion with the `java.lang.RuntimeException` are much more generic errors and are less useful than that final bottom-most "Caused by" exception captured above which points to the real culprit.

### Identify the Exception

Next, we need to identify the actual issue that this stack trace is warning us about. To do this, we need to identify the following elements of the problem:

1. File name
2. Line number
3. Exception type
4. Exception message

Consider the exception we copied above:

<img src="https://i.imgur.com/xHOq5oE.gif" width="900" />

This exception needs to be translated to identifying the following elements:

| Element               | Identified culprit |
| -------               | ----               |
| **Filename**          | VideoActivity.java |
| **Line Number**       | Line 13            |
| **Exception Type**    | `java.lang.NullPointerException` |
| **Exception Message** | `Attempt to invoke virtual method 'java.lang.String android.content.Intent.getStringExtra(java.lang.String)' on a null object reference`

### Open the Offending File 

Next, we need to open up the offending file and line number. In this case, the `VideoActivity.java` to line 13 which looks like this:

```java
public class VideoActivity extends YouTubeBaseActivity{
    // -----> LINE 13 is immediately below <--------
    String videoKey = getIntent().getStringExtra("video_key");

    // ...
    protected void onCreate(Bundle savedInstanceState) {
       // ....
    }
}
```

Therefore, we know the crash is happening on line 13:

```java
String videoKey = getIntent().getStringExtra("video_key");
```

This is a great first step to now solving the problem because we know exactly where the exception is being triggered.

### Diagnose the Problem

Next, we need to use the exception type and message to diagnose the problem at that line. The type in this case is `java.lang.NullPointerException` and the message was `Attempt to invoke virtual method 'java.lang.String android.content.Intent.getStringExtra(java.lang.String)' on a null object reference`.

The type is the most important part because there are a limited number of types. If the type is `java.lang.NullPointerException` then we know that **some object is null when it shouldn't be**. In other words, we are calling a method on an object or passing an object that is null (has no value). This usually means **we forgot to set the value** or **the value is being set incorrectly**. 

The message above gives you the specifics that the method `getStringExtra` is being called on a null object. This tell us that `getIntent()` is actually null since this is the object `getStringExtra` is being called on. That might seem strange, why is the `getIntent()` null here? How do we fix this?

### Google for the Exception

Often we won't know what is going wrong even after we've diagnosed the issue. We know that "`getIntent()` is null and shouldn't be". But we don't know why or how to fix.

At this stage, we need to **google cleverly for the solution**. Any problem you have, stackoverflow probably has the answer. We need to identify a search query that is likely to find us answers. The recipe is generally a query like `android [exception type] [partial exception message]`. The type in this case is `java.lang.NullPointerException` and the message was `Attempt to invoke virtual method 'java.lang.String android.content.Intent.getStringExtra(java.lang.String)' on a null object reference`. 

We might start by googling: `android java.lang.NullPointerException android.content.Intent.getStringExtra(java.lang.String)`. [These results get returned](https://www.google.com/#q=android+java.lang.NullPointerException+android.content.Intent.getStringExtra\(java.lang.String\)). 

You generally want to look for "stackoverflow" links but in this case, the first result [is this other forum](http://www.coderanch.com/t/654627/Android/Mobile/string-intent-null-object-reference). 

Scroll to the bottom and you will find this message by the original author:

> I realize my mistake now. I was calling the getIntent() method outside of the onCreate method. 
> As soon as I moved the code calling getIntent() inside the onCreate method it's all working fine.

Let's give this solution a try! He seems to suggest the issue is that `getIntent()` is being called outside of the `onCreate` block and we need to move it inside that method.

### Address the Problem

Based on the advice given on stackoverflow or other sites from Google, we can then apply a potential fix. In this case, the `VideoActivity.java` can then be changed to:

```java
public class VideoActivity extends YouTubeBaseActivity{
    // ...move the call into the onCreate block
    String videoKey; // declare here
    protected void onCreate(Bundle savedInstanceState) {
       // ....
       // set the value inside the block
       videoKey = getIntent().getStringExtra("video_key");
    }
}
```

Now, we can run the app and see if things work as expected!

### Verify the Error is Fixed

Let's re-run the app and try this out again:

<img src="https://i.imgur.com/qtcxeLA.gif" width="600" />

Great! The exception seems to have been fixed!

### Rinse and Repeat

Sometimes the first fix we try after googling doesn't work. Or it makes things worse. Sometimes the solutions are wrong. Sometimes we have to try different queries. Here's a few guidelines:

 * It's normal to have to try 3-4 solutions before one actually works.
 * Try to stick with [stackoverflow](http://stackoverflow.com/) results from Google first
 * Open up multiple stackoverflow pages and **look for answers you see repeated multiple times**
 * Look for stackoverflow answers that have a  **green check mark and many upvotes**.

Don't get discouraged! Look through a handful of sites and you are bound to find the solution in 80% of cases as long as you have the **right query**. Try multiple variations of your query until you find the right solution.

In certain cases, we need to investigate the problem further. The methods for investigating why something is broken are outlined below.

## Investigation Methodologies

In addition to finding and googling errors, often additional methods have to be applied to figure out what is going wrong. This usually becomes helpful when something isn't as we expected in the app and we need to figure out why. The three most common investigation techniques in Android are:

1. **Toasts** - Using [[toasts|Displaying-Toasts]] to alert us to failures
2. **Logging** - Using the [logging system](https://developer.android.com/studio/debug/index.html#systemLogWrite) to print out values
3. **Breakpoints** - Using the [breakpoint system](https://developer.android.com/studio/debug/index.html#breakPoints) to investigate values

Both methods are intended for us to be able to determine why a value isn't as we expect. Let's take a look at how to use each. 

Let's start with the same app from above and suppose we are trying to get the app so that when the image at the top of the movie details page is clicked, then the movie trailer begins playing.

```java
public class InfoActivity extends YouTubeBaseActivity {
    private ImageView backdropTrailer;
    private String videoKey;

    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_info);
        // ...
        backdropTrailer = (ImageView) findViewById(R.id.ivPoster);
        // Trigger video when image is clicked
        backdropTrailer.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) { 
                // Launch youtube video with key if not null
                if (videoKey != null) {
                    Intent i = new Intent(InfoActivity.this, VideoActivity.class);
                    i.putExtra("videoKey", videoKey);
                    startActivity(i);
                }
            }
        });
    }
}
```

Unfortunately when testing, we see that the trailer does not come up as expected when we run the app and click the top image:

<img src="https://i.imgur.com/qLSCwQK.gif" width="300" />

Why doesn't the video launch when you click on the image as expected? Let's investigate.

### Notifying Failure with Toasts

The video activity isn't launching when the user presses the image but let's see if we can narrow down the problem. First, let's add a [[toast message|Displaying-Toasts]] to make sure we can begin to understand the issue. Toasts are messages that popup within the app. For example:

```java
Toast.makeText(this, "Message saved as draft.", Toast.LENGTH_SHORT).show();
```

would produce:

![Toast](http://developer.android.com/images/toast.png)

In `InfoActivity.java`, we will add the following toast message inside the `onClick` listener method:

```java
public class InfoActivity extends YouTubeBaseActivity {
    // ...
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_info);
        // ...
        backdropTrailer = (ImageView) findViewById(R.id.ivPoster);
        backdropTrailer.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) { 
                // Launch youtube video with key if not null
                if (videoKey != null) {
                    Intent i = new Intent(InfoActivity.this, VideoActivity.class);
                    i.putExtra("videoKey", videoKey);
                    startActivity(i);
                } else {
                    // ----> ADD A TOAST HERE. This means video key is null
                    Toast.makeText(InfoActivity.this, "The key is null!", Toast.LENGTH_SHORT).show();
                } 
            }
        });
    }
}
```

With the toast added, running this again, we see the problem is confirmed:


<img src="https://i.imgur.com/kXe4fLF.gif" width="300" />

The problem is that the **youtube video key is null** where as it should have a value. We haven't fixed the problem, but we at least know what the initial issue is. 

Using a [[toast message|Displaying-Toasts]] we can easily notify ourselves when something goes wrong in our app. We don't even have to check the logs. Be sure to **add toasts whenever there are failure cases** for networking, intents, persistence, etc so you can easily spot when things go wrong. 

### Investigating using Logging

Next, let's fix the problem the toast clarified for us. We now know the problem is that the **youtube video key is null** where as it should have a value. Let's take a look at the method that fetches the video key for us:

```java
public void fetchMovies(int videoId) {
    // URL should be: https://api.themoviedb.org/3/movie/246655/videos?api_key=KEY
    String url = "https://api.themoviedb.org/3/movie" + movie_id + "/videos?api_key=" + KEY;
    client.get(url, new JsonHttpResponseHandler(){
        @Override
        public void onSuccess(int statusCode, Headers headers, JSON response) {
            JSONArray movieJsonResults = null;
            try {
                movieJsonResults = response.getJSONArray("results");
                JSONObject result = movieJsonResults.getJSONObject(0);
                // Get the key from the JSON
                videoKey = result.getString("key");
            } catch (JSONException e) {
                e.printStackTrace();
            }
        }
    });
}
```

This method is somehow not fetching the key that we'd expect. Let's start by logging inside the `onSuccess` method to see if we are getting inside there as we expect. 

The [Android logger](https://developer.android.com/reference/android/util/Log.html) is pretty easy to use. The log options are as follows:

| Level   | Method   |
| -----   | -----    |
| Verbose | `Log.v()` | 
| Debug   | `Log.d()` |
| Info    | `Log.i()` |
| Warn    | `Log.w()` |
| Error   | `Log.e()` |

They are sorted by relevance with `Log.i()` being the least important one. The first parameter of these methods is the category string (can be any label we choose) and the second is the actual message to display. 

We can use the logger by adding two lines to the code: one at the top before the network call and one inside `onSuccess` to see if they both display:

```java
Log.e("VIDEOS", "HELLO"); // <------------ LOGGING
client.get(url, new JsonHttpResponseHandler(){
    @Override
    public void onSuccess(int statusCode, Headers headers, JSON response) {
        JSONArray movieJsonResults = null;
        try {
            // LOG the entire JSON response string below
            Log.e("VIDEOS", response.toString()); // <------------ LOGGING
            movieJsonResults = response.getJSONArray("results");
            JSONObject result = movieJsonResults.getJSONObject(0);
            videoKey = result.getString("key");
        } catch (JSONException e) {
            e.printStackTrace();
        }
    }
});
```

When running the app, the first log does show up as expected but the one inside `onSuccess` does **not show up at all** in the Android monitor:

<img src="https://i.imgur.com/LFOUrf5.gif" width="900" />

Notice that we see `HELLO` but no other line logs. This means that we now know that **the onSuccess method is never called**. This means our network request sent out is failing for some reason. We are one step closer to fixing this issue.

### Investigating using Breakpoints

In order to investigate why the network request sent out is failing, we are going to bring out the most powerful debugging tool we have which is the breakpointing engine in Android Studio which allows us to stop the app and investigate our environment thoroughly.

First, we have to decide at which points we want to stop the app and investigate. This is done by setting breakpoints. Let's set two breakpoints to investigate the network request:

<img src="https://i.imgur.com/63e6ZMI.gif " width="600" />

Now, we need to run the app using the "debugger" rather than the normal run command:

<img src="https://i.imgur.com/A4gqK6r.gif" />

Once the debugger connects, we can click on the movie to trigger the code to run. Once the code hits the spot with a breakpoint, the entire code pauses and let's us inspect everything:

<img src="https://i.imgur.com/TFfpOSi.gif" />

Here we were able to inspect the URL and compare the URL against the expected value. Our actual URL was "https://api.themoviedb.org/3/movie246655/videos?api_key=KEY" while the expected URL was "https://api.themoviedb.org/3/movie/246655/videos?api_key=KEY". Extremely subtle difference. Can you spot it?

Then we can hit "resume" (<img src="https://i.imgur.com/shxFUhJ.png" />) to continue until the next breakpoint or stop debugging (<img src="https://i.imgur.com/PETVfws.png" />) to end the session.

Breakpoints are incredibly powerful and worthy of additional investigation. To learn more about breakpoints, check out [this official Android Guide on debugging](https://developer.android.com/studio/debug/index.html#breakPoints) and [this third-party breakpoints guide](https://www.learnhowtoprogram.com/android/user-interface-basics-637d41b1-35dc-400a-bcc3-65794760474d/debugging-breakpoints-and-the-android-debugger).

### Fixing the Issue

Now that we know the issue is the URL being incorrectly formed, we can fix that in the original code in `InfoActivity.java`:

```java
public void fetchMovies(int videoId) {
    // ADD trailing slash to the URL to fix the issue
    String url = "https://api.themoviedb.org/3/movie/" + // <----- add trailing slash
       movie_id + "/videos?api_key=" + KEY;
    client.get(url, new JsonHttpResponseHandler(){ 
      // ...same as before...
    });
}    
```

and remove any old log statements we don't need. Now, we can try running the app again:

<img src="https://i.imgur.com/qtcxeLA.gif" width="600" />

Great! The video now plays exactly as expected!

### Wrapping Up

In this section, we looked at three important ways to investigate your application:

1. **Toasts** - Display messages inside the app emulator. Good for notifying you of common failure cases.
2. **Logs** - Display messages in the Android monitor. Good to printing out values and seeing if code is running.
3. **Breakpoints** - Halt execution of your code with breakpoints and inspect the entire environment to figure out which values are incorrect or what code hasn't run. 

With these tools and the power of Google, you should be well-equipped to debug any errors or issues that come up as you are developing your apps. Good luck!

## References

* <http://andressjsu.blogspot.com/2016/07/android-debugging.html>
* <https://developer.android.com/studio/debug/index.html>
* <http://blog.strv.com/debugging-in-android-studio-as/>
* <https://www.youtube.com/watch?v=2c1L19ZP5Qg>
* <https://docs.google.com/presentation/d/1DUigTm6Uh43vatHkB4rFkVVIt1zf7zB7Z5tpGTy2oFY/edit?usp=sharing>
