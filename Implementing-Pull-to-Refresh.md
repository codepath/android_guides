## Overview

In Android, the common "pull to refresh" UX concept is not built in to a ListView. However, many Android applications would like to make use of this concept for their feeds. This is useful for all sorts of feeds such as a Twitter timeline.

Thankfully, there is an excellent third-party [pull-to-refresh](https://github.com/erikwt/PullToRefresh-ListView) library that can be used to make this interaction extremely easy to implement. 

### Installation

Download the zip of [pull-to-refresh](https://github.com/erikwt/PullToRefresh-ListView/archive/master.zip) and then [import the library folder](http://imgur.com/a/N8baF) into Eclipse with **File => Import => Existing Android Code Into Workspace**. Call the project "PullToRefresh" and then start the import. This folder is now a "library project" and can then be included as a library in your Android apps. Make sure to mark this project as a library and then include the library in your application Check the [Detailed Instructions Here](http://imgur.com/a/N8baF) for a step-by-step of this process.

### Replace ListView

Next, for each ListView that we want to add pull-to-refresh functionality to, we need to replace the `ListView` in the XML with the new third-party `eu.erikw.PullToRefreshListView` which has all the same features of a ListView. For example:

```xml
<ListView
    android:id="@+id/pull_to_refresh_listview"
    android:layout_height="fill_parent"
    android:layout_width="fill_parent" />
```

would be replaced with:

```xml
<eu.erikw.PullToRefreshListView
    android:id="@+id/pull_to_refresh_listview"
    android:layout_height="fill_parent"
    android:layout_width="fill_parent" />
```

All other properties in the XML can stay the same.

## References

* <https://github.com/erikwt/PullToRefresh-ListView> - Third-party library for pull to refresh