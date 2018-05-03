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

<img alt="fonts" src="https://i.imgur.com/BES7g98.png" width="400" />

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

A sampling of styles can be seen below:

<img alt="style" src="https://i.imgur.com/BcX2r9O.png" width="400" />

### Text Size

`android:textSize` specifies the font size. Its value must consist of two parts: a floating-point number followed by a unit. It is generally a good practice to use the `sp` unit so the size can scale depending on user settings.

```xml
<TextView
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:text="14sp is the 'normal' size."
    android:textSize="14sp"
/>
```

A sampling of styles can be seen below:

<img alt="style" src="https://i.imgur.com/4pimMzN.png" width="400" />

Too many type sizes and styles at once can wreck any layout. The basic set of styles are based on a typographic scale of 12, 14, 16, 20, and 34. Refer to this [typography styles guide](https://www.google.com/design/spec/style/typography.html#typography-styles) for more details. 

### Text Truncation

There are a few ways to truncate text within a `TextView`. First, to restrict the total number of lines of text we can use `android:maxLines` and `android:minLines`:

```xml
<TextView
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:minLines="1"
    android:maxLines="2"
/>
```

In addition, we can use `android:ellipsize` to begin truncating text 

```xml
<TextView
    ...
    android:ellipsize="end"
    android:singleLine="true"
/>
```

Following values are available for `ellipsize`: `start` for `...bccc`, `end` for `aaab...`, `middle` for `aa...cc`, and `marquee` for `aaabbbccc` sliding from left to right. Example:

<img alt="style" src="https://i.imgur.com/NoKo7Ou.png" width="400" />

There is a known issue with **ellipsize and multi-line text**, see [this MultiplelineEllipsizeTextView library](https://github.com/IPL/MultiplelineEllipsizeTextView) for an alternative.

### Text Color

The `android:textColor` and `android:textColorLink` attribute values are hexadecimal RGB values with an optional alpha channel, similar to what's found in CSS:

```xml
<TextView
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:text="A light blue color."
    android:textColor="#00ccff"
    android:textColorLink="#8DE67F"
/>
```

The `android:textColorLink` attribute controls the highlighting for [[hyperlinks embedded within the TextView|Working-with-the-TextView#inserting-html-formatting]]. This results in:

![](https://i.imgur.com/UlLSrEG.png)

We can edit the color at runtime with:

```java
// Based on hex value
textView.setColor(Color.setTextColor(Color.parseColor("#000000"));
// based on a color resource file
textView.setTextColor(ContextCompat.getColor(context, R.color.your_color));
// based on preset colors
textView.setColor(Color.setTextColor(Color.RED));
```

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
    android:shadowRadius="2"
    android:shadowDx="1"
    android:shadowDy="1"
/>
```

This results in:

![](https://i.imgur.com/blFEHxX.png)

### Various Text Properties

There are many other text properties including `android:lineSpacingMultiplier`, `android:letterSpacing`, `android:textAllCaps`, `android:includeFontPadding` and [many others](http://developer.android.com/reference/android/widget/TextView.html#nestedclasses):

```xml
<TextView
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:lineSpacingMultiplier="1.1"
    android:textAllCaps="true"
/>
```

`android:includeFontPadding` removes the extra padding around large fonts. `android:lineSpacingMultiplier` governs the spacing between lines with a default of "1".

## Inserting HTML Formatting

TextView natively supports [HTML](http://developer.android.com/reference/android/text/Html.html) by translating HTML tags to [spannable](http://developer.android.com/reference/android/text/Spannable.html) sections within the view. To apply basic HTML formatting to text, add text to the TextView with:

```java
TextView view = (TextView)findViewById(R.id.sampleText);
String formattedText = "This <i>is</i> a <b>test</b> of <a href='http://foo.com'>html</a>";
// or getString(R.string.htmlFormattedText);
view.setText(Html.fromHtml(formattedText));
```

This results in:

![](https://i.imgur.com/PEl2EKl.png)

Note that all tags are not supported. See [this article](http://javatechig.com/android/display-html-in-android-textview) for a more detailed look at supported tags and usages. 

### Android N Usage

As of Android N, the signature for the method changed, requiring there to be a check depending on version to remove the deprecation warning:

```java
@SuppressWarnings("deprecation")
public static Spanned fromHtml(String html){
    Spanned result;
    if (android.os.Build.VERSION.SDK_INT >= android.os.Build.VERSION_CODES.N) {
       result = Html.fromHtml(html,Html.FROM_HTML_MODE_LEGACY);
    } else {
       result = Html.fromHtml(html);
    }
    return result;
}
```

You can [read more about the html modes here](https://developer.android.com/reference/android/text/Html.html#FROM_HTML_MODE_COMPACT).

### Setting Font Colors

For setting font colors, we can use the `<font>` tag as shown:

```java
Html.fromHtml("Nice! <font color='#c5c5c5'>This text has a color</font>. This doesn't"); 
```

And you should be all set. 

### Storing Long HTML Strings

If you want to store your HTML text within `res/values/strings.xml`, you have to use CDATA to escape such as:

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

This results in:

![](https://i.imgur.com/73bwaRm.png)

### Issues with ListView

One known issue when using `android:autoLink` or the `Linkify` class is that it may break the ability to respond to events on the ListView through `setOnItemClickListener`.  Check out [this solution](http://www.michaelevans.org/blog/2013/03/29/clickable-links-in-android-listviews/) which extends `TextView` in order to modify the `onTouchEvent` to correctly propagate the click. You basically need to create a `LinkifiedTextView` and use this special View in place of any of your TextView's that need auto-link detection.

In addition, review these alternate solutions which may be effective as well:

 * [This stackoverflow post](http://stackoverflow.com/questions/26980204/listview-with-textview-autolink-not-receiving-onitemclicklistener) or [this other post](https://stackoverflow.com/a/8654237/313399)
 * [This android issue](https://code.google.com/p/android/issues/detail?id=3414) for additional context.

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

![Contacts View](https://i.imgur.com/LoN8jpH.png)

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

![EditText with drawable](https://i.imgur.com/GZiIf1C.png)

The relevant attributes here are `drawableLeft`, `drawableRight`, `drawableTop` and `drawableBottom` along with `drawablePadding`. Check out [this TextView article](http://antonioleiva.com/textview_power_drawables/) for a more detailed look at how to use this functionality. 

Note that if you want to be able to better control the size or scale of the drawables, check out [this handy TextView extension](http://stackoverflow.com/a/31916731/313399) or [this bitmap drawable approach](http://stackoverflow.com/a/29804171/313399). You can also make calls to [setCompoundDrawablesWithIntrinsicBounds](https://groups.google.com/forum/#!topic/android-developers/_Gzbe0KCP_0) on the `TextView`.

## Using Custom Fonts

We can actually use any custom font that we'd like within our applications. Check out [fontsquirrel](http://www.fontsquirrel.com/) for an easy source of free fonts. For example, we can download [Chantelli Antiqua](http://www.fontsquirrel.com/fonts/Chantelli-Antiqua) as an example. 

Fonts are stored in the "assets" folder. In Android Studio, `File > New > folder > Assets Folder`. Now download any font and **place the TTF file in the `assets/fonts` directory**:

![](https://i.imgur.com/2dxTeGY.png)

We're going to use a basic layout file with a `TextView`, marked with an id of "custom_font" so we can access it in our code.

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

To set the custom font manually, open your activity file and insert this into the `onCreate()` method:

```java
// Get access to our TextView
TextView txt = (TextView) findViewById(R.id.custom_font);
// Create the TypeFace from the TTF asset
Typeface font = Typeface.createFromAsset(getAssets(), "fonts/Chantelli_Antiqua.ttf");
// Assign the typeface to the view
txt.setTypeface(font);
```

Alternatively, you can use the third-party [calligraphy library](https://github.com/chrisjenx/Calligraphy):

```
<TextView fontPath="fonts/Chantelli_Antiqua.ttf"/>
```

Either method will will result in:

<img alt="custom" src="https://i.imgur.com/jlTQpEY.png" width="400" />

You'll also want to keep an eye on the total size of your custom fonts, as this can grow quite large if you're using a lot of different typefaces. 

## Using Spans to Style Sections of Text

Spans come in really handy when we want to apply styles to portions of text within the same TextView. We can change the text color, change the typeface, add an underline, etc, and apply these to only certain portions of the text. The [full list of spans](http://developer.android.com/reference/android/text/style/package-summary.html) shows all the available options.

As an example, let's say we have a single TextView where we want the first word to show up in red and the second word to have a strikethrough:
 
![Custom](https://i.imgur.com/X9tKFmv.png)

We can accomplish this with spans using the code below:

```java
String firstWord = "Hello";
String secondWord = "World!";

TextView tvHelloWorld = (TextView)findViewById(R.id.tvHelloWorld);

// Create a span that will make the text red
ForegroundColorSpan redForegroundColorSpan = new ForegroundColorSpan(
        getResources().getColor(android.R.color.holo_red_dark));

// Use a SpannableStringBuilder so that both the text and the spans are mutable
SpannableStringBuilder ssb = new SpannableStringBuilder(firstWord);

// Apply the color span
ssb.setSpan(
        redForegroundColorSpan,            // the span to add
        0,                                 // the start of the span (inclusive)
        ssb.length(),                      // the end of the span (exclusive)
        Spanned.SPAN_EXCLUSIVE_EXCLUSIVE); // behavior when text is later inserted into the SpannableStringBuilder
                                           // SPAN_EXCLUSIVE_EXCLUSIVE means to not extend the span when additional
                                           // text is added in later

// Add a blank space
ssb.append(" ");

// Create a span that will strikethrough the text
StrikethroughSpan strikethroughSpan = new StrikethroughSpan();

// Add the secondWord and apply the strikethrough span to only the second word
ssb.append(secondWord);
ssb.setSpan(
        strikethroughSpan,
        ssb.length() - secondWord.length(),
        ssb.length(),
        Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);

// Set the TextView text and denote that it is Editable
// since it's a SpannableStringBuilder
tvHelloWorld.setText(ssb, TextView.BufferType.EDITABLE);
```

Note: There are 3 different classes that can be used to represent text that has markup attached. [SpannableStringBuilder](http://developer.android.com/reference/android/text/SpannableStringBuilder.html) (used above) is the one to use when dealing with mutable spans and mutable text. [SpannableString](http://developer.android.com/reference/android/text/SpannableString.html) is for mutable spans, but immutable text. And [SpannedString](http://developer.android.com/reference/android/text/SpannedString.html) is for immutable spans and immutable text.

### Creating Clickable Styled Spans

In certain cases, we might want different substrings in a `TextView` to different styles and then clickable to trigger an action. For example, rendering tweet items where `@foo` can be clicked in a message to view a user's profile. For this, you should copy over the [PatternEditableBuilder.java](https://gist.github.com/nesquena/f2504c642c5de47b371278ee61c75124#file-patterneditablebuilder-java) utility into your app. You can then use this utility to make clickable spans. For example:

```java
// Set text within a `TextView`
TextView textView = (TextView) findViewById(R.id.textView);
textView.setText("Hey @sarah, where did @jim go? #lost");
// Style clickable spans based on pattern
new PatternEditableBuilder().
    addPattern(Pattern.compile("\\@(\\w+)"), Color.BLUE,
       new PatternEditableBuilder.SpannableClickedListener() {
            @Override
            public void onSpanClicked(String text) {
                Toast.makeText(MainActivity.this, "Clicked username: " + text,
                    Toast.LENGTH_SHORT).show();
            }
       }).into(textView);
```

and this results in the following:

<img src="https://i.imgur.com/OHFMMTc.gif" width="500" />

For more details, [view the README](https://gist.github.com/nesquena/f2504c642c5de47b371278ee61c75124#file-readme-md) for more usage examples. 

## References

* <http://code.tutsplus.com/tutorials/customize-android-fonts--mobile-1601>
* <http://www.androidhive.info/2012/02/android-using-external-fonts/>
* <http://stackoverflow.com/questions/3651086/android-using-custom-font>
* <http://www.tutorialspoint.com/android/android_custom_fonts.htm>
* <http://antonioleiva.com/textview_power_drawables/>
