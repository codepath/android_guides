### Fragments and Tabs

There are several ways to setup tabs with fragments. The easiest is using ActionBar tabs. Note: ActionBar tabs are not supported in Gingerbread, so many people use [ActionBarSherlock](http://actionbarsherlock.com/) when Gingerbread must be supported. Thankfully, the API is more or less identical with a few class name tweaks.

#### Without ActionBarSherlock

To setup tabs using ActionBar and fragments, you need to add a `TabListener` implementation to your application which defines the behavior of a tab when activated. A good default implementation is just [adding this](https://gist.github.com/nesquena/f54a991ccb4e5929e0ec) to `FragmentTabListener.java`. 

Once you have created the `FragmentTabListener` [from this snippet](https://gist.github.com/nesquena/f54a991ccb4e5929e0ec) within your app, simply setup the ActionBar and define which tabs you would like to display and attaches listeners for each tab:

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

#### With ActionBarSherlock

Using [ActionBarSherlock](http://actionbarsherlock.com/), the code looks almost exactly the same. First you must download [ActionBarSherlock.zip](https://api.github.com/repos/JakeWharton/ActionBarSherlock/zipball/4.4.0). Import the code into Eclipse and mark the project as a library. Next, add the library to your application. Watch the [video](http://www.youtube.com/watch?v=4GJ6yY1lNNY) on the [FAQ](http://actionbarsherlock.com/faq.html) page for a detailed guide.

With ActionBarSherlock installed and added to your app, add [the gist code](https://gist.github.com/nesquena/0f0a1f50a2a64721309e) as the `SherlockTabListener` within your application.

Once you have created the `SherlockTabListener` [from this snippet](https://gist.github.com/nesquena/0f0a1f50a2a64721309e) within your app, simply setup the ActionBar and define which tabs you would like to display and attaches listeners for each tab:

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