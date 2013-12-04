## Overview

Layout that allows the user to swipe left and right through "pages" of content which are usually different fragments. This is a common navigation mode to use instead of [[ActionBar Tabs with Fragments]].

![ViewPager](http://i.imgur.com/75zqnxN.png)

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

If you want an "indicator" that displays the pages available at the top as shown in the screenshot above, you need to include a nested indicator view called a [PagerTabStrip](http://developer.android.com/reference/android/support/v4/view/PagerTabStrip.html):

```xml
<android.support.v4.view.ViewPager
   android:id="@+id/vpPager"
   android:layout_width="match_parent"
   android:layout_height="wrap_content">

   <android.support.v4.view.PagerTabStrip
        android:id="@+id/pager_header"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_gravity="top"
        android:paddingBottom="4dp"
        android:paddingTop="4dp" />

</android.support.v4.view.ViewPager>
```

which will automatically display the page indicator for your pager. You might want to check out the popular [ViewPagerIndicator](http://viewpagerindicator.com/) for an improved page indicator. 

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

For more complex cases with many pages, check out the [[more dynamic approach|ViewPager-with-FragmentPagerAdapter#dynamic-viewpager-fragments]] with `SmartFragmentStatePagerAdapter` explained later.

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
	
	// ...
    }

}
```

And now we have a basic functioning `ViewPager` with any number of fragments as pages which can be swiped between. 

### Setup OnPageChangeListener

If the Activity needs to be able listen for changes to the page selected or other events surrounding the `ViewPager`, then we just need to hook into the [ViewPager.OnPageChangeListener](http://developer.android.com/reference/android/support/v4/view/ViewPager.OnPageChangeListener.html) on the `ViewPager` to handle the events:

```java
// Attach the page change listener inside the activity
vpPager.setOnPageChangeListener(new OnPageChangeListener() {
	
	// This method will be invoked when a new page becomes selected.
	@Override
	public void onPageSelected(int position) {
		Toast.makeText(HomeActivity.this, 
                    "Selected page position: " + position, Toast.LENGTH_SHORT).show();
	}
	
	// This method will be invoked when the current page is scrolled
	@Override
	public void onPageScrolled(int position, float positionOffset, int positionOffsetPixels) {
		// Code goes here
	}
	
	// Called when the scroll state changes: 
	// SCROLL_STATE_IDLE, SCROLL_STATE_DRAGGING, SCROLL_STATE_SETTLING
	@Override
	public void onPageScrollStateChanged(int state) {
		// Code goes here
	}
});
```

## Dynamic ViewPager Fragments

In certain cases, we may require a dynamic `ViewPager` with pages being added or removed on the fly. If your ViewPager is more dynamic with many pages and fragments, we will want to use the the alternate [FragmentStatePagerAdapter](http://developer.android.com/reference/android/support/v4/app/FragmentStatePagerAdapter.html) instead. Below shows us how to use this and also intelligently cache the fragments for easy lookup.

First, copy in the [SmartFragmentStatePagerAdapter.java](https://gist.github.com/nesquena/c715c9b22fb873b1d259) which provides the intelligent caching of registered fragments for our `ViewPager`. This solves the common problem of needing to **access the current item within the ViewPager**.

Now, we want to extend from `SmartFragmentStatePagerAdapter` copied above when declaring our adapter so we can take advantage of the better memory management of the state pager:

```java
public class MainActivity extends FragmentActivity {
    // ...
    private SmartFragmentStatePagerAdapter adapterViewPager;
    
    // Extend from SmartFragmentStatePagerAdapter now instead for more dynamic ViewPager items
    public static class MyPagerAdapter extends SmartFragmentStatePagerAdapter {
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

Now with this adapter in place, we can also easily access any fragments within the `ViewPager` with:

```java
adapterViewPager.getRegisteredFragment(0); 
// returns first Fragment item within the pager
```

and we can easily access the "current" pager item with:

```java
adapterViewPager.getRegisteredFragment(vpPager.getCurrentItem());
// returns current Fragment item displayed within the pager
```

This pattern should save your app quite a deal of memory and allow for much easier management of fragments within your pager for the right situation.
 
## References

* <http://architects.dzone.com/articles/android-tutorial-using>
* <http://developer.android.com/training/animation/screen-slide.html>
* <http://developer.android.com/reference/android/support/v4/view/ViewPager.html>
* <http://android-developers.blogspot.com/2011/08/horizontal-view-swiping-with-viewpager.html>
* <http://viewpagerindicator.com/>
* <http://mobile.tutsplus.com/tutorials/android/android-user-interface-design-horizontal-view-paging/>
* <http://tamsler.blogspot.com/2011/10/android-viewpager-and-fragments.html>
* <http://www.truiton.com/2013/05/android-fragmentpageradapter-example/>