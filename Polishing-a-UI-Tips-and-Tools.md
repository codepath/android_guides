## Overview

Building beautiful Android apps starts with understanding how to approach building a delightful UI and each of the components that contributes:

<a href="http://androidniceties.tumblr.com/">
  <img src="https://i.imgur.com/Zt9ZIys.jpg" alt="design" width="250" />&nbsp;
  <img src="https://i.imgur.com/UECNpcx.png" alt="design" width="250" />&nbsp;
  <img src="https://i.imgur.com/I4qigt5.jpg" alt="design" width="250" />
</a>

## Critical Guidelines

Be sure to pay attention to the following elements of beautiful visual design. The devil is in the details. Trust that small improvements can make a big difference on the overall experience:

"Perfection is achieved not when there is nothing more to add, but when there is nothing left to take away."

 - **Hierarchy.** Set a clear hierarchy on the screen. One thing should clearly be the most important (i.e. larger, more colorful). If other things are on the screen, then they should clearly be less important (i.e. less contrast, less space around them). A lot of design mistakes can be cleared up by answering "What is the most important thing here? What is second and third?"
 - **Colors.** Start with one color plus light and dark grey. Add another color if you want to. You could add a third, but more than three colors is usually unnecessary. Two-to-six shades of light and dark grey will usually be enough. Avoid shades of grey in the middle range.
 - **Typography.** Establish a clear size hierarchy. As a rule of thumb, have three basic sizes: headline, title, and body (regular) size. The font-size menus in word processors show certain numbers (18, 36, 64 etc.) because their ratios go well together, so that is a good place to pick from. 
 - **Iconography.** Icons should generally be a muted single-color (gray, faded colors) and not be too eye-catching unless they are actions or meant to draw attention. 
 - **Spacing.** Make sure you to tighten up spacing of elements. Spacing on the top and bottom should be consistent. Pay attention to proper spacing between elements as well. 
 - **Alignment.** Make sure elements are aligned to a grid. In other words, each view should be aligned to surrounding elements (i.e text to the middle or edge of an image). Here is a useful exercise: Pick the left edge of any object in a design. Now draw a vertical line from the top to the bottom of the screen, aligned with that edge. Now do that for every element on screen, unless it already rests on a line. How many lines do you end up with? Try to reduce that number by aligning elements to the same vertical lines.
 - **Information.** Present only the information that is necessary in a given moment. Having multiple pieces of text that are all near each other, or several items that are exactly the same size color and style, is usually not a good practice. 
 - **Tabs.** Tabs should be a consistent with the background color of the Toolbar above them.

## Links and Resources

Polishing up the user interface of your application starts with the following enhancements:

 1. Benchmarking to **Adhere to Good Designs** - Check out the [[following sites|Android-Design-Guidelines#benchmarking]] or [these material design examples](http://www.materialup.com/) for looking at how popular apps look and feel.
 2. Pick a **Vibrant Color Scheme** - Pick a primary color and a secondary color for [coloring your app](http://www.google.com/design/spec/style/color.html#color-ui-color-application) using a [sensible color scheme](http://www.colourlovers.com/palettes/new/past-month/meta?page=1). Check out [material palette](http://www.materialpalette.com/) or the [coolors generator](https://coolors.co/) for other takes on picking colors. 
 3. Use **Proper Icons and Colorful Images** - Use images, icons and backgrounds for your UIs leveraging resources like [MaterialDesignIcons](http://materialdesignicons.com/), [IconFinder](https://www.iconfinder.com/), [iconmonstr](http://iconmonstr.com/), [NounProject](http://thenounproject.com/), [flickr](https://www.flickr.com/search/) and [Google Image Search](http://www.google.com/imghp) to locate relevant assets. Adhere carefully to the [iconography style guidelines](http://developer.android.com/design/style/iconography.html) for Android
 4. Review **Typography** - Check out the [typography guide](http://developer.android.com/design/style/typography.html) to understand the common font types for Android apps and default type colors and sizes. See the [calligraphy library](https://github.com/chrisjenx/Calligraphy) for easy custom fonts. 
 5. Apply **backgrounds and borders** to views and layouts - Use [[shape and layer drawables|Drawables]] cliffnotes to create colorful backgrounds and borders to your [[buttons|Drawables#customizing-a-button]], [[listviews|Drawables#customizing-a-listview]], and other views. See the [[material card view|Using-the-CardView]] for a look at modern styles for lists.
 6. Improve **ActionBar and Navigation Appearance** -  Review our style guides for the [[ActionBar|Extended-ActionBar-Guide#custom-actionbar-styles]] and [[Tabs guide|Google-Play-Style-Tabs-using-TabLayout#styling-the-tablayout]]. See generators linked in next section.
 7. Follow **Android UI Standards** - Use [[modern material design guidelines|Material-Design-Primer]], [[common navigation styles|Android-Design-Guidelines#common-patterns]]. Review the [[Android Design Guidelines]] with [proper app structure](http://developer.android.com/design/patterns/app-structure.html) and be sure to [design for Android](http://developer.android.com/design/patterns/pure-android.html).
 8. Implement **Intermediate UI Elements** - Make sure to [[add progress bars|Handling-ProgressBars]] when loading, along with placeholders for images and empty states in cases when there's no content.

## Further Reading

Additional reading:
  
  * Review the [[Android Design Guidelines]] page.
  * Review the [[screen styling FAQ|Styling-UI-Screens-FAQ]].
  * Review the [[material design primer|Material-Design-Primer]].
  * Review the [[complete drawables|Drawables]] cliffnotes.
  * Review the [[styles and themes|Styles-and-Themes]] cliffnotes.
  * Review our [[styling the ActionBar guide|Extended-ActionBar-Guide#custom-actionbar-styles]].
  * Review our [[styling tabs guide|Sliding-Tabs-with-PagerSlidingTabStrip#customize-tab-styles]]. 

## References

* [original gist here](https://gist.github.com/nesquena/6c567083aec13d868017)