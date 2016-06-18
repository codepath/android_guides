## Overview

When building Android apps, your app is bound to crash from time to time. You know you have experienced a runtime exception when you see this in your emulator or device:

<img src="http://i.imgur.com/lMTPBfk.png" />

Don't worry! This is normal and there's a specific set of steps you can take to solve this. 

### Setting Up Error Filter

First, within Android Studio, be sure to setup your Android Monitor to filter for "Errors" only to reduce noise:

<img src="http://i.imgur.com/fbUq9ok.gif" width="900" />

1. Select "Error" as the log level to display
2. Select "Show only selected application" to filter messages

This will set you up to see only serious issues as they come up.

## Debugging Procedure

Now that we filtered our logs for errors. Let's get started fixing our crash! The steps are outlined below:

1. Find the final exception stack trace within the Android Monitor (logcat)
2. Identify the exception type, message, and file with line number
3. Open the file within your app and find the line number
4. Look at the exception type and message to diagnose the problem
5. If the problem is not familiar or a `NullPointerException`, google for the message
6. Make fixes based on the issue and re-run the app
7. Repeat until the crash no longer occurs

First off though when you see the crash dialog, **don't press OK** on the dialog when you see that error until **after you've already went through these steps**.

### Witnessing the Crash

First, suppose we were building a simple movie trailer app that let's the users browse new movies and watch trailers from Youtube. Imagine we ran the app, and we wanted to play the trailer and we saw this crash instead:

<img src="http://i.imgur.com/Z22GWZe.gif" width="300" />

### Find the Stack Trace

Don't click "OK" on the crash. Instead, go into Android Studio and select open up the "Android Monitor". Expand the monitor so you can read the log messages easily. 

<img src="http://i.imgur.com/1atGM14.gif" width="900" />

1. Scroll to **the bottom of the error** looking for a line that says `Caused by` all the way at the bottom. The "first cause" is the important part of the error, ignore most of the rest of the stack trace.
2. Find the first `Caused by` line and also the line that has a blue link with the name of your activity `VideoActivity.java:13`. Copy them onto your clipboard and paste them into a separate text file.

### Identify the Exception

Next, we need to identify the actual issue that this stack trace is warning us about. To do this, we need to identify the following elements of the problem:

1. File name
2. Line number
3. Exception type
4. Exception message

Consider the exception we copied above:

<img src="http://i.imgur.com/xHOq5oE.gif" width="900" />

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

```
String videoKey = getIntent().getStringExtra("video_key");
```

This is a great first step to now solving the problem because we know exactly where the exception is being triggered.

### Diagnose the Problem

Next, we need to use the exception type and message to diagnose the problem at that line. The type in this case is `java.lang.NullPointerException` and the message was `Attempt to invoke virtual method 'java.lang.String android.content.Intent.getStringExtra(java.lang.String)' on a null object reference`.

The type is the most important part because there are a limited number of types. If the type is `java.lang.NullPointerException` then we know that **some object is null when it shouldn't be**. In other words, we are calling a method on an object or passing an object that is null (has no value). This usually means **we forget to set the value** or **the value is being set incorrectly**. 

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

<img src="http://i.imgur.com/qtcxeLA.gif" width="600" />

Great! The exception seems to have been fixed!

### Rinse and Repeat

Sometimes the first fix we try after googling doesn't work. Or it makes things worse. Sometimes the solutions are wrong. Sometimes we have to try different queries. Here's a few guidelines:

 * It's normal to have to try 3-4 solutions before one actually works.
 * Try to stick with [stackoverflow](http://stackoverflow.com/) results from Google first
 * Open up multiple stackoverflow answers and **look for answers you see multiple times**
 * Look for stackoverflow answers that have a green check mark and **many upvotes**.

Don't get discouraged! Look through a handful of sites and you are bound to find the solution in 80% of cases as long as you have the **right query**. Try multiple variations of your query until you find the right solution.