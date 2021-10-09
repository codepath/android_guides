## Overview

The View Binding library makes it easy to reduce the need to use `findViewById()` lookups.  Once this option is enabled in your project, special binding classes will be generated from any of your layout XML files.  If you have a layout called `activity_simple.xml`, for instance, a special class called `ActivitySimpleBinding` will be generated that will have references automatically created for you.  Some major advantages are:
* improved null safety
* better type safety
* greater efficiency with complex views

NOTE: the View Binding library is separate from the [Data Binding Library](https://guides.codepath.com/android/Applying-Data-Binding-for-Views), which provides two-way and layout variables support.  

## Setup

Add the following to `app/build.gradle` file:

```gradle
android {
  buildFeatures {
        viewBinding true
    }
}
```

Once you have enabled this option, make sure to click `Rebuild Project`.  You will need to rebuild your entire project in order to make sure the view binding class are created.

## Activity View Lookups

Once you have enabled and recompiled your project, the binding classes should be available to you.  In this example, a `ActivitySimpleBinding` class should have been generated for a `activity_simple.xml` file.  Instead of using `setContentView(activity_simple.xml)`, we replace that line with a call with `ActivitySimpleBinding.inflate()`:

```java
class ExampleActivity extends AppCompatActivity {
  private TextView title;
  private ActivitySimpleBinding binding;

  @Override 
  public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    // activity_simple.xml -> ActivitySimpleBinding
    binding = ActivitySimpleBinding.inflate(getLayoutInflater());

    // layout of activity is stored in a special property called root
    View view = binding.getRoot();
    setContentView(view);

    // set bindings more efficiently through bindings
    title = binding.title;       // was title = findViewById(R.id.title);
    title.setText("My title");

    // alternately, access views through binding when needed, instead of variables
    binding.title.setText("My title");

  }
}
```

```kotlin
class ExampleActivity : AppCompatActivity() {
  private lateinit var title: TextView
  private lateinit var binding: ActivitySimpleBinding  

  override fun onCreate(savedInstanceState: Bundle?) {
      super.onCreate(savedInstanceState)
      
      // activity_simple.xml -> ActivitySimpleBinding
      binding = ActivitySimpleBinding.inflate(layoutInflater)
  
      // layout of activity is stored in a special property called root
      val view = binding.root
      setContentView(view)

      // set bindings more efficiently through bindings
      title = binding.title       // was title = findViewById(R.id.title)
      title.text = "My title"

      // alternately, access views through binding when needed, instead of variables
      binding.title.text = "My title"
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

  @Override 
  public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) { 
    
    // fancy_fragment.xml -> FancyFragmentBinding
    binding = FancyFragmentBinding.inflate(getLayoutInflater(), container, false);

    // layout of fragment is stored in a special property called root
    View view = binding.getRoot();
    			 
    // TODO Use fields...
    // binding.
    return view;
  }

  @Override 
  public void onDestroyView() {
    super.onDestroyView();
    binding = null;
  }
}
```

```kotlin
class FancyFragment : Fragment() {
  private lateinit var button1: Button
  private lateinit var button2: Button

  // Used to inflate and destroy the view
  private var _binding: FancyFragmentBinding? = null

  // Used throughout the class
  private val binding get() = _binding!!

  override fun onCreateView(
        inflater: LayoutInflater, container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View {
    
    // fancy_fragment.xml -> FancyFragmentBinding
    _binding = FancyFragmentBinding.inflate(getLayoutInflater(), container, false)

    // layout of fragment is stored in a special property called root
    View view = binding.getRoot()
    			 
    // TODO Use fields...
    // binding.
    return view
  }

  override fun onDestroyView() {
    super.onDestroyView()
    _binding = null
  }
}
```

## Troubleshooting

* If you are missing a view binding class even though it is imported, try to `Build` -> `Rebuild Project`.

## References

* [Android Jetpack: Replace findViewById with view binding](https://www.youtube.com/watch?v=W7uujFrljW0)
* [View Binding Guide from Google](https://developer.android.com/topic/libraries/view-binding)