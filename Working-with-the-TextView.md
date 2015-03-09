## Overview

Every Android device comes with a collection of standard fonts: Droid Sans, Droid Sans Mono and Droid Serif. They were designed to be optimal for mobile displays, so these are the three fonts you will be working with most of the time and they can be styled using a handful of XML attributes. You might, however, see the need to use custom fonts for special purposes. 

This guide will take a look at the [TextView](http://developer.android.com/reference/android/widget/TextView.html) and discuss common properties associated with this view as well as how to setup custom typefaces.

## Text Attributes

### Typeface

As stated in the overview, there are three different default typefaces which are known as the Droid family of fonts: `sans`, `monospace` and `serif`. You can specify any one of them as the value for the `android:typeface` attribute in the XML:

```xml
<TextView
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:text="This is a 'sans' demo!"
    android:typeface="sans"
/>
```

Here's how they look:

![Fonts](http://i.imgur.com/or5z86M.png)

In addition to the above, there is another attribute value named "normal" which defaults to the sans typeface.

### Text Style

The `android:textStyle` attribute can be used to put emphasis on the text. The possible values are: `normal`, `bold`, `italic`. You can also specify `bold|italic`.

```xml
<TextView
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:text="This is bold!"
    android:textStyle="bold"
/>
```

### Text Size

`android:textSize` specifies the font size. Its value must consist of two parts: a floating-point number followed by a unit. It is generally a good practice to use the `sp` unit so the size can scale depending on user settings.

```xml
<TextView
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:text="15sp is the 'normal' size."
    android:textSize="15sp"
/>
```

### Text Truncation

There are a few ways to truncate text within a `TextView`. First, to restrict the total number of lines of text we can use `android:maxLines` and `android:minLines`:

```xml
<TextView
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:minLines="1"
    android:maxLines="3"
/>
```

In addition, we can use `android:ellipsize` to begin truncating text 

```xml
<TextView
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:ellipsize="end"
    android:singleLine="true"
/>
```

Following values are available for `ellipsize`: `start` for `...bccc`, `end` for `aaab...`, `middle` for `aa...cc`, and `marquee` for `aaabbbccc` sliding from left to right. There is a known issue with **ellipsize and multi-line text**, see [this MultiplelineEllipsizeTextView library](https://github.com/IPL/MultiplelineEllipsizeTextView) for an alternative.

### Text Shadow

You can use three different attributes to customize the appearance of your text shadow:

 * `android:shadowColor` - Shadow color in the same format as textColor.
 * `android:shadowRadius` - Radius of the shadow specified as a floating point number.
 * `android:shadowDx` - The shadow's horizontal offset specified as a floating point number.
 * `android:shadowDy` - The shadow's vertical offset specified as a floating point number.

The floating point numbers don't have a specific unit - they are merely arbitrary factors.

```xml
<TextView
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:text="A light blue shadow."
    android:shadowColor="#00ccff"
    android:shadowRadius="1.5"
    android:shadowDx="1"
    android:shadowDy="1"
/>
```

### Text Color

The `android:textColor` attribute's value is a hexadecimal RGB value with an optional alpha channel, similar to what's found in CSS:

```xml
<TextView
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:text="A light blue color."
    android:textColor="#00ccff"
/>
```

### Related Text Colors

There are several related text color properties in addition such as `android:textColorHighlight`, `android:textColorHint`, and `android:textColorLink` which affect the properties for highlighting, hint, and link color respectively:

```xml
<TextView
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:textColorHighlight="#7C82D2"
    android:textColorHint="#2DC942"
    android:textColorLink="#8DE67F"
/>
```

In cases where we want to change the TextView colors within the theme itself, we need to [modify the styles.xml](http://stackoverflow.com/a/21984419) in these cases.

## Inserting HTML Formatting

TextView natively supports [HTML](http://developer.android.com/reference/android/text/Html.html) by translating HTML tags to [spannable](http://developer.android.com/reference/android/text/Spannable.html) sections within the view. To apply basic HTML formatting to text, add text to the TextView with:

```java
TextView view = (TextView)findViewById(R.id.sampleText);
String formattedText = "This <i>is</i> a <b>test</b>";
// or getString(R.string.htmlFormattedText);
view.setText(Html.fromHtml(formattedText));
```

Note that all tags are not supported. See [this article](http://javatechig.com/android/display-html-in-android-textview) for a more detailed look at supported tags and usages. If you want to store your HTML text within `res/values/strings.xml`, you have to use CDATA to escape such as:

```xml
<?xml version="1.0" encoding="utf-8"?>
<string name="htmlFormattedText">
    <![CDATA[
        Please <a href="http://highlight.com">let us know</a> if you have <b>feedback on this</b> or if 
        you would like to log in with <i>another identity service</i>. Thanks!   
    ]]>
</string>
```

and access the content with `getString(R.string.htmlFormattedText)` to load this within the TextView.

For more advanced cases, you can also check out the [html-textview](https://github.com/dschuermann/html-textview) library which adds support for almost any HTML tag within this third-party TextView.

## Autolinking URLs

TextView has [native support](http://developer.android.com/reference/android/widget/TextView.html#attr_android:autoLink) for automatically locating URLs within the their text content and making them clickable links which can be opened in the browser. To do this, enable the `android:autolink` property:

```xml
<TextView
     android:id="@+id/custom_font"
     android:layout_width="match_parent"
     android:layout_height="wrap_content"
     android:autoLink="all"
     android:linksClickable="true"
/>
```

### Issues with ListView

One known issue when using `android:autoLink` or the `Linkify` class is that it may break the ability to respond to events on the ListView through `setOnItemClickListener`.  Check out [this solution](http://www.michaelevans.org/blog/2013/03/29/clickable-links-in-android-listviews/) which extends `TextView` in order to modify the `onTouchEvent` to correctly propagate the click.   You basically need to create a `LinkifiedTextView` and use this special View in place of any of your TextView's that need auto-link detection.

In addition, review [this stackoverflow post](http://stackoverflow.com/questions/26980204/listview-with-textview-autolink-not-receiving-onitemclicklistener) or [this android issue](https://code.google.com/p/android/issues/detail?id=3414) for additional context.

## Using Custom Fonts

We can actually use any custom font that we'd like within our applications. Check out [fontsquirrel](http://www.fontsquirrel.com/) for an easy source of free fonts. For example, we can download [Chantelli Antiqua](http://www.fontsquirrel.com/fonts/Chantelli-Antiqua) as an example. Download it and **place the TTF file in the ./assets/fonts directory**.

We're going to use a basic layout file with a TextView, marked with an id of "custom_font" so we can access it in our code.

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:orientation="vertical"
              android:layout_width="match_parent"
              android:layout_height="match_parent">
 
    <TextView
            android:id="@+id/custom_font"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="This is the Chantelli Antiqua font."
    />
</LinearLayout>
```

Open your main activity file and insert this into the `onCreate()` method:

```java
// Get access to our TextView
TextView txt = (TextView) findViewById(R.id.custom_font);
// Create the TypeFace from the TTF asset
Typeface font = Typeface.createFromAsset(getAssets(), "fonts/Chantelli_Antiqua.ttf");
// Assign the typeface to the view
txt.setTypeface(font);
```

And this will result in:

![Custom](http://i.imgur.com/WW2QWpe.png)

You'll also want to keep an eye on the total size of your custom fonts, as this can grow quite large if you're using a lot of different typefaces.

## Displaying Images within a TextView

A TextView is actually surprisingly powerful and actually supports having images displayed as a part of it's content area. Any images stored in the "drawable" folders can actually be embedded within a TextView at several key locations in relation to the text using the [android:drawableRight](http://developer.android.com/reference/android/widget/TextView.html#attr_android:drawableRight) and the `android:drawablePadding` property. For example:

```xml
<TextView xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"     
    android:gravity="center"
    android:text="@string/my_contacts"
    android:drawableRight="@drawable/ic_action_add_group"
    android:drawablePadding="8dp"
/>
```

Which results in:

![Contacts View](http://i.imgur.com/LoN8jpH.png)

In Android, many views inherit from `TextView` such as `Button`s, `EditText`s, `RadioButton`s which means that all of these views support the same functionality. For example, we can also do:

```xml
<EditText
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:hint="@string/user_name"
    android:drawableLeft="@drawable/ic_action_person"
    android:drawablePadding="8dp"
/>
```

Which results in:

![EditText with drawable](http://i.imgur.com/GZiIf1C.png)

The relevant attributes here are `drawableLeft`, `drawableRight`, `drawableTop` and `drawableBottom` along with `drawablePadding`. Check out [this TextView article](http://antonioleiva.com/textview_power_drawables/) for a more detailed look at how to use this functionality. 

## References

* <http://code.tutsplus.com/tutorials/customize-android-fonts--mobile-1601>
* <http://www.androidhive.info/2012/02/android-using-external-fonts/>
* <http://stackoverflow.com/questions/3651086/android-using-custom-font>
* <http://www.tutorialspoint.com/android/android_custom_fonts.htm>
* <http://antonioleiva.com/textview_power_drawables/>