# Customizing with Drawables

## Overview

A drawable resource is a general concept for a graphic that can be drawn to the screen. Drawables are used to define shapes, colors, borders, gradients, et al which can then be applied to views within an Activity.

This is typically used for customizing the view graphics that are displayed within a particular view or context. Drawables tend to be defined in XML and can then be applied to a view via XML or Java.

## Usage

Drawables can be an initially overwhelming topic because there are many drawable types used in different situations such as drawing shapes, setting state behaviors for buttons, creating stretchable button backgrounds and creating compound drawable layers.

Let's explore the drawable file types one by one and take a look at examples of each.

### Drawing Shapes

The Shape Drawable is an XML file that defines a geometric shape, including colors and gradients. This is used to create a complex shape that can then be attached as the background of a layout or a view on screen. For example, you can use a shape drawable to change the shape, border, and gradient of a button background.

A shape is simply a collection of properties that are combined to describe a background. The shape can be described with properties such as `corners` for rounding, `gradient` for backgrounds, `padding` for spacing, `solid` for background colors, and `stroke` for border. 

#### Solid Color Shapes

Here's an example of drawing a rounded rectangle with a border in `res/layout/drawable/solid_color_shape.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="rectangle" >
    <corners android:radius="4dp" />
    <stroke android:width="4dp" android:color="#C1E1A6" /> 
    <solid android:color="#118C4E"/> 
    <padding android:left="20dp" android:top="20dp" 
             android:right="20dp" android:bottom="20dp" /> 
</shape>
```

and then applied to a TextView:

```xml
<TextView
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"
   android:background="@drawable/shape_test"
   android:textColor="#ffffff"
   android:text="@string/hello_world" />
```

and the resulting view looks like:

![Solid Shape](http://i.imgur.com/RPzvPc7.png)

#### Gradient Colored Shapes

Shapes also support [gradients backgrounds](http://developer.android.com/guide/topics/resources/drawable-resource.html#gradient-element) as well supporting properties such as `startColor`, `centerColor`, `endColor`, `angle`. Different gradients such as radial, linear or sweep can be selected using the `type` property. 

Here's an example of a simple linear gradient shape specified in `res/layout/drawable/gradient_shape.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="rectangle" >
    <corners android:radius="4dp" />
    <stroke android:width="1dp" android:color="#0078a5" /> 
    <gradient
        android:startColor="#0078a5" android:endColor="#00adee" 
        android:angle="90"/>
    <padding android:left="8dp" android:top="2dp" 
             android:right="8dp" android:bottom="2dp" /> 
</shape>
```

applied to a button and the resulting view looks like:

![Gradient Shape](http://i.imgur.com/g8b9XWb.png)

Using solid color shapes and gradients we can customize the appearance of buttons, layouts and other views without requiring the use of any images.

### State List

A [StateListDrawable](http://developer.android.com/guide/topics/resources/drawable-resource.html#StateList) is a drawable object defined in XML that uses a several different images to represent the same graphic, depending on the state of the object. For example, a Button widget can exist in one of several different states (pressed, focused, or niether) and, using a state list drawable, you can provide a different background image for each state. The state list supports different view states such as `android:state_pressed`, `android:state_focused`, `android:state_hovered`, `android:state_selected`, among many others.

For example, a state list XML for a button might look like the following:

```xml
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item
        android:state_pressed="true"
        android:state_enabled="true"
        android:drawable="@drawable/button_pressed" />
    <item
        android:state_focused="true"
        android:state_enabled="true"
        android:drawable="@drawable/button_focused" />
    <item
        android:state_enabled="true"
        android:drawable="@drawable/button_enabled" />
</selector>
```

Now, when the view (i.e button) is pressed or focused, the drawable used for the view will change accordingly. 

### Creating a Layer List

A [LayerDrawable](http://developer.android.com/guide/topics/resources/drawable-resource.html#LayerList) is a drawable object that manages an array of other drawables. Each drawable in the list is drawn in the order of the list—the last drawable in the list is drawn on top. Each drawable is represented by an `<item>` element inside a single `<layer-list>` element.

The LayerList can be used to draw multiple other drawables (shapes, images, etc) and position them in relationship to one another. For example, the following layer list draws several shapes:

```xml
<?xml version="1.0" encoding="utf-8"?>
<layer-list xmlns:android="http://schemas.android.com/apk/res/android" >
    <item>
      <shape 
        android:shape="rectangle">
            <stroke android:width="1dp" android:color="#585858" />
            <solid android:color="#FF9009" />
            <padding android:bottom="1dp"/>
        </shape>
   </item>
   <item android:left="10dp" android:top="20dp" android:bottom="20dp" android:right="150dp">
      <shape 
        android:shape="oval">
            <stroke android:width="1dp" android:color="#ffffff" />
            <solid android:color="#118C4E" />
            <padding android:bottom="1dp"/>
        </shape>
   </item>
   <item android:left="150dp" android:top="20dp" android:bottom="20dp" android:right="10dp">
      <shape 
        android:shape="rectangle">
            <stroke android:width="1dp" android:color="#ffffff" />
            <solid android:color="#C1E1A6" />
            <padding android:bottom="1dp"/>
        </shape>
   </item>
