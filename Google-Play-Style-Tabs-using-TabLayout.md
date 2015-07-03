Tabs are now best implemented by leveraging the [[ViewPager|ViewPager-with-FragmentPagerAdapter]] with a custom "tab indicator" on top. In this guide, we will be using Google's new [TabLayout](https://developer.android.com/reference/android/support/design/widget/TabLayout.html) included in the support design library release for Android "M".

Prior to Android "M", the easiest way to setup tabs with Fragments was to use ActionBar Tabs as described in [ActionBar Tabs with Fragments](http://guides.codepath.com/android/ActionBar-Tabs-with-Fragments) guide. However, all methods related to navigation modes in the ActionBar class (such as `setNavigationMode()`, `addTab()`, `selectTab()`, etc.) are now deprecated.

### Design Support Library

To implement Google Play style sliding tabs, make sure to follow the [[Design Support Library]] setup instructions first.

### Sliding Tabs Layout

Simply add `android.support.design.widget.TabLayout`, which will be used for rendering the different tab options.  The `android.support.v4.view.ViewPager` component will be used to page between the various fragments we will create.

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <android.support.design.widget.TabLayout
        android:id="@+id/sliding_tabs"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        app:tabMode="scrollable" />

    <android.support.v4.view.ViewPager
        android:id="@+id/viewpager"
        android:layout_width="match_parent"
        android:layout_height="0px"
        android:layout_weight="1"
        android:background="@android:color/white" />

</LinearLayout>
```

### Create Fragment

Now that we have the `ViewPager` and our tabs in our layout, we should start defining the content of each of the tabs. Since each tab is just a fragment being displayed, we need to create and define the `Fragment` to be shown. You may have one or more fragments in your application depending on your requirements.

In `res/layout/fragment_page.xml` define the XML layout for the fragment which will be displayed on screen when a particular tab is selected:

```xml
<?xml version="1.0" encoding="utf-8"?>
<TextView xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:gravity="center" />
```

In `PageFragment.java` define the inflation logic for the fragment of tab content:

```java
// In this case, the fragment displays simple text based on the page
public class PageFragment extends Fragment {
    public static final String ARG_PAGE = "ARG_PAGE";

    private int mPage;

    public static PageFragment newInstance(int page) {
        Bundle args = new Bundle();
        args.putInt(ARG_PAGE, page);
        PageFragment fragment = new PageFragment();
        fragment.setArguments(args);
        return fragment;
    }

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        mPage = getArguments().getInt(ARG_PAGE);
    }

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.fragment_page, container, false);
        TextView textView = (TextView) view;
        textView.setText("Fragment #" + mPage);
        return view;
    }
}
```

### Implement FragmentPagerAdapter

The next thing to do is to implement the adapter for your `ViewPager` which controls the order of the tabs, the titles and their associated content. The most important methods to implement here are `getPageTitle(int position)` which is used to get the title for each tab and `getItem(int position)` which determines the fragment for each tab.

```java
public class SampleFragmentPagerAdapter extends FragmentPagerAdapter {
    final int PAGE_COUNT = 3;
    private String tabTitles[] = new String[] { "Tab1", "Tab2", "Tab3" };
    private Context context;

    public SampleFragmentPagerAdapter(FragmentManager fm, Context context) {
        super(fm);
        this.context = context;
    }

    @Override
    public int getCount() {
        return PAGE_COUNT;
    }

    @Override
    public Fragment getItem(int position) {
        return PageFragment.newInstance(position + 1);
    }

    @Override
    public CharSequence getPageTitle(int position) {
        // Generate title based on item position
        return tabTitles[position];
    }
}
```

### Setup Sliding Tabs

Finally, we need to attach our `ViewPager` to the `SampleFragmentPagerAdapter` and then configure the sliding tabs with a two step process:

* In the `onCreate()` method of your activity, find the `ViewPager` and connect the adapter.
* Set the `ViewPager` on the `TabLayout` to connect the pager with the tabs.

```java
public class MainActivity extends FragmentActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        // Get the ViewPager and set it's PagerAdapter so that it can display items
        ViewPager viewPager = (ViewPager) findViewById(R.id.viewpager);
        viewPager.setAdapter(new SampleFragmentPagerAdapter(getSupportFragmentManager(), 
            MainActivity.this));

        // Give the TabLayout the ViewPager
        TabLayout tabLayout = (TabLayout) findViewById(R.id.sliding_tabs);
        tabLayout.setupWithViewPager(viewPager);
    }

}
``` 

Heres the output:

![Screen 1](http://i.imgur.com/rhRXjLIl.png)

### Customize Tab Indicator Color

Normally, the tab indicator color chosen is the [accent color](http://www.google.com/design/spec/style/color.html#color-color-palette) defined for your Material Design theme.  If you intend to override this color, you will need to create a style that overrides this color.

```xml
<style name="MyCustomTabLayout" parent="Widget.Design.TabLayout">
        <item name="tabIndicatorColor">#0000FF</item>
</style>
```

You can then override this style for your TabLayout:

```xml
<android.support.design.widget.TabLayout
        android:id="@+id/tabs"
        style="@style/MyCustomTabLayout"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"></android.support.design.widget.TabLayout>
