## Overview

Layout that allows the user to swipe left and right through "pages" of content which are usually different fragments. This is a common navigation mode to use instead of [[ActionBar Tabs with Fragments]].

## Usage

### Layout ViewPager

A `ViewPager` is a layout which can be added to any layout XML file inside a root layout:

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">
 
    <android.support.v4.view.ViewPager
        android:id="@+id/vpPager"
        android:layout_width="match_parent"
        android:layout_height="wrap_content">
    </android.support.v4.view.ViewPager>
</LinearLayout>
```

### Define Fragments

Next, let's suppose we have defined two fragments `FirstFragment` and `SecondFragment` both of which contain a label in the layout and have implementations such as:

```java
public class FirstFragment extends Fragment {
	// Store instance variables
	private String title;
	private int page;

	// newInstance constructor for creating fragment with arguments
	public static FirstFragment newInstance(int page, String title) {
		FirstFragment fragmentFirst = new FirstFragment();
		Bundle args = new Bundle();
		args.putInt("someInt", page);
		args.putString("someTitle", title);
		fragmentFirst.setArguments(args);
		return fragmentFirst;
	}

	// Store instance variables based on arguments passed
	@Override
	public void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		page = getArguments().getInt("someInt", 0);
		title = getArguments().getString("someTitle");
	}

	// Inflate the view for the fragment based on layout XML
	@Override
	public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
		View view = inflater.inflate(R.layout.fragment_first, container, false);
		TextView tvLabel = (TextView) view.findViewById(R.id.tvLabel);
		tvLabel.setText(page + " -- " + title);
		return view;
	}
}
```

### Setup FragmentPagerAdapter

Now we need to define the adapter that will properly determine how many pages exist and which fragment to display for each page of the adapter by creating a [FragmentPagerAdapter](http://developer.android.com/reference/android/support/v4/app/FragmentPagerAdapter.html):

```java
public class MainActivity extends FragmentActivity {
	// ...
	
    public static class MyPagerAdapter extends FragmentPagerAdapter {
	private static int NUM_ITEMS = 3;
		
        public MyPagerAdapter(FragmentManager fragmentManager) {
            super(fragmentManager);
        }
        
        // Returns total number of pages
        @Override
        public int getCount() {
            return NUM_ITEMS;
        }
 
        // Returns the fragment to display for that page
        @Override
        public Fragment getItem(int position) {
            switch (position) {
            case 0: // Fragment # 0 - This will show FirstFragment
                return FirstFragment.newInstance(0, "Page # 1");
            case 1: // Fragment # 0 - This will show FirstFragment different title
                return FirstFragment.newInstance(1, "Page # 2");
            case 2: // Fragment # 1 - This will show SecondFragment
                return SecondFragment.newInstance(2, "Page # 3");
            default:
            	return null;
            }
        }
        
        // Returns the page title for the top indicator
        @Override
        public CharSequence getPageTitle(int position) {
        	return "Page " + position;
        }
        
    }

}
```

### Apply the Adapter

Finally, let's associate the `ViewPager` with a new instance of our adapter:

```java
public class MainActivity extends FragmentActivity {
	FragmentPagerAdapter adapterViewPager;

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_home);
		ViewPager vpPager = (ViewPager) findViewById(R.id.vpPager);
		adapterViewPager = new MyPagerAdapter(getSupportFragmentManager());
		vpPager.setAdapter(adapterViewPager);
	}
	
	public static class MyPagerAdapter extends FragmentPagerAdapter {
	  // ...
        }
    }

}
```

And now we have a basic functioning `ViewPager`. We might want to check out the [ViewPagerIndicator](http://viewpagerindicator.com/) for a better top indicator that is quite popular. We may also want to keep in mind the alternate [FragmentStatePagerAdapter](http://developer.android.com/reference/android/support/v4/app/FragmentStatePagerAdapter.html) in cases where there are many fragment pages and the pages are of a more dynamic nature.

## References

* <http://architects.dzone.com/articles/android-tutorial-using>
* <http://developer.android.com/training/animation/screen-slide.html>
* <http://developer.android.com/reference/android/support/v4/view/ViewPager.html>
* <http://android-developers.blogspot.com/2011/08/horizontal-view-swiping-with-viewpager.html>
* <http://viewpagerindicator.com/>
* <http://mobile.tutsplus.com/tutorials/android/android-user-interface-design-horizontal-view-paging/>
* <http://tamsler.blogspot.com/2011/10/android-viewpager-and-fragments.html>
* <http://www.truiton.com/2013/05/android-fragmentpageradapter-example/>