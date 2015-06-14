## Overview

Building beautiful Android apps starts with understanding how to approach building a delightful UI and each of the components that contributes:

<a href="http://androidniceties.tumblr.com/">
  <img src="http://i.imgur.com/Zt9ZIys.jpg" alt="design" width="250" />&nbsp;
  <img src="http://i.imgur.com/UECNpcx.png" alt="design" width="250" />&nbsp;
  <img src="http://i.imgur.com/I4qigt5.jpg" alt="design" width="250" />
</a>

Polishing up the user interface of your application starts with the following enhancements:

 1. Benchmarking to **Adhere to Good Designs** - Check out the [[following sites|Android-Design-Guidelines#benchmarking]] or [these material design examples](http://www.materialup.com/) for looking at how popular apps look and feel.
 2. Pick a **Vibrant Color Scheme** - Pick a primary color and a secondary color for [coloring your app](http://www.google.com/design/spec/style/color.html#color-ui-color-application) using a [sensible color scheme](http://www.colourlovers.com/palettes/new/past-month/meta?page=1). Check out [material palette](http://www.materialpalette.com/) for another take on picking colors. 
 3. Use **Proper Icons and Colorful Images** - Use images, icons and backgrounds for your UIs leveraging resources like [MaterialDesignIcons](http://materialdesignicons.com/), [IconFinder](https://www.iconfinder.com/), [iconmonstr](http://iconmonstr.com/), [NounProject](http://thenounproject.com/), [flickr](https://www.flickr.com/search/) and [Google Image Search](http://www.google.com/imghp) to locate relevant assets. Adhere carefully to the [iconography style guidelines](http://developer.android.com/design/style/iconography.html) for Android
 4. Review **Typography** - Check out the [typography guide](http://developer.android.com/design/style/typography.html) to understand the common font types for Android apps and default type colors and sizes. See the [calligraphy library](https://github.com/chrisjenx/Calligraphy) for easy custom fonts. 
 5. Apply **backgrounds and borders** to views and layouts - Use [[shape and layer drawables|Drawables]] cliffnotes to create colorful backgrounds and borders to your [[buttons|Drawables#customizing-a-button]], [[listviews|Drawables#customizing-a-listview]], and other views. See the [material card view](https://developer.android.com/training/material/lists-cards.html#CardView) for a look at modern styles for lists.
 6. Improve **ActionBar and Navigation Appearance** -  Review our style guides for the [[ActionBar|Extended-ActionBar-Guide#custom-actionbar-styles]] and [[ActionBar Tabs guide|ActionBar-Tabs-with-Fragments#styling-tabs]]. See generators linked in next section.
 7. Follow **Android UI Standards** - Use [[modern material design guidelines|Material-Design-Primer]], [common navigation styles](http://guides.codepath.com/android/Android-Design-Guidelines#common-patterns) and [proper app guidelines](http://developer.android.com/design/patterns/app-structure.html) and be sure to [design for Android](http://developer.android.com/design/patterns/pure-android.html).
 8. Implement **Intermediate UI Elements** - Make sure to [add progress bars](http://guides.codepath.com/android/Handling-ProgressBars) when loading, along with placeholders for images and empty states in cases when there's no content.

## Tips and Tools

Simple guide for improving the UI for any application including links to tools:

1. **Catchy Title** - Pick a creative single word name for your application
2. **Launcher Icon** - Select a pleasant launcher icon (create a [launcher icon](http://imgur.com/a/8cmLM) and update in manifest)
3. **Design Guidelines** - Review these [[design cliffnotes|Android-Design-Guidelines]] for an  overview of design guidelines and patterns.
  * [[Material Design Primer|Material-Design-Primer]] - Quick overview of all things related to material design
  * [Core Principles](http://developer.android.com/design/get-started/principles.html) - Core motivating principles of Android UI
  * [Pure Android](http://developer.android.com/design/patterns/pure-android.html) - Simple guidelines for following Android standards
  * [App Structure](http://developer.android.com/design/patterns/app-structure.html) - Guidelines for general app structure
4. **Benchmarking** - Check out the [[following sites|Android-Design-Guidelines#benchmarking]] for looking at how popular apps look and feel
5. **Styling with Generators**
  * **Style ActionBar** - Customize the ActionBar with [this generator](http://jgilfelt.github.io/android-actionbarstylegenerator/), copy over the files, and apply the theme. 
  * **Style Views** - Customize the View control colors using the [Holo Colors Generator](http://android-holo-colors.com/)
  * **Style Buttons** - Customize the buttons using the [Button Style Generator](http://angrytools.com/android/button/)

## Further Reading

Additional reading:
  
  * Review the [[screen styling FAQ|Styling-UI-Screens-FAQ]].
  * Review the [[material design primer|Material-Design-Primer]].
  * Review the [[complete drawables|Drawables]] cliffnotes 
  * Review the [[styles and themes|Styles-and-Themes]] cliffnotes
  * Review our [[styling the ActionBar guide|Extended-ActionBar-Guide#custom-actionbar-styles]]
  * Review our [[styling tabs guide|Sliding-Tabs-with-PagerSlidingTabStrip#customize-tab-styles]] 

The original gist source for the page can be found [here](https://gist.github.com/nesquena/6c567083aec13d868017).