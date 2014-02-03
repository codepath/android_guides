# CodePath Android Cliffnotes

Welcome to the open-source [Codepath](http://thecodepath.com) Android Cliffnotes! Our goal is to become the **central crowdsourced resource** for complete and up-to-date practical Android developer guides for any topic. [[Just take me to the notes|Home#structure]]!

[![CodePath](http://i.imgur.com/XgxWfyF.png)](http://thecodepath.com)

We have guides for everyone whether you are a **beginner, intermediate or expert**. Want to learn how to [[use the ActionBar|Defining the ActionBar]] or the [[ins and outs of fragments|Creating and Using Fragments]]? We got that. Want to learn [[about automated integration testing|Android Unit and Integration Testing]] or how to [[build flexible user interfaces for multiple devices|Flexible User Interfaces]]? We got you covered. We don't waste time with the "theoretical approach" you might get from a book. We cover **exactly the things we use every day** as we are developing apps for contracts.

## Motivation

Ever been **frustrated finding information on outdated one-off blog posts and tutorials** that have long since become irrelevant? How many times have you been googling to find your answer only exists on an obscure **2 year old StackOverflow post**? We believe there's got to be a better way. Why not have the community work together to create useful and detailed documentation for every aspect of Android (or any platform)? There's absolutely no reason that we should have to make do with vague and outdated content anymore.

Read about our [[mission to change the way engineers learn new technologies|The CodePath Goal]] and check out how you can [[get involved|The-CodePath-Goal#how-do-i-help]]! In addition, we are at present a fledgling startup so if you like this guide and what we are trying to do, please consider following us on twitter [@thecodepath](https://twitter.com/thecodepath) or [donating to the cause](https://www.gittip.com/nesquena/)! 

## Overview

The cliffnotes below are **categorized by their topic**, so you can easily find guides on related topics whether that is views, styling, testing, or using sensors. If you see an error, incorrect explanation or deprecated solution, **why not contribute back** and make these cliffnotes better for the next person? That in a nutshell is the core spirit of this initiative. Check out the [list of contributors](https://github.com/thecodepath/android_guides/blob/master/README.md#contributors) to this project.

**Disclaimer:** We have scoured the web endlessly for content while creating these guides and adapted content from many source including the [Google Official Docs](http://developer.android.com/guide/components/index.html), [Vogella Tutorials](http://www.vogella.com/android.html) and countless other sources that had hidden gems of information. At the bottom of each guide, there are citations for the content we used. **We don't claim the content is original** (although we did develop quite a bit ourselves), but unlike those other sources listed, it is freely community editable. We have openly adapted, modified and brought together this content from all the sources we could find for the benefit of every engineer.

Read more about [[us and our vision for all this|The CodePath Goal]]. If you want to **contribute to this guide**, please read the [[Contributing Guidelines]].

## Getting Started

**Totally new to Android?** Start here.

* [Setting up the Android Development Tools](http://goo.gl/Ml9YN) (Installation Slides)
* [Architecture of Mobile Apps](http://goo.gl/AAsGLx) (Concept Slides)
* [Developing our First App](http://goo.gl/pBKfYP) (Step-by-Step Todo App)
* [[Troubleshooting Common Issues]] (Running into problems?)
* [[Beginning Android Resources]] (Detailed Post With Many Links)
* [[Sample Android Apps]] (Code repositories)

## Structure

Exploring the foundations of app development:

* [[Android Directory Structure]] (Files and Folders for Android apps)
* [[Organizing your Source Files]] (Cleaning up your Apps)
* [[Using Resource Files|Using String Resources]] (Understanding String Resources)

## Views and Layouts

Exploring the gritty details of views, layout, styling and common UI patterns:

* [[Constructing View Layouts]] (How to layout views)
* [[Defining Views and their Attributes]]
* [[Working with Input Views]] (Spinner, RatingBar, etc)
* [[Working with the Soft Keyboard]]

### Designing and Styling Views

* [[Drawables]] (and how to polish UI views)
* [[Styles and Themes]] (consolidating view styles)
* [[Animations]] (animating views, layouts, activities and more)
* [Polishing a UI Tips and Tools](https://gist.github.com/nesquena/6c567083aec13d868017)
* [[Cloning a Login Screen Layout]] Guide (Creating attractive UIs, Q&A)

### AdapterViews

* [[Using an ArrayAdapter with ListView]] (with custom list items)
* [[Endless Scrolling with AdapterViews]] (infinite pagination)
* [[Implementing Pull to Refresh]] Guide
* [[Implementing a Horizontal ListView]] Guide (scrolls horizontally)
* [[Implementing a Heterogenous ListView]] (with different item types)

### Custom Views

* [[Basic Painting with Views]] (Simple drawing app tutorial)
* [[Defining Custom Views]] (**Needs Attention**)
* [[Extending SurfaceView]] (**Needs Attention**)

## Interaction

Exploring how to allow user interaction and navigation within an app:

* [[View Event Listeners|Basic Event Listeners]] (Clicks, Key Presses, Updates)
* [[Displaying Toasts]] (Quick notices and includes custom views)
* [[Exploring the ActionBar|Defining the ActionBar]] (includes adding ActionItems)
* [[Extended ActionBar Guide]] (Split-bar, Custom ActionBar, etc)
* [[Gestures and Touch Events]] (Swipe, Shake, or Dragging Events)
* [[Menus and Popups]] (Context Menu, PopupMenu, PopupWindow)
* [[Dialogs with DialogFragment|Using DialogFragment]] (Displaying a content overlay)
* [[Implementing a Rate Me Feature]] (For getting Play Store ratings)

## Navigation

* [[Navigating Activities with Intents|Using Intents to Create Flows]] (Communicating between Activities)
* [[Common Navigation Paradigms]] (i.e Tabs, Swipe-able Views, Pull-out Drawer)
* [[Common Implicit Intents]] (Making a Call, Sending a Text, Opening a URL)
* [[Navigation and Task Stacks]] (Controlling the behavior of the task stack)
* [[Sharing Content with Intents]] (and ShareActionProvider)

## Networking, Models and Persistence

Diving into the networking, model and persistence layers for data-driven apps:

* [[Creating and Executing Async Tasks]] (Long-running Background Tasks)
* [[Handling ProgressBars]] (with Long-Running Tasks)
* [[Sending and Managing Network Requests]] (API Calls, Image Downloading)
* [[Persisting Data to the Device]] (Preferences, Files, SQLite, ORMs)
* [[Converting JSON to Models]] (JSON => Objects Deserialization)
* [[ActiveAndroid ORM References|ActiveAndroid Guide]] (and Q&A)
* [[Using Parcelable]] (Pass data fast between activities)
* [[RottenTomatoes Networking Tutorial]]
* [[Networking with the Volley Library]]
* [[Storing and Accessing SharedPreferences]]
* [[Building Data-driven Apps with Parse]]

## Fragments

Understanding how to build powerful and flexible views using Fragments:

* [[Creating and Using Fragments]]
* [[ActionBar Tabs with Fragments]]
* [[Fragment Navigation Drawer]]
* [[DialogFragment Overview|Using DialogFragment]]
* [[Flexible User Interfaces]] (with Fragments)
* [[ViewPager with FragmentPagerAdapter]]
* [[Settings with PreferenceFragment]] (**Needs Attention**)

## Providers, Sensors and Components

Exploring sensors, data and components available via the Android SDK:

* [[Loading Contacts with Content Providers]]
* [[Using Hardware, Sensors and Device Data]] (Camera, Photo Roll, Location)
* [[Google Maps Fragment Guide]] (and [Setup Genymotion](https://github.com/thecodepath/android_guides/wiki/Google-Maps-Fragment-Guide#genymotion))
* [[Creating Content Providers]] (**Needs Attention**)
* [[Video and Audio Playback]] (**Needs Attention**)

## Services

Digging into how to run background services or leverage Android system services:

* [[Starting Background Services]] (with ServiceIntent and Receivers)
* [[Notifications]] (Persistent Notices on the Dashboard)
* [[Push Messaging]] (**Needs Attention**)

## Workflow Guides

Focused on issues like deployment, testing, dependency management, etc:

* [[Getting Started with Gradle]]
* [[Android Unit and Integration Testing]] (with Roboelectric and Robotium)
* [[Robolectric Installation for Unit Testing]]
* [[Must Have Libraries]] Guide
* [Publishing to the Play Store](http://goo.gl/mUlGL1) (Slides)
* [[Using Android Studio]] (**Needs Attention**, highly experimental)