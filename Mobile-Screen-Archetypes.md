## Overview

In mobile application development, mobile apps tend to adhere to a set of standard screen "archetypes" that appear again and again. There are over a dozen screen archetypes but there are six core archetypes that appear in nearly every app:

 * [Login / Register](#1-login) - User signs up or logs into their account
 * [Stream](#2-stream) - User can scroll through important resources in a list
 * [Detail](#3-detail) - User can view the specifics of a particular resource
 * [Creation](#4-creation) - User can create a new resource
 * [Profile](#5-profile) - User can view their identity and stats
 * [Settings](#6-settings) - User can configure app options

In addition to these six, there's other "extended" archetypes often encountered, including:

 * **Splash Pages** - Displayed while app is loading (and generally discouraged)
 * **Onboarding** - First-time user visual tutorial for how the app works 
 * **Map View** - Often visualizing location-based information 
 * **Messaging** - Real-time chat and group conversations
 * **Calendar**  - Visualize dates or events into calendar form
 * **Media Players** - Allowing the control of media playback

A large part of building mobile apps is about understanding how to build the UI and data back-end to power these common screen types above. Therefore when learning about Android development, you must ensure you have built at least all six of these core types when working on early projects. 

## Archetypes

The six archetypes are provided in more detail below.

### 1. Login

This archetype is focused on the "signed out" view. This is where the user signs in or registers for a new account. This can usually take the form of username / password although commonly includes third-party authentication such as facebook or twitter connect for easy access. 

Login screens usually require the following components:

 * Accepting varying types of form input from user for basic login / signup
   * See: [[text fields|Working with the EditText]], [[input fields|Working with Input Views]]
 * Validation of form inputs for invalid data (i.e bad emails, duplicate emails)
 * Sending requests to server to authenticate or create new user accounts 
   * See: [[sending requests|Sending and Managing Network Requests]]
 * Integrating third-party connection SDKs (i.e Facebook SDK) 
   * See: [[parse sdk|Building Data driven Apps with Parse]]
 * Loading states during authentication or creation 
   * See: [[progress bars|Handling ProgressBars]]

**Examples:**

First, sign in pages:

<img src="http://cdn.pttrns.com/357/6082_w250.jpg" height="420" alt="Sign In" />&nbsp;
<img src="http://cdn.pttrns.com/356/6045_w250.jpg" height="420" alt="Sign In 2" />&nbsp;
<img src="http://cdn.pttrns.com/422/4511_w250.jpg" height="420" alt="Sign In 3" />

And of course, signup:

<img src="http://cdn.pttrns.com/399/4181_w250.jpg" height="420" alt="Sign Up" />&nbsp;
<img src="http://cdn.pttrns.com/531/6208_w250.jpg" height="420" alt="Sign Up 2" />&nbsp;
<img src="http://cdn.pttrns.com/336/6137_w250.jpg" height="420" alt="Sign Up 3" />

Check out more examples of login screens on [pttrns](https://pttrns.com/android-patterns?scid=17) and sign up pages on [pttrns](https://pttrns.com/android-patterns?scid=9).

### 2. Stream

This archetype is focused on the primary content or data that a user consumes within the application. This is typically time-based displaying activities from other users that might be of interest. This is usually a list of discrete items which contain different data properties. The primary data is usually text or media content while secondary data includes the timestamp and the author.

Streams usually require the following components:

 * Sending network requests to retrieve lists of content data to display 
   * See: [[sending requests|Sending and Managing Network Requests]]
 * Creating a list of items based on that source data including displaying text and media
   * See: [[listview|Using an ArrayAdapter with ListView]]
 * Endless scrolling which paginates as user consumes content
   * See: [[endless scrolling|Endless-Scrolling-with-AdapterViews-and-RecyclerView]]
 * Handling cases where the user wants to view more information on a piece of content
   * See: [[events|Basic-Event-Listeners]]
 * Allowing the user to take primary actions on this content such as deleting or editing 
 * Optimizing the display of items such that scrolling the stream is smooth 
   * See: [[viewholder|Using-an-ArrayAdapter-with-ListView#improving-performance-with-the-viewholder-pattern]]

**Examples:**

<img src="http://cdn.pttrns.com/534/6259_w250.jpg" height="420" alt="Stream" />&nbsp;
<img src="http://cdn.pttrns.com/396/4146_w250.jpg" height="420" alt="Stream" />&nbsp;
<img src="http://cdn.pttrns.com/353/6108_w250.jpg" height="420" alt="Stream" />

Check out more examples of streams on [pttrns](https://pttrns.com/android-patterns?scid=11).

### 3. Detail

This archetype is focused on displaying all relevant information about a single discrete item within the application. This usually is a view that is reached when a user is consuming content in a stream that they would like to view in more detail or interact with. Typically this view contains additional data not displayed on the stream as well as actions a user can perform such as editing their items, liking or commenting on other user's content.

Detail views usually require the following components:

* Sending network request to retrieve additional details or media for the data item 
  * See: [[sending requests|Sending and Managing Network Requests]]
* Action buttons that allow user to interact with the item 
  * See: [[events|Basic-Event-Listeners]]
* Option for user to share the content out to other apps 
  * See: [[sharing|Sharing Content with Intents]]
* Scrollable text or media content to read about the item 
  * See: [[scrollview|Working with the ScrollView]]

**Examples:**

<img src="http://68.media.tumblr.com/0738655266cd17a0c96d5943e8cef331/tumblr_o60r3ewWOW1r2wjwko5_250.png" height="420" alt="Detail" />&nbsp;
<img src="http://cdn.pttrns.com/351/3493_w250.jpg" height="420" alt="Detail" />&nbsp;
<img src="http://68.media.tumblr.com/3bf1585c110ec3493c5756296eaaa4bf/tumblr_ngodqroIOp1r2wjwko5_250.png" height="420" alt="Detail" />

Check out more examples of detail screens on [pttrns](https://pttrns.com/android-patterns?scid=54).

### 4. Creation

This archetype is focused on allowing the user to create a new item by filling in all the properties for the item using a creation flow. This typically involves presenting the user with a series of input fields and allowing them to attach media such as a photo from the camera or metadata such as their location. Usually this is broken up into discrete steps and/or a great deal of fields are optional.

Creation screens usually require the following tasks:

* Input fields of various types to collect information 
  * See: [[input fields|Working with Input Views]]
* Ability to validate fields for correctness before creation
* Sending network request to create new valid content item 
  * See: [[sending requests|Sending and Managing Network Requests]]
* Option to capture a photo or select from photo gallery 
  * See: [[camera and media|Accessing the Camera and Stored Media]]
* Option to capture current location of device 
  * See: [[location|Retrieving Location with LocationServices API]]
* Option to share or include friends related to the item 
  * See: [[parse sdk|Building Data driven Apps with Parse]]

**Examples:**

<img src="http://68.media.tumblr.com/e1df69f85950abdfc18209437e075496/tumblr_nubrrgrwuI1r2wjwko5_250.png" height="420" alt="Creation" />&nbsp;
<img src="http://68.media.tumblr.com/24ae04fb6c007c9bc771cc269e97eb87/tumblr_ntrvs7zGvr1r2wjwko5_250.png" height="420" alt="Creation" />&nbsp;
<img src="http://68.media.tumblr.com/ecfd265ee9185621a4df6ca1a6111f0f/tumblr_mz9930Lwvb1r2wjwko2_250.png" height="420" alt="Creation" />
<img src="http://68.media.tumblr.com/7d99e88a28cf4aac41d0a9011720d53c/tumblr_n1mxepbR2s1r2wjwko3_250.png" height="420" alt="Creation" />


Check out more examples of creation screens on [pttrns](https://pttrns.com/android-patterns?scid=19).

### 5. Profile

This archetype is focused on allowing the user to view information about their own account, view their own content, and provide them account related action items. Typically, the profile contains important statistics about the user (i.e number of followers), displays recent content, and provides access to actions such as editing their profile, changing their picture, logging out, etc.

Profile screens usually require the following components:

* Grid or list of recent content items for user
  * See: [[listview|Using an ArrayAdapter with ListView]]
* Images associated with the user's identity 
  * See: [[picasso|Sending-and-Managing-Network-Requests#displaying-remote-images-the-easy-way]]
* List of related users (followers, following)
* Action items when on another user's account 
  * See: [[events|Basic-Event-Listeners]]

**Examples:**

<img src="http://cdn.pttrns.com/353/6124_w250.jpg" height="420" alt="Profile" />&nbsp;
<img src="http://68.media.tumblr.com/42420c6bbe58829c3fd56e2d85e544eb/tumblr_nqu31wL6WT1r2wjwko5_250.png" />&nbsp;
<img src="http://68.media.tumblr.com/64d0d7baecd9b76f3badc0628137f1fe/tumblr_o60rraLQzl1r2wjwko3_250.png" height="420" alt="Profile" />

Check out more examples of profile screens on [pttrns](https://pttrns.com/android-patterns?scid=20).

### 6. Settings

This archetype is focused on giving the user the ability to tune preferences associated with their account and/or the behavior of the app. The settings available range from notification and privacy settings, to profile settings such as username or email and to preferences affecting the behavior of the app.

Settings screens usually require the following components:

* Persisting settings after they are saved 
  * See: [[preferences|Storing-and-Accessing-SharedPreferences]]
* Input fields and labels for modifying settings 
  * See: [[input fields|Working with Input Views]]
* Validating new input from the user
* Connecting with third-party accounts
* Affecting behavior of push notifications 
  * See: [[push messaging|Push Messaging]], [[notifications|Notifications]]

**Examples:**

<img src="http://cdn.pttrns.com/532/6227_w250.jpg" height="420" alt="Profile" />&nbsp;
<img src="http://cdn.pttrns.com/345/6033_w250.jpg" height="420" alt="Profile" />&nbsp;
<img src="http://68.media.tumblr.com/479eeffd3b1174453a6ceb732c92eaa1/tumblr_mohaz18J5z1r2wjwko6_250.png" height="420" alt="Profile" />

Check out more examples of settings screens on [pttrns](https://pttrns.com/android-patterns?scid=16&y=2016).

## References

* <https://pttrns.com/android-patterns/>
* <http://androidniceties.tumblr.com//>