</layer-list>
```

and this results in the following:

![Layer List](http://i.imgur.com/T1BN9tf.png)

Keep in mind that a [LayerList](http://developer.android.com/guide/topics/resources/drawable-resource.html#LayerList) can also draw images and any other types of drawables. You can use this to create more complex drawings and overlay multiple drawables ontop of each other. See more examples within the [official docs](http://developer.android.com/guide/topics/resources/drawable-resource.html#LayerList).

### Stretchable Nine-Patch Image

A [NinePatch](http://developer.android.com/guide/topics/graphics/2d-graphics.html#nine-patch) is a PNG image in which you can define stretchable regions that Android scales when content within the View exceeds the normal image bounds. You typically assign this type of image as the background of a View which has width set to `wrap_content`. The most common use is a button that has to stretch based on the text displayed inside.

NinePatch are images that have a "9" in the name such as `button_background.9.png` which signifies that this is a stretchable image. The [draw9patch](http://developer.android.com/tools/help/draw9patch.html) tool is used to create special nine-patch images.  a NinePatch is defined and stored, setting the background works the same as with any image:

```xml
<Button android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Submit"
        android:background="@drawable/button_background"/>
```

Read more about NinePatch images in the [offical docs](http://developer.android.com/guide/topics/graphics/2d-graphics.html#nine-patch). Once

### Customizing a Button

Creating a custom button require combining at least a state list drawable with a shape drawable. First, let's create our shape drawable which represents the "default" button background in `res/drawable/nice_button_enabled.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android" android:shape="rectangle">
    <gradient
        android:startColor="#0078a5"
        android:endColor="#00adee"
        android:angle="90"/>
    <padding android:left="15dp"
        android:top="1dp"
        android:right="15dp"
        android:bottom="1dp" />
    <stroke
        android:width="1dp"
        android:color="#0076a3" />
    <corners android:radius="8dp" />
</shape>
```

Let's also create a style (set of view properties) that includes setting the background in `res/values/styles.xml`:

```xml
<style name="NiceButton" parent="@android:style/Widget.Button">
    <item name="android:gravity">center_vertical|center_horizontal</item>
    <item name="android:textColor">#FFFFFF</item>
    <item name="android:background">@drawable/nice_button_enabled</item>
    <item name="android:textSize">16sp</item>
    <item name="android:textStyle">bold</item>
    <item name="android:focusable">true</item>
    <item name="android:clickable">true</item>
</style>
```

This now represents the shape and background of the button in the default state along with all the other properties. We could apply this by setting the `background` of the button:

```xml
<Button
    android:id="@+id/btnGo"
    style="@style/NiceButton"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
/>
```

If we run that, we will see the following button:

![Button Graphic 1](http://i.imgur.com/UJc7SQd.png)

Now the button shows up nicely but doesn't have any "pressed" or "focused" states. To do this, we need to create a **State List** drawable that describes the drawables for each state in `res/drawable/states_nice_button.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item
        android:state_pressed="true"
        android:state_enabled="true"
        android:drawable="@drawable/nice_button_pressed" />
    <item
        android:state_focused="true"
        android:state_enabled="true"
        android:drawable="@drawable/nice_button_focused" />
    <item
        android:state_enabled="true"
        android:drawable="@drawable/nice_button_enabled" />
</selector>
```

This describes the look of the button in all three major states (default, pressed and focused). Now we need to create two more Shape drawable states. One for `res/drawable/nice_button_pressed.xml` and another for `res/drawable/nice_button_focused.xml`:

```
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android" android:shape="rectangle">
    <gradient
        android:startColor="#00adee"
        android:endColor="#0078a5"
        android:angle="90"/>
    <padding android:left="15dp"
        android:top="1dp"
        android:right="15dp"
        android:bottom="1dp" />
    <stroke
        android:width="1dp"
        android:color="#0076a3" />
    <corners android:radius="8dp" />
</shape>
```

For this guide the pressed and focused states will appear the same but of course these could be distinct visual states as well. Now, we need to change the style to use the `res/drawable/states_nice_button.xml`:

```xml
<style name="NiceButton" parent="@android:style/Widget.Button">
    <item name="android:gravity">center_vertical|center_horizontal</item>
    <item name="android:textColor">#FFFFFF</item>
    <item name="android:background">@drawable/states_nice_button</item>
    <item name="android:textSize">16sp</item>
    <item name="android:textStyle">bold</item>
    <item name="android:focusable">true</item>
    <item name="android:clickable">true</item>
</style>
```

Now we have a button that has a nice shape drawable background and changes visual state when pressed all without requiring a single image asset! Be sure to check out the [Button Generator](http://angrytools.com/android/button/) for a tool that allows you to make your own buttons via a web interface.

### Additional Drawable Types

 * [LevelList](http://developer.android.com/guide/topics/resources/drawable-resource.html#LevelList) - A Drawable that manages a number of alternate Drawables, each assigned a maximum numerical value.
 * [TransitionDrawable](http://developer.android.com/guide/topics/resources/drawable-resource.html#Transition) - A drawable object that can cross-fade between the two drawable resources. Used for animating between two drawables.
 * [InsetDrawable](http://developer.android.com/guide/topics/resources/drawable-resource.html#Inset) - A drawable defined in XML that insets another drawable by a specified distance. This is useful when a View needs a background that is smaller than the View's actual bounds.
 * [ClipDrawable](http://developer.android.com/guide/topics/resources/drawable-resource.html#Clip) - A drawable defined in XML that clips another drawable based on this Drawable's current level. Most often used to implement things like progress bars.
 * [ScaleDrawable](http://developer.android.com/guide/topics/resources/drawable-resource.html#Scale) - A drawable defined in XML that changes the size of another drawable based on its current level.

## Tools

 * <http://androiddrawables.com/>
 * <http://angrytools.com/android/button/>

## References

 * <http://developer.android.com/guide/topics/resources/drawable-resource.html>
 * <http://www.vogella.com/articles/AndroidDrawables/article.html>
 * <http://developer.android.com/reference/android/graphics/drawable/Drawable.html>
 * <http://android-dev-tips-and-tricks.blogspot.com/2012/08/xml-drawables-part-i.html>
 * <http://androidcookbook.com/Recipe.seam;jsessionid=8BED36512503CA63614CA9237248CBE7?recipeId=3307>