```

### Add Icons to TabLayout

Currently, the TabLayout class does not provide a clean abstraction model that allows for icons in your tab.  There are many posted workarounds, one of which is to return a `SpannableString`, containing your icon in an `ImageSpan`, from your PagerAdapter's `getPageTitle(position)` method as shown in the code snippet below:

```java
private int[] imageResId = {
        R.drawable.ic_one,
        R.drawable.ic_two,
        R.drawable.ic_three
};

// ...

@Override
public CharSequence getPageTitle(int position) {
    // Generate title based on item position
    // return tabTitles[position];
    Drawable image = context.getResources().getDrawable(imageResId[position]);
    image.setBounds(0, 0, image.getIntrinsicWidth(), image.getIntrinsicHeight());
    SpannableString sb = new SpannableString(" ");
    ImageSpan imageSpan = new ImageSpan(image, ImageSpan.ALIGN_BOTTOM);
    sb.setSpan(imageSpan, 0, 1, Spannable.SPAN_EXCLUSIVE_EXCLUSIVE);
    return sb;
}
```

By default, the tab created by TabLayout sets the `textAllCaps` property to be true, which prevents ImageSpans from being rendered.  You can override this behavior by changing the `tabTextAppearance` property.

```xml
  <style name="MyCustomTabLayout" parent="Widget.Design.TabLayout">
        <item name="tabTextAppearance">@style/MyCustomTextAppearance</item>
  </style>

  <style name="MyCustomTextAppearance" parent="TextAppearance.Design.Tab">
        <item name="textAllCaps">false</item>
  </style>
```

Sliding tabs with images:

![Slide 2](http://i.imgur.com/dYvY5NKl.jpg)

### Add Icons+Text to TabLayout

Since we are using `SpannableString` to add icons to `TabLayout`, it becomes easy to have text next to the icons by manipulating the `SpannableString` object.

```java
@Override
public CharSequence getPageTitle(int position) {
    // Generate title based on item position
    Drawable image = context.getResources().getDrawable(imageResId[position]);
    image.setBounds(0, 0, image.getIntrinsicWidth(), image.getIntrinsicHeight());
    // Replace blank spaces with image icon
    SpannableString sb = new SpannableString("   " + tabTitles[position]);
    ImageSpan imageSpan = new ImageSpan(image, ImageSpan.ALIGN_BOTTOM);
    sb.setSpan(imageSpan, 0, 1, Spannable.SPAN_EXCLUSIVE_EXCLUSIVE);
    return sb;
}
```

Note the additional spaces that are added before the tab title while instantiating `SpannableString` class. The blank spaces are used to place the image icon so that the actual title is displayed completely. Depending on where you want to position your icon, you can specify the range start…end of the span in `setSpan()` method.

![Slide 3](http://i.imgur.com/A8xEpKsl.jpg)

### Add Custom View to TabLayout

In certain cases, instead of the default tab view we may want to apply a custom XML layout for each tab. To achieve this, iterate over all the `TabLayout.Tab`s after attaching the sliding tabs to the pager:

```java
public class MainActivity extends FragmentActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        // Get the ViewPager and set it's PagerAdapter so that it can display items
        ViewPager viewPager = (ViewPager) findViewById(R.id.viewpager);
        SampleFragmentPagerAdapter pagerAdapter = 
            new SampleFragmentPagerAdapter(getSupportFragmentManager(), MainActivity.this);
        viewPager.setAdapter(pagerAdapter);

        // Give the TabLayout the ViewPager
        TabLayout tabLayout = (TabLayout) findViewById(R.id.sliding_tabs);
        tabLayout.setupWithViewPager(viewPager);

        // Iterate over all tabs and set the custom view
        for (int i = 0; i < tabLayout.getTabCount(); i++) {
            TabLayout.Tab tab = tabLayout.getTabAt(i);
            tab.setCustomView(pagerAdapter.getTabView(i));
        }
    }

    //...
}
```

Next, we add the `getTabView(position)` method to the `SampleFragmentPagerAdapter` class:

```java
public class SampleFragmentPagerAdapter extends FragmentPagerAdapter {
   private String tabTitles[] = new String[] { "Tab1", "Tab2" };
   private int[] imageResId = { R.drawable.ic_one, R.drawable.ic_two };

    public View getTabView(int position) {
        // Given you have a custom layout in `res/layout/custom_tab.xml` with a TextView and ImageView
        View v = LayoutInflater.from(context).inflate(R.layout.custom_tab, null);
        TextView tv = (TextView) v.findViewById(R.id.textView);
        tv.setText(tabTitles[position]);
        ImageView img = (ImageView) v.findViewById(R.id.imgView);
        img.setImageResource(imageResId[position]);
        return v;
    }

}
```

With this you can setup any custom tab content for each page in the adapter. 

## References

* <https://android.googlesource.com/platform/frameworks/support.git/+/master/design/src/android/support/design/widget/TabLayout.java>
* <https://android.googlesource.com/platform/frameworks/support.git/+/master/design/res/values/styles.xml>