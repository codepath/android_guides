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
	public View onCreateView(LayoutInflater inflater, ViewGroup container, 
            Bundle savedInstanceState) {
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
```

And now we have a basic functioning `ViewPager` with any number of fragments as pages which can be swiped between. 

### Selecting or Getting the Page

We can access the selected page within the `ViewPager` at any time with the [getCurrentItem](http://developer.android.com/reference/android/support/v4/view/ViewPager.html#getCurrentItem\(\)) method which returns the current page:

```java
vpPager.getCurrentItem(); // --> 2
```

The current page can also be changed programmatically with the 

```java
vpPager.setCurrentItem(2)
```

With this getter and setter, we can easily access or modify the selected page at runtime.

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
**Note:** if you're using the PagerSlidingTabStrip library (see below), call `setOnPageChangeListener` on your PagerSlidingTabStrip object instead of on your ViewPager. You must do this because the ViewPager only supports notifying one listener.

## Tabbed Interface with Pager

We can use the ViewPager to display a tabbed indicator in order to create tabs to display our fragments. 
At Google I/O 2015, Google announced a new `TabLayout` class that makes creating this tabbed interface fairly easy to do.  See [[Google Play Style Tabs using TabLayout]] for a walkthrough.

An alternative approach to achieve this is to use the third-party [[PagerSlidingTabStrip|Sliding-Tabs-with-PagerSlidingTabStrip]] library. 

![Tabs](http://i.imgur.com/a2wpJ80.png)

In this way, we can use the same pager system described above and augment the pager with a tabbed navigation indicator.

## Dynamic ViewPager Fragments

In certain cases, we may require a dynamic `ViewPager` where we want to get access to fragment instances or with pages being added or removed at runtime. If your ViewPager is more dynamic with many pages and fragments, we will want to use an implementation of the alternate [FragmentStatePagerAdapter](http://developer.android.com/reference/android/support/v4/app/FragmentStatePagerAdapter.html) instead. Below shows us how to use this and also intelligently cache the fragments for easy lookup.

### Setup SmartFragmentStatePagerAdapter

First, copy in the [SmartFragmentStatePagerAdapter.java](https://gist.github.com/nesquena/c715c9b22fb873b1d259) which provides the intelligent caching of registered fragments within our `ViewPager`. This solves the common problem of needing to **access the current item within the ViewPager**.

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

### Access Fragment Instances

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

## Set Offscreen Page Limit

Alternatively, you can use the method `setOffscreenPageLimit(int limit)` provided by ViewPager to set how many page instances you want the system to keep in memory on either side of your current page. As a result, more memory will be consumed. So be careful when tweaking this setting if your pages have complex layouts. 

For example, to let the system keep 3 page instances on both sides of the current page:

```java
vpPager.setOffscreenPageLimit(3);
```

This can be useful in order to preload more fragments for a smoother viewing experience trading off with memory usage.

## ViewPager with Visible Adjacent Pages

If you are interested in a ViewPager with visible adjacent pages that are partially visible:

![ViewPager Adjacent](http://i.stack.imgur.com/ddm1j.jpg)

We can do that with by tuning a few properties of our pager. First, here's how the `ViewPager` might be defined in the XML Layout:

```xml
<android.support.v4.view.ViewPager
  	android:id="@+id/pager"
  	android:gravity="center"
  	android:layout_width="match_parent"
  	android:layout_height="0px"
  	android:paddingLeft="24dp"
  	android:paddingRight="12dp"
  	android:layout_weight="1" />
```

Next, we need to tune these properties of the pager in the containing fragment or activity:

```java
ViewPager vpPager = (ViewPager) view.findViewById(R.id.vpPager);
vpPager.setClipToPadding(false);
vpPager.setPageMargin(12);
// Now setup the adapter as normal
```

Finally we need to adjust the width inside the adapter:

```java
class MyPageAdapter : FragmentStatePagerAdapter {
    @Override
    public float getPageWidth (int position) {
        return 0.93f;
    }	
    
    // ...
}
```

For more details, you can follow these guides:

* [ViewPager with Protruding Children](http://blog.neteril.org/blog/2013/10/14/android-tip-viewpager-with-protruding-children/)
* [ViewPager with Page Boundaries](http://stackoverflow.com/questions/13914609/viewpager-with-previous-and-next-page-boundaries)

## Animating the Scroll with PageTransformer

We can customize how the pages animate as they are being swiped between using the [PageTransformer](http://developer.android.com/reference/android/support/v4/view/ViewPager.PageTransformer.html). This transformer exists within the support library and is compatible with API 11 or greater. Usage is pretty straightforward, just attach a PageTransformer to the ViewPager:

```java
vpPager.setPageTransformer(false, new ViewPager.PageTransformer() { 
    @Override
    public void transformPage(View page, float position) {
        // Do our transformations to the pages here
    }
});
```

The first argument is set to true if the **supplied PageTransformer requires page views to be drawn from last to first** instead of first to last. The second argument is the transformer which requires defining the `transformPage` method to define the sliding page behavior. 

The `transformPage` method accepts two parameters: `page` which is the particular page to be modified and `position` which indicates where a given page is located **relative to the center of the screen**. The page **which fills the screen** is at **position 0**. The page immediately to the right is at **position 1**. If the user scrolls halfway between pages one and two, page one has a position of -0.5 and page two has a position of 0.5.

```java
vpPager.setPageTransformer(false, new ViewPager.PageTransformer() { 
    @Override
    public void transformPage(View page, float position) {
        int pageWidth = view.getWidth();
        int pageHeight = view.getHeight();

        if (position < -1) { // [-Infinity,-1)
            // This page is way off-screen to the left.
            view.setAlpha(0);
        } else if(position <= 1){ // Page to the left, page centered, page to the right
           // modify page view animations here for pages in view 
        } else { // (1,+Infinity]
            // This page is way off-screen to the right.
            view.setAlpha(0);
        }
    }
});
```

For more details, check out [the official guide](http://developer.android.com/training/animation/screen-slide.html#pagetransformer) or [this guide](http://howrobotswork.wordpress.com/2013/06/27/create-viewpager-transitions-a-pagertransformer-example/). You can also review this cool [rotating page transformer effect](https://stevenkideckel.wordpress.com/tag/pagetransformer/) for another example.
 
## References

* <http://architects.dzone.com/articles/android-tutorial-using>
* <http://developer.android.com/training/animation/screen-slide.html>
* <http://developer.android.com/reference/android/support/v4/view/ViewPager.html>
* <http://android-developers.blogspot.com/2011/08/horizontal-view-swiping-with-viewpager.html>
* <http://viewpagerindicator.com/>
* <http://mobile.tutsplus.com/tutorials/android/android-user-interface-design-horizontal-view-paging/>
* <http://tamsler.blogspot.com/2011/10/android-viewpager-and-fragments.html>
* <http://www.truiton.com/2013/05/android-fragmentpageradapter-example/>