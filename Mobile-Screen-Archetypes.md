## Overview

In mobile application development, mobile apps tend to adhere to a set of standard screen "archetypes" that appear again and again. There are over a dozen screen archetypes but there are six core archetypes that appear in nearly every app:

 * Login / Register
 * Stream
 * Detail
 * Creation
 * Profile
 * Settings

A large part of building mobile apps is about understanding how to build the UI and data back-end to power these common screen types above. Therefore when learning about Android development, you must ensure you have built at least all six of these core types when working on early projects. 

## Archetypes

The six archetypes are provided in more detail below.

### 1. Login

This archetype is focused on the "signed out" view. This is where the user signs in or registers for a new account. This can usually take the form of username / password although commonly includes third-party authentication such as facebook or twitter connect for easy access. 

Login screens usually require the following tasks:

 * Accepting varying types of form input from user for basic login / signup
 * Validation of form inputs for invalid data (i.e bad emails, duplicate emails)
 * Sending requests to server to authenticate or create new user accounts
 * Integrating third-party connection SDKs (i.e Facebook SDK)
 * Loading states during authentication or creation

**Examples:**

First, sign in pages:

<img src="http://www.android-app-patterns.com/img/screens/52935e9985697.png" height="420" alt="Sign In" />&nbsp;
<img src="http://www.android-app-patterns.com/img/sets/login-screens/155_2_login-screens.png" height="420" alt="Sign In 2" />&nbsp;
<img src="http://www.android-app-patterns.com/img/sets/login-screens/facebook-login.jpeg" height="420" alt="Sign In 3" />

And of course, signup:

<img src="http://www.android-app-patterns.com/img/screens/52409664366a5.png" height="420" alt="Sign Up" />&nbsp;
<img src="http://www.android-app-patterns.com/img/sets/signup/149_7_signup.png" height="420" alt="Sign Up 2" />&nbsp;
<img src="http://www.android-app-patterns.com/img/sets/signup/foodspotting-signup.png" height="420" alt="Sign Up 3" />

Check out more examples of login screens on [mobile app patterns](http://www.android-app-patterns.com/category/login-screens).

### 2. Stream

This archetype is focused on the primary content or data that a user consumes within the application. This is typically time-based displaying activities from other users that might be of interest. This is usually a list of discrete items which contain different data properties. The primary data is usually text or media content while secondary data includes the timestamp and the author.

Streams usually require the following tasks:

 * Sending network requests to retrieve lists of content data to display
 * Creating a list of items based on that source data including displaying text and media
 * Handling cases where the user wants to view more information on a piece of content
 * Allowing the user to take primary actions on this content such as deleting or editing
 * Optimizing the display of items such that scrolling the stream is smooth

**Examples:**

<img src="http://www.android-app-patterns.com/img/sets/streams/144_4_streams.png" height="420" alt="Stream" />&nbsp;
<img src="http://www.android-app-patterns.com/img/sets/streams/twitter-stream-2.png" height="420" alt="Stream" />&nbsp;
<img src="http://www.android-app-patterns.com/img/sets/streams/facebook-stream.jpeg" height="420" alt="Stream" />

Check out more examples of streams on [mobile app patterns](http://www.android-app-patterns.com/category/streams).

### 3. Detail

This archetype is focused on displaying all relevant information about a single discrete item within the application. This usually is a view that is reached when a user is consuming content in a stream that they would like to view in more detail or interact with. Typically this view contains additional data not displayed on the stream as well as actions a user can perform such as editing their items, liking or commenting on other user's content.

**Examples:**

<img src="http://www.android-app-patterns.com/img/screens/52935ea54ecfa.png" height="420" alt="Detail" />&nbsp;
<img src="http://www.android-app-patterns.com/img/sets/content-item/164_9_content-item.png" height="420" alt="Detail" />&nbsp;
<img src="http://www.android-app-patterns.com/img/sets/content-item/148_5_content-item.png" height="420" alt="Detail" />

Check out more examples of detail screens on [mobile app patterns](http://www.android-app-patterns.com/category/content-item).

### 4. Creation

This archetype is focused on allowing the user to create a new item by filling in all the properties for the item using a creation flow. This typically involves presenting the user with a series of input fields and allowing them to attach media such as a photo from the camera or metadata such as their location. Usually this is broken up into discrete steps and/or a great deal of fields are optional.

**Examples:**

<img src="http://www.android-app-patterns.com/img/sets/content-creation/144_15_content-creation.png" height="420" alt="Creation" />&nbsp;
<img src="http://www.android-app-patterns.com/img/screens/537fede2cda9d.png" height="420" alt="Creation" />&nbsp;
<img src="http://www.android-app-patterns.com/img/sets/content-creation/Google%20Plus_4_content-creation.png" height="420" alt="Creation" />

Check out more examples of creation screens on [mobile app patterns](http://www.android-app-patterns.com/category/content-creation).

### 5. Profile

This archetype is focused on allowing the user to view information about their own account, view their own content, and provide them account related action items. Typically, the profile contains important statistics about the user (i.e number of followers), displays recent content, and provides access to actions such as editing their profile, changing their picture, logging out, etc.

**Examples:**

<img src="http://www.android-app-patterns.com/img/sets/profile-pages/159_15_profile-pages.png" height="420" alt="Profile" />&nbsp;
<img src="http://www.android-app-patterns.com/img/sets/profile-pages/149_5_profile-pages.png" height="420" alt="Profile" />&nbsp;
<img src="http://www.android-app-patterns.com/img/sets/profile-pages/Pinterest_4_profile-pages.png" height="420" alt="Profile" />

Check out more examples of profile screens on [mobile app patterns](http://www.android-app-patterns.com/category/profile-pages).

### 6. Settings

This archetype is focused on giving the user the ability to tune preferences associated with their account and/or the behavior of the app. The settings available range from notification and privacy settings, to profile settings such as username or email and to preferences affecting the behavior of the app.

**Examples:**

<img src="http://www.android-app-patterns.com/img/sets/settings/148_12_settings.png" height="420" alt="Profile" />&nbsp;
<img src="http://www.android-app-patterns.com/img/sets/settings/155_10_settings.png" height="420" alt="Profile" />&nbsp;
<img src="http://www.android-app-patterns.com/img/sets/settings/Pinterest_1_settings.png" height="420" alt="Profile" />

Check out more examples of settings screens on [mobile app patterns](http://www.android-app-patterns.com/category/settings).

## References

* <http://www.android-app-patterns.com/>