## Overview

DialogFragment is a specialized Fragment used when you want to display an overlay modal window within an activity that floats on top of the rest of the content. 

This is typically used for displaying an alert dialog, a confirm dialog, or prompting the user for information within an overlay without having to switch to another Activity.

DialogFragment is now the canonical way to display overlays; using Dialog directly is considered bad practice.

## Usage

The minimum that must be implemented when creating a DialogFragment is either the `onCreateView` method or the `onCreateDialog` method. Use `onCreateView` when the entire view of the dialog is going to be defined via custom XML. Use `onCreateDialog` when you just need to construct and configure a standard Dialog class (such as AlertDialog) to display.

**Note:** The entire guide below requires every fragment related class imported to use the `android.support.v4.app` namespace and **not** the `android.app` namespace. If any imported class (FragmentManager, DialogFragment, etc) uses the `android.app` namespace, compile-time errors will occur.

### Custom View

Let's start by providing the code for creating a completely custom dialog based on an XML view. First, an example fragment XML file in `res/layout/fragment_edit_name.xml`:

```xml
<!-- fragment_edit_name.xml -->
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
	android:id="@+id/edit_name"
	android:layout_width="wrap_content" android:layout_height="wrap_content"
	android:layout_gravity="center" android:orientation="vertical"  >

	<TextView
		android:id="@+id/lbl_your_name" android:text="Your name" 
		android:layout_width="wrap_content" android:layout_height="wrap_content" />

	<EditText
		android:id="@+id/txt_your_name"
		android:layout_width="match_parent"  android:layout_height="wrap_content" 
		android:inputType="text"
		android:imeOptions="actionDone" />
</LinearLayout>
```

and defining the fragment itself extending from the **support version of dialog fragment**:

```java
import android.support.v4.app.DialogFragment;
// ...

public class EditNameDialog extends DialogFragment {

	private EditText mEditText;

	public EditNameDialog() {
		// Empty constructor is required for DialogFragment
                // Make sure not to add arguments to the constructor
                // Use `newInstance` instead as shown below
	}
	
	public static EditNameDialog newInstance(String title) {
		EditNameDialog frag = new EditNameDialog();
		Bundle args = new Bundle();
		args.putString("title", title);
		frag.setArguments(args);
		return frag;
	}

	@Override
	public View onCreateView(LayoutInflater inflater, ViewGroup container,
			Bundle savedInstanceState) {
		return inflater.inflate(R.layout.fragment_edit_name, container);
	}

	@Override
	public void onViewCreated(View view, @Nullable Bundle savedInstanceState) {
		super.onViewCreated(view, savedInstanceState);
		// Get field from view
		mEditText = (EditText) view.findViewById(R.id.txt_your_name);
		// Fetch arguments from bundle and set title
		String title = getArguments().getString("title", "Enter Name");
		getDialog().setTitle(title);
		// Show soft keyboard automatically and request focus to field
		mEditText.requestFocus();
		getDialog().getWindow().setSoftInputMode(
		    WindowManager.LayoutParams.SOFT_INPUT_STATE_VISIBLE);
	}
}
```
and showing the dialog in an Activity extending `AppCompatActivity`:

```java
// Note: `FragmentActivity` works here as well
public class FragmentDialogDemo extends AppCompatActivity {
  @Override
  public void onCreate(Bundle savedInstanceState) {
      super.onCreate(savedInstanceState);
      setContentView(R.layout.main);
      showEditDialog();
  }
  
  private void showEditDialog() {
      FragmentManager fm = getSupportFragmentManager();
      EditNameDialog editNameDialog = EditNameDialog.newInstance("Some Title");
      editNameDialog.show(fm, "fragment_edit_name");
  }
}
```

For this to work properly make sure that all the fragment related items (`FragmentDialog`) are from the `android.support.v4.app` namespace.

### Build Dialog

Second, let's take a look at how to build a dialog simply by customizing the available
Dialog objects that we can construct. For more details about the different types of dialogs, check out the "Things to Note" section below.

```java
class MyAlertDialogFragment extends DialogFragment {
    public MyAlertDialogFragment() {
          // Empty constructor required for DialogFragment
    }
    
    public static MyAlertDialogFragment newInstance(String title) {
        MyAlertDialogFragment frag = new MyAlertDialogFragment();
    	Bundle args = new Bundle();
    	args.putString("title", title);
    	frag.setArguments(args);
    	return frag;
    }
    
    @Override
    public Dialog onCreateDialog(Bundle savedInstanceState) {
         String title = getArguments().getString("title");
         AlertDialog.Builder alertDialogBuilder = new AlertDialog.Builder(getActivity());
         alertDialogBuilder.setTitle(title);
         alertDialogBuilder.setMessage("Are you sure?");
         alertDialogBuilder.setPositiveButton("OK",  new DialogInterface.OnClickListener() {
             @Override
             public void onClick(DialogInterface dialog, int which) {
               	   // on success
             }
         });
         alertDialogBuilder.setNegativeButton("Cancel", new DialogInterface.OnClickListener() {
             @Override
             public void onClick(DialogInterface dialog, int which) {
                 dialog.dismiss();
             }
         });
         
         return alertDialogBuilder.create();
    }
}
```

