Tabs are now best implemented by leveraging the [[ViewPager|ViewPager-with-FragmentPagerAdapter]] with a custom "tab indicator" on top. In this guide, we will be using Google's new [TabLayout](https://developer.android.com/reference/android/support/design/widget/TabLayout.html) included in the support design library release for Android "M".

Prior to Android "M", the easiest way to setup tabs with Fragments was to use ActionBar Tabs as described in [[ActionBar Tabs with Fragments|ActionBar-Tabs-with-Fragments]] guide. However, all methods related to navigation modes in the ActionBar class (such as `setNavigationMode()`, `addTab()`, `selectTab()`, etc.) are now deprecated.

### Design Support Library

To implement Google Play style sliding tabs, make sure to follow the Design Support Library [[setup instructions|Design-Support-Library#setup]] first.

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
        app:tabMode="fixed" />

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
public class MainActivity extends AppCompatActivity {

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

![Screen 1](https://i.imgur.com/rhRXjLIl.png)

### Configuring the TabLayout

There are many attributes you can use to customize the behavior of your `TabLayout` as shown below:

```xml
<android.support.design.widget.TabLayout
    android:id="@+id/sliding_tabs"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    app:tabMaxWidth="0dp"
    app:tabGravity="fill"
    app:tabMode="fixed" />
```

The most important properties available are listed below:

| Name                 | Options           | Description |
| -----                | --------          | ----------- |
| `tabBackground`      | `@drawable/image` | Background applied to the tabs |
| `tabGravity`         | `center`, `fill`  | Gravity of the tabs |
| `tabIndicatorColor`  | `@color/blue`     | Color of the tab indicator line |
| `tabIndicatorHeight` | `@dimen/tabh`     | Height of the tab indicator line |
| `tabMaxWidth`        | `@dimen/tabmaxw`  | Maximum width of the tab |
| `tabMode`            | `fixed`, `scrollable` | Small number of fixed tabs or scrolling list |
| `tabTextColor`       | `@color/blue`     | Color of the text on the tab |

You can see all the [properties available here](https://developer.android.com/reference/android/support/design/widget/TabLayout.html#lattrs).

### Styling the TabLayout

Normally, the tab indicator color chosen is the [accent color](http://www.google.com/design/spec/style/color.html#color-color-palette) defined for your Material Design theme.  We can override this color by defining a custom style in `styles.xml` and then applying the style to your `TabLayout`:

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
        android:layout_height="wrap_content">
</android.support.design.widget.TabLayout>
```

There are several other styles that can be configured for the `TabLayout`:

```xml
<style name="MyCustomTabLayout" parent="Widget.Design.TabLayout">
    <item name="tabMaxWidth">@dimen/tab_max_width</item>
    <item name="tabIndicatorColor">?attr/colorAccent</item>
    <item name="tabIndicatorHeight">2dp</item>
    <item name="tabPaddingStart">12dp</item>
    <item name="tabPaddingEnd">12dp</item>
    <item name="tabBackground">?attr/selectableItemBackground</item>
    <item name="tabTextAppearance">@style/MyCustomTabTextAppearance</item>
    <item name="tabSelectedTextColor">?android:textColorPrimary</item>
</style>
<style name="MyCustomTabTextAppearance" parent="TextAppearance.Design.Tab">
    <item name="android:textSize">14sp</item>
    <item name="android:textColor">?android:textColorSecondary</item>
    <item name="textAllCaps">true</item>
</style>
```

### Add Icons to TabLayout

Currently, the TabLayout class does not provide a clean abstraction model that allows for icons in your tab.  However, you can manually add the icons after you have setup your TabLayout.

Inside your FragmentPagerAdapter, you can delete the `getPageTitle()` line or simply return null:

```java
// ...

@Override
public CharSequence getPageTitle(int position) {
    return null;
}
```

After you setup your TabLayout, you can use the `getTabAt()` function to set the icon:

```java
// setup TabLayout first

// configure icons
private int[] imageResId = {
        R.drawable.ic_one,
        R.drawable.ic_two,
        R.drawable.ic_three };

for (int i = 0; i < imageResId.length; i++) {
       tabLayout.getTabAt(i).setIcon(imageResId[i]);
}
```

Sliding tabs with images:

![Slide 2](https://i.imgur.com/dYvY5NKl.jpg)

We can also apply color tinting to the tabs by following [[this guide|Drawables#applying-tints-to-drawables]].

### Add Icons+Text to TabLayout

Another approach is to use `SpannableString` to add icons and text to `TabLayout`:

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

By default, the tab created by TabLayout sets the textAllCaps property to be true, which prevents ImageSpans from being rendered. You can override this behavior by changing the tabTextAppearance property.

```xml
<style name="MyCustomTabLayout" parent="Widget.Design.TabLayout">
     <item name="tabTextAppearance">@style/MyCustomTextAppearance</item>
</style>

<style name="MyCustomTextAppearance" parent="TextAppearance.Design.Tab">
     <item name="textAllCaps">false</item>
</style>
```

Note the additional spaces that are added before the tab title while instantiating `SpannableString` class. The blank spaces are used to place the image icon so that the actual title is displayed completely. Depending on where you want to position your icon, you can specify the range start…end of the span in `setSpan()` method.

![Slide 3](https://i.imgur.com/A8xEpKsl.jpg)

### Add Custom View to TabLayout

In certain cases, instead of the default tab view we may want to apply a custom XML layout for each tab. To achieve this, iterate over all the `TabLayout.Tab`s after attaching the sliding tabs to the pager:

```java
public class MainActivity extends AppCompatActivity {

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

### Getting or Selected the Current Page

With the [recent updates](https://plus.google.com/+AndroidDevelopers/posts/XTtNCPviwpj) to the design support library, you can also get the selected tab position by calling `getSelectedTabPosition()`.  If you need to save or restore the selected tab position during screen rotation or other configuration changes, this method is helpful for restoring the original tab position.

First, move your `tabLayout` and `viewPager` as member variables of your main activity:

```java
public class MainActivity extends AppCompatActivity {
    TabLayout tabLayout;
    ViewPager viewPager;
```

Next, we can save and restore the last known tab position by implementing methods on `onSaveInstanceState()` and `onRestoreInstanceState()` to persist this data.  When the view is recreated, we can use this data and set the current tab to the last selected tab value.

```java
    public static String POSITION = "POSITION";

    @Override
    public void onSaveInstanceState(Bundle outState) {
        super.onSaveInstanceState(outState);
        outState.putInt(POSITION, tabLayout.getSelectedTabPosition());
    }

    @Override
    protected void onRestoreInstanceState(Bundle savedInstanceState) {
        super.onRestoreInstanceState(savedInstanceState);
        viewPager.setCurrentItem(savedInstanceState.getInt(POSITION));
    }
```

## References

* <http://www.truiton.com/2015/06/android-tabs-example-fragments-viewpager/>
* <https://android.googlesource.com/platform/frameworks/support.git/+/master/design/src/android/support/design/widget/TabLayout.java>
* <https://android.googlesource.com/platform/frameworks/support.git/+/master/design/res/values/styles.xml>