The following tutorial explain how to build a application that can switch between multiple distinct themes. At the end of this exercise, you will have better understanding of some of the core features of Android like - `drawables`, `styles` and `themes`. For more general overview of these concepts, check out [[Styles and Themes|Styles-and-Themes]] cliffnotes.

A `style` in Android is a collection of attribute/value pairs applied to a view. A `style` is a `xml` resource and it separates the design attributes from `XML` layout. Styles in Android is similar in concept to CSS on web because it separates design from the content. A `Theme` is a `Style` that applies to the entire application or a certain `Activity`.

We will be defining three themes in our app and use a spinner view to switch between themes.  By the end of this exercise, you should know how to define a theme in your resources in an XML file, how to define attributes of the theme, how to apply those to your layout file, and finally how to dynamically change the theme of an activity. Below is the final output.

![Imgur](http://i.imgur.com/a7glYfd.png)

## 1. Create a New Android Application Project

* Open Eclipse IDE and go to File > New > Project > Android Application Project. 
* Enter App name: ThemeSwitcher (minSDK 14)
* Name the first activity "ThemeActivity"
* Keep other defualt selections, go Next till you reach Finish.

## 2. Design Layout

Create a simple layout for our app. Later on we'll be applying all our styles and themes to this layout file. Open `res/layout/activity_theme.xml` file and go to the respective xml tab. Then paste the code below.

```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent" >

    <TextView
        android:id="@+id/tvSelectTheme"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/settings_text_select_theme" />

    <Spinner
        android:id="@+id/spThemes"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignBaseline="@+id/tvSelectTheme"
        android:layout_alignParentRight="true"
        android:layout_toRightOf="@+id/tvSelectTheme"
        android:entries="@array/theme_array"
        android:spinnerMode="dropdown" />

    <RelativeLayout
        android:id="@+id/rlCredentials"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_below="@+id/tvSelectTheme" >

        <TextView
            android:id="@+id/tvCredentials"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/settings_text_credentials" />

        <EditText
            android:id="@+id/tvUsername"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_below="@+id/tvCredentials"
            android:hint="@string/settings_text_username_hint"
            android:inputType="text"
            android:lines="1" />

        <EditText
            android:id="@+id/tvpassword"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_below="@+id/tvUsername"
            android:hint="@string/settings_text_password_hint"
            android:inputType="textPassword"
            android:lines="1" />
    </RelativeLayout>

    <TextView
        android:id="@+id/tvSync"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_below="@+id/rlCredentials"
        android:text="@string/settings_text_sync_automatically"
        android:textSize="17sp" />

    <CheckBox
        android:id="@+id/checkbox_sync_automatically"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignBaseline="@+id/tvSync"
        android:layout_alignParentRight="true"
        android:checked="true" />

    <TextView
        android:id="@+id/tvLocation"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_below="@+id/tvSync"
        android:text="@string/settings_text_location"
        android:textSize="17sp" />

    <Switch
        android:id="@+id/toggle_google"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignBaseline="@+id/tvLocation"
        android:layout_alignParentRight="true"
        android:layout_toRightOf="@+id/tvLocation"
        android:textOff="@string/settings_text_state_off"
        android:textOn="@string/settings_text_state_on" />

    <Button
        android:id="@+id/btnClearData"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentBottom="true"
        android:layout_centerHorizontal="true"
        android:text="@string/settings_text_clear_data" />

</RelativeLayout>
```

Also add the following to `res/values/strings.xml`.

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>

    <string name="app_name">Theme Switcher</string>
    
    <string name="settings_text_select_theme">Select Theme:</string>
    <string name="settings_text_credentials">Credentials</string>
    <string name="settings_text_username_hint">username</string>
    <string name="settings_text_password_hint">password</string>
    <string name="settings_text_sync_automatically">Sync automatically</string>
    <string name="settings_text_location">Location</string>
    <string name="settings_text_state_on">On</string>
    <string name="settings_text_state_off">Off</string>
    <string name="settings_text_clear_data">Clear Data</string>

    <string-array name="theme_array">
        <item>Holo-Light</item>
        <item>Console</item>
        <item>Navy</item>
    </string-array>

</resources>
```

Note that the spinner is bound to a string array. Spinner will display the theme names that we will be defining later on.

If you run your application now, you should see the following output.

![Imgur](http://i.imgur.com/l2cW1Tj.png)

## 3. Custom Attributes

Next, we need to declare the styleable attributes so we can use them in XML markup. These are the custom attributes you would like your theme to define. Create a file called `attrs.xml` inside `/res/values/` and add the following

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>

    <attr name="pageBackground" format="reference" />
    <attr name="textSubheader" format="reference" />
    <attr name="textLarge" format="reference" />
    <attr name="textRegular" format="reference" />
    <attr name="whiteBackground" format="reference" />
    <attr name="button" format="reference" />
    <attr name="spinner" format="reference" />

</resources>
```

The format attribute indicates what type of values you can define. In this case, it is a reference to another attribute Other examples of possible formats are pixels, color, boolean, integer, and float.

## 4. Dimensions

Create `/res/values/dimens.xml` file. This is a XML resource that carries dimension values.

```xml
<resources>

    <dimen name="activity_horizontal_padding">16dp</dimen>
    <dimen name="content_left_margin_wh">8sp</dimen>
    <dimen name="text_large_margin_top">40dp</dimen>
    <dimen name="text_large_text_size">18sp</dimen>
    <dimen name="text_subheader_text_size">16sp</dimen>
    <dimen name="text_regular_text_size">15sp</dimen>
    <dimen name="view_margin">20dp</dimen>
    <dimen name="white_background_padding">20dp</dimen>
    <dimen name="edittext_margin_top">10dp</dimen>
    <dimen name="spinner_margin_left">10dp</dimen>
    <dimen name="card_margin_top">40dp</dimen>

</resources>
```

## 5. Custom Styles and Drawables 

A more sophisticated method to setting the properties of your views in the layout is give a view a style, which is a group of attributes, instead of defining the values of individual attributes. For example, you could set the styles of all of your title TextViews to have the style textTitle.  This style could have custom text color, font, and margin properties. In addition to styles, you will be using drawables to customize your views. A drawable resource is a general concept for a graphic that can be drawn to the screen. For more information, refer the cliffnotes on [[drawables|Drawables]].

Open `res/values/styles.xml` file. This is where youâ€™ll define your styles.

`res/values/styles.xml`:
```xml
<resources>

    <!--
        Base application theme, dependent on API level. This theme is replaced
        by AppBaseTheme from res/values-vXX/styles.xml on newer devices.
    -->
    <style name="AppBaseTheme" parent="android:Theme.Light">
        <!--
            Theme customizations available in newer API levels can go in
            res/values-vXX/styles.xml, while customizations related to
            backward-compatibility can go here.
        -->
    </style>

    <!-- Application theme. -->
    <style name="AppTheme" parent="AppBaseTheme">
        <!-- All customizations that are NOT specific to a particular API-level can go here. -->
    </style>

    <style name="switch_text_appearance" parent="@android:style/TextAppearance.Holo.Small">
        <item name="android:textColor">#FFF</item>
    </style>

    <!-- =============================================================== -->
    <!-- Holo-Light styles -->
    <!-- =============================================================== -->

    <style name="page_background_wh">
        <item name="android:background">@drawable/white_gray_gradient_background</item>
        <item name="android:paddingLeft">@dimen/activity_horizontal_padding</item>
        <item name="android:paddingRight">@dimen/activity_horizontal_padding</item>
    </style>

    <style name="text_subheader_wh">
        <item name="android:textColor">#000</item>
        <item name="android:textSize">@dimen/text_subheader_text_size</item>
        <item name="android:shadowDy">1.0</item>
        <item name="android:shadowRadius">1</item>
        <item name="android:shadowColor">#000</item>
    </style>

    <style name="text_large_wh">
        <item name="android:textColor">@android:color/background_dark</item>
        <item name="android:textSize">@dimen/text_large_text_size</item>
        <item name="android:shadowDy">1.0</item>
        <item name="android:shadowRadius">1</item>
        <item name="android:shadowColor">#888</item>
        <item name="android:textStyle">bold</item>
        <item name="android:layout_marginTop">@dimen/text_large_margin_top</item>
    </style>

    <style name="text_regular_wh">
        <item name="android:textColor">@android:color/background_dark</item>
        <item name="android:textSize">@dimen/text_regular_text_size</item>
    </style>

    <style name="white_background_wh">
        <item name="android:background">@android:drawable/dialog_holo_light_frame</item>
        <item name="android:layout_marginTop">@dimen/card_margin_top</item>
        <item name="android:layout_marginBottom">@dimen/card_margin_top</item>
        <item name="android:padding">@dimen/white_background_padding</item>
    </style>

    <style name="button_wh" parent="text_large_wh">
        <item name="android:background">@drawable/button_wh</item>
        <item name="android:layout_marginTop">@dimen/view_margin</item>
    </style>

    <style name="spinner_wh">
        <item name="android:background">@drawable/spinner_wh_background</item>
    </style>

    <style name="horizontal_line_wh">
        <item name="android:background">#00000000</item>
    </style>
    
    <style name="action_bar_wh" parent="android:Widget.Holo.ActionBar">
        <item name="android:background">#AFAFAF</item>
    </style>
    
    <!-- =============================================================== -->
    <!-- Console styles -->
    <!-- Define your styles here. -->
    <!-- =============================================================== -->
    
    <!-- =============================================================== -->
    <!-- Navy styles -->
    <!-- Define your styles here. -->
    <!-- =============================================================== -->

</resources>
```

Add a folder called `drawable` under `res` folder.

`res/drawable/white_gray_gradient_background.xml`:
```xml
<shape xmlns:android="http://schemas.android.com/apk/res/android" >

    <gradient
        android:angle="270"
        android:endColor="#AFAFAF"
        android:startColor="#FFFFFF" />

</shape>
```

`res/drawable/button_wh_normal.xml`:
```xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="rectangle" >

    <gradient
        android:angle="270"
        android:centerColor="#FFCECFCE"
        android:endColor="#FFBEBEBE"
        android:startColor="#FFF7F7F7" />

    <padding
        android:bottom="8dp"
        android:left="16dp"
        android:right="16dp"
        android:top="8dp" />

    <corners android:radius="48dp" />

</shape>
```

`res/drawable/button_wh_pressed.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="rectangle" >

    <gradient
        android:angle="270"
        android:centerColor="#FF207FB9"
        android:endColor="#FF0060B8"
        android:startColor="#FF66CFE6" />

    <padding
        android:bottom="8dp"
        android:left="8dp"
        android:right="8dp"
        android:top="8dp" />

    <corners android:radius="48dp" />

</shape>
```


`res/drawable/button_wh.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">

    <item android:drawable="@drawable/button_wh_pressed" android:state_pressed="true"/>
    <item android:drawable="@drawable/button_wh_normal"/>

</selector>
```

For the spinner, you can find the nine-patch file for the corner triangle at this location: `/Android/android-sdks/plataforms//data/res/spinner_default_holo_light.9.png`. Copy this to your `drawable` folder.

`res/drawable/spinner_wh_background.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<layer-list xmlns:android="http://schemas.android.com/apk/res/android"
    android:opacity="transparent" >

    <item android:state_pressed="true">
        <shape android:shape="rectangle" >
            <solid android:color="#00000000" />
        </shape>
    </item>
    <item android:state_selected="true">
        <shape android:shape="rectangle" >
            <solid android:color="#00000000" />
        </shape>
    </item>
    <item android:drawable="@drawable/spinner_default_holo_light">
    </item>

</layer-list>
```

## 6. Create `themes.xml` file

To define our theme go to `res/values/themes.xml` file. In our theme definition, we set some custom styles using the `item` element. Note how the default OS attribute `android:actionBarStyle` has been overridden to style the action bar along with the custom attributes. For more information on styling action bar, check out [[styling the action bar|Extended-ActionBar-Guide#custom-actionbar-styles]] cliffnotes.

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources xmlns:android="http://schemas.android.com/apk/res/android">

    <style name="Theme.Holo_Light" parent="android:Theme.Holo.Light">
        <item name="pageBackground">@style/page_background_wh</item>
        <item name="textSubheader">@style/text_subheader_wh</item>
        <item name="textLarge">@style/text_large_wh</item>
        <item name="textRegular">@style/text_regular_wh</item>
        <item name="whiteBackground">@style/white_background_wh</item>
        <item name="button">@style/button_wh</item>
        <item name="spinner">@style/spinner_wh</item>
        <item name="android:actionBarStyle">@style/action_bar_wh</item>
    </style>

    <style name="Theme.Console" parent="android:Theme.Holo">
    	<!-- Define your styles here. -->
    </style>
    

    <style name="Theme.Navy" parent="android:Theme.Holo">
    	<!-- Define your styles here. -->
    </style>

</resources>
```

To have multiple themes, you will want to create multiple theme definitions in `themes.xml` as should in the XML above.

## 7. Apply Custom Styles

Update your layout file and apply the custom styles to your views. Note the `style` attribute applied to many of the views below.

`res/layout/activity_theme.xml`:

```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    style="?pageBackground"
    android:layout_width="match_parent"
    android:layout_height="match_parent" >

    <TextView
        android:id="@+id/tvSelectTheme"
        style="?textLarge"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/settings_text_select_theme" />

    <Spinner
        android:id="@+id/spThemes"
        style="?spinner"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignBaseline="@+id/tvSelectTheme"
        android:layout_alignParentRight="true"
        android:layout_marginLeft="@dimen/spinner_margin_left"
        android:layout_toRightOf="@+id/tvSelectTheme"
        android:entries="@array/theme_array"
        android:spinnerMode="dropdown" />

    <RelativeLayout
        android:id="@+id/rlCredentials"
        style="?whiteBackground"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_below="@+id/tvSelectTheme" >

        <TextView
            android:id="@+id/tvCredentials"
            style="?textSubheader"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/settings_text_credentials" />

        <EditText
            android:id="@+id/tvUsername"
            style="?textRegular"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_below="@+id/tvCredentials"
            android:hint="@string/settings_text_username_hint"
            android:inputType="text"
            android:lines="1" />

        <EditText
            android:id="@+id/tvpassword"
            style="?textRegular"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_below="@+id/tvUsername"
            android:layout_marginTop="@dimen/edittext_margin_top"
            android:hint="@string/settings_text_password_hint"
            android:inputType="textPassword"
            android:lines="1" />
    </RelativeLayout>

    <TextView
        android:id="@+id/tvSync"
        style="?textRegular"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_below="@+id/rlCredentials"
        android:text="@string/settings_text_sync_automatically"
        android:textSize="17sp" />

    <CheckBox
        android:id="@+id/checkbox_sync_automatically"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignBaseline="@+id/tvSync"
        android:layout_alignParentRight="true"
        android:checked="true" />

    <TextView
        android:id="@+id/tvLocation"
        style="?textRegular"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_below="@+id/tvSync"
        android:layout_marginTop="@dimen/view_margin"
        android:text="@string/settings_text_location"
        android:textSize="17sp" />

    <Switch
        android:id="@+id/toggle_google"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignBaseline="@+id/tvLocation"
        android:layout_alignParentRight="true"
        android:layout_toRightOf="@+id/tvLocation"
        android:switchTextAppearance="@style/switch_text_appearance"
        android:textOff="@string/settings_text_state_off"
        android:textOn="@string/settings_text_state_on" />

    <Button
        android:id="@+id/btnClearData"
        style="?button"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentBottom="true"
        android:layout_centerHorizontal="true"
        android:layout_marginBottom="@dimen/view_margin"
        android:text="@string/settings_text_clear_data" />

</RelativeLayout>
```

## 8. Apply Dynamic Themes

We can set the theme of our application or individual activities in the manifest file when we are not dealing with multiple themes. But for our app, since we are dynamically changing the theme from the spinner we'll have to do this programatically. This is done by calling `setTheme()` in the activity's `onCreate()` method, before any call to `setContentView()`. To change the theme, you simply need to restart your activity.

`ThemeApplication.java`:

```java
public class ThemeApplication extends Application {
	// App level variable to retain selected spinner value
	public static int currentPosition;
}
```

`Utils.java`:

```java
public class Utils {
	private static int sTheme;

	public final static int THEME_HOLO_LIGHT = 0;
	public final static int THEME_CONSOLE = 1;
	public final static int THEME_NAVY = 2;

	public static void changeToTheme(Activity activity, int theme) {
		sTheme = theme;
		activity.finish();
		activity.startActivity(new Intent(activity, activity.getClass()));
		activity.overridePendingTransition(android.R.anim.fade_in,
				android.R.anim.fade_out);
	}

	public static void onActivityCreateSetTheme(Activity activity) {
		switch (sTheme) {
		default:
		case THEME_HOLO_LIGHT:
			activity.setTheme(R.style.Theme_Holo_Light);
			break;
		case THEME_CONSOLE:
			activity.setTheme(R.style.Theme_Console);
			break;
		case THEME_NAVY:
			activity.setTheme(R.style.Theme_Navy);
			break;
		}
	}
}
```

`ThemeActivity.java`:

```java
public class ThemeActivity extends Activity {
	private Spinner spThemes;

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		Utils.onActivityCreateSetTheme(this);
		setContentView(R.layout.activity_theme);

		setupSpinnerItemSelection();
	}

	private void setupSpinnerItemSelection() {
		spThemes = (Spinner) findViewById(R.id.spThemes);
		spThemes.setSelection(ThemeApplication.currentPosition);
		ThemeApplication.currentPosition = spThemes.getSelectedItemPosition();

		spThemes.setOnItemSelectedListener(new OnItemSelectedListener() {

			@Override
			public void onItemSelected(AdapterView<?> parent, View view,
					int position, long id) {
				if (ThemeApplication.currentPosition != position) {
					Utils.changeToTheme(ThemeActivity.this, position);
				}
				ThemeApplication.currentPosition = position;
			}

			@Override
			public void onNothingSelected(AdapterView<?> parent) {

			}
		});

	}
}
```


If you run your app at this point, you should have applied the styles for 'Holo-Light' theme. It is upto the reader to define various styles for 'console' and 'navy' themes.