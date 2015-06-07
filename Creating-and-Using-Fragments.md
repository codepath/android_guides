## Overview

A fragment is a reusable class implementing a portion of an activity.  A Fragment typically defines a part of a user interface. Fragments must be embedded in activities; they cannot run independently of activities.

![Fragments](http://developer.android.com/images/fundamentals/fragments.png)

A fragment has a separate layout XML file. A Fragment encapsulates functionality so that it is easier to reuse within activities and layouts. Fragments are standalone components that can contain views, events and logic. Within a fragment-oriented architecture, applications become navigation containers that are primarily responsible for navigation to other activities, presenting fragments and passing data.

### Defining a Fragment

A fragment, like an activity, has an XML layout file and a Java class that represents the [Fragment](http://developer.android.com/reference/android/app/Fragment.html) controller.

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
      // etFoo = (EditText) view.findViewById(R.id.etFoo);
      return view;
    }
}
```

### Embedding a Fragment in an Activity

There are two ways to add a fragment to an activity: dynamically using **Java** and statically using **XML**. 

Before embedding a "support" fragment in an Activity make sure the Activity is changed to extend from `FragmentActivity` or `AppCompatActivity` which adds support for the fragment manager to all Android versions. Any activity using fragments should make sure to extend from `FragmentActivity` or `AppCompatActivity`:

```java
import android.support.v7.app.AppCompatActivity;

public class MainActivity extends AppCompatActivity {
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