and to display the alert dialog in an activity extending `AppCompatActivity`:

```java
public class FragmentDialogDemo extends AppCompatActivity {
  @Override
  public void onCreate(Bundle savedInstanceState) {
  	super.onCreate(savedInstanceState);
  	setContentView(R.layout.main);
  	showAlertDialog();
  }

  private void showAlertDialog() {
  	FragmentManager fm = getSupportFragmentManager();
  	MyAlertDialogFragment alertDialog = MyAlertDialogFragment.newInstance("Some title");
  	alertDialog.show(fm, "fragment_alert");
  }
}
```

### Passing Data to Activity

To pass data from a dialog to an Activity, use the same approach you would use for any fragment. This is called the "fragment interface pattern". This is demonstrated for the `onCreateView` approach 
but works similarly for the AlertDialog (just use the listeners defined for the buttons):

```java
public class EditNameDialog extends DialogFragment implements OnEditorActionListener {
    private EditText mEditText;

    public interface EditNameDialogListener {
        void onFinishEditDialog(String inputText);
    }
	
    // ...
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
        Bundle savedInstanceState) {
        // ...
        mEditText.setOnEditorActionListener(this);
    }
	
    // Fires whenever the textfield has an action performed
    // In this case, when the "Done" button is pressed
    // REQUIRES a 'soft keyboard' (virtual keyboard)
    @Override
    public boolean onEditorAction(TextView v, int actionId, KeyEvent event) {
        if (EditorInfo.IME_ACTION_DONE == actionId) {
            // Return input text to activity
            EditNameDialogListener listener = (EditNameDialogListener) getActivity();
            listener.onFinishEditDialog(mEditText.getText().toString());
            dismiss();
            return true;
        }
        return false;
    }
}
```

and have the activity define the action to take when the dialog has the information:

```java
public class FragmentDialogDemo extends AppCompatActivity implements EditNameDialogListener {
  // ...
  
  @Override
  public void onFinishEditDialog(String inputText) {
  	Toast.makeText(this, "Hi, " + inputText, Toast.LENGTH_SHORT).show();
  }
}
```

