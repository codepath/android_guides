## Overview
A drawable resource is a general concept for a graphic that can be drawn to the screen. Drawables are used to define shapes, colors, borders, gradients, etc. which can then be applied to views within an Activity.

This is typically used for customizing the view graphics that are displayed within a particular view or context. Drawables tend to be defined in XML and can then be applied to a view via XML or Java.

## Usage

Drawables can be an initially overwhelming topic because there are many drawable types used in different situations such as drawing shapes, setting state behaviors for buttons, creating stretchable button backgrounds and creating compound drawable layers.

There are at least 17 types of drawables but there are five that are most important to understand:

1. **Shape Drawables** - Defines a shape with properties such as stroke, fill, and padding
2. **StateList Drawables** - Defines a list of drawables to use for different states
3. **LayerList Drawables** - Defines a list of drawables grouped together into a composite result
4. **NinePatch Drawables** - A PNG file with stretchable regions to allow proper resizing
5. **Vector Drawables** - Defines complex XML-based vector images

Let's explore these drawable file types one by one and take a look at examples of usage.

### Drawing Shapes

The [Shape Drawable](http://developer.android.com/guide/topics/resources/drawable-resource.html#Shape) is an XML file that defines a geometric shape, including colors and gradients. This is used to create a complex shape that can then be attached as the background of a layout or a view on screen. For example, you can use a shape drawable to change the shape, border, and gradient of a button background.

A shape is simply a collection of properties that are combined to describe a background. The shape can be described with properties such as `corners` for rounding, `gradient` for backgrounds, `padding` for spacing, `solid` for background colors, and `stroke` for border. 

#### Solid Color Shapes

Here's an example of drawing a rounded rectangle with a border in `res/layout/drawable/solid_color_shape.xml`:

```xml
<?xml version="1.0" enc