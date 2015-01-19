Prior to Android "L" preview, the easiest way to setup tabs with Fragments was to use ActionBar Tabs as described in [ActionBar Tabs with Fragments](http://guides.codepath.com/android/ActionBar-Tabs-with-Fragments) guide. However, all methods related to navigation modes in the ActionBar class (such as setNavigationMode(), addTab(), selectTab() etc.) are now deprecated.

As a result, tabs are now best implemented by leveraging the [[ViewPager|ViewPager-with-FragmentPagerAdapter]] with a custom "tab indicator" on top.

### SlidingTabs Layout

To implement Google Play style sliding tabs, you first need to copy the following two java source files into your application. Please make sure to copy the files directly from the Google IO links specified below. 
  
* [SlidingTabLayout.java](https://github.com/google/iosched/blob/0a90bf8e6b90e9226f8c15b34eb7b1e4bf6d632e/android/src/main/java/com/google/samples/apps/iosched/ui/widget/SlidingTabLayout.java)
* [SlidingTabStrip.java](https://github.com/google/iosched/blob/0a90bf8e6b90e9226f8c15b34eb7b1e4bf6d632e/android/src/main/java/com/google/samples/apps/iosched/ui/widget/SlidingTabStrip.java)

You may choose to move them to a suitable package in your project. Once you have included `SlidingTabLayout.java` and `SlidingTabStrip.java` files within your app, you can use the `SlidingTabLayout` in your layout file to display tabs. Your layout file will have tabs on the top and a `ViewPager` on the bottom as shown in the code snippet below:

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <com.example.android.slidingtabsexample.SlidingTabLayout
        android:id="@+id/sliding_tabs"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />

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

In `res/layout/fragment_page.xml` define the XML layout for the fragment:

```xml
<?xml version="1.0" encoding="utf-8"?>
<TextView xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:gravity="center" />
```

In `PageFragment.java` define the inflation logic for the fragment:

```java
// In this case, the fragment displays simple text based on the page
public class PageFragment extends Fragment {
    public static final String ARG_PAGE = "ARG_PAGE";

    private int mPage;

    public static PageFragment create(int page) {
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

### Setup Sliding Tabs

Next, we need to attach our `ViewPager` to the 
Setting up the sliding tabs is a two step process:

* In the `onCreate()` method of your activity, find the `ViewPager` and set its adapter.
* Set the `ViewPager` on the `SlidingTabLayout`.

```java
public class MainActivity extends FragmentActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        // Get the ViewPager and set it's PagerAdapter so that it can display items
        ViewPager viewPager = (ViewPager) findViewById(R.id.viewpager);
        viewPager.setAdapter(new SampleFragmentPagerAdapter(getSupportFragmentManager()));

        // Give the SlidingTabLayout the ViewPager
        SlidingTabLayout slidingTabLayout = (SlidingTabLayout) findViewById(R.id.sliding_tabs);
        // Center the tabs in the layout
        slidingTabLayout.setDistributeEvenly(true);
        slidingTabLayout.setViewPager(viewPager);
    }

}
```

Please note that your `SlidingTabLayout.java` may not have `setDistributeEvenly()` method is you are using the old version. To solve that update your view files to the latest version as mentioned above. 

### Implement FragmentPagerAdapter

The last thing to do is to implement the adapter for your `ViewPager`. The most important method to implement here is `getPageTitle()` which is used to get the title for each tab.

```java
public class SampleFragmentPagerAdapter extends FragmentPagerAdapter {
    final int PAGE_COUNT = 3;
    private String tabTitles[] = new String[] { "Tab1", "Tab2", "Tab3" };

    public SampleFragmentPagerAdapter(FragmentManager fm) {
        super(fm);
    }

    @Override
    public int getCount() {
        return PAGE_COUNT;
    }

    @Override
    public Fragment getItem(int position) {
        return PageFragment.create(position + 1);
    }

    @Override
    public CharSequence getPageTitle(int position) {
        // Generate title based on item position
        return tabTitles[position];
    }
}
```

Heres the output:

![Screen 1](http://i.imgur.com/rhRXjLIl.png)

### Customize Tab Indicator Color

You can use `setCustomTabColorizer()` to change the color of the tab indicator:

```java
// Customize tab color
slidingTabLayout.setCustomTabColorizer(new SlidingTabLayout.TabColorizer() {
    @Override
    public int getIndicatorColor(int position) {
        return Color.BLUE;
    }
});
```

### Add Icons to SlidingTabLayout

The key to add icons to your sliding tabs is to return a `SpannableString`, containing your icon in an `ImageSpan`, from your PagerAdapter's `getPageTitle(position)` method as shown in the code snippet below:

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

You'll also have to use a custom tab view since the default tab created by SlidingTabLayout makes a call to `textView.setAllCaps(true)` which effectively disables all ImageSpans.

res/layout/custom_tab.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<TextView xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:textSize="12sp"
    android:textStyle="bold"
    android:background="?android:selectableItemBackground"
    android:padding="16dp"
    android:gravity="center"
/>
```

To set your icon at the center of the tab, add `android:gravity="center"` attribute to your view.

Next, set the custom tab in the `onCreate()` of your `MainActivity.java` file by calling `setCustomTabView()` method on your `slidingTabLayout`. Make sure to call `setCustomTabView()` before `setViewPager()`.

```java
public class MainActivity extends ActionBarActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        // Get the ViewPager and set it's PagerAdapter so that it can display items
        ViewPager viewPager = (ViewPager) findViewById(R.id.viewpager);
        viewPager.setAdapter(new SampleFragmentPagerAdapter(getSupportFragmentManager(),
           MainActivity.this));

        // Give the SlidingTabLayout the ViewPager
        SlidingTabLayout slidingTabLayout = (SlidingTabLayout) findViewById(R.id.sliding_tabs);
        // Set custom tab layout
        slidingTabLayout.setCustomTabView(R.layout.custom_tab, 0);
        // Center the tabs in the layout
        slidingTabLayout.setDistributeEvenly(true);
        // Customize tab color
        slidingTabLayout.setCustomTabColorizer(new SlidingTabLayout.TabColorizer() {
            @Override
            public int getIndicatorColor(int position) {
                return Color.RED;
            }
        });
        slidingTabLayout.setViewPager(viewPager);
    }
}
```

Sliding tabs with images:

![Slide 2](http://i.imgur.com/dYvY5NKl.jpg)

## References

* [SlidingTabsBasic](http://developer.android.com/samples/SlidingTabsBasic/index.html) - Official google sample
* [SlidingTabsColors](http://developer.android.com/samples/SlidingTabsColors/index.html) - Google sample with colors