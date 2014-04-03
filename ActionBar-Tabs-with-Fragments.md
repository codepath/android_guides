## Fragments and Tabs

There are several ways to setup tabs with fragments. The easiest is using ActionBar tabs. Note: Standard ActionBar tabs are not supported in Gingerbread, so many people use [ActionBarSherlock](http://actionbarsherlock.com/) when Gingerbread must be supported. Google has also released a support `ActionBarActivity` class which can be used for compatible tabs. Thankfully, both the support approaches are more or less identical in code with a few class name tweaks.

### Without Gingerbread Support

To setup tabs using ActionBar and fragments that are not gingerbread compatible, you need to add a `TabListener` implementation to your application which defines the behavior of a tab when activated. A good default implementation is just [adding this](https://gist.github.com/nesquena/f54a991ccb4e5929e0ec) to `FragmentTabListener.java`. 

Once you have created the `FragmentTabListener` [from this snippet](https://gist.github.com/nesquena/f54a991ccb4e5929e0ec) within your app, setup the ActionBar and define which tabs you would like to display and attach listeners for each tab:

```java
public class MainActivity extends FragmentActivity {

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		setupTabs();
	}

	private void setupTabs() {
		ActionBar actionBar = getActionBar();
		actionBar.setNavigationMode(ActionBar.NAVIGATION_MODE_TABS);
		actionBar.setDisplayShowTitleEnabled(true);

		Tab tab1 = actionBar
			.newTab()
			.setText("First")
			.setIcon(R.drawable.ic_home)
			.setTag("HomeTimelineFragment")
			.setTabListener(
				new FragmentTabListener<FirstFragment>(R.id.flContainer, this, "first",
								FirstFragment.class));

		actionBar.addTab(tab1);
		actionBar.selectTab(tab1);

		Tab tab2 = actionBar
			.newTab()
			.setText("Second")
			.setIcon(R.drawable.ic_mentions)
			.setTag("MentionsTimelineFragment")
			.setTabListener(
			    new FragmentTabListener<SecondFragment>(R.id.flContainer, this, "second",
								SecondFragment.class));

		actionBar.addTab(tab2);
	}
}
```

Now you have fragments that work for any Android version above 3.0. If you need gingerbread support, check out the approaches below.

### With ActionBarActivity Support

Google has released an updated support library "android-support-v7-appcompat" which includes official support for the ActionBar with Gingerbread compatibility. To use this, first, we need to **upgrade to latest support library** by opening the Android SDK Manager and verifying we have the latest "Android Support Library".

Now, we need to import the support library as a library project following the [usual steps](http://imgur.com/a/N8baF). `File => Import => Existing Android Code...` and then go to your **sdk folder** and select `sdk => extras => android => support => v7 => appcompat`. 

![Screen 1](http://i.imgur.com/pRffDgs.png)

Now, for your app, right-click and select "Properties..." and then add "android-support-v7-appcompat" as a library.

![Screen 2](http://i.imgur.com/E5IYm69.png)

Make sure to setup your app to use the correct support theme within the `AndroidManifest.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.codepath.apps.codepathbootcampapp"
    ...>
    <application
        android:theme="@style/Theme.Base.AppCompat.Light.DarkActionBar" 
        ...>
</manifest>
```

For these compatibility items, you also need to be careful to **change the menu items** to use a [custom prefix](http://developer.android.com/guide/topics/ui/actionbar.html#ActionItems) in `res/menu/example.xml` for the `showAsAction` instead of `android:showAsAction`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<menu
  xmlns:android="http://schemas.android.com/apk/res/android"
  xmlns:myapp="http://schemas.android.com/apk/res-auto">
    <item android:id="@+id/item_menu_ok" android:icon="@drawable/ic_action_ok"
        android:title="@string/ok" myapp:showAsAction="ifRoom"></item>
    <item android:id="@+id/item_menu_cancel" android:icon="@drawable/ic_action_cancel"
        android:title="@string/cancel" myapp:showAsAction="ifRoom"></item>
</menu>
```

To setup tabs using ActionBar and fragments, you need to add a `TabListener` implementation to your application which defines the behavior of a tab when activated. A good default implementation is just [adding this](https://gist.github.com/nesquena/8b9f9ec29582afd4d138) to `SupportFragmentTabListener.java`. 

Once you have created the `SupportFragmentTabListener` [from this snippet](https://gist.github.com/nesquena/8b9f9ec29582afd4d138) within your app, setup the ActionBar and define which tabs you would like to display and attach listeners for each tab:

```java
public class MainActivity extends ActionBarActivity {

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		setupTabs();
	}

	private void setupTabs() {
		ActionBar actionBar = getSupportActionBar();
		actionBar.setNavigationMode(ActionBar.NAVIGATION_MODE_TABS);
		actionBar.setDisplayShowTitleEnabled(true);

		Tab tab1 = actionBar
		    .newTab()
		    .setText("First")
		    .setIcon(R.drawable.ic_home)
		    .setTag("HomeTimelineFragment")
		    .setTabListener(new SupportFragmentTabListener<FirstFragment>(R.id.flContainer, this,
                        "first", FirstFragment.class));

		actionBar.addTab(tab1);
		actionBar.selectTab(tab1);

		Tab tab2 = actionBar
		    .newTab()
		    .setText("Second")
		    .setIcon(R.drawable.ic_mentions)
		    .setTag("MentionsTimelineFragment")
		    .setTabListener(new SupportFragmentTabListener<SecondFragment>(R.id.flContainer, this,
                        "second", SecondFragment.class));
		actionBar.addTab(tab2);
	}

}
```

### With ActionBarSherlock

Using [ActionBarSherlock](http://actionbarsherlock.com/), the code looks almost exactly the same but **automatically supports gingerbread and below**. First you must download [ActionBarSherlock.zip](https://api.github.com/repos/JakeWharton/ActionBarSherlock/zipball/4.4.0). Import the code into Eclipse and mark the project as a library. Next, add the library to your application. Watch the [video](http://www.youtube.com/watch?v=4GJ6yY1lNNY) on the [FAQ](http://actionbarsherlock.com/faq.html) page for a detailed guide.

With ActionBarSherlock installed and added to your app, add [the gist code](https://gist.github.com/nesquena/0f0a1f50a2a64721309e) as the `SherlockTabListener` within your application.

Once you have created the `SherlockTabListener` [from this snippet](https://gist.github.com/nesquena/0f0a1f50a2a64721309e) within your app, setup the ActionBar and define which tabs you would like to display and attaches listeners for each tab:

```java
public class ActionBarSherlockActivity extends SherlockFragmentActivity {

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_action_bar_sherlock);
		setupTabs();
	}

	private void setupTabs() {
		ActionBar actionBar = getSupportActionBar();
		actionBar.setNavigationMode(ActionBar.NAVIGATION_MODE_TABS);
		actionBar.setDisplayShowHomeEnabled(true);
		actionBar.setDisplayShowTitleEnabled(true);

		Tab tabFirst = actionBar
			.newTab()
			.setText("First")
			.setIcon(R.drawable.ic_launcher)
			.setTabListener(
				new SherlockTabListener<FirstFragment>(R.id.flContainer, this, "First",
								FirstFragment.class));

		actionBar.addTab(tabFirst);
		actionBar.selectTab(tabFirst);

		Tab tabSecond = actionBar
			.newTab()
			.setText("Second")
			.setIcon(R.drawable.ic_launcher)
			.setTabListener(
				new SherlockTabListener<SecondFragment>(R.id.flContainer, this, "Second",
							SecondFragment.class));

		actionBar.addTab(tabSecond);
	}
}
```

With this approach you can have easy tab navigation switching quickly between fragments.

### Reference a Fragment in the Activity

If you need to reference a fragment instance from within the activity, you can refer to the fragment by it's "tag". For example, if you created the following tab:

```
private final String FIRST_TAB_TAG = "first";
Tab tab1 = actionBar
  ...
  .setTabListener(new SupportFragmentTabListener<FirstFragment>(R.id.flContainer, this,
      FIRST_TAB_TAG, FirstFragment.class));
```

You have assigned the tag "First" to that fragment during construction and you can access the fragment for this tab later using `findFragmentByTag`:

```java
FirstFragment fragmentFirst = getSupportFragmentManager().findFragmentByTag(FIRST_TAB_TAG);
```

This will return the fragment instance embedded for that tab.

### Tab Selection

Once the ActionBar has been set to `ActionBar.NAVIGATION_MODE_TABS` navigation mode, we can look at the currently selected tab index with [getSelectedNavigationIndex()](http://developer.android.com/reference/android/app/ActionBar.html#getSelectedNavigationIndex\(\)):

```java
// Returns tab index currently selected
getSupportActionBar().getSelectedNavigationIndex();
```

We can set the currently selected tab by index with [setSelectedNavigationItem](http://developer.android.com/reference/android/app/ActionBar.html#setSelectedNavigationItem\(int\)):

```java
// sets current tab by index to the first tab on the bar
getSupportActionBar().setSelectedNavigationItem(0);
```

We can also access arbitrary tabs, check the number of tabs or remove them:

```java
// returns the tab object at position 0
getSupportActionBar().getTabAt(0); 
// returns the total number of tabs
getSupportActionBar().getTabCount();
// Removes the first tab
getSupportActionBar().removeTabAt(0); 
// Returns the currently selected tab object
getSupportActionBar().getSelectedTab();
// Select a tab given the tab object
getSupportActionBar().selectTab(someTabObject);
```

Using these methods described above, we can modify the tabs and their selected states at runtime.

**Note:** Occasionally, you might attempt to switch the current tab and load a fragment in the wrong event (for example `onActivityResult`) and encounter an error such as `IllegalStateException: cannot perform this action after onSaveInstanceState`. This is typically tied to replacing fragments onto the content pane at a time where state loss could occur. Read [this article](http://www.androiddesignpatterns.com/2013/08/fragment-transaction-commit-state-loss.html) for a detailed description of the issue. See [this solution](http://stackoverflow.com/a/18345899) for one common approach to solving this.

### Styling Tabs

Styling the ActionBar requires using the [[Styles|Styles and Themes]] system for declaring the look of the ActionBar and tabs. We can tweak the styles of these by building a custom theme in the `res/values-v14/styles.xml`:

```xml
<style name="MyTheme" parent="android:Theme.Holo.Light.DarkActionBar">
    <item name="android:actionBarTabStyle">@style/MyTheme.ActionBar.Tab</item>
</style>
 
<style name="MyTheme.ActionBar.Tab">
    <item name="android:background">@drawable/tab_bar_background</item>
</style>
```

There are actually three related aspects to styling different parts of the ActionBar:

* `actionBarTabStyle` – This determines the style of the individual tabs themselves including the indicator.
* `actionBarTabBarStyle` – This determines the style of the overall tab bar which contains the tabs.

Check out [this article](http://blog.alwold.com/2013/08/28/styling-tabs-in-the-android-action-bar/) for a detailed look at styling tabs in the action bar which also requires an understanding of [[Drawables]]. Also be sure to browse the [Styling ActionBar](https://developer.android.com/training/basics/actionbar/styling.html) official guide as well even though it is a bit verbose.