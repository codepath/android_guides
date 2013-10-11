## Overview

In Android, the common "pull to refresh" UX concept is not built in to a ListView. However, many Android applications would like to make use of this concept for their feeds. This is useful for all sorts of feeds such as a Twitter timeline.

Thankfully, there is an excellent third-party [pull-to-refresh](https://github.com/erikwt/PullToRefresh-ListView) library that can be used to make this interaction extremely easy to implement. 

### Installation

Download the zip of [pull-to-refresh](https://github.com/erikwt/PullToRefresh-ListView/archive/master.zip) and then import the library folder into Eclipse with **File => Import => Existing Android Code Into Workspace**. Call the project "PullToRefresh" and then start the import. This folder is now a "library project" and can then be included as a library in your Android apps.

## References

* <https://github.com/erikwt/PullToRefresh-ListView> - Third-party library for pull to refresh