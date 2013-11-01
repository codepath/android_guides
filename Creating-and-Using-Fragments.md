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
        
        // This event fires 1st, before creation of fragment or any views
 	// The onAttach method is called when the Fragment instance is associated with an Activity instance. 
	// This does not mean the Activity is fully initialized.
	@Override
	public void onAttach(Activity activity) {
		super.onAttach(activity);
		this.listener = (FragmentActivity) activity;
	}
       
        // This event fires 2nd, before views are created for the fragment
	// The onCreate method is called when the Fragment instance is being created, or re-created.
	// Use onCreate for any standard setup that does not require the activity to be completed
	@Override
	public void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		ArrayList<Thing> things = new ArrayList<Thing>();
		adapter = new ThingsAdapter(getActivity(), things);
	}
	
        // This event fires 3rd, and is the first time views are available in the fragment
	// The onCreateView method is called when Fragment should create its View object hierarchy. 
	// Use onCreateView to get a handle to views as soon as they are freshly inflated
	@Override
	public View onCreateView(LayoutInflater inf, ViewGroup parent, Bundle savedInstanceState) {
		View v =  inf.inflate(R.layout.fragment_some, parent, false);
		ListView lv = (ListView) v.findViewById(R.id.lvSome);
		lv.setAdapter(adapter);
		return v;
	}
	
        // This fires 4th, and this is the first time the Activity is fully created.
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

Fragments should not directly communicate with each other, only through an activity.  Fragments should be modular and reusable components. Let the activity respond to intents and fragment callbacks in most cases.

There are three ways a fragment and an activity can communicate:

1. Activity can construct a fragment and set arguments
2. Activity can call methods on a fragment instance
3. Fragment can fire listener events on an activity via an interface

#### Fragment with Arguments

In certain cases, your fragment may want to accept certain arguments. A common pattern is to create a static `newInstance` method for creating a Fragment with arguments. This is because a Fragment **must have only a constructor with no arguments**. Instead we want to use the `setArguments` method such as:

```java
public class DemoFragment extends Fragment {
    public static DemoFragment newInstance(int someInt, String someTitle) {
        DemoFragment fragmentDemo = new DemoFragment();
        Bundle args = new Bundle();
        args.putInt("someInt", someInt);
        args.putString("someTitle", someTitle);
        fragmentDemo.setArguments(args);
        return fragmentDemo;
    }
}
```

This sets certain arguments into the Fragment for later use. You can access the argument later by using:

```java
public class DemoFragment extends Fragment {
   @Override
   public void onCreate(Bundle savedInstanceState) {
       super.onCreate(savedInstanceState);
       // Get back arguments
       int SomeInt = getArguments().getInt("someInt", 0);	
       String someTitle = getArguments().getString("someTitle", "");	
   }
}
```

Now we can load a fragment dynamically in an Activity with:

```java
FragmentTransaction ft = getSupportFragmentManager().beginTransaction();
DemoFragment fragmentDemo = DemoFragment.newInstance(5, "my title");
ft.replace(R.id.your_placeholder, fragmentDemo);
ft.commit();
```

This pattern makes passing arguments to fragments fairly straightforward.

#### Fragment Methods

If an activity needs to make a fragment perform an action, the easiest way is by having the activity invoke a method on the fragment instance. In the fragment, add a method:

```java
public class DemoFragment extends Fragment {
  public void doSomething(String param) {
     // do something in fragment
  }
}
```

and then in the activity, get access to the fragment using the fragment manager and call the method:

```java
@Override
public void onCreate(Bundle savedInstanceState) {
     super.onCreate(savedInstanceState);
     DemoFragment fragmentDemo = (DemoFragment) 
         getSupportFragmentManager().findFragmentById(R.id.fragmentDemo)
     fragmentDemo.doSomething("some param");
}
```

and then the activity can communicate directly with the fragment through these methods.

#### Fragment Listener

If a fragment needs to communicate events to the activity, the fragment should define an interface as an inner type and require that the activity must implement this interface:

```java
import android.support.v4.app.Fragment;

public class MyListFragment extends Fragment {
  // ...
  // Define the listener of the interface type
  // listener is the activity itself
  private OnItemSelectedListener listener;
  
  // Define the events that the fragment will use to communicate
  public interface OnItemSelectedListener {
    public void onRssItemSelected(String link);
  }
  
  // Store the listener (activity) that will have events fired once the fragment is attached
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
 
  // Now we can fire the event when the user selects something in the fragment
  public void onSomeClick(View v) {
     listener.onRssItemSelected("some link");
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
  
  // Now we can define the action to take in the activity when the fragment event fires
  @Override
  public void onRssItemSelected(String link) {
      if (fragment != null && fragment.isInLayout()) {
          fragment.setText(link);
      }
  }
}
```

in order to keep the fragment as re-usable as possible.

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

### Navigating Between Fragments

There are several methods for navigating between different fragments within a single Activity. The primary options are:

 1. [[ActionBar Tabs | ActionBar Tabs with Fragments]] - Tabs at the top
 2. [[Fragment Navigation Drawer]] - Slide out navigation menu
 3. [[ViewPager | ViewPager with FragmentPagerAdapter]] - Swiping between fragments

Check the guides linked above for detailed steps for each of these approaches.

## References

 * <http://developer.android.com/reference/android/app/Fragment.html>
 * <http://developer.android.com/training/basics/fragments/creating.html>
 * <http://developer.android.com/guide/components/fragments.html>
 * <http://www.vogella.com/articles/AndroidFragments/article.html>