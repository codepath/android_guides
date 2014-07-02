# CodePath Android Cliffnotes

Welcome to the open-source [Codepath](http://thecodepath.com) Android Cliffnotes! Our goal is to become the **central crowdsourced resource** for complete and up-to-date practical Android developer guides for any topic. [[Just take me to the notes|Home#getting-started]]!

[![CodePath](http://i.imgur.com/XgxWfyF.png)](http://thecodepath.com)

We have Android guides for everyone whether you are a **beginner, intermediate or expert**. Want to learn how to [[use the ActionBar|Defining the ActionBar]] or the [[ins and outs of fragments|Creating and Using Fragments]]? We got that. Want to learn [[about automated integration testing|Android Unit and Integration Testing]] or how to [[build flexible user interfaces for multiple devices|Flexible User Interfaces]]? We got you covered. We don't waste time with the "theoretical approach" you might get from a book. We cover **exactly the things we use every day** as we are developing apps for contracts.

**Need Help?** Please join this [gitter real-time chat](https://gitter.im/thecodepath/public) or the [google groups](https://groups.google.com/forum/#!forum/codepath-android-guides) for these guides where you can post related questions.

## Motivation

Ever been **frustrated finding information on outdated one-off blog posts and tutorials** that have long since become irrelevant? How many times have you been googling to find your answer only exists on an obscure **2 year old StackOverflow post**? We believe there's got to be a better way. Why not have the community work together to create useful and detailed documentation for every aspect of Android (or any platform)? There's absolutely no reason that we should have to make do with vague and outdated content anymore.

Read about our [[mission to change the way engineers learn new technologies|The CodePath Goal]] and check out how you can [[get involved|The-CodePath-Goal#how-do-i-help]]! In addition, we are at present a fledgling startup so if you like this guide and what we are trying to do, please consider following us on twitter [@thecodepath](https://twitter.com/thecodepath) or [donating to the cause](https://www.gittip.com/nesquena/)! 

## Overview

The cliffnotes below are **categorized by their topic**, so you can easily find guides on related topics whether that is views, styling, testing, or using sensors. If you see an error, incorrect explanation or deprecated solution, **why not contribute back** and make these cliffnotes better for the next person? That in a nutshell is the core spirit of this initiative. Check out the [list of contributors](https://github.com/thecodepath/android_guides/blob/master/README.md#contributors) to this project.

**Disclaimer:** We have scoured the web endlessly for content while creating these guides and adapted content from many source including the [Google Official Docs](http://developer.android.com/guide/components/index.html), [Vogella Tutorials](http://www.vogella.com/android.html) and countless other sources that had hidden gems of information. At the bottom of each guide, there are citations for the content we used. **We don't claim the content is original** (although we did develop quite a bit ourselves), but unlike those other sources listed, it is freely community editable. We have openly adapted, modified and brought together this content from all the sources we could find for the benefit of every engineer.

Read more about [[us and our vision for all this|The CodePath Goal]]. If you want to **contribute to this guide**, please read the [[Contributing Guidelines]].

## Live in San Francisco?

Located in the San Francisco Bay Area and interested in learning with others in a more structured program? Check out our local [Android](http://www.meetup.com/Learning-Android-Development) or [iOS](http://www.meetup.com/Learning-iOS-Development-SF/) meetup events. We have free evening events and at-cost 1-day workshops to make learning social and connect you with others passionate about mobile.

If you are an experienced engineer (2+ years of professional experience in software development) and serious about learning Android, check out our [free evening 8-week Android bootcamp](http://thecodepath.com/androidbootcamp). Learn how to build mobile apps while collaborating with other engineers and designers. Work on solving important problems for non-profits with our free 8-week accelerated evening mobile bootcamp. Compete for $10,000 worth of cash prizes. [Learn more and apply here](https://gist.github.com/nesquena/638c239b723159aaa096).

## Getting Started

**Totally new to Android?** Start here.

* [[Beginning Android Resources]] (Detailed Post With Many Links)
* [Setting up the Android Development Tools](http://goo.gl/Ml9YN) (Installation Slides)
* [Architecture of Mobile Apps](http://goo.gl/AAsGLx) (Concept Slides)
* [Developing our First App Using Eclipse](http://goo.gl/pBKfYP) (Step-by-Step Todo App)
* [[Troubleshooting Common Issues]] (Running into problems?)
* [[Sample Android Apps]] (Code repositories)

**Using Android Studio?** See below.

* [Setting up Android Studio](http://goo.gl/X2SVFR) (Experimental Install Slides)
* [Developing our First App Using Android Studio](http://goo.gl/iwozwi) (Step-by-Step Todo App)

## Structure

Exploring the foundations of app development:

* [[Android Directory Structure]] (Files and Folders for Android apps)
* [[Organizing your Source Files]] (Cleaning up your Apps)
* [[Using Resource Files|Using String Resources]] (Understanding String Resources)
* [[Handling Configuration Changes]] (Screen Rotation)

## Views and Layouts

Exploring the gritty details of views, layout, styling and common UI patterns:

* [[Constructing View Layouts]] (How to layout views)
* [[Defining Views and their Attributes]] (Gravity, Margin, Padding, etc)
* [[Working with the TextView]] (Properties, Drawables, Custom Fonts)
* [[Working with the EditText]] (Properties)
* [[Working with the ImageView]] (Size, Scale, Density, Raw Bitmaps)
* [[Working with Input Views]] (Spinner, RatingBar, etc)
* [[Working with the Soft Keyboard]]
* [[Working with the WebView]]
* [[Working with the ScrollView]]

### Designing and Styling Views

* [[Drawables]] (and how to polish UI views)
* [[Styles and Themes]] (Consolidating view styles)
* [[Animations]] (Animating views, layouts, activities and more)
* [[Polishing a UI Tips and Tools]] (Links to key resources)
* [[Android Design Guidelines]] (Overview of Android design standards)
* [[Styling UI Screens FAQ]] (Answers to common questions around building screens)
* [[Cloning a Login Screen Layout Guide]] (Creating attractive UIs, Q&A)

### AdapterViews

* [[Using an ArrayAdapter with ListView]] (with custom list items)
* [[Endless Scrolling with AdapterViews]] (Infinite pagination)
* [[Implementing Pull to Refresh Guide]]
* [[Implementing a Horizontal ListView Guide]] (Scrolls horizontally)
* [[Implementing a Heterogenous ListView]] (with different item types)

### Custom Views

* [[Basic Painting with Views]] (Simple drawing app tutorial)
* [[Defining Custom Views]] (**Needs Attention**)
* [[Extending SurfaceView]] (**Needs Attention**)

## Interaction

Exploring how to allow user interaction and navigation within an app:

* [[View Event Listeners|Basic Event Listeners]] (Clicks, Key Presses, Updates)
* [[Displaying Toasts]] (Quick notices and includes custom views)
* [[Exploring the ActionBar|Defining the ActionBar]] (Includes adding ActionItems)
* [[Extended ActionBar Guide]] (Split-bar, Custom ActionBar, etc)
* [[Gestures and Touch Events]] (Swipe, Shake, or Dragging Events)
* [[Menus and Popups]] (Context Menu, PopupMenu, PopupWindow)
* [[Dialogs with DialogFragment|Using DialogFragment]] (Displaying a content overlay)
* [[Implementing a Rate Me Feature]] (For getting Play Store ratings)

## Navigation

* [[Navigating Activities with Intents|Using Intents to Create Flows]] (Communicating between Activities)
* [[Common Navigation Paradigms]] (Tabs, Swipe-able Views, Pull-out Drawer)
* [[Common Implicit Intents]] (Making a Call, Sending a Text, Opening a URL)
* [[Navigation and Task Stacks]] (Controlling the behavior of the task stack)
* [[Sharing Content with Intents]] (and ShareActionProvider)
* [[Using Parcelable]] (Pass data fast between activities)

## Networking and Models

Diving into the networking and model layers for data-driven apps:

* [[Sending and Managing Network Requests]] (API Calls, Image Downloading)
* [[Converting JSON to Models]] (JSON to Objects Deserialization)
* [[Creating and Executing Async Tasks]] (Long-running Background Tasks)
* [[Handling ProgressBars]] (with Long-Running Tasks)
* [[RottenTomatoes Networking Tutorial]]
* [[Networking with the Volley Library]]
* [[Building Data-driven Apps with Parse]]

## Persistence

Exploring the strategies for data persistence:

* [[Persisting Data to the Device]] (Preferences, Files, SQLite, ORMs)
* [[ActiveAndroid ORM References|ActiveAndroid Guide]] (and Q&A)
* [[Clean Persistence with Sugar ORM]] (Integration, Installation, Queries, Migrations)
* [[Storing and Accessing SharedPreferences]]
* [[Settings with PreferenceFragment]] (**Needs Attention**)

## Fragments

Understanding how to build powerful and flexible views using Fragments:

* [[Creating and Using Fragments]]
* [[ActionBar Tabs with Fragments]]
* [[Fragment Navigation Drawer]]
* [[DialogFragment Overview|Using DialogFragment]]
* [[Flexible User Interfaces]] (with Fragments)
* [[ViewPager with FragmentPagerAdapter]]

## Providers, Sensors and Components

Exploring sensors, data and components available via the Android SDK:

* [[Loading Contacts with Content Providers]]
* [[Using Hardware, Sensors and Device Data]] (Camera, Photo Roll, Location)
* [[Video and Audio Playback and Recording]] (MediaPlayer, VideoView)
* [[Google Maps Setup Guide|Google Maps Fragment Guide]] (and [[Setup Genymotion|Google-Maps-Fragment-Guide#setup-emulator]])
* [[Google Maps API v2 Usage]] (Markers, InfoWindow)
* [[Creating Content Providers]] (**Needs Attention**)

## Services

Digging into how to run background services or leverage Android system services:

* [[Starting Background Services]] (with ServiceIntent and Receivers)
* [[Notifications]] (Persistent Notices on the Dashboard)
* [[Push Messaging]] (**Needs Attention**)

## Workflow Guides

Focused on issues like deployment, testing, dependency management, etc:

* [[Getting Started with Gradle]]
* [[Building Gradle Projects with Jenkins CI]]
* [[Android Unit and Integration Testing]] (with Roboelectric and Robotium)
* [[Robolectric Installation for Unit Testing]]
* [[Must Have Libraries]] (Networking, Persistence, Compatibility, Convenience, etc)
* [Publishing to the Play Store](http://goo.gl/mUlGL1) (Slides)
* [[Popular External Tools]] (Analytics, Crash Reporting)
* [[Using Android Studio]] (**Needs Attention**, highly experimental)