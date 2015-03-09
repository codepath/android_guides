## Overview

Traditionally transitions between different activities or fragments involved enter and exit transitions that animated entire view hierarchies independant to each other. Example of such transitions are a fade transition, slide transition or the newly introduced explode transition.

Default Activity Transition:

<iframe title="YouTube video player" width="480" height="390" src="https://www.youtube.com/watch?v=XxnsSNX22hY?autoplay=1" frameborder="0" allowfullscreen></iframe>

However, many times, there are elements common to both activities and providing the ability to transition these shared elements separately emphasizes continuity between transitions and breaks activity boundaries as the user navigates the app.

![SharedElement](http://3.bp.blogspot.com/-DnRuIu0QqgE/VAeEWFgCVdI/AAAAAAAAqak/t5NF8kHVRG8/s1600/heroview.png)

The nature of this transition forces the human eye to focus on the content and its representation in the new activity instead of the actual activity frame sliding or fading which makes the experience a lot more seamless.

<iframe title="YouTube video player" width="480" height="390" src="http://youtu.be/XkWI1FKKrs4?autoplay=1" frameborder="0" allowfullscreen></iframe>
