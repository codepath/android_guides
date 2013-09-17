## Overview

A fragment is a reusable class implementing a portion of an activity.  A Fragment typically defines a part of a user interface. Fragments must be embedded in activities; they cannot run independent of activities.

A fragment has a separate layout XML file. A Fragment encapsulates functionality so that it is easier to reuse within activity and layouts.

Fragments can be dynamically (Java) or statically (XML) added to an activity layout.

### Defining a Fragment

Creating a fragment, like an activity, has an XML layout file and a Java class that represents the [Fragment ](http://developer.android.com/reference/android/app/Fragment.html) controller.

The XML layout file is just like any other layout file, and can be named `fragment_foo.xml`. Think of them as a partial (re-usable) activity:

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent" android:layout_height="match_parent"
    android:orientation="vertical" >

    <TextView
        android:id="@+id/textView1"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="TextView" />

    <Button
        android:id="@+id/button1"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Button" />

</LinearLayout>
```

The Java controller for a fragment looks like:

```java
import android.support.v4.app.Fragment;

public class FooFragment extends Fragment {
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
      Bundle savedInstanceState) {
      // Defines the xml file for the fragment
      View view = inflater.inflate(R.layout.foo, container, false);
      // Setup handles to view objects here
      // etFoo = (EditText) v.findViewById(R.id.etFoo);
      return view;
    }
}
```

### Embedding a Fragment in an Activity

There are two ways to add a fragment to an activity: dynamically using **Java** and statically using **XML**. 

Before embedding a "support" fragment in an Activity make sure the Activity is changed to extend from `FragmentActivity` which adds support for the fragment manager to all Android versions. Any activity using fragments should make sure to extend from `FragmentActivity`:

```java
import android.support.v4.app.FragmentActivity;

public class MainActivity extends FragmentActivity {
    // ...
}
```

#### Statically

To add the fragment **statically**, simply embed the fragment in the activity's xml layout file:

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical" >

   <TextView
        android:id="@+id/textView1"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="TextView" />

    <fragment
        android:id="@+id/fooFragment"
        android:layout_height="match_parent"
        android:layout_marginTop="?android:attr/actionBarSize"
        class="com.example.android.FooFragment" ></fragment>

</LinearLayout>
```

#### Dynamically

