## Overview

DialogFragment is a specialized Fragment used when you want to display an overlay modal window within an activity that floats on top of the rest of the content. 

This is typically used for displaying an alert dialog, a confirm dialog, or prompting the user for information within an overlay without having to switch to another Activity.

DialogFragment is now the canonical way to display overlays; using Dialog directly [is considered bad practice](https://stackoverflow.com/a/13765411).

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

public class EditNameDialogFragment extends DialogFragment {

	private EditText mEditText;

	public EditNameDialogFragment() {
		// Empty constructor is required for DialogFragment
                // Make sure not to add arguments to the constructor
                // Use `newInstance` instead as shown below
	}
	
	public static EditNameDialogFragment newInstance(String title) {
		EditNameDialogFragment frag = new EditNameDialogFragment();
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
public class DialogDemoActivity extends AppCompatActivity {
  @Override
  public void onCreate(Bundle savedInstanceState) {
      super.onCreate(savedInstanceState);
      setContentView(R.layout.main);
      showEditDialog();
  }
  
  private void showEditDialog() {
      FragmentManager fm = getSupportFragmentManager();
      EditNameDialogFragment editNameDialogFragment = EditNameDialogFragment.newInstance("Some Title");
      editNameDialogFragment.show(fm, "fragment_edit_name");
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
            if (dialog != null && dialog.isShowing()) { 
                 dialog.dismiss();
              }
            }

        });

        return alertDialogBuilder.create();
    }
}
```

and to display the alert dialog in an activity extending `AppCompatActivity`:

```java
public class DialogDemoActivity extends AppCompatActivity {
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

To pass data from a dialog to an Activity, use the same approach you would use for any fragment which is [creating a custom listener](http://guides.codepath.com/android/Creating-Custom-Listeners). In short, you will need to do the following:

1. Define an interface with methods that can be invoked to pass data result to the activity
2. Setup a [view event](http://guides.codepath.com/android/Basic-Event-Listeners) which invokes the custom listener passing data through the method
3. Implement the interface in the Activity defining behavior for when the event is triggered

This example below demonstrates how to pass a data result back to the activity when the "Done" button is pressed on the keyboard but this works similarly for other cases or for the `AlertDialog` (just use the listeners defined for each of the buttons):

```java
public class EditNameDialogFragment extends DialogFragment implements OnEditorActionListener {
    private EditText mEditText;

    // 1. Defines the listener interface with a method passing back data result.
    public interface EditNameDialogListener {
        void onFinishEditDialog(String inputText);
    }
	
    // ...
    @Override
    public void onViewCreated(View view, @Nullable Bundle savedInstanceState) {
        // ...
        // 2. Setup a callback when the "Done" button is pressed on keyboard
        mEditText.setOnEditorActionListener(this);
    }
	
    // Fires whenever the textfield has an action performed
    // In this case, when the "Done" button is pressed
    // REQUIRES a 'soft keyboard' (virtual keyboard)
    @Override
    public boolean onEditorAction(TextView v, int actionId, KeyEvent event) {
        if (EditorInfo.IME_ACTION_DONE == actionId) {
            // Return input text back to activity through the implemented listener
            EditNameDialogListener listener = (EditNameDialogListener) getActivity();
            listener.onFinishEditDialog(mEditText.getText().toString());
            // Close the dialog and return back to the parent activity
            dismiss();
            return true;
        }
        return false;
    }
}
```

and have the activity define the action to take when the dialog has the information:

```java
public class DialogDemoActivity extends AppCompatActivity implements EditNameDialogListener {
  // ...
  
  // 3. This method is invoked in the activity when the listener is triggered
  // Access the data result passed to the activity here
  @Override
  public void onFinishEditDialog(String inputText) {
  	Toast.makeText(this, "Hi, " + inputText, Toast.LENGTH_SHORT).show();
  }
}
```

**Note:** `setOnEditorActionListener` used above to dismiss requires the use of the soft keyboard in the emulator which [can be enabled through AVD](https://imgur.com/a/kf1s9) or by testing on a device. If you don't want to enable soft keyboard, you may want to dismiss on a button click or on a keypress instead.

### Passing Data to Parent Fragment

In certain situations, the a dialog fragment may be invoked **within the context of another fragment**. For example, a screen has tabs with a form contained in a fragment. The form has an input for selecting dates using a date picker in a dialog. In this case, we may want to pass the date back not to the activity but instead to the parent fragment. This data can be passed back directly to the parent fragment:

```java
import android.support.v4.app.DialogFragment;

public class EditNameDialogFragment extends DialogFragment {
    // Defines the listener interface
    public interface EditNameDialogListener {
        void onFinishEditDialog(String inputText);
    }

    // Call this method to send the data back to the parent fragment
    public void sendBackResult() {
      // Notice the use of `getTargetFragment` which will be set when the dialog is displayed
      EditNameDialogListener listener = (EditNameDialogListener) getTargetFragment();
      listener.onFinishEditDialog(mEditText.getText().toString());
      dismiss();
    }
}
```

And then the dialog must be displayed within a parent fragment passing the target via `setTargetFragment` with:

```java
import android.support.v4.app.Fragment;

public class MyParentFragment extends Fragment implements EditNameDialogListener {
    // Call this method to launch the edit dialog
    private void showEditDialog() {
        FragmentManager fm = getFragmentManager();
        EditNameDialogFragment editNameDialogFragment = EditNameDialog.newInstance("Some Title");
        // SETS the target fragment for use later when sending results
        editNameDialogFragment.setTargetFragment(MyParentFragment.this, 300);
        editNameDialogFragment.show(fm, "fragment_edit_name");
    }

    // This is called when the dialog is completed and the results have been passed
    @Override
    public void onFinishEditDialog(String inputText) {
        Toast.makeText(this, "Hi, " + inputText, Toast.LENGTH_SHORT).show();
    }
}
```

**Troubleshooting**

If you are having any issues, be sure to use the checklist below:

 * In parent fragment, before calling `dialogFragment.show()`, are you calling `setTargetFragment` and passing in the correct fragment as the target?
 * In the dialog fragment, before calling `dismiss()`, are you calling `listener.someCallbackMethod()` on a listener casted from the `getTargetFragment()` passed in above? 
 * Have you correctly implemented the interface and callback method fired i.e `listener.someCallbackMethod()` inside of the parent fragment?
 * Try breakpointing each of those lines to make sure the target fragment is set property and the callback method is being executed. 

With that, the two fragments are able to pass data back and forth. 

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

#### Dialog Styling Workaround

**Note:** There is currently a bug in the support library that **causes styles not to show up properly**. Changing the `DialogFragment` in the `onCreateView` to use the activity's inflater seems to resolve the issue:

```java
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
			Bundle savedInstanceState) {
        return getActivity().getLayoutInflater().inflate(R.layout.fragment_edit_name, container);
    }
```
All dialog widgets should now be properly themed. Check out this [stackoverflow post](http://stackoverflow.com/a/32791569) for details.

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

In certain situations, you may want to explicitly set the height and width of the `DialogFragment` at runtime during creation. This can be done easily with `getDialog().getWindow()` as follows. In the XML simply set the root layout to `wrap_content` with:

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
    // Call super onResume after sizing
    super.onResume();
}
```

See [this stackoverflow post](http://stackoverflow.com/questions/12478520/how-to-set-dialogfragments-width-and-height) for more information. Using this approach we could set the dialog's width as a percentage of the screen within the `DialogFragment` with:

```java
public void onResume() {
    // Store access variables for window and blank point
    Window window = getDialog().getWindow();
    Point size = new Point();
    // Store dimensions of the screen in `size`
    Display display = window.getWindowManager().getDefaultDisplay();
    display.getSize(size);
    // Set the width of the dialog proportional to 75% of the screen width
    window.setLayout((int) (size.x * 0.75), WindowManager.LayoutParams.WRAP_CONTENT);
    window.setGravity(Gravity.CENTER);
    // Call super onResume after sizing
    super.onResume();
}
```

See [this stackoverflow post](http://stackoverflow.com/a/28596836/313399) for the source reference.

### Sizing Adjustments for Soft Keyboard

When displaying a dialog that is accepting text input, there can often be limited space on screen because the soft keyboard on screen eats up a lot of room. To account for this, you may want to modify the `android:windowSoftInputMode` property for the activity within the `AndroidManifest.xml` file:

```xml
<!-- Configures the UI to be resized to make room for the keyboard -->
<activity
    android:name="com.example.myactivity"
    android:windowSoftInputMode="adjustResize" />
```

See the full details in the [[working with the soft keyboard|Working-with-the-Soft-Keyboard#changing-ui-reaction]] guide. Alternatively, we could perform the resize directly at runtime within the `onCreateView` method of the fragment:

```java
public class EditNameDialog extends DialogFragment {
    // ...
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup parent, Bundle bundle) {
        // Set to adjust screen height automatically, when soft keyboard appears on screen 
        getDialog().getWindow().setSoftInputMode(WindowManager.LayoutParams.SOFT_INPUT_ADJUST_RESIZE);
        return inflater.inflate(R.layout.fragment_edit_name, container);
    }
}
```

See [this stackoverflow post](http://stackoverflow.com/a/26434420) for additional details. Keep in mind that for either of these to work, the layout of the dialog must be configured to resize properly as the height changes. 

### Full-Screen Dialog

In other cases, we want the dialog to fill the entire screen. First, in the XML layout for the dialog simply set the root layout to `match_parent` with:

```xml
<!-- fragment_edit_name.xml -->
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/edit_name"
    android:layout_width="match_parent" android:layout_height="match_parent" >
  <!-- ...subviews here... -->
</LinearLayout>
```

Next, within the `onResume` method of the `DialogFragment` we need to set the rules on the `getDialog().getWindow()` object to `WindowManager.LayoutParams.MATCH_PARENT` with:

```java
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

With this code above, the dialog may still not appear to be entirely full-screen until we configure the background color and padding inside `res/values/styles.xml` with the following:

```xml
<style name="Dialog.FullScreen" parent="Theme.AppCompat.Dialog">
    <item name="android:padding">0dp</item>
    <item name="android:windowBackground">@android:color/white</item>
</style>
```

and then this style can be applied when creating the fragment with:

```java
EditNameDialogFragment editNameDialogFragment = new EditNameDialogFragment();
editNameDialogFragment.setStyle(DialogFragment.STYLE_NORMAL, R.style.Dialog_FullScreen);
editNameDialogFragment.show(getSupportFragmentManager(), "fragment_edit_name");
```

See [this stackoverflow post](http://stackoverflow.com/questions/27202382/android-dialog-fragment-width-keeps-on-matching-parent) for more details. Refer to [this post](http://stackoverflow.com/questions/7189948/full-screen-dialogfragment-in-android/31597906#31597906) for the customized dialog styles.

## Specialized Dialog Types

When using the `onCreateDialog` method there are many built-in Dialog types to take advantage of:

* [AlertDialog](http://developer.android.com/reference/android/app/AlertDialog.html) - Base dialog that can display a message, an icon and 1-3 buttons with customized text. 
* [ProgressDialog](http://developer.android.com/reference/android/app/ProgressDialog.html) - Dialog showing a progress indicator and an optional text message 
* [TimePickerDialog](http://developer.android.com/reference/android/app/TimePickerDialog.html) - Dialog that allows a user to select a time.
* [DatePickerDialog](http://developer.android.com/reference/android/app/DatePickerDialog.html) - Dialog that allows a user to select a date.
* BottomSheetDialog - Dialog that slides from the bottom.
* Other dialogs not worth discussing here: [1](http://developer.android.com/reference/android/text/method/CharacterPickerDialog.html) [2](http://developer.android.com/reference/android/support/v7/app/MediaRouteChooserDialog.html)

### Displaying a ProgressDialog

When running a long running background task, one easy way to notify users the app is loading is to display a [ProgressDialog](http://developer.android.com/intl/es/reference/android/app/ProgressDialog.html).

<img src="https://i.imgur.com/apLzWt6.png" width="250" />

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

### Displaying Date or Time Picker Dialogs

The native date and time pickers for Android are another example of specialized dialog fragments.  Please note that the date/time pickers that comes with the Android SDK differ depending on the Android device version.  See [this section](https://github.com/codepath/android_guides/wiki/Working-with-Input-Views#date-and-time-pickers) for more information.

<img src="https://imgur.com/8YqcxA7.png" width="300"/>
<img src="https://imgur.com/s358M0K.png" width="300"/>

If you wish for the containing activity to receive the date or time selected by the dialog, you should ensure that the Activity implements the respective interface. If we want the date picker to be shown from within another dialog fragment, refer to [[setting a target fragment|Using-DialogFragment#passing-data-to-parent-fragment]]. For instance, for a date picker fragment, you will want to ensure that the activity implements the `OnDateSetListener` interface:

```java

import java.util.Calendar;  // do not import java.icu.utils.Calendar

public class DatePickerFragment extends DialogFragment {

    @Override
    public Dialog onCreateDialog(Bundle savedInstanceState) {
        // Use the current time as the default values for the picker
        final Calendar c = Calendar.getInstance();
        int year = c.get(Calendar.YEAR);
        int month = c.get(Calendar.MONTH);
        int day = c.get(Calendar.DAY_OF_MONTH);

        // Activity needs to implement this interface
        DatePickerDialog.OnDateSetListener listener = (DatePickerDialog.OnDateSetListener) getActivity();

        // Create a new instance of TimePickerDialog and return it
        return new DatePickerDialog(getActivity(), listener, year, month, day);
    }
```

The Activity, which also is responsible for instantiating this dialog fragment, simply needs to implement the `onDateSet` method of this interface.

```java
public class MyActivity extends AppCompatActivity implements DatePickerDialog.OnDateSetListener {

  // attach to an onclick handler to show the date picker
  public void showDatePickerDialog(View v) {
     DatePickerFragment newFragment = new DatePickerFragment();
     newFragment.show(getSupportFragmentManager(), "datePicker");
  }

  // handle the date selected
  @Override
  public void onDateSet(DatePicker view, int year, int monthOfYear, int dayOfMonth) {
    // store the values selected into a Calendar instance
    final Calendar c = Calendar.getInstance();
    c.set(Calendar.YEAR, year);
    c.set(Calendar.MONTH, monthOfYear);
    c.set(Calendar.DAY_OF_MONTH, dayOfMonth);
  }
```

A similar approach can be done with the time picker too:

```java
public class TimePickerFragment extends DialogFragment {

    @Override
    public Dialog onCreateDialog(Bundle savedInstanceState) {
        // Use the current time as the default values for the picker
        final Calendar c = Calendar.getInstance();
        int hour = c.get(Calendar.HOUR_OF_DAY);
        int minute = c.get(Calendar.MINUTE);

        // Activity has to implement this interface
        TimePickerDialog.OnTimeSetListener listener = (TimePickerDialog.OnTimeSetListener) getActivity();

        // Create a new instance of TimePickerDialog and return it
        return new TimePickerDialog(getActivity(), listener, hour, minute,
                DateFormat.is24HourFormat(getActivity()));
    }

    public void onTimeSet(TimePicker view, int hourOfDay, int minute) {
        // Do something with the time chosen by the user
    }
}
```

### Modal Bottom Sheets

With the [[support design library|Design-Support-Library]], it also fairly easy to convert your dialog to use modal bottom sheets.  Instead of `DialogFragment`, you can extend from `BottomSheetDialogFragment`:

```java
public class MyBottomSheetDialogFragment extends BottomSheetDialogFragment {

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
            Bundle savedInstanceState) {
        return inflater.inflate(R.layout.fragment_bottom_sheet, container);
    }
}
```

When you show this fragment, you will notice that it appears from the bottom:

```java
MyBottomSheetDialogFragment myDialog = new MyBottomSheetDialogFragment();

FragmentManager fm = getSupportFragmentManager();
myDialog.show(fm, "test");
```

## Things To Note

* Notice that we are using the support library version of fragments for better compatibility in our code samples. The non-support version works identically.
* Dialogs are just classes that extend `DialogFragment` and define the view to display in the floating content area.
* `DialogFragment` classes must define an empty constructor as shown in the code samples, otherwise the Android system will raise an exception when it attempts to instantiate the fragment.
* After loading the initial view, the activity immediately shows the dialog using the show() method which allows the fragment manager to keep track of the state and gives us certain things for free such as the back button dismissing the fragment.
* In the code snippets above, notice the use of `requestFocus` and input modes to control the appearance of the soft keyboard when the dialog appears.
* We can dismiss a dialog one of two ways. Here we are calling `dismiss()` within the Dialog class itself. It could also be called from the Activity like the `show()` method. In API 13, `removeDialog(int id)` and `dismissDialog(int id)` were deprecated. `dismiss()` directly on the dialog is now the recommended approach [as outlined here](http://stackoverflow.com/a/36120864/313399).

## References

 * <http://android-developers.blogspot.com/2012/05/using-dialogfragments.html>
 * <http://codebaum.wordpress.com/2013/04/10/dialog-fragments/>
 * <http://developer.android.com/reference/android/app/DialogFragment.html>
 * <http://www.coderzheaven.com/2013/02/14/dialogfragments-android-simple-example/>
 * <http://stackoverflow.com/questions/12912181/simplest-yes-no-dialog-fragment>
