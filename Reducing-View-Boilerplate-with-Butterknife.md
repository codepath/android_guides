## Overview

[Butterknife](http://jakewharton.github.io/butterknife/) is a popular View "injection" library for Android. This means that the library writes common boilerplate view code for you based on annotations to save you time and significantly reduce the lines of boilerplate code written. 

<img src="http://i.imgur.com/8QU88W8.png" width="250" />

This guide covers the most common usages of the library.

### Performance

Butterknife uses **compile-time annotations** which means there is no additional cost at run-time. Instead of slow reflection, code is generated ahead of time. Calling bind delegates to this generated code that you can see and debug. This means that **Butterknife does not slow down your app at all!**

### Compared to Dagger?

Butterknife is about reducing view boilerplate. [[Dagger 2|Dependency-Injection-with-Dagger-2]] is dependency injection for arbitrary components. Dagger is very flexible but is not intended for view injection. Think of Butter Knife as a means of binding views rather than injection. In other words, **Butterknife and Dagger 2 are complementary** and many projects include both for different purposes.

## Setup

Using Gradle, add the following line to `app/build.gradle` file:

```gradle
dependencies {
  compile 'com.jakewharton:butterknife:7.0.1'
}
```

Use [gradleplease](http://gradleplease.appspot.com/#butterknife) to get the latest version. See [this page](https://github.com/JakeWharton/butterknife/blob/master/README.md#download) for alternate installation methods.

## Usage

There are three major features of ButterKnife:

1. Improved View Lookups
2. Improved Listener Attachments
3. Improved Resource Lookups

### 1. Improved View Lookups 

### Activity View Lookups

Eliminate `findViewById` calls by using `@Bind` on fields:

```java
class ExampleActivity extends Activity {
  // Automatically finds each field by the specified ID.
  @Bind(R.id.title) TextView title;
  @Bind(R.id.subtitle) TextView subtitle;
  @Bind(R.id.footer) TextView footer;

  @Override public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.simple_activity);
    ButterKnife.bind(this);
    // TODO Use fields...
  }
}
```

### Fragment View Lookups

This can be done within `Activity`, `Fragment`, or `Adapter` classes. For example, fragment usage would look like:

```java
public class FancyFragment extends Fragment {
  @Bind(R.id.button1) Button button1;
  @Bind(R.id.button2) Button button2;

  @Override public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
    View view = inflater.inflate(R.layout.fancy_fragment, container, false);
    ButterKnife.bind(this, view);
    // TODO Use fields...
    return view;
  }

  // When binding a fragment in onCreateView, set the views to null in onDestroyView. 
  // Butter Knife has an unbind method to do this automatically.
  @Override public void onDestroyView() {
    super.onDestroyView();
    ButterKnife.unbind(this);
  }
}
```

### Adapter View Lookups

Within a `ViewHolder` inside of a `ListView` adapter:

```java
public class MyAdapter extends BaseAdapter {
  @Override public View getView(int position, View view, ViewGroup parent) {
    ViewHolder holder;
    if (view != null) {
      holder = (ViewHolder) view.getTag();
    } else {
      view = inflater.inflate(R.layout.whatever, parent, false);
      holder = new ViewHolder(view);
      view.setTag(holder);
    }
    holder.name.setText("John Doe");
    // etc...
    return view;
  }

  static class ViewHolder {
    @Bind(R.id.title) TextView name;
    @Bind(R.id.job_title) TextView jobTitle;

    public ViewHolder(View view) {
      ButterKnife.bind(this, view);
    }
  }
}
```

This will save you the need to ever write `findViewById` ever again!

### 2. Improved Listener Attachments

Eliminate anonymous inner-classes for listeners by annotating methods with `@OnClick` and others:

```java
@OnClick(R.id.submit)
public void sayHi(Button button) {
  button.setText("Hello!");
}
```

We can attach multiple views to the same listener with:

```java
@OnClick({ R.id.door1, R.id.door2, R.id.door3 })
public void pickDoor(DoorView door) {
  if (door.hasPrizeBehind()) {
    Toast.makeText(this, "You win!", LENGTH_SHORT).show();
  } else {
    Toast.makeText(this, "Try again", LENGTH_SHORT).show();
  }
}
```

The following event listeners are supported out of the box: `OnClick`, `OnLongClick`, `OnEditorAction`, `OnFocusChange`, `OnItemClick`, `OnItemLongClick`,`OnItemSelected`, `OnPageChange`, `OnTextChanged`, `OnTouch`, `OnCheckedChanged`.

### 3. Improved Resource Lookups

Eliminate resource lookups in your Java code by using resource annotations on fields:

```java
class ExampleActivity extends Activity {
  @BindString(R.string.title) String title;
  @BindDrawable(R.drawable.graphic) Drawable graphic;
  @BindColor(R.color.red) int red; // int or ColorStateList field
  @BindDimen(R.dimen.spacer) Float spacer; // int (for pixel size) or float (for exact value) field
  // ...
}
```

The following resource types are available: `BindArray`, `BindBitmap`, `BindBool`,`BindColor`,`BindDimen`,`BindDrawable`,`BindInt`,`BindString`.

## Advanced Usage

There are two advanced features:

1. Acting on Multiple Views In a List
2. Type Inference for View Lookups

### 1. Acting on Multiple Views In a List

You can group multiple views into a List and perform actions on them as group:

```
// Group the views together
@Bind({ R.id.first_name, R.id.middle_name, R.id.last_name })
List<EditText> nameViews;
```

The `apply` method allows you to act on all the views in a list at once:

```java
ButterKnife.apply(nameViews, DISABLE);
ButterKnife.apply(nameViews, ENABLED, false);
```

This requires writing `Action` or `Setter` interfaces allow specifying the action to perform:

```java
static final ButterKnife.Action<View> DISABLE = new ButterKnife.Action<View>() {
  @Override public void apply(View view, int index) {
    view.setEnabled(false);
  }
};

static final ButterKnife.Setter<View, Boolean> ENABLED = new ButterKnife.Setter<View, Boolean>() {
  @Override public void set(View view, Boolean value, int index) {
    view.setEnabled(value);
  }
};
```

An Android property can also be used with the `apply` method.

```java
ButterKnife.apply(nameViews, View.ALPHA, 0.0f);
```

### 2. Type Inference for View Lookups

Included are `findById` methods which simplify code for view lookups. It uses generics to infer the return type:

```java
TextView firstName = ButterKnife.findById(view, R.id.first_name);
TextView lastName = ButterKnife.findById(view, R.id.last_name);
ImageView photo = ButterKnife.findById(view, R.id.photo);
```

Add a static import for `ButterKnife.findById` and enjoy even more fun.

## Workarounds

If you are using the [[Navigation Drawer|Fragment-Navigation-Drawer]] from the latest version of the [[support library|Design-Support-Library]], you cannot use `@InjectView` on elements defined in the header layout because a RecyclerView is used instead of ListView in the newer versions, causing the header not be available immediately when the view is first created.  To get a reference, you need to first get a reference to the header view and use the `ButterKnife.findById` call once a reference to the header is obtained:

```java
View headerView = navigationView.getHeaderView(0);
TextView textView = ButterKnife.findById(headerView, R.id.tvName);
```

## Limitations

* ButterKnife cannot be used when creating your own Android libraries.  This limitation is described in [[this section|Building-your-own-Android-library#using-with-butterknife]] of the guide.

## References

* <http://jakewharton.github.io/butterknife/>