**Note:** `setOnEditorActionListener` used above to dismiss requires the use of the soft keyboard in the emulator which [can be enabled through AVD](http://imgur.com/a/kf1s9) or by testing on a device. If you don't want to enable soft keyboard, you may want to dismiss on a button click or on a keypress instead.

## Styling Dialogs

### Styling Custom Dialog

Styling a DialogFragment with a custom layout works just the [[same as styling any views|Styles-and-Themes]]. Styling a dialog or `AlertDialog` requires changing several key properties in `styles.xml` such as the `dialogTheme` and `alertDialogTheme` as shown in this app [here](https://github.com/irccloud/android/blob/master/res/values/styles.xml) and shown below in `res/values/styles.xml`:

```xml
<!-- In res/values/colors.xml -->
<color name="dark_blue">#180065</color>
<color name="light_blue">#334ee9ff</color>
<color name="medium_green">#3d853e</color>
<color name="light_green">#3c2ae668</color>

<!-- In res/values/styles.xml -->
<style name="AppTheme" parent="Theme.AppCompat.Light">
    <!-- Apply default style for dialogs -->
    <item name="android:dialogTheme">@style/AppDialogTheme</item>
    <!-- Apply default style for alert dialogs -->
    <item name="android:alertDialogTheme">@style/AppAlertTheme</item>
</style>

<!-- Define your custom dialog theme here extending from base -->
<style name="AppDialogTheme" parent="Theme.AppCompat.Light.Dialog">
    <!-- Define color properties as desired -->
    <item name="colorPrimary">@color/dark_blue</item>
    <item name="colorPrimaryDark">#000</item>
    <item name="android:textColorHighlight">@color/light_blue</item>
    <item name="colorAccent">@color/dark_blue</item>
    <item name="colorControlNormal">@color/dark_blue</item>
    <!-- Define window properties as desired -->
    <item name="android:windowNoTitle">false</item>
    <item name="android:windowFullscreen">false</item>
    <item name="android:windowBackground">@color/medium_green</item>
    <item name="android:windowIsFloating">true</item>
    <item name="android:windowCloseOnTouchOutside">true</item>
</style>

<!-- Define your custom alert theme here extending from base -->
<style name="AppAlertTheme" parent="Theme.AppCompat.Light.Dialog.Alert">
    <item name="colorPrimary">@color/dark_blue</item>
    <item name="colorAccent">@color/dark_blue</item>
    <item name="colorPrimaryDark">#000</item>
    <item name="colorControlNormal">@color/dark_blue</item>
    <item name="android:textColorHighlight">@color/light_blue</item>
</style>
```

### Styling Titlebar of Dialog

The titlebar can be styled using the "android:windowTitleStyle" as follows:

```xml
<style name="AppTheme" parent="Theme.AppCompat.Light">
    <!-- Apply default style for dialogs -->
    <item name="android:dialogTheme">@style/AppDialogTheme</item>
    <!-- Apply default style for alert dialogs -->
    <item name="android:alertDialogTheme">@style/AppAlertTheme</item>
</style>

<style name="AppDialogTheme" parent="Theme.AppCompat.Light.Dialog">
    <item name="android:windowTitleStyle">@style/DialogWindowTitle</item>
    <!-- ...other stuff here... -->
</style>


<style name="AppAlertTheme" parent="Theme.AppCompat.Light.Dialog.Alert">
    <item name="android:windowTitleStyle">@style/DialogWindowTitle</item>
    <!-- ...other stuff here... -->
</style>

<style name="DialogWindowTitle" parent="Base.DialogWindowTitle.AppCompat">
    <item name="android:background">@color/light_green</item>
    <item name="android:gravity">center</item>
    <item name="android:textAppearance">@style/DialogWindowTitleText</item>
</style>

<style name="DialogWindowTitleText" parent="@android:style/TextAppearance.DialogWindowTitle">
    <item name="android:textSize">24sp</item>
</style>
```

### Removing the TitleBar from the Dialog

The TitleBar can be easily removed from your `DialogFragment` by overriding the `onCreateDialog` method:

```java
@Override
public Dialog onCreateDialog(Bundle savedInstanceState) {
    Dialog dialog = super.onCreateDialog(savedInstanceState);
    // request a window without the title
    dialog.getWindow().requestFeature(Window.FEATURE_NO_TITLE);
    return dialog;
}
```

This will give you a dialog box without a title bar. Read more [in this StackOverflow post](http://stackoverflow.com/questions/15277460/how-to-create-a-dialogfragment-without-title)

### Transparent Dialogs

We can make the dialog (or the title of the dialog) translucent using the `android:windowBackground` property:

```xml
<style name="AppDialogTheme" parent="Theme.AppCompat.Light.Dialog">
    <item name="android:windowIsTranslucent">true</item>
    <item name="android:windowBackground">@android:color/transparent</item>
    <!-- ...other stuff here... -->
</style>
```

Note that this removes the [default background image](https://github.com/android/platform_frameworks_support/blob/master/v7/appcompat/res/drawable/abc_dialog_material_background_light.xml) from the dialog `@drawable/abc_dialog_material_background_light` and as a result the shadow and border is removed.

To complete the transparent effect, make sure to **set the alpha channel of the background colors** as [outlined here](http://stackoverflow.com/a/16890937) to make any background colors semi-transparent.

### Styling with Third-Party Libraries

Note that [[third party material libraries|Material-Design-Primer#dialog-styles]] such as [material-dialogs](https://github.com/afollestad/material-dialogs) can be used to simplify and improve dialog styling as well.

![Material Dialog](https://i.imgur.com/U4mr2BB.jpg)

In order to use the "material-dialogs" library, you will need to add in maven repository to your build.gradle file. Your gradle file should look [something like this](https://github.com/afollestad/impression/blob/master/app/build.gradle#L38-L47).

## Sizing Dialogs

### Runtime Dimensions

In certain situations, you may want to set the height and width of the `DialogFragment` at runtime during creation. This can be done easily with `getDialog().getWindow()` as follows. In the XML simply set the root layout to `wrap_content` with:

```xml
<!-- fragment_edit_name.xml -->
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/edit_name"
    android:layout_width="wrap_content" android:layout_height="wrap_content" >
  <!-- ...subviews here... -->
</LinearLayout>
```

In the `DialogFragment` java source we can set the width and height `onResume` with:

```java
public void onResume() {
    int width = getResources().getDimensionPixelSize(R.dimen.popup_width);
    int height = getResources().getDimensionPixelSize(R.dimen.popup_height);        
    getDialog().getWindow().setLayout(width, height);
}
```

See [this stackoverflow post](http://stackoverflow.com/questions/12478520/how-to-set-dialogfragments-width-and-height) for more information.

### Full-Screen Dialog

In other cases, we want the dialog to fill the entire screen. First, in the XML layout for the dialog simply set the root layout to `match_parent` with:

```xml
<!-- fragment_edit_name.xml -->
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/edit_name"
    android:layout_width="wrap_content" android:layout_height="wrap_content" >
  <!-- ...subviews here... -->
</LinearLayout>
```

Next, within the `onResume` method of the `DialogFragment` we need to set the rules on the `getDialog().getWindow()` object to `WindowManager.LayoutParams.MATCH_PARENT` with:

```
@Override
public void onResume() {
    // Get existing layout params for the window
    ViewGroup.LayoutParams params = getDialog().getWindow().getAttributes();
    // Assign window properties to fill the parent
    params.width = WindowManager.LayoutParams.MATCH_PARENT;
    params.height = WindowManager.LayoutParams.MATCH_PARENT;
    getDialog().getWindow().setAttributes((android.view.WindowManager.LayoutParams) params);
    // Call super onResume after sizing
    super.onResume();
}
```

See [this stackoverflow post](http://stackoverflow.com/questions/27202382/android-dialog-fragment-width-keeps-on-matching-parent) for more details.

## Specialized Dialog Types

When using the `onCreateDialog` method there are many built-in Dialog types to take advantage of:

* [AlertDialog](http://developer.android.com/reference/android/app/AlertDialog.html) - Base dialog that can display a message, an icon and 1-3 buttons with customized text. 
* [ProgressDialog](http://developer.android.com/reference/android/app/ProgressDialog.html) - Dialog showing a progress indicator and an optional text message 
* [TimePickerDialog](http://developer.android.com/reference/android/app/TimePickerDialog.html) - Dialog that allows a user to select a time.
* [DatePickerDialog](http://developer.android.com/reference/android/app/DatePickerDialog.html) - Dialog that allows a user to select a date.
* Other dialogs [not](http://developer.android.com/reference/android/text/method/CharacterPickerDialog.html) [worth](http://developer.android.com/reference/android/support/v7/app/MediaRouteChooserDialog.html) discussing here.

### Displaying a ProgressDialog

When running a long running background task, one easy way to notify users the app is loading is to display a [ProgressDialog](http://developer.android.com/intl/es/reference/android/app/ProgressDialog.html).

<img src="http://i.imgur.com/apLzWt6.png" width="250" />

A `ProgressDialog` can be created anytime with the following:

```java
ProgressDialog pd = new ProgressDialog(context);
pd.setTitle("Loading...");
pd.setMessage("Please wait.");
pd.setCancelable(false);
```

The dialog can be displayed with:

```java
pd.show();
```

and hidden anytime with:

```java
pd.dismiss();
```

`ProgressDialog` can be safely paired with an [[AsyncTask|Creating-and-Executing-Async-Tasks]]. Refer to this [ProgressDialog tutorial](http://www.quicktips.in/show-progressdialog-android/) for a code sample. The dialog progress animation can be customized by supplying your own animation drawable [using this tutorial](http://islandofatlas.net/2014/03/29/android-custom-progress-dialog.html).

Check out the CodePath [android-view-helpers](https://github.com/codepath/android-view-helpers) library for an easier way to create simple alert and progress modals.

## Things To Note

* Notice that we are using the support library version of fragments for better compatibility in our code samples. The non-support version works identically.
* Dialogs are just classes that extend `DialogFragment` and define the view to display in the floating content area.
* `DialogFragment` classes must define an empty constructor as shown in the code samples, otherwise the Android system will raise an exception when it attempts to instantiate the fragment.
* After loading the initial view, the activity immediately shows the dialog using the show() method which allows the fragment manager to keep track of the state and gives us certain things for free such as the back button dismissing the fragment.
* In the code snippets above, notice the use of `requestFocus` and input modes to control the appearance of the soft keyboard when the dialog appears.
* We can dismiss a dialog one of two ways. Here we are calling `dismiss()` within the Dialog class itself. It could also be called from the Activity like the `show()` method.

## References

 * <http://android-developers.blogspot.com/2012/05/using-dialogfragments.html>
 * <http://codebaum.wordpress.com/2013/04/10/dialog-fragments/>
 * <http://developer.android.com/reference/android/app/DialogFragment.html>
 * <http://www.coderzheaven.com/2013/02/14/dialogfragments-android-simple-example/>
 * <http://stackoverflow.com/questions/12912181/simplest-yes-no-dialog-fragment>