The second way is by adding the fragment **dynamically** in Java using the `FragmentManager`. The `FragmentManager` class and the [FragmentTransaction class](http://developer.android.com/reference/android/app/FragmentTransaction.html) allow you to add, remove and replace fragments in the layout of your activity.

In this case, you need a "placeholder" that can later be replaced with the fragment:

```java
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical" >

 <TextView
        android:id="@+id/textView1"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="TextView" />

    <FrameLayout
       android:id="@+id/your_placeholder"
       android:layout_height="match_parent">
    </FrameLayout>

</LinearLayout>
```

and then you can use the [FragmentManager](http://developer.android.com/reference/android/app/FragmentManager.html) to create a [FragmentTransaction](http://developer.android.com/reference/android/app/FragmentTransaction.html) which allows us to replace the FrameLayout with any fragment at runtime:

```java
FragmentTransaction ft = getSupportFragmentManager().beginTransaction();
ft.replace(R.id.your_placeholder, new FooFragment());
// or ft.add(R.id.your_placeholder, new FooFragment());
ft.commit();
```

If the fragment should always be in the activity, use XML to statically add but if it's more complex use the Java-based approach.

### Fragment Lifecycle

Fragment has many methods which can be overriden to plug into the lifecycle (similar to an Activity):

- `onAttach()` is called when a fragment is connected to an activity
- `onCreate()` is called to do initial creation of the fragment.
- `onCreateView()` is called by Android once the Fragment should create
- `onActivityCreated()` is called when host activity has completed its `onCreate()`.
- `onDestroyView()` is called when fragment is being destroyed
- `onStart()` is called once the fragment gets visible
- `onResume()` - Allocate “expensive” resources such as registering for location, sensor updates, etc.
- `onPause()` - Release “expensive” resources. Commit any changes.

The most common are `onCreateView` which is in almost every fragment to setup the view, `onCreate` for any initialization and `onActivityCreated` used for setting up things that can only take place once the Activity is created.

Here's an example of how you might use the various fragment lifecycle events:

```java
public class SomeFragment extends Fragment {
	ThingsAdapter adapter;
	FragmentActivity listener;
        
	// The onAttach method is called when the Fragment instance is associated with an Activity instance. 
	// This does not mean the Activity is fully initialized.
	@Override
	public void onAttach(Activity activity) {
		super.onAttach(activity);
		this.listener = (FragmentActivity) activity;
	}

	// The onCreate method is called when the Fragment instance is being created, or re-created.
	// Use onCreate for any standard setup that does not require the activity to be completed
	@Override
	public void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		ArrayList<Thing> things = new ArrayList<Thing>();
		adapter = new ThingsAdapter(getActivity(), things);
	}
	
	// The onCreateView method is called when Fragment should create its View object hierarchy. 
	// Use onCreateView to get a handle to views as soon as they are freshly inflated
	@Override
	public View onCreateView(LayoutInflater inf, ViewGroup parent, Bundle savedInstanceState) {
		View v =  inf.inflate(R.layout.fragment_some, parent, false);
		ListView lv = (ListView) v.findViewById(R.id.lvSome);
		lv.setAdapter(adapter);
		return v;
	}
	
	// Accessing the view hierarchy of the parent activity must be done in the onActivityCreated
	// At this point, it is safe to search for View objects by their ID, for example.
	@Override
	public void onActivityCreated(Bundle savedInstanceState) {
		super.onActivityCreated(savedInstanceState);
	}
}
```

[This chart](http://developer.android.com/images/fragment_lifecycle.png) has the lifecycle displayed visually.

### Fragment <-> Activity Communication

Fragments should not directly communicate with each other, only through an activity.  Fragments should be modular and reusable components. Let the activity respond to intents and fragment callbacks.

Fragments should define an interface as an inner type and require that the activity must implement this interface:

```java
import android.support.v4.app.Fragment;

public class MyListFragment extends Fragment {
  private OnItemSelectedListener listener;

  @Override
  public View onCreateView(LayoutInflater inflater, ViewGroup container,
      Bundle savedInstanceState) {
      // ...
  }

  public interface OnItemSelectedListener {
    public void onRssItemSelected(String link);
  }

  @Override
  public void onAttach(Activity activity) {
    super.onAttach(activity);
      if (activity instanceof OnItemSelectedListener) {
        listener = (OnItemSelectedListener) activity;
      } else {
        throw new ClassCastException(activity.toString()
            + " must implement MyListFragment.OnItemSelectedListener");
      }
  }
}
```

and then in the activity:

```java
import android.support.v4.app.FragmentActivity;

public class RssfeedActivity extends FragmentActivity implements
  MyListFragment.OnItemSelectedListener {
    DetailFragment fragment;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_rssfeed);
        fragment = (DetailFragment) getSupportFragmentManager()
              .findFragmentById(R.id.detailFragment);
    }

  @Override
    public void onRssItemSelected(String link) {
          if (fragment != null && fragment.isInLayout()) {
            fragment.setText(link);
          }
    }
}
```

in order to keep the fragment as re-usable as possible.

### Fragments and Tabs

There are several ways to setup tabs with fragments. The easiest is using ActionBar tabs. Note: ActionBar tabs are not supported in Gingerbread, so many people use [ActionBarSherlock](http://actionbarsherlock.com/) when Gingerbread must be supported. Thankfully, the API is more or less identical with a few class name tweaks.

#### Without ActionBarSherlock

To setup tabs using ActionBar and fragments, you need to add a `TabListener` implementation to your application which defines the behavior of a tab when activated. A good default implementation is just [adding this](https://gist.github.com/nesquena/f54a991ccb4e5929e0ec) to `FragmentTabListener.java`. 

Once you have created the `FragmentTabListener` [from this snippet](https://gist.github.com/nesquena/f54a991ccb4e5929e0ec) within your app, simply setup the ActionBar and define which tabs you would like to display and attaches listeners for each tab:

```java
public class MainActivity extends FragmentActivity {

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		setupTabs();
	}

	private void setupTabs() {
		ActionBar actionBar = getActionBar();
		actionBar.setNavigationMode(ActionBar.NAVIGATION_MODE_TABS);
		actionBar.setDisplayShowTitleEnabled(true);

		Tab tab1 = actionBar
			.newTab()
			.setText("First")
			.setIcon(R.drawable.ic_home)
			.setTag("HomeTimelineFragment")
			.setTabListener(
				new FragmentTabListener<FirstFragment>(R.id.flContainer, this, "first",
								FirstFragment.class));

		actionBar.addTab(tab1);
		actionBar.selectTab(tab1);

		Tab tab2 = actionBar
			.newTab()
			.setText("Second")
			.setIcon(R.drawable.ic_mentions)
			.setTag("MentionsTimelineFragment")
			.setTabListener(
			    new FragmentTabListener<SecondFragment>(R.id.flContainer, this, "second",
								SecondFragment.class));

		actionBar.addTab(tab2);
	}
}
```

#### With ActionBarSherlock

Using [ActionBarSherlock](http://actionbarsherlock.com/), the code looks almost exactly the same. First you must download [ActionBarSherlock.zip](https://api.github.com/repos/JakeWharton/ActionBarSherlock/zipball/4.4.0). Import the code into Eclipse and mark the project as a library. Next, add the library to your application. Watch the [video](http://www.youtube.com/watch?v=4GJ6yY1lNNY) on the [FAQ](http://actionbarsherlock.com/faq.html) page for a detailed guide.

With ActionBarSherlock installed and added to your app, add [the gist code](https://gist.github.com/nesquena/0f0a1f50a2a64721309e) as the `SherlockTabListener` within your application.

Once you have created the `SherlockTabListener` [from this snippet](https://gist.github.com/nesquena/0f0a1f50a2a64721309e) within your app, simply setup the ActionBar and define which tabs you would like to display and attaches listeners for each tab:

```java
public class ActionBarSherlockActivity extends SherlockFragmentActivity {

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_action_bar_sherlock);
		setupTabs();
	}

	private void setupTabs() {
		ActionBar actionBar = getSupportActionBar();
		actionBar.setNavigationMode(ActionBar.NAVIGATION_MODE_TABS);
		actionBar.setDisplayShowHomeEnabled(true);
		actionBar.setDisplayShowTitleEnabled(true);

		Tab tabFirst = actionBar
			.newTab()
			.setText("First")
			.setIcon(R.drawable.ic_launcher)
			.setTabListener(
				new SherlockTabListener<FirstFragment>(R.id.flContainer, this, "First",
								FirstFragment.class));

		actionBar.addTab(tabFirst);
		actionBar.selectTab(tabFirst);

		Tab tabSecond = actionBar
			.newTab()
			.setText("Second")
			.setIcon(R.drawable.ic_launcher)
			.setTabListener(
				new SherlockTabListener<SecondFragment>(R.id.flContainer, this, "Second",
							SecondFragment.class));

		actionBar.addTab(tabSecond);
	}
}
```

With this approach you can have easy tab navigation switching quickly between fragments.

### ActionBar Menu Items and Fragments

One common case is the need for fragment-specific menu items that only show up for that fragment. This can be done by adding an `onCreateOptionsMenu` method to the fragment directly. This works just like the one for the activity:

```java
@Override
public void onCreateOptionsMenu(Menu menu, MenuInflater inflater) {
   inflater.inflate(R.menu.fragment_menu, menu);
}
```

You then also need to notify the fragment that it's menu items should be loaded within the fragment's `onCreate` method:

```java
@Override
public void onCreate(Bundle savedInstanceState) {
     super.onCreate(savedInstanceState);
     setHasOptionsMenu(true);
}  
```

Clicks can be handled using `onClick` property as usual or more typically in this case, using the `onOptionsItemSelected` method in the fragment:

```java
@Override
public boolean onOptionsItemSelected(MenuItem item) {
   // handle item selection
   switch (item.getItemId()) {
      case R.id.edit_item:
         // do s.th.
         return true;
      default:
         return super.onOptionsItemSelected(item);
   }
}
```

Note that the fragment’s method is called only when the Activity didn’t consume the event first. Be sure to check out a more detailed guide about [fragments and action bar](http://www.grokkingandroid.com/adding-action-items-from-within-fragments/) if you have more questions.

## References

 * <http://developer.android.com/reference/android/app/Fragment.html>
 * <http://developer.android.com/training/basics/fragments/creating.html>
 * <http://developer.android.com/guide/components/fragments.html>
 * <http://www.vogella.com/articles/AndroidFragments/article.html>