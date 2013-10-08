## Overview

This guide is going to walk us through the step-by-step process of recreating this screenshot in an Android app:

<img width="400" src="http://www.android-app-patterns.com/img/sets/login-screens/155_2_login-screens.png" />

### Cutting Assets

First, I will make the downloadable assets available. Keep in mind these were stolen right out of the screenshot and therefore their quality is pretty poor. But it demonstrates the concepts nonetheless. To create the mockup, we need the following assets:

 * [Highlight Launcher Icon](http://i.imgur.com/pysgYQU.png/ic_highlight.png)
 * [Facebook Connect](http://i.imgur.com/TJxrci5.png/facebook_connect.png)
 * [LinkedIn Connect](http://i.imgur.com/dhYw2xF.png/linkedin_connect.png)

Download those and **drag the button images into the drawable-xhdpi folder**. Note that in production applications you would multiple sizes of these images for [different image densities](http://developer.android.com/guide/practices/screens_support.html#DesigningResources). We are going to ignore that and provide only extra-high density for this demo (they will be resampled for other densities).

### Create Project

Generate a new Android project that uses the highlight icon as the launcher icon:

<img width="400" src="http://imgur.com/GiWY8gB.png" />

Generate the project with no additional changes.

### Start Basic Layout

There are **seven views** that will be placed within the layout:

 * TextView (To use highlight...)
 * ImageButton (Facebook)
 * ImageButton (LinkedIn)
 * TextView (Why not email?)
 * TextView (Highlight is based...)
 * TextView (Please let us know...)
 * TextView (We won't post things without...)

#### Step 1: Layout Background

Let's start by setting the layout background to white:

```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="#ffffff"
    ...>
</RelativeLayout>
```

Now the entire background for the screen appears white like the mockup.

#### Step 2: ImageButtons

Let's start with the ImageButtons. Drag the two image buttons to the screen and select facebook and linkedin as sources respectively. Now go into the XML and tweak the `background` property of each to "@null" to remove the button borders.

```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    ...>

    <ImageButton
        android:id="@+id/btnFacebook"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentTop="true"
        android:layout_centerHorizontal="true"
        android:background="@null"
        android:contentDescription="@string/facebook_desc"
        android:src="@drawable/facebook_connect" />

    <ImageButton
        android:id="@+id/btnLinkedIn"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignLeft="@+id/btnFacebook"
        android:layout_below="@+id/btnFacebook"
        android:layout_marginTop="5dp"
        android:background="@null"
        android:contentDescription="@string/linkedin_desc"
        android:src="@drawable/linkedin_connect" />

</RelativeLayout>
```

Here we've aligned the first button to the top of the parent, used `layout_centerHorizontal`, set `android:background` to null to avoid the button border, set text descriptions for `android:contentDescription` and assigned the `android:src` to the respective buttons. Now our screen looks like:

<img width="400" src="http://i.imgur.com/AffrW7w.png" />

#### Step 3: Basic TextViews

Now, let's drag in the TextViews for the screen. (There's five of these). First, let's store the strings for each of these TextView controls in the `res/values/strings.xml` file for use in our TextViews later:

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string name="highlight_preamble">To use Highlight, please sign in with one of the services below:</string>
    <string name="why_not_email_title">Why not email?</string>
    <string name="why_not_email_body">Highlight is based on real identity and mutual friends. 
        Using these services allows us to show you how you\'re connected to the people around you. 
        It also makes it super easy to create a Highlight profile.
    </string>
    <string name="feedback_label">
    <![CDATA[
        Please <a href="http://highlight.com">let us know</a> if you have feedback on this or if 
        you would like to log in with another identity service. Thanks!   
    ]]>
    </string>
    <string name="permission_label">We won\'t post things without your permission.</string>
</resources>
```

Now let's drag on the different TextView and set the appropriate text values. Without any styling or proper positioning it might look like this screen:

<img width="400" src="http://i.imgur.com/udYbT3n.png" />

#### TextView Styling

Let's add basic styling:

 * TextView should be set to "14sp" and `textColor` of "#7e7e7e"
 * "To use Highlight" view should be set to `textColor` of "#575757"
 * "Why not email?" view `textStyle` should be set to bold 
 * "We won't post things" should be set to `textColor` of "#bbbbbb" and `textSize` of 12sp, and use `layout_centerHorizontal` to center the text.
 * Estimate margin between elements based on mockup

The resulting XML might look like:

```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="#ffffff"
    ... >

    <TextView
        android:id="@+id/textView1"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentLeft="true"
        android:layout_alignParentTop="true"
        android:text="@string/highlight_preamble"
        android:textColor="#575757"
        android:textSize="14sp" />

    <ImageButton
        android:id="@+id/btnFacebook"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignLeft="@+id/textView1"
        android:layout_below="@+id/textView1"
        android:layout_marginTop="10dp"
        android:background="@null"
        android:contentDescription="@string/facebook_desc"
        android:src="@drawable/facebook_connect" />

    <ImageButton
        android:id="@+id/btnLinkedIn"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignLeft="@+id/btnFacebook"
        android:layout_below="@+id/btnFacebook"
        android:layout_marginTop="10dp"
        android:background="@null"
        android:contentDescription="@string/linkedin_desc"
        android:src="@drawable/linkedin_connect" />

    <TextView
        android:id="@+id/textView2"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignLeft="@+id/btnLinkedIn"
        android:layout_below="@+id/btnLinkedIn"
        android:layout_marginTop="23dp"
        android:text="@string/why_not_email_title"
        android:textColor="#7e7e7e"
        android:textStyle="bold" />

    <TextView
        android:id="@+id/textView3"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignLeft="@+id/textView2"
        android:layout_below="@+id/textView2"
        android:layout_marginTop="10dp"
        android:text="@string/why_not_email_body"
        android:textColor="#7e7e7e" />

    <TextView
        android:id="@+id/textView4"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignLeft="@+id/textView3"
        android:layout_below="@+id/textView3"
        android:layout_marginTop="10dp"
        android:text="@string/feedback_label"
        android:textColor="#7e7e7e" />

    <TextView
        android:id="@+id/textView5"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_below="@+id/textView4"
        android:layout_centerHorizontal="true"
        android:layout_marginTop="10dp"
        android:text="@string/permission_label"
        android:textColor="#bbbbbb"
        android:textSize="12sp" />

</RelativeLayout>
```

## References

* <http://www.android-app-patterns.com/category/login-screens>
* <http://www.android-app-patterns.com/img/sets/login-screens/155_2_login-screens.png>