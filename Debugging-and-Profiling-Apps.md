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

In addition, the following concepts need to be included:

* [Profiling GPU Rendering](http://developer.android.com/tools/performance/profile-gpu-rendering/index.html) - quick visual representation of how much time it takes to render the frames of a UI window relative to the 16-ms-per-frame benchmark.
* [Visualizing Overdraw](http://developer.android.com/tools/performance/debug-gpu-overdraw/index.html) - Shows on the device where an app might be doing more rendering work than necessary. Helping you see where you might be able to reduce rendering overhead.
* [Heap Viewer](http://developer.android.com/tools/performance/heap-viewer/index.html) - Identifying memory leaks
* [LeakCanary](https://corner.squareup.com/2015/05/leak-canary.html) for finding memory leaks easily
* [Allocation Tracker](http://developer.android.com/tools/performance/allocation-tracker/index.html) - Finding the places in your code that may contribute to memory trashing.
* [Batterystats](http://developer.android.com/tools/performance/batterystats-battery-historian/index.html) - Shows where and how processes are drawing current from the battery.
* [Batteryhistorian](https://github.com/google/battery-historian) - Visualize system and application level events on a timeline. [Guide to getting started](https://docs.google.com/document/d/1CSTRAaCtbjTe2rs2vzra6-PViMkfPJC9DVWg7NbXqYk)

## Optimizing Performance

Check out the following links regarding optimizing performance:

* [[Layout Performance|Constructing-View-Layouts#optimizing-layout-performance]]
* [[ViewHolder with ListView|Using-an-ArrayAdapter-with-ListView#improving-performance-with-the-viewholder-pattern]]
* [Displaying Bitmaps Effectively](http://developer.android.com/training/displaying-bitmaps/index.html)
* [Caching Bitmaps](http://developer.android.com/training/displaying-bitmaps/cache-bitmap.html)
* [Android Performance Tips](http://developer.android.com/training/articles/perf-tips.html)
* [Managing App Memory](http://developer.android.com/training/articles/memory.html#YourApp)
* [Layout Performance Guide](http://developer.android.com/training/improving-layouts/index.html)
* [Keeping Your App Responsive](http://developer.android.com/training/articles/perf-anr.html)

## Memory Management

* [Stopping Memory Leaks](http://blog.nimbledroid.com/2016/09/06/stop-memory-leaks.html)
* [Memory Performance 101 Video](https://www.youtube.com/watch?v=OrLEoIsMIAc)

## References

* [Top Android performance problems](http://www.androidauthority.com/top-android-performance-problems-666234/)
* [FlatBuffers in Android](http://frogermcs.github.io/flatbuffers-in-android-introdution/)
* [Android Performance Patterns Video Series](https://www.youtube.com/playlist?list=PLWz5rJ2EKKc9CBxr3BVjPTPoDPLdPIFCE)
* [Reducing Overdraw: Optimizing Layouts in Android](http://riggaroo.co.za/optimizing-layouts-in-android-reducing-overdraw/)