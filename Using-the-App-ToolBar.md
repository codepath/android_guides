## Overview

`ToolBar` was introduced in Android Lollipop, API 21 release and is the spiritual successor of the ActionBar. It's a `ViewGroup` that can be placed anywhere in your layout. ToolBar's appearance can be more easily customized than the ActionBar. 

<img src="http://i.imgur.com/0auGknf.png" width="500" />

ToolBar works well with apps targeted to API 21 and above. However, Android has updated the AppCompat support libraries so the ToolBar can be used on lower Android OS devices as well. In AppCompat, ToolBar is implemented in the `android.support.v7.widget.Toolbar` class.

There are two ways to use Toolbar:

 1. Use a `Toolbar` as an Action Bar when you want to use the existing Action Bar facilities (such as menu inflation and selection, `ActionBarDrawerToggle`, and so on) but want to have more control over its appearance.
 2. Use a standalone `Toolbar` when you want to use the pattern in your app for situations that an Action Bar would not support; for example, showing multiple toolbars on the screen, spanning only part of the width, and so on.

### Using ToolBar as ActionBar

To use Toolbar as an ActionBar, first ensure the AppCompat-v7 support library is added to your application `build.gradle` (Module:app) file:

```gradle
dependencies {
  ...
  compile 'com.android.support:appcompat-v7:23.1.0+'
}
```

Second, let's disable the theme-provided ActionBar. The easiest way is to have your theme extend from `Theme.AppCompat.NoActionBar` (or the light variant) within the `res/styles.xml` file:

```xml
<resources>
  <!-- Base application theme. -->
  <style name="AppTheme" parent="Theme.AppCompat.Light.NoActionBar">
  </style>
</resources>
```

Now you need to add a `Toolbar` to your Activity layout file. One of the biggest advantages of using the Toolbar widget is that you can place the view anywhere within your layout. Below we place the toolbar at the top of a LinearLayout like the standard ActionBar:

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <android.support.v7.widget.Toolbar
      android:id="@+id/toolbar"
      android:minHeight="?attr/actionBarSize"  
      android:layout_width="match_parent"
      android:layout_height="wrap_content"
      app:titleTextColor="@android:color/white"
      android:background="?attr/colorPrimary">
    </android.support.v7.widget.Toolbar>

    <!-- Layout for content is here. This can be a RelativeLayout  -->

</LinearLayout>
```

As Toolbar is just a `ViewGroup` and can be **styled and positioned like any other view**. Note that this means if you are in a `RelativeLayout`, you need to ensure that all other views are positioned below the toolbar explicitly. The toolbar is not given any special treatment as a view.


Next, in your Activity or Fragment, set the Toolbar to act as the ActionBar by calling the  `setSupportActionBar(Toolbar)` method:

**Note:** When using the support library, make sure that you are importing `android.support.v7.widget.Toolbar` and not `android.widget.Toolbar`.

```java
import android.support.v7.app.AppCompatActivity;
import android.support.v7.widget.Toolbar;

public class MyActivity extends AppCompatActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_my);

        // Find the toolbar view inside the activity layout
        Toolbar toolbar = (Toolbar) findViewById(R.id.toolbar);
        // Sets the Toolbar to act as the ActionBar for this Activity window.
        // Make sure the toolbar exists in the activity and is not null
        setSupportActionBar(toolbar);
    }

    // Menu icons are inflated just as they were with actionbar
    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        // Inflate the menu; this adds items to the action bar if it is present.
        getMenuInflater().inflate(R.menu.menu_main, menu);
        return true;
    }
}
```

The above code results in the toolbar fully replacing the ActionBar at the top:

<img src="http://i.imgur.com/lucP1wY.png" width="400" />

From this point on, all menu items are displayed in your Toolbar, populated via the standard options menu callbacks.

### Styling the Toolbar

The Toolbar can be customized in many ways leveraging various style properties including `android:theme`, `app:titleTextAppearance`, `app:popupTheme`. Each of these can be mapped to a style. Start with:

```xml
<android.support.v7.widget.Toolbar
    android:id="@+id/toolbar"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:minHeight="?attr/actionBarSize"  
    android:theme="@style/ToolbarTheme"
    app:titleTextAppearance="@style/Toolbar.TitleText"
    app:popupTheme="@style/ThemeOverlay.AppCompat.Light"
/>
```

Now, we need to create the custom styles in `res/styles.xml` with:

```xml
<!-- Base application theme. -->
<style name="AppTheme" parent="Theme.AppCompat.Light.NoActionBar">
  <!-- Customize your theme here. -->
  <item name="colorPrimary">@color/colorPrimary</item>
  <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
  <item name="colorAccent">@color/colorAccent</item>
</style>

<style name="ToolbarTheme" parent="@style/ThemeOverlay.AppCompat.Dark.ActionBar">
  <!-- This would set the toolbar's background color -->
  <item name="android:background">@color/colorPrimary</item>
  <!-- android:textColorPrimary is the color of the title text in the Toolbar  -->
  <item name="android:textColorPrimary">@android:color/holo_blue_light</item>
  <!-- android:actionMenuTextColor is the color of the text of action (menu) items  -->
  <item name="actionMenuTextColor">@android:color/holo_green_light</item>
  <!-- Enable these below if you want clicking icons to trigger a ripple effect -->
  <!-- 
  <item name="selectableItemBackground">?android:selectableItemBackground</item>
  <item name="selectableItemBackgroundBorderless">?android:selectableItemBackground</item> 
  -->
</style>

<!-- This configures the styles for the title within the Toolbar  -->
<style name="Toolbar.TitleText" parent="TextAppearance.Widget.AppCompat.Toolbar.Title">
    <item name="android:textSize">21sp</item>
    <item name="android:textStyle">italic</item>
</style>
```

This results in:

<img src="http://i.imgur.com/uLWp8Rl.png" width="450" />

### Custom Title View

A `Toolbar` is just a decorated `ViewGroup` and as a result, the title contained within can be completely customized by embedding a view within the Toolbar such as:

```xml
<android.support.v7.widget.Toolbar
    android:id="@+id/toolbar"
    android:minHeight="?attr/actionBarSize"  
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    app:titleTextColor="@android:color/white"
    android:background="?attr/colorPrimary">

     <TextView
        android:id="@+id/toolbar_title"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Toolbar Title"
        android:textColor="@android:color/white"
        style="@style/TextAppearance.AppCompat.Widget.ActionBar.Title"
        android:layout_gravity="center"
     />

</android.support.v7.widget.Toolbar>
```

This means that you can style the `TextView` like any other. You can access the `TextView` inside your activity with:

```java
/* Inside the activity */
// Sets the Toolbar to act as the ActionBar for this Activity window.
Toolbar toolbar = (Toolbar) findViewById(R.id.toolbar);
setSupportActionBar(toolbar);
// Remove default title text
getSupportActionBar().setDisplayShowTitleEnabled(false);
// Get access to the custom title view
TextView mTitle = (TextView) toolbar.findViewById(R.id.toolbar_title);
```

This results in:

<img src="http://i.imgur.com/2Lu7Eru.png" width="450" />