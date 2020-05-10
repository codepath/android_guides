## Overview

The View Binding library makes it easy to reduce the need to use `findViewById`() lookups.  Once this option is enabled in your project, special binding classes will be generated from any of your layout XML files.  If you have a layout called `simple_activity.xml` for instance, a special class called `SimpleActivityBinding` will be generated that will have references automatically created for you.  One major advantage is that using this class helps to avoid null pointer issues and provides type safety support, making it less likely your app will crash.

NOTE: the View Binding library is separate from the [[Applying-Data-Binding-for-Views|Data Binding Library]], which provides two-way and layout variables support.  

## Setup

Add the following to `app/build.gradle` file:

```gradle
android {
  viewBinding {
        enabled = true
    }
}
```

Confirm you have updated to Android Studio v3.6 by going to `Android Studio` -> `About Android Studio`.

Once you have enabled this option, make sure to click `Rebuild Project`.  You will need to rebuild your entire project in order to make sure the view binding class are created.

## Activity View Lookups

Once you have enabled and recompiled your project, the binding classes should be available to you.  In this example, a `SimpleActivityBinding` class should have been generated for a `simple_activity.xml` file.  Instead of using `setContentView(simple_activity.xml)`, we replace that line with a call with `SimpleActivityBinding.inflate()`:

```java
class ExampleActivity extends Activity {
  // Automatically finds each field by the specified ID.
  TextView title;
  TextView subtitle;
  TextView footer;

  @Override public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    // simple_activity.xml -> SimpleActivityBinding
    SimpleActivityBinding binding = SimpleActivityBinding.inflate(getLayoutInflater());

    // layout of activity is stored in a special property called root
    View view = binding.getRoot();
    setContentView(view);

    // TODO Use fields (binding.title, binding.subtitle, binding.footer, etc.)
  }
}
```

#### Fragment View Lookups

This can be done within `Activity`, `Fragment`, or `Adapter` classes. For example, fragment usage would look like:

```java
public class FancyFragment extends Fragment {
  Button button1;
  Button button2;

  FancyFragmentBinding binding;

  @Override public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) { 
    
    // fancy_fragment.xml -> FancyFragmentBinding
    binding = FancyFragmentBinding.inflate(getLayoutInflater(), container, false);

    // layout of fragment is stored in a special property called root
    View view = binding.getRoot();
    			 
    // TODO Use fields...
    // binding.
    return view;
  }

  @Override public void onDestroyView() {
    super.onDestroyView();
    binding = null;
  }
}
```

## Troubleshooting

* If you are missing a view binding class even though it is imported, try to `Build` -> `Rebuild Project`.

## References

* [https://www.youtube.com/watch?v=W7uujFrljW0](Android Jetpack: Replace findViewById with view binding)
* [https://developer.android.com/topic/libraries/view-binding?hl=ca](View Binding Guide from Google)