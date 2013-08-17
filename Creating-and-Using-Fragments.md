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

There are two ways to add a fragment to an activity: dynamically using **Java** and statically using **XML**. To add the fragment statically, simply embed the fragment in the activity's xml layout file:

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

The second way is by adding the fragment dynamically in Java using the `FragmentManager`. The `FragmentManager` class and the [FragmentTransaction class](http://developer.android.com/reference/android/app/FragmentTransaction.html) allow you to add, remove and replace fragments in the layout of your activity.

```java
FragmentTransaction ft = getSupportFragmentManager().beginTransaction();
ft.replace(R.id.your_placeholder, new FooFragment());
// or ft.add(R.id.your_placeholder, new FooFragment());
ft.commit();
```

If the fragment should always be in the activity, use XML to statically add but if it's more complex use the Java-based approach.

### Fragment Lifecycle

- `onAttach()` is called when a fragment is connected to an activity
- `onCreateView()` method is called by Android once the Fragment should create
- `onActivityCreated()` is called when host activity is created
- `onStart()` method is called once the fragment gets visible
- `onDestroyView()` method is called when fragment is being destroyed
- `onResume()` - Allocate “expensive” resources such as registering for location, sensor updates, etc.
- `onPause()` - Release “expensive” resources. Commit any changes.

### Fragment <-> Activity Communication

Fragments should not directly communicate with each other, only through an activity.  Fragments should be modular and reusable components. Let the activity respond to intents and fragment callbacks.

Fragments should define an interface as an inner type and require that the activity must implement this interface:

```java
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
            + " must implemenet MyListFragment.OnItemSelectedListener");
      }
  }
}
```

and then in the activity:

```java
public class RssfeedActivity extends Activity implements
  MyListFragment.OnItemSelectedListener {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_rssfeed);
    }

  @Override
    public void onRssItemSelected(String link) {
      DetailFragment fragment = (DetailFragment) getFragmentManager()
              .findFragmentById(R.id.detailFragment);
          if (fragment != null && fragment.isInLayout()) {
            fragment.setText(link);
          }
    }
}
```

in order to keep the fragment as re-usable as possible.

## References

 * <http://developer.android.com/reference/android/app/Fragment.html>
 * <http://developer.android.com/training/basics/fragments/creating.html>
 * <http://developer.android.com/guide/components/fragments.html>
 * <http://www.vogella.com/articles/AndroidFragments/article.html>