## Prerequisite

Please make sure you are familiar with [RecyclerView] (https://developer.android.com/reference/android/support/v7/widget/RecyclerView.html) by going through the following guide for [basic usage of a `RecyclerView`] (https://github.com/codepath/android_guides/wiki/Using-the-RecyclerView). We will be building on top of the classes from the above guide so it is very important that you have the basic `RecyclerView` up and running.

## Overview

`RecyclerView` can also be used to inflate multiple view types in situations where your list might be heterogeneous, in the sense, based on the response from the server, there might be a requirement for inflating different types of layouts (example: Consider facebook home feed where there are a variety of stories such as a status update, location update, single image, image album, video, etc). This guide will explain how to inflate multiple view types inside your `RecyclerView` widget based on the item type. 

Note: You can refer to the [following guide] (https://github.com/codepath/android_guides/wiki/Implementing-a-Heterogenous-ListView) on how to do the same with a `ListView`.



