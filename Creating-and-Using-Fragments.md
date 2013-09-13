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
- `onCreateView()` method is called by Android once the Fragment should create
- `onDestroyView()` method is called when fragment is being destroyed
- `onActivityCreated()` is called when host activity is created
- `onStart()` method is called once the fragment gets visible
- `onResume()` - Allocate “expensive” resources such as registering for location, sensor updates, etc.
- `onPause()` - Release “expensive” resources. Commit any changes.

The most common are `onCreateView` which is in almost every fragment to setup the view and `onActivityCreated` used for setting up things that can only take place once the Activity is created.

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

To setup tabs using ActionBar and fragments, you need to add a `TabListener` implementation to your application which defines the behavior of a tab when activated. A good default implementation is:

```java
package com.codepath.example.simpletabsdemo;

import android.app.ActionBar.Tab;
import android.app.ActionBar.TabListener;
import android.support.v4.app.Fragment;
import android.support.v4.app.FragmentActivity;
import android.support.v4.app.FragmentTransaction;

public class FragmentTabListener<T extends Fragment> implements TabListener {
	private Fragment mFragment;
	private final FragmentActivity mActivity;
	private final String mTag;
	private final Class<T> mClass;
	private final int mfragmentContainerId;
        
        // This version defaults to replacing the entire activity content area
        // new FragmentTabListener<SomeFragment>(this, "first", SomeFragment.class))
	public FragmentTabListener(FragmentActivity activity, String tag, Class<T> clz) {
		mActivity = activity;
		mTag = tag;
		mClass = clz;
		mfragmentContainerId = android.R.id.content;
	}
        
        // This version supports specifying the container to replace with fragment content
        // new FragmentTabListener<SomeFragment>(R.id.flContent, this, "first", SomeFragment.class))
	public FragmentTabListener(int fragmentContainerId, FragmentActivity activity, 
            String tag, Class<T> clz) {
		mActivity = activity;
		mTag = tag;
		mClass = clz;
		mfragmentContainerId = fragmentContainerId;
	}

	/* The following are each of the ActionBar.TabListener callbacks */

	public void onTabSelected(Tab tab, android.app.FragmentTransaction ft) {
		FragmentTransaction sft = mActivity.getSupportFragmentManager().beginTransaction();
		// Check if the fragment is already initialized
		if (mFragment == null) {
			// If not, instantiate and add it to the activity
			mFragment = Fragment.instantiate(mActivity, mClass.getName());
			sft.add(mfragmentContainerId, mFragment, mTag);
		} else {
			// If it exists, simply attach it in order to show it
			sft.attach(mFragment);
		}
		sft.commit();
	}

	public void onTabUnselected(Tab tab, android.app.FragmentTransaction ft) {
		FragmentTransaction sft = mActivity.getSupportFragmentManager().beginTransaction();
		if (mFragment != null) {
			// Detach the fragment, because another one is being attached
			sft.detach(mFragment);
		}
		sft.commit();
	}

	public void onTabReselected(Tab tab, android.app.FragmentTransaction ft) {
		// User selected the already selected tab. Usually do nothing.
	}
}
```

Next, simply setup the ActionBar and define which tabs you would like to display and attaches listeners for each tab:

```java
ActionBar actionBar = getActionBar();
actionBar.setNavigationMode(ActionBar.NAVIGATION_MODE_TABS);
actionBar.setDisplayShowTitleEnabled(true);

Tab tab1 = actionBar
		.newTab()
		.setText("First")
		.setTabListener(
				new FragmentTabListener<FirstFragment>(R.id.flContainer, this, "first",
						FirstFragment.class))
                .setTag("HomeTimelineFragment")
		.setIcon(R.drawable.ic_first);

actionBar.addTab(tab1);
actionBar.selectTab(tab1);

Tab tab2 = actionBar
		.newTab()
		.setText("Second")
		.setTabListener(
				new FragmentTabListener<SecondFragment>(R.id.flContainer, this, "second",
						SecondFragment.class))
                .setTag("MentionsTimelineFragment")
		.setIcon(R.drawable.ic_second);

actionBar.addTab(tab2);
```

#### With ActionBarSherlock

## References

 * <http://developer.android.com/reference/android/app/Fragment.html>
 * <http://developer.android.com/training/basics/fragments/creating.html>
 * <http://developer.android.com/guide/components/fragments.html>
 * <http://www.vogella.com/articles/AndroidFragments/article.html>