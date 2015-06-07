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
		// Empty constructor required for DialogFragment
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
		View view = inflater.inflate(R.layout.fragment_edit_name, container);
		mEditText = (EditText) view.findViewById(R.id.txt_your_name);
		String title = getArguments().getString("title", "Enter Name");
		getDialog().setTitle(title);
		// Show soft keyboard automatically
		mEditText.requestFocus();
		getDialog().getWindow().setSoftInputMode(
				WindowManager.LayoutParams.SOFT_INPUT_STATE_VISIBLE);
		return view;
	}
}
```
and showing the dialog in an Activity (extending FragmentActivity or AppCompatActivity):

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

and to display the alert dialog in an activity (extending `FragmentActivity` or `AppCompatActivity`):

```java
// Note: `AppCompatActivity` works here as well
public class FragmentDialogDemo extends FragmentActivity {
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
public class FragmentDialogDemo extends FragmentActivity implements EditNameDialogListener {
  // ...
  
  @Override
  public void onFinishEditDialog(String inputText) {
  	Toast.makeText(this, "Hi, " + inputText, Toast.LENGTH_SHORT).show();
  }
}
```

**Note:** `setOnEditorActionListener` used above to dismiss requires the use of the soft keyboard in the emulator which [can be enabled through AVD](http://imgur.com/a/kf1s9) or by testing on a device. If you don't want to enable soft keyboard, you may want to dismiss on a button click or on a keypress instead.

## Styling Dialogs

Styling a DialogFragment with a custom layout works just the [[same as styling any views|Styles-and-Themes]]. Styling an `AlertDialog` requires changing several key properties in `styles.xml` such as the `android:alertDialogTheme` as shown in this app [here](https://github.com/aliHafizji/Cheddar-Android/blob/master/res/values/styles_cheddar.xml#L15) and defining your own dialog style extending from `@android:style/Theme.Holo.Light.Dialog` as shown [here](https://github.com/aliHafizji/Cheddar-Android/blob/master/res/values/styles_cheddar.xml#L15).

## Things To Note

* Notice that we are using the support library version of fragments for better compatibility in our code samples. The non-support version works identically.
* Dialogs are just classes that extend `DialogFragment` and define the view to display in the floating content area.
* `DialogFragment` classes must define an empty constructor as shown in the code samples, otherwise the Android system will raise an exception when it attempts to instantiate the fragment.
* After loading the initial view, the activity immediately shows the dialog using the show() method which allows the fragment manager to keep track of the state and gives us certain things for free such as the back button dismissing the fragment.
* In the code snippets above, notice the use of `requestFocus` and input modes to control the appearance of the soft keyboard when the dialog appears.
* We can dismiss a dialog one of two ways. Here we are calling `dismiss()` within the Dialog class itself. It could also be called from the Activity like the `show()` method.

### Different Dialog Types

When using the `onCreateDialog` method there are many built-in Dialog types to take advantage of:

* [AlertDialog](http://developer.android.com/reference/android/app/AlertDialog.html) - Base dialog that can display a message, an icon and 1-3 buttons with customized text. 
* [ProgressDialog](http://developer.android.com/reference/android/app/ProgressDialog.html) - Dialog showing a progress indicator and an optional text message 
* [TimePickerDialog](http://developer.android.com/reference/android/app/TimePickerDialog.html) - Dialog that allows a user to select a time.
* [DatePickerDialog](http://developer.android.com/reference/android/app/DatePickerDialog.html) - Dialog that allows a user to select a date.
* Other dialogs [not](http://developer.android.com/reference/android/text/method/CharacterPickerDialog.html) [worth](http://developer.android.com/reference/android/support/v7/app/MediaRouteChooserDialog.html) discussing here.

## Libraries

* CodePath [android-view-helpers](https://github.com/codepath/android-view-helpers) for an easier way to create simple alert and progress modals.

## References

 * <http://android-developers.blogspot.com/2012/05/using-dialogfragments.html>
 * <http://codebaum.wordpress.com/2013/04/10/dialog-fragments/>
 * <http://developer.android.com/reference/android/app/DialogFragment.html>
 * <http://www.coderzheaven.com/2013/02/14/dialogfragments-android-simple-example/>
 * <http://stackoverflow.com/questions/12912181/simplest-yes-no-dialog-fragment>