    <fragment
        android:name="com.example.android.FooFragment"
        android:id="@+id/fooFragment"
        android:layout_width="match_parent" 
        android:layout_height="match_parent" />

</LinearLayout>
```

#### Dynamically

The second way is by adding the fragment **dynamically** in Java using the `FragmentManager`. The `FragmentManager` class and the [FragmentTransaction class](http://developer.android.com/reference/android/app/FragmentTransaction.html) allow you to add, remove and replace fragments in the layout of your activity at runtime.

In this case, you want to add a "placeholder" container (usually a `FrameLayout`) to your activity where the fragment is inserted at runtime:

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical" >

  <FrameLayout
       android:id="@+id/your_placeholder"
       android:layout_height="match_parent">
  </FrameLayout>

</LinearLayout>
```

and then you can use the [FragmentManager](http://developer.android.com/reference/android/app/FragmentManager.html) to create a [FragmentTransaction](http://developer.android.com/reference/android/app/FragmentTransaction.html) which allows us to add fragments to the `FrameLayout` at runtime:

```java
// Begin the transaction
FragmentTransaction ft = getSupportFragmentManager().beginTransaction();
// Replace the contents of the container with the new fragment
ft.replace(R.id.your_placeholder, new FooFragment());
// or ft.add(R.id.your_placeholder, new FooFragment());
// Complete the changes added above
ft.commit();
```

If the fragment should always be within the activity, use XML to statically add the fragment but in more complex cases be sure to use the Java-based approach.

### Fragment Lifecycle

Fragment has many methods which can be overridden to plug into the lifecycle (similar to an Activity):

- `onAttach()` is called when a fragment is connected to an activity.
- `onCreate()` is called to do initial creation of the fragment.
- `onCreateView()` is called by Android once the Fragment should inflate a view.
- `onActivityCreated()` is called when host activity has completed its `onCreate()` method.
- `onStart()` is called once the fragment gets visible.
- `onResume()` - Allocate “expensive” resources such as registering for location, sensor updates, etc.
- `onPause()` - Release “expensive” resources. Commit any changes.
- `onDestroyView()` is called when fragment's view is being destroyed, but the fragment is still kept around.
- `onDestroy()` is called when fragment is no longer in use.
- `onDetach()` is called when fragment is no longer connected to the activity.

The lifecycle execution order is mapped out below:

<a href="http://i.imgur.com/0EVReuq.png"><img src="http://i.imgur.com/0EVReuq.png" alt="lifecycle" width="500" /></a>

The most common ones to override are `onCreateView` which is in almost every fragment to setup the inflated view, `onCreate` for any data initialization and `onActivityCreated` used for setting up things that can only take place once the Activity has been fully created.

Here's an example of how you might use the various fragment lifecycle events:

```java
public class SomeFragment extends Fragment {
	ThingsAdapter adapter;
	FragmentActivity listener;
        
	// This event fires 1st, before creation of fragment or any views
 	// The onAttach method is called when the Fragment instance is associated with an Activity. 
	// This does not mean the Activity is fully initialized.
	@Override
	public void onAttach(Activity activity) {
		super.onAttach(activity);
		this.listener = (FragmentActivity) activity;
	}
       
	// This event fires 2nd, before views are created for the fragment
	// The onCreate method is called when the Fragment instance is being created, or re-created.
	// Use onCreate for any standard setup that does not require the activity to be fully created
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
	// At this point, it is safe to search for activity View objects by their ID, for example.
	@Override
	public void onActivityCreated(Bundle savedInstanceState) {
		super.onActivityCreated(savedInstanceState);
	}
}
```

[This chart](http://developer.android.com/images/fragment_lifecycle.png) has the lifecycle displayed visually.

### Looking Up a Fragment Instance

Often we need to lookup or find a fragment instance within an activity layout file. There are a few methods for looking up an existing fragment instance:

 1. **ID** - Lookup a fragment by calling `findFragmentById` on the `FragmentManager`
 2. **Tag** - Lookup a fragment by calling `findFragmentByTag` on the `FragmentManager`
 3. **Pager** - Lookup a fragment by calling `getRegisteredFragment` on a `PagerAdapter`

Each method is outlined in more detail below.

#### Finding Fragment By ID

If the fragment was statically embedded in the XML within an activity and given an `android:id` such as `fragmentDemo` then we can lookup this fragment by id by calling `findFragmentById` on the `FragmentManager`:

```java
public class MainActivity extends FragmentActivity {
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        if (savedInstanceState == null) {
          DemoFragment fragmentDemo = (DemoFragment) 
              getSupportFragmentManager().findFragmentById(R.id.fragmentDemo);
        }
    }
}
```

#### Finding Fragment By Tag

If the fragment was dynamically added at runtime within an activity then we can lookup this fragment by tag by calling `findFragmentByTag` on the `FragmentManager`:

```java
public class MainActivity extends FragmentActivity {
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        if (savedInstanceState == null) {
          // Let's first dynamically add a fragment into a frame container
          getSupportFragmentManager().beginTransaction(). 
              replace(R.id.flContainer, new DemoFragment(), "SOMETAG").
              commit();
          // Now later we can lookup the fragment by tag
          DemoFragment fragmentDemo = (DemoFragment) 
              getSupportFragmentManager().findFragmentByTag("SOMETAG");
        }
    }
}
```

#### Finding Fragment Within Pager

If the fragment was dynamically added at runtime within an activity into a `ViewPager` using a [[FragmentPagerAdapter|ViewPager-with-FragmentPagerAdapter#setup-fragmentpageradapter]] then we can lookup the fragment by upgrading to a `SmartFragmentStatePagerAdapter` as [[described in the ViewPager guide|ViewPager-with-FragmentPagerAdapter#setup-smartfragmentstatepageradapter]]. Now with the adapter in place, we can also easily access any fragments within the ViewPager using `getRegisteredFragment`:

```java
// returns first Fragment item within the pager
adapterViewPager.getRegisteredFragment(0); 
```

Note that the `ViewPager` loads the fragment instances lazily similar to the a `ListView` recycling items as they appear on screen. If you attempt to access a fragment that is not on screen, the lookup will return `null`.

### Communicating with Fragments

Fragments should not directly communicate with each other, only through an activity.  Fragments should be modular and reusable components. Let the activity respond to intents and fragment callbacks in most cases.

There are three ways a fragment and an activity can communicate:

1. **Bundle** - Activity can construct a fragment and set arguments
2. **Methods** - Activity can call methods on a fragment instance
3. **Listener** - Fragment can fire listener events on an activity via an interface

#### Fragment with Arguments

In certain cases, your fragment may want to accept certain arguments. A common pattern is to create a static `newInstance` method for creating a Fragment with arguments. This is because a Fragment **must have only a constructor with no arguments**. Instead, we want to use the `setArguments` method such as:

```java
public class DemoFragment extends Fragment {
    // Creates a new fragment given an int and title
    // DemoFragment.newInstance(5, "Hello");
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

This sets certain arguments into the Fragment for later access within `onCreate`. You can access the arguments later by using:

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
// Within the activity
FragmentTransaction ft = getSupportFragmentManager().beginTransaction();
DemoFragment fragmentDemo = DemoFragment.newInstance(5, "my title");
ft.replace(R.id.your_placeholder, fragmentDemo);
ft.commit();
```

This pattern makes passing arguments to fragments for initialization fairly straightforward.

#### Fragment Methods

If an activity needs to make a fragment perform an action after initialization, the easiest way is by having the activity invoke a method on the fragment instance. In the fragment, add a method:

```java
public class DemoFragment extends Fragment {
  public void doSomething(String param) {
      // do something in fragment
  }
}
```

and then in the activity, get access to the fragment using the fragment manager and call the method:

```java
public class MainActivity extends FragmentActivity {
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        DemoFragment fragmentDemo = (DemoFragment) 
            getSupportFragmentManager().findFragmentById(R.id.fragmentDemo);
        fragmentDemo.doSomething("some param");
    }
}
```

and then the activity can communicate directly with the fragment by invoking this method.

#### Fragment Listener

If a fragment needs to communicate events to the activity, the fragment should [[define an interface|Creating-Custom-Listeners]] as an inner type and require that the activity must implement this interface:

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

in order to keep the fragment as re-usable as possible. For more details about this pattern, check out our detailed [[Creating Custom Listeners]] guide.

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

 1. [[TabLayout| Google Play Style Tabs using TabLayout]] - Tabs at the top
 2. [[Fragment Navigation Drawer]] - Slide out navigation menu
 3. [[ViewPager | ViewPager with FragmentPagerAdapter]] - Swiping between fragments

Check the guides linked above for detailed steps for each of these approaches.

### Managing Fragment Backstack

A record of all Fragment transactions is kept for each Activity by the FragmentManager.  When used properly, this allows the user to hit the device’s back button to remove previously added Fragments (not unlike how the back button removes an Activity). Simply call [addToBackstack](http://developer.android.com/reference/android/app/FragmentTransaction.html#addToBackStack\(java.lang.String\)) on each FragmentTransaction that should be recorded:

```java
// Create the transaction
FragmentTransaction fts = getSupportFragmentManager().beginTransaction();
// Replace the content of the container
fts.replace(R.id.flContainer, new FirstFragment());	
// Append this transaction to the backstack
fts.addToBackStack("optional tag");
// Commit the changes
fts.commit();
```

Programmatically, you can also pop from the back stack at any time through the manager:

```java
FragmentManager fragmentManager = getSupportFragmentManager();
if (fragmentManager.getBackStackEntryCount() > 0) {
    fragmentManager.popBackStack();
}
```

With this approach, we can easily keep the history of which fragments have appeared dynamically on screen and allow the user to easily navigate to previous fragments.

### Fragment Hiding vs Replace

In many of the examples above, we call `transaction.replace(...)` to load a dynamic fragment which first removes the existing fragment from the activity invoking `onStop` and `onDestroy` for that fragment before adding the new fragment to the container. This can be good because this will release memory and make the UI snappier. However, in many cases, we may want to keep both fragments around in the container and simply toggle their visibility. This allows all fragments to maintain their state because they are never removed from the container. To do this, we might modify this code:

```java
// Within an activity

private FragmentA fragmentA;
private FragmentB fragmentB;
private FragmentC fragmentC;

@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    if (savedInstanceState == null) {
        fragmentA = FragmentA.newInstance("foo");
        fragmentB = FragmentB.newInstance("bar");
        fragmentC = FragmentC.newInstance("baz");
    }
}

protected void displayFragmentA() {
    FragmentTransaction ft = getSupportFragmentManager().beginTransaction();
    // removes the existing fragment calling onDestroy
    ft.replace(R.id.flContainer, fragmentA); 
    ft.commit();
}
```

to this approach instead leveraging `add`, `show`, and `hide` in the `FragmentTransaction`:

```java
// ...onCreate stays the same

// Replace the switch method
protected void displayFragmentA() {
    FragmentTransaction ft = getSupportFragmentManager().beginTransaction();
    if (fragmentA.isAdded()) { // if the fragment is already in container
        ft.show(fragmentA);
    } else { // fragment needs to be added to frame container
        ft.add(R.id.flContainer, fragmentA, "A");
    }
    // Hide fragment B
    if (fragmentB.isAdded()) { ft.hide(fragmentB); }
    // Hide fragment C
    if (fragmentC.isAdded()) { ft.hide(fragmentC); }
    // Commit changes
    ft.commit();
}
```

Using this approach, all three fragments will remain in the container once added initially and then we are simply revealing the desired fragment and hiding the others within the container. Check out [this stackoverflow](http://stackoverflow.com/a/13185025/313399) for a discussion on deciding when to replace vs hide and show.

### Nesting Fragments within Fragments

Inevitably in certain cases you will want to **embed a fragment within another fragment**. Since [Android 4.2](http://developer.android.com/about/versions/android-4.2.html#NestedFragments), you have the ability to embed a fragment within another fragment. This nested fragment is known as a **child fragment**.
A common situation where you might want to nest fragments is when you are using a sliding drawer for top-level navigation and one of the fragments needs to display tabs. 

Note that one limitation is that nested (or child) fragments **must be dynamically added at runtime** to their parent fragment and cannot be statically added using the `<fragment>` tag. To nest a fragment in another fragment, first we need a `<FrameLayout>` or alternatively a [[ViewPager|ViewPager-with-FragmentPagerAdapter]] to contain the dynamic child fragment in the `res/layout/fragment_parent.xml` layout:

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="fill_parent" android:layout_height="fill_parent"
    android:orientation="vertical" >
<TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="I am the parent fragment" />
<FrameLayout
        android:id="@+id/child_fragment_container"
        android:layout_width="fill_parent"
        android:layout_height="wrap_content" />
</LinearLayout>
```

Notice that there's a `FrameLayout` with the id of `@+id/child_fragment_container` in which the child fragment will be inserted. Inflation of the `ParentFragment` view is within the `onCreateView` method, just as was outlined in earlier sections. In addition, we would also define a `ChildFragment` that would have its own distinct layout file:

```java
// Top-level fragment that will contain the child
public class ParentFragment extends Fragment {
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
        Bundle savedInstanceState) {
      View view = (View) inflater.inflate(R.layout.fragment_parent, container, false);
      return view;
   }
}

// Child fragment that will be dynamically embedded in the parent
public class ChildFragment extends Fragment {
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
        Bundle savedInstanceState) {
       // Need to define the child fragment layout
       View view = (View) inflater.inflate(R.layout.fragment_child, container, false);
       return view;
   }
}
```

Now we can add the child fragment to the parent at runtime using the `getChildFragmentManager` method:

```java
// Top-level fragment that will contain the child
public class ParentFragment extends Fragment {
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
        Bundle savedInstanceState) {
      View view = (View) inflater.inflate(R.layout.fragment_parent, container, false);
      insertNestedFragment();
      return view;
   }
  
   // Embeds the child fragment dynamically
   private void insertNestedFragment() {
      Fragment childFragment = new ChildFragment();
      FragmentTransaction transaction = getChildFragmentManager().beginTransaction();
      transaction.replace(R.id.child_fragment_container, childFragment).commit();
   }
}
```

Note that **you must always use `getChildFragmentManager`** when interacting with nested fragments instead of using `getSupportFragmentManager`. Read [this stackoverflow post](http://stackoverflow.com/a/14775322) for an explanation of the difference between the two.

In the child fragment, we can use `getParentFragment()` to get the reference to the parent fragment, similar to a fragment's `getActivity()` method that gives reference to the parent Activity. See [the docs](http://developer.android.com/reference/android/app/Fragment.html#getParentFragment\(\)) for more information. 

### Managing Configuration Changes 

When you are working with fragment as with activities, you need to make sure to [[handle configuration changes|Handling-Configuration-Changes#saving-and-restoring-fragment-state]] such as screen rotation or the activity being closed. Be sure to review [[the configuration changes guide|Handling-Configuration-Changes#saving-and-restoring-fragment-state]] for more details on how to save and restore fragment state.

## References

 * <http://developer.android.com/reference/android/app/Fragment.html>
 * <http://developer.android.com/training/basics/fragments/creating.html>
 * <http://developer.android.com/guide/components/fragments.html>
 * <http://www.vogella.com/articles/AndroidFragments/article.html>
 * <http://xperiment-andro.blogspot.com/2013/02/nested-fragments.html>