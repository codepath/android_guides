## Overview

(**Needs Attention**)

* [Analyzing Display and Performance](http://developer.android.com/tools/debugging/systrace.html) (`systrace`)
* [Optimizing your View and Inspecting Hierarchy](http://developer.android.com/tools/debugging/debugging-ui.html) (`hierarchyviewer`)
* [Investigating your RAM Usage](http://developer.android.com/tools/debugging/debugging-memory.html) (`monitor`)
* [Profiling with Traceview](http://developer.android.com/tools/debugging/debugging-tracing.html) (`traceview`)
* [Profiling Android Apps - Performance Turning](https://www.youtube.com/watch?v=88nwiiVTh5w)
* [Vogella Android Analysis Tools](http://www.vogella.com/tutorials/AndroidTools/article.html)
* [Performance Course on Udacity](https://www.udacity.com/course/ud825)
* [Performance Doc from Udacity](https://docs.google.com/document/d/1EKVq2FzcLVJFbwUtaC3QRddSwtzs0BSKZahkQyeGyHo/pub?embedded=true)

In addition, this should include:

* Profiling GPU Rendering - quick visual representation of how much time it takes to render the frames of a UI window relative to the 16-ms-per-frame benchmark.
* Visualizing Overdraw - Shows on the device where an app might be doing more rendering work than necessary. Helping you see where you might be able to reduce rendering overhead.
* Heap Viewer - Identifying memory leaks
* LeakCanary for finding memory leaks easily
* Allocation Tracker - Finding the places in your code that may contribute to memory trashing.
* Batterystats - Shows where and how processes are drawing current from the battery.

## Optimizing Performance

Check out the following links regarding optimizing performance:

* [[Layout Performance|Constructing-View-Layouts#optimizing-layout-performance]]
* [[ViewHolder with ListView|Using-an-ArrayAdapter-with-ListView#improving-performance-with-the-viewholder-pattern]]
* [Displaying Bitmaps Effectively](http://developer.android.com/training/displaying-bitmaps/index.html)
* [Android Performance Tips](http://developer.android.com/training/articles/perf-tips.html)
* [Managing App Memory](http://developer.android.com/training/articles/memory.html#YourApp)
* [Layout Performance Guide](http://developer.android.com/training/improving-layouts/index.html)