# CodePath Android Cliffnotes

Welcome to the open-source [Codepath](http://thecodepath.com) Android Guides! Our goal is to become the **central crowd-sourced resource** for complete and up-to-date Android content and tutorials.

![CodePath](http://i.imgur.com/XgxWfyF.png) 

We have guides for everyone whether you are **beginner, intermediate or advanced**. Want to learn how to [[use the ActionBar|Defining the ActionBar]] or the [[ins and outs of fragments|Creating and Using Fragments]]? We got that. Want to learn [[about testing|Android Unit and Integration Testing]] or how to [[build flexible user interfaces for multiple devices|Flexible User Interfaces]]? We got you covered. We don't teach you the "theoretical way" from a book. We cover **exactly the things we use every day** as we are developing apps for contracts.

Ever been **frustrated finding information on outdated one-off blog posts and tutorials** that has since become irrelevant? How many times were you googling only to find your answer only on a **2 year old StackOverflow post**? We believe there's got to be a better way. Why not have the community work together to create useful and detailed documentation for every aspect of Android (or any platform)? There's absolutely no reason that we should have to make due with outdated, vague or un-editable content anymore.

Read about our [[mission to change the way engineers learn new technologies|The CodePath Goal]] and we would love for you to [get involved](https://github.com/thecodepath/android_guides/wiki/The-CodePath-Goal#how-do-i-help)!

## Introduction

The guides below are **categorized by their topic**, so you can easily find guides on related topics whether that is views, styling, testing, or using sensors. If you see an error, incorrect explanation or deprecated solution, **why not contribute back** and make this better for the next person reading the guide? That's the spirit of this project.

**Disclaimer:** We have scoured the web endlessly for content while creating these guides and adapted content from many source including the [Google Official Docs](http://developer.android.com/guide/components/index.html), [Vogella Tutorials](http://www.vogella.com/android.html) and countless others that had hidden gems. At the bottom of every guide, there is citations for the content we adapted. **We don't claim the content is original** (although we did develop much ourselves), but unlike those other sources listed, it is freely community-contributable and editable. We have openly adapted, modified and brought together content from all the sources we could find for the benefit of everybody.

Read more about [[us and our vision for this|The CodePath Goal]].

## Structure

Exploring the foundations and structure of apps:

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

* [[Drawables]] (and how to polish UI)
* [[Animations]] (animating views)
* [[Styles and Themes]] (consolidating view styles)
* [Polishing a UI Tips and Tools](https://gist.github.com/nesquena/6c567083aec13d868017)
* [[Cloning a Login Screen Layout]] Guide (creating attractive UIs)

### AdapterViews

* [[Using an ArrayAdapter with ListView]] (with custom list items)
* [[Implementing a Horizontal ListView]] Guide
* [[Endless Scrolling with AdapterViews]]
* [[Implementing Pull to Refresh]] Guide

## Interaction and Navigation

Exploring how to allow user interaction and navigation within an app:

* [[View Event Listeners|Basic Event Listeners]] (Clicks, Key Presses, Updates)
* [[Navigating Activities with Intents|Using Intents to Create Flows]] (Communicating between Activities)
* [[Exploring the ActionBar|Defining the ActionBar]] (includes adding ActionItems)
* [[Displaying Toasts]] (Quick notices and includes custom views)
* [[Common Navigation Paradigms]] (i.e Tabs, Swipe-able Views, Pull-out Drawer)
* [[Extended ActionBar Guide]] (Split-bar, Custom ActionBar, etc)
* [[Common Implicit Intents]] (Making a Call, Sending a Text, Opening a URL)
* [[Gestures and Touch Events]] (Swipe, Shake, or Dragging Events)
* [[Navigation and Task Stacks]] (Controlling the behavior of the task stack)
* [[Sharing Content with Intents]] (and ShareActionProvider)

## Networking, Models and Persistence

Diving into the networking, model and persistence layers for data-driven apps:

* [[Creating and Executing Async Tasks]] (Long-running Background Tasks)
* [[Handling ProgressBars]] (with Long-Running Tasks)
* [[Sending and Managing Network Requests]] (API Calls, Image Downloading)
* [[Persisting Data to the Device]] (Preferences, Files, SQLite, ORMs)
* [[Converting JSON to Models]] (JSON => Objects Deserialization)
* [[ActiveAndroid Guide]] (ORM Guide)

## Fragments

Understanding how to build powerful and flexible using Fragments:

* [[Creating and Using Fragments]]
* [[Fragment Navigation Drawer]]
* [[Using DialogFragment]]
* [[Flexible User Interfaces]] (with Fragments)

## Providers, Sensors and Components

Exploring sensors, data and components available via the Android SDK:

* [[Loading Contacts with Content Providers]]
* [[Using Hardware, Sensors and Device Data]]
* [[Google Maps Fragment Guide]]

## Services

Digging into how to run background services or leverage Android system services:

* [[Starting Background Services]] (with ServiceIntent and Receivers)
* [[Notifications]] (Persistent Notices on the Dashboard)

## Workflow Guides

Focused on issues like deployment, testing, dependency management, etc:

* [[Getting Started with Gradle]]
* [[Android Unit and Integration Testing]] (with Roboelectric and Robotium)
* [[Robolectric Installation for Unit Testing]]
* [[Must Have Libraries]] Guide