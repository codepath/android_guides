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

#### 

Next, let's drag on the basic elements of our login screen:

 * TextView (To use highlight...)
 * ImageButton (Facebook)
 * ImageButton (LinkedIn)
 * TextView (Why not email?)
 * TextView (Highlight is based...)
 * TextView (Please let us know...)
 * TextView (We won't post things without...)

The RelativeLayout might look like the following:

```xml

```

## References

* <http://www.android-app-patterns.com/category/login-screens>
* <http://www.android-app-patterns.com/img/sets/login-screens/155_2_login-screens.png>