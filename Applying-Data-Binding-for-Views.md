## Overview

Android has now released a stable data-binding library which allows you to connect views with data in a much more powerful way than was possible previously. Applying data binding can improve your app by removing boilerplate for data-driven UI and allowing for two-way binding between views and data objects. 

The Data Binding Library is a support library that is compatible with all recent Android versions. See [this official video from Google](https://www.youtube.com/watch?v=5sCQjeGoE7M) for a brief overview.
 
### Setup

To get started with data binding, we need to make sure to upgrade to the latest version of the [[Android Gradle plugin|Getting-Started-with-Gradle#upgrading-gradle]].

To configure your app to use data binding, add the `dataBinding` element to your `app/build.gradle` file:

```gradle
apply plugin: 'com.android.application'

android {
    // Previously there
    // Add this next line
    dataBinding.enabled = true // <----
    // ...
```

### Eliminating View Lookups

The most basic thing we get with data binding is the elimination of `findViewById`. To enable this to work for a layout file, first we need to change the layout file by making the outer tag `<layout>` instead of whatever ViewGroup you use (note that the XML namespaces should also be moved):

```xml
<layout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:tools="http://schemas.android.com/tools">
    <RelativeLayout
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:paddingLeft="@dimen/activity_horizontal_margin"
            android:paddingRight="@dimen/activity_horizontal_margin"
            android:paddingTop="@dimen/activity_vertical_margin"
            android:paddingBottom="@dimen/activity_vertical_margin"
            tools:context=".MainActivity">

        <TextView
                android:id="@+id/tvLabel"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"/>

    </RelativeLayout>
</layout>
```

The `layout` container tag tells Android Studio that this layout should take the extra processing during compilation time to find all the interesting Views and note them for use with the data binding library.  

Essentially creating this outer `layout` tag causes a special reserved class file to be generated at compile time based on the name of the file.  For instance, `activity_main.xml` will generate a class called `ActivityMainBinding`.  Although this class is not generated at compile-time, it can still be referenced in Android Studio thanks to [integrated support](https://developer.android.com/topic/libraries/data-binding/index.html#studio_support) for data binding.
  
Now, we can use the `Binding` object to access the view.  In our activity, we can now inflate the layout content using the binding:
 
```java
public class MainActivity extends AppCompatActivity {
    // Store the binding
    private ActivityMainBinding binding;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        // Inflate the content view (replacing `setContentView`)
        binding = DataBindingUtil.setContentView(this, R.layout.activity_main);
        // Store the field now if you'd like without any need for casting
        TextView tvLabel = binding.tvLabel;
        tvLabel.setAllCaps(true);
        // Or use the binding to update views directly on the binding
        binding.tvLabel.setText("Foo");
    }
}
```

Note that this binding object is generated automatically with the following rules in place:

 * Layout file `activity_main.xml` becomes `ActivityMainBinding` binding object
 * View IDs within a layout become variables inside the binding object (i.e `binding.tvLabel`)

The binding process makes a single pass on all views in the layout to assign the views to the fields. Since this only happens once, this can actually be faster than calling `findViewById`. 

Refer to [this guide for more details about bindings and inflating views](https://medium.com/google-developers/no-more-findviewbyid-457457644885#.p1ie9j52a).

### One Way Data Binding

The simplest binding is to automatically load data from an object into a view directly using a new syntax in the template. For example, suppose we want to display a user's name inside a `TextView`. Assuming a simple user class:

```java
public class User {
   public String firstName;
   public String lastName;
}
```

Next, we need to wrap our existing layout inside a `<layout>` tag to indicate we want data binding enabled:

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">
    <RelativeLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:paddingLeft="@dimen/activity_horizontal_margin"
        android:paddingRight="@dimen/activity_horizontal_margin"
        android:paddingTop="@dimen/activity_vertical_margin"
        android:paddingBottom="@dimen/activity_vertical_margin"
        tools:context=".MainActivity">

        <TextView
            android:id="@+id/tvFullName"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Hello World" />

    </RelativeLayout>
</layout>
```

Next, we need to indicate that we want to load data from a particular object by declaring `variable` nodes in a `data` section of the `<layout>`:

```xml
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">
    <data>
       <variable name="user" type="com.example.User"/>
    </data>
    <!-- ... rest of layout here -->
</layout>
```

The `user` variable within the data block describes a property that can now be used within this layout. Next, we can now reference this object inside of our views using `@{variable.field}` syntax such as:

```xml
<TextView
    android:id="@+id/tvFullName"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text='@{user.firstName + " " + user.lastName}' />
```

We can use conditional logic and other operations as part of the [binding expression language](https://developer.android.com/topic/libraries/data-binding/index.html#expression_language) for more complex cases. Finally, we need to assign that user data variable to the binding at runtime:

```java
public class MainActivity extends AppCompatActivity {
    private ActivityMainBinding binding;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        // Inflate the `activity_main` layout
        binding = DataBindingUtil.setContentView(this, R.layout.activity_main);
        // Create or access the data to bind
        User user = new User("Sarah", "Gibbons");
        // Attach the user to the binding
        binding.setUser(user);
    }
}
```

That's all! When running the app now, you'll see the name is populated into the layout automatically:

<img src="https://i.imgur.com/Q8kscZQ.png" width="400" />

#### Controlling visibility

You can use binding expressions to control the visibility of an item.  

First, make sure you have access to the View properties (i.e. `View.INVISIBLE` and `View.GONE`) by importing
the object into the template.

```xml
<data>
   <import type="android.view.View" />
</data>
```

Next, you can test for a condition (i.e. first name is null) and determine whether to show or hide the text view  accordingly:

```xml
<TextView
   android:visibility="@{user.firstName != null ? View.VISIBLE: View.GONE}">

</TextView>
```

#### Image Loading

Unlike TextViews, you cannot bind values directly.  In this case, you need to create a custom  attribute.

First, make sure to define the `res-auto` namespace in your layout.  You cannot declare it in the `<layout>` outer tag so should put it on the outermost tag after the `<data>` tag:

```xml
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">
  
  <data>

  </data>

  <RelativeLayout xmlns:app="http://schemas.android.com/apk/res-auto">
  </RelativeLayout>
```

and:

```xml
<ImageView app:imageUrl=“@{mymodel.imageUrl}”>
```

You then need to annotate a static method that maps the custom attribute:

```java
public class BindingAdapterUtils {
  @BindingAdapter({"bind:imageUrl"})
  public static void loadImage(ImageView view, String url) {
     Picasso.with(view.getContext()).load(url).into(view); 
  }
}
```

### Event Handling

Data Binding allows you to write expressions handling events that are dispatched from the views (e.g. onClick). To attach events to views, we can use [method references](https://developer.android.com/topic/libraries/data-binding/index.html#method_references) or [listener bindings](https://developer.android.com/topic/libraries/data-binding/index.html#listener_bindings).

### Using Data Binding inside RecyclerView 

We first need to modify the ViewHolder class to include a reference to the data binding class:

```java
public class SamplesViewHolder extends RecyclerView.ViewHolder {
  final ItemUserBinding binding;  // this will be used by onBindViewHolder()

  public ItemViewHolder(View rootView) {
     super(rootView);
   
     // Since the layout was already inflated within onCreateViewHolder(), we 
     // can use this bind() method to associate the layout variables
     // with the layout.
     binding = ItemUserBinding.bind(rootView);

  }
}
```

Next, we modify the `onBindViewHolder()` to associate the User object with the user at the given position and then update the views with the newly bound references:

```java
@Override 
   public void onBindViewHolder(BindingHolder holder, int position) { 
      final User user = users.get(position); 

      // add these lines
      holder.binding.setUser(user);  // setVariable(BR.user, user) would also work
      holder.binding.executePendingBindings();   // update the view now
   } 
```

Refer also to these tutorials for how to work with data binding in `RecyclerView` or `ListView`:

 * [Databinding within RecyclerView](https://medium.com/google-developers/android-data-binding-recyclerview-db7c40d9f0e4)
 * [Using data binding in RecyclerView](http://mutualmobile.com/posts/using-data-binding-api-in-recyclerview).
 * [RecyclerView and Data Binding](https://www.jayway.com/2015/12/08/recyclerview-and-databinding/)
 * [Simple code for data binding in the RecyclerView](https://newfivefour.com/android-databinding-recyclerview.html)
 * [Descent into data binding tutorial](https://www.bignerdranch.com/blog/descent-into-databinding/)

### Data Binding and the Include Tag

Refer to the following resources related to the include tag and binding:

 * [Includes tag with binding](https://developer.android.com/topic/libraries/data-binding/index.html#includes)
 * [Data binding with include tag](https://medium.com/google-developers/android-data-binding-that-include-thing-1c8791dd6038#.ptysrnqqo)
 * [Imports and Includes with Data Binding](http://mobikul.com/imports-includes-data-binding/)

### Two Way Data Binding

If you want to have a two-way binding between the view and the data source, check out this [handy 2-way data binding tutorial](https://medium.com/google-developers/android-data-binding-2-way-your-way-ccac20f6313). You can also check [this reference on 2-way data binding](https://medium.com/google-developers/android-data-binding-2-way-your-way-ccac20f6313) and [this related post on inverse functions](https://medium.com/google-developers/android-data-binding-inverse-functions-95aab4b11873).

### Advanced Data Binding

There is a great [series of posts](https://medium.com/@georgemount007) outlining advanced data binding features with the most important highlighted below:

 * [Animating UI Updates with Data Binding](https://medium.com/google-developers/android-data-binding-animations-55f6b5956a64)
 * [Dynamic List Tricks and Data Binding](https://medium.com/google-developers/android-data-binding-list-tricks-ef3d5630555e)
 * [Databinding Dependent Properties](https://medium.com/google-developers/android-data-binding-dependent-properties-516d3235cd7c)

## Databinding Best Practices

While MVVM with Data Binding does remove a good deal of this boilerplate you also run into new issues where logic is now present in the views, like this code similar to the section shown above:

```xml
<TextView 
    ...
    android:visibility="@{post.hasComments ? View.Visible : View.Gone}" />
```

This now buries the logic in an Android XML view and its near impossible to test. Instead, we should use a utility attribute:

```java
public class BindingAdapterUtils {
  @BindingAdapter({"isVisible"})
  public static void setIsVisible(View view, boolean isVisible) {
      if (isVislble) {
        view.visibility = View.VISIBLE
      } else {
        view.visibility = View.GONE
      }
  }
}
```
```kotlin
@BindingAdapter("isVisible")
fun setIsVisible(view: View, isVisible: Boolean) {
  if (isVislble) {
    view.visibility = View.VISIBLE
  } else {
    view.visibility = View.GONE
  }
}
```

The logic for showing a view is now determined by a Boolean value. To use this in an MVVM Data Binding XML View you’d do the following in your view:

```xml
<TextView 
    ...
    app:isVisible="@{post.hasComments()}" />
```

For more best practices, check out this [source article here](http://www.donnfelker.com/android-mvvm-with-databinding-removing-logic-from-your-views-with-bindingadapters/). 

## MVVM Architecture and Data Binding 

The Data binding framework can be used in conjunction with an MVVM architecture to help to decouple the View from the Model. In this approach, the binding framework connects with the ViewModel, which exposes data from the Model, into the View (xml layout). 

[<img src="https://i.imgur.com/Beq1Zry.png" width="700" />](https://labs.ribot.co.uk/approaching-android-with-mvvm-8ceec02d5442)

Check out [this blog post](https://labs.ribot.co.uk/approaching-android-with-mvvm-8ceec02d5442) for a detailed overview. 

## Troubleshooting

**Issues with the Binding Class**

If you see an error message such as `cannot resolve symbol 'ActivityMainBinding'` then this means that the data binding auto-generated class has not been created. Check the following to resolve the issue:

 1. Make sure you have the proper `dataBinding.enabled = true` in gradle and trigger "Sync with Gradle"
 2. Open the layout file and ensure that the XML file is valid and is wrapped in a `<layout>` tag.
 3. Check the **layout file for the correct name** i.e `activity_main.xml` maps to `ActivityMainBinding.java`.
 4. Run `File => Invalidate Caches / Restart` to clear the caches. 
 4. Run `Project => Clean` and `Project => Re-Build` to regenerate the class file.
 5. Restart Android Studio again and then try the above steps again.
 
**Databinding does not exist messages**

If you see an error message such as `**.**.databinding does not exist`, it's likely that there is an error in your data binding XML templates that needs to be resolved.  Make sure to look for errors (i.e. forgetting to import a Java class when referencing it within your template). 

Make sure that **all your XML files with data binding have their errors fully resolved** and the files are saved. Then be sure to `Project -> Clean` and `Project -> Rebuild Project` the project in order to regenerate the binding classes with latest properties. 

**Databinding is not picking up certain layout fields**

Be sure to `Project -> Clean` and `Project -> Rebuild Project` the project in order to regenerate the binding classes with latest layout properties. 

**Identifiers must have user defined types from the XML file. View is missing it**

This usually means that you have a missing import statement or a mismatch between the type of variable in the XML and the actual object of the class passed in while binding. Check out these possible posts for solutions: 

 * <https://stackoverflow.com/questions/32068675/data-binding-expression-not-compiling>
 * <https://stackoverflow.com/q/41609252/313399>
 * <https://stackoverflow.com/q/36229529/313399>
 * <https://stackoverflow.com/a/36938816/313399>
 * <https://stackoverflow.com/questions/31980342/android-data-binding-style>

Please refer to these and carefully check for missing imports or type mismatches. Also try `Build -> Clean -> Rebuild Project` to recompile the binding classes as well to see if this helps. 

**Dagger 2 compatibility errors**

If you are using the data binding library with [[Dagger 2|Dependency Injection With Dagger 2]], you may see errors such as `NoSuchMethodError: ...FluentIterable.append(...)`.  The solution to this fix is to add the Guava library before the Dagger compiler.  There appears to be a known [issue](https://code.google.com/p/android/issues/detail?id=205589) that can only be resolved by forcing the Dagger compiler to use a newer Guava version (Dagger 2 appears to have an older version of Guava bundled and without an explicit dependency it uses this version).

```gradle
apt 'com.google.guava:guava:19.0' // add above dagger-compiler
apt 'com.google.dagger:dagger-compiler:2.5' 
```

## References

* <https://developer.android.com/topic/libraries/data-binding/index.html>
* <https://medium.com/google-developers/no-more-findviewbyid-457457644885#.p1ie9j52a>
* <https://realm.io/news/data-binding-android-boyar-mount/>
* <https://www.captechconsulting.com/blogs/android-data-binding-tutorial>
* <http://www.survivingwithandroid.com/2015/08/android-data-binding-tutorial-2.html>
* <https://www.bignerdranch.com/blog/descent-into-databinding/>
* <https://stfalcon.com/en/blog/post/faster-android-apps-with-databinding>
