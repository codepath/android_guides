## Overview

Apps often need to provide the user with a consistent experience for executing actions that are contextually specific. The most common actions for an activity live on the default [[Action Bar|Defining The ActionBar]] but actions that are more specific to an item or element can be displayed contextually using menus and popups. In addition to menus, apps often need to display popup overlays that contain informational content. This guide covers how to manage menus and popups within your Android apps.

## Usage

In modern Android apps, there are a few mechanisms for displaying secondary content or actions related to an Activity:

* [Contextual Action Modes](http://developer.android.com/guide/topics/ui/menus.html#CAB) - An "action mode" which is enabled when a user selects an item. Upon the item selection, the actionbar switches to a contextual mode that presents relevant actions.
* [PopupMenu](http://developer.android.com/guide/topics/ui/menus.html#PopupMenu) - A modal menu that is anchored to a particular view within an activity. The menu appears below that view when it is selected. Used to provide an overflow menu that allows for secondary actions on an item.
* [PopupWindow](http://mrbool.com/how-to-implement-popup-window-in-android/28285) - A simple dialog box that gains focus when appearing on screen. It is used to display additional information on screen and is simpler but less flexible than a DialogFragment.
* [[DialogFragment|Using DialogFragment]] - A fully customizable dialog overlay that appears ontop of the activity and can contain arbitrary content as defined within the fragment. This is the most flexible but also the heaviest approach to displaying overlay content.

Let's take a look at each one of these.

### Contextual Action Modes

Contextual Action Modes can be enabled when a user selects or focuses on any item. Upon the item selection, the ActionBar switches to a contextual mode that presents relevant actions for said item.

Important actions related to the view item should be presented by constructing these contextual action modes whenever possible. The action mode is typically presented **upon long click of a view** or by selecting a checkbox within the activity. 

#### Defining the ActionMode

Let's take a look at how to create a contextual action mode for a view which will be presented when a view is long clicked. First, let's define a menu XML file to be displayed in a `res/menu/actions_textview.xml` file:

```xml
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:id="@+id/menu_edit"
          android:title="Edit"
          android:icon="@android:drawable/ic_menu_edit"/>
    <item android:id="@+id/menu_delete"
          android:title="Delete"
          android:icon="@android:drawable/ic_menu_delete"/>
</menu>
```

Next, let's define a [ActionMode.Callback](http://developer.android.com/reference/android/view/ActionMode.Callback.html) which will be manage the behavior of our contextual action menu:

```java
public class MainActivity extends Activity {
  // Tracks current contextual action mode
  private ActionMode currentActionMode; 
  // Define the callback when ActionMode is activated
  private ActionMode.Callback modeCallBack = new ActionMode.Callback() {
    // Called when the action mode is created; startActionMode() was called
    @Override
    public boolean onCreateActionMode(ActionMode mode, Menu menu) {
      mode.setTitle("Actions");
      mode.getMenuInflater().inflate(R.menu.actions_textview, menu);
      return true;
    }
    
    // Called each time the action mode is shown.
    @Override
    public boolean onPrepareActionMode(ActionMode mode, Menu menu) {
      return false; // Return false if nothing is done
    }
    
    // Called when the user selects a contextual menu item
    @Override
    public boolean onActionItemClicked(ActionMode mode, MenuItem item) {
      switch (item.getItemId()) {
      case R.id.menu_edit:
        Toast.makeText(MainActivity.this, "Editing!", Toast.LENGTH_SHORT).show();
        mode.finish(); // Action picked, so close the contextual menu
        return true;
      case R.id.menu_delete:
        // Trigger the deletion here
        mode.finish(); // Action picked, so close the contextual menu
        return true;
      default:
        return false;
      }
    }
    
    // Called when the user exits the action mode
    @Override
    public void onDestroyActionMode(ActionMode mode) {
      currentActionMode = null; // Clear current action mode
    } 
  };
}
```

Now that we have an ActionMode we can enable for any view (i.e TextView) or AdapterView (such as a ListView) which will display a contextual action bar for that element.

#### Enabling for Simple Views

To activate the contextual action mode defined above for a simple TextView, we just need to attach the `OnLongClickListener` for the view and then trigger the action mode using `startActionMode`:

```java
public class MainActivity extends Activity {
  // TextView with context menu
  private TextView tvName;
  // Tracks current contextual action mode
  private ActionMode currentActionMode;
  
  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    tvName = (TextView) findViewById(R.id.tvName);
    tvName.setOnLongClickListener(new View.OnLongClickListener() {
      // Called when the user long-clicks on someView
      public boolean onLongClick(View view) {
         if (currentActionMode != null) { return false; }
         startActionMode(modeCallBack);
         view.setSelected(true);
         return true;
      }
    }
  }
  
  // ... action mode definitions ....
}
```

That's all and now you have a TextView that upon long clicking will reveal the contextual action bar menu as show below:

![ViewActionMenu](http://i.imgur.com/h0r3PRS.png)

#### Enabling for Adapter Views

We can also enable contextual menus for ListView items as well. This is as simple as setting the `setOnItemLongClickListener` property for the list items and then presenting the action mode. Given the same action mode defined the previous sections, we can apply the mode to items with:

```java
public class MainActivity extends Activity {
  // ListView and adapter
  private ListView lvItems;
  private ArrayList<String> arrayItems;
  private ArrayAdapter<String> adapterItems;
  // Tracks current menu item
  private int currentListItemIndex;
  // Tracks current contextual action mode
  private ActionMode currentActionMode;

  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    populateListView(); // Fill array with string items and attach to listview
    lvItems = (ListView) findViewById(R.id.lvItems);
    lvItems.setAdapter(adapterItems);
    // Setup contextual action mode when item is clicked
    lvItems.setOnItemLongClickListener(new AdapterView.OnItemLongClickListener() {
      public boolean onItemLongClick(AdapterView<?> parent, View view, int position, long id) {
        if (currentActionMode != null) { return; }
        currentListItemIndex = position;
        currentActionMode = startActionMode(modeCallBack);
        view.setSelected(true);
        return true;
      }
    });
  }
  
  // ... action mode definitions ....
}
```

This will trigger the contextual action menu whenever a list item is long-clicked. Notice that we also store the `currentListItemIndex` so we can act directly on the correct item after an action is selected. For example, the `modeCallback` might have a `onActionItemClicked` defined as:

```java
private ActionMode.Callback modeCallBack = new ActionMode.Callback() {
  // ...
  // Called when the user selects a contextual menu item
  @Override
  public boolean onActionItemClicked(ActionMode mode, MenuItem item) {
    switch (item.getItemId()) {
    case R.id.menu_delete:
      arrayItems.remove(currentListItemIndex); // Remove current item
      adapterItems.notifyDataSetChanged(); // Refresh adapter
      mode.finish(); // Action picked, so close the contextual menu
      return true;
    default:
      return false;
    }
  }
  // ...
};
```

This will result in the following after being defined correctly:

![ListView CAB](http://i.imgur.com/hdSxf9e.png)

Using this approach we can easily allow users to act on items within a list. Note that in certain cases we may want to also **support batch modes** where more then one item can be affected by a contextual action. In that case, we need to use a [MultiChoiceModeListener](http://developer.android.com/reference/android/widget/AbsListView.MultiChoiceModeListener.html) as the ActionMode as explained in [the official menus docs](http://developer.android.com/guide/topics/ui/menus.html#CABforListView).

### Popup Menu

A modal menu that is anchored to a particular view within an activity and the menu appears below that view when displayed. Used to provide an overflow menu that allows for secondary actions on an item. 

Note that if the items change the selected content, consider using the "Contextual Action Mode" explained above. Instead, use this for actions that **relate to the content**, for example triggering a reply to a message.

Defining a PopupMenu for a view is as simple as constructing the `PopupMenu` object and then defining the menu XML that you would like to display. For example, if we wanted to present a popup menu for filtering items when a button was clicked, first we define the menu XML for that button in `res/menu/popup_filters.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:id="@+id/menu_keyword"
          android:title="Keyword" />
    <item android:id="@+id/menu_popularity"
          android:title="Popularity" />
</menu>
```

Next, we define a click handler for the button that displays the `PopupMenu`:

```java
public class MainActivity extends Activity {
  private Button btnFilter;
  
  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    // Locate filter button and attach click listener
    btnFilter = (Button) findViewById(R.id.btnFilter);
    btnFilter.setOnClickListener(new OnClickListener() {
        @Override
        public void onClick(View v) {
            showFilterPopup(v);
        }
    });
    // ...
  }
  
  // Display anchored popup menu based on view selected
  private void showFilterPopup(View v) {
    PopupMenu popup = new PopupMenu(this, v);
    // Inflate the menu from xml
    popup.getMenuInflater().inflate(R.menu.popup_filter, popup.getMenu());
    // Setup menu item selection
    popup.setOnMenuItemClickListener(new PopupMenu.OnMenuItemClickListener() {
        public boolean onMenuItemClick(MenuItem item) {
            switch (item.getItemId()) {
            case R.id.menu_keyword:
              Toast.makeText(MainActivity.this, "Keyword!", Toast.LENGTH_SHORT).show();
              return true;
            case R.id.menu_popularity:
              Toast.makeText(MainActivity.this, "Popularity!", Toast.LENGTH_SHORT).show();
              return true;
            default:
              return false;
            }
        }
    });
    // Handle dismissal with: popup.setOnDismissListener(...);
    // Show the menu
    popup.show();
  }
}
```

With that setup, whenever the button is pressed, the menu will be displayed and a different action will be taken based on the item selected. Here's the result:

![PopupMenu](http://i.imgur.com/r97rvpp.png)

### Popup Window

In addition to contextual action modes and popup menus, there are also a more customizable [PopupWindow](http://developer.android.com/reference/android/widget/PopupWindow.html) concept. `PopupWindow` is a floating content container that appears over the current activity like a simplified [[DialogFragment|Using DialogFragment]] that is less flexible but easier to implement. Popups are usually used to show some additional information or something user wants to know after an event takes place.

To display a `PopupWindow`, let's first [download a background](http://i.imgur.com/C89eYBC.png/popup_bg.9.png) to use for the popup within `res/drawable/popup_bg.9.png`. The background is a nine-patch image which will be used as the background for the popup content. 

Next, let's define an arbitrary XML layout file which will be **displayed within the popup** at `res/layout/popup_content.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:background="@drawable/popup_bg">
    <TextView
        android:id="@+id/tvCaption"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_centerHorizontal="true"
        android:layout_centerVertical="true"
        android:text="This is a popup!"
        android:textAppearance="?android:attr/textAppearanceLarge"
        android:textColor="#ffffff" />
</RelativeLayout>
```

Now, we have the background and the content layout defined so we can **setup the popup to display when a button is clicked** with:

```java
public class DemoWindowActivity extends Activity {
  private Button btnShowPopup;

  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_demo_window);
    // Locate the show popup button and attach click listener
    btnShowPopup = (Button) findViewById(R.id.btnShowPopup);
    btnShowPopup.setOnClickListener(new OnClickListener() {
      @Override
      public void onClick(View v) {
        // Display popup attached to the button as a position anchor
        displayPopupWindow(v);
      }
    });
  }
  
  // ...
}
```

Finally, we need to define the `displayPopupWindow` method to actually construct and display the window based on our content area and background image:

```java
public class DemoWindowActivity extends Activity {
  // ...onCreate above...
  // Display popup attached to the button as a position anchor
  private void displayPopupWindow(View anchorView) {
    PopupWindow popup = new PopupWindow(DemoWindowActivity.this);
    View layout = getLayoutInflater().inflate(R.layout.popup_content, null);
    popup.setContentView(layout);
    // Set content width and height
    popup.setHeight(WindowManager.LayoutParams.WRAP_CONTENT);
    popup.setWidth(WindowManager.LayoutParams.WRAP_CONTENT);
    // Closes the popup window when touch outside of it - when looses focus
    popup.setOutsideTouchable(true);
    popup.setFocusable(true);
    // Show anchored to button
    popup.setBackgroundDrawable(new BitmapDrawable());
    popup.showAsDropDown(anchorView);
  }
}  
```

The result of this is a dismissable popup window with our content and which looks like:

![PopupWindow](http://i.imgur.com/CfTnHyX.png)

Read more details on the [PopupWindow tutorial](http://mrbool.com/how-to-implement-popup-window-in-android/28285). You might also want to check out this extended [DismissablePopupWindow](https://gist.github.com/raquibulbari/6059845) for a simple re-usable popup that had a dismiss button embedded within the window.

### Deprecated Menu Strategies

Prior to Android 3.0, there were several other relevent menu strategies which are now discouraged. The strategies that have been deprecated in modern apps are listed below:

 * [Options Menu](http://developer.android.com/guide/topics/ui/menus.html#options-menu) - This was a menu that appeared when a hardware "menu button" was pressed. This menu button has been deprecated and should no longer be used in modern apps. You can read more in the [Creating an Options Menu](http://developer.android.com/guide/topics/ui/menus.html#options-menu) official guide.
 * [Floating Context Menu](http://developer.android.com/guide/topics/ui/menus.html#FloatingContextMenu) - This was a contextual menu that appeared to float above the activity with a list of menu items to select. This approach is deprecated in favor of the aforementioned Contextual Action Mode in modern apps. See [this tutorial](http://mobile.tutsplus.com/tutorials/android/android-sdk-context-menus/) for another look at the deprecated floating context approach.
 
### Menu Groups

In certain cases, we may want to group menu items together within a menu XML. To group menu items simply wrap them in a `group` node in the menu xml:

```xml
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:id="@+id/menu_save"
          android:icon="@drawable/menu_save"
          android:title="@string/menu_save" />
    <!-- menu group -->
    <group android:id="@+id/group_delete">
        <item android:id="@+id/menu_archive"
              android:title="@string/menu_archive" />
        <item android:id="@+id/menu_delete"
              android:title="@string/menu_delete" />
    </group>
</menu>
```

Check out more details in the [Menu Groups](http://developer.android.com/guide/topics/ui/menus.html#groups) official docs which shows you how to create "checkable" items as well.

## Libraries

* [Android-SlideExpandableListView](https://github.com/tjerkw/Android-SlideExpandableListView) - Custom listview in which each list item has an area that can slide-out when activated.
* [android-swipelistview](https://github.com/47deg/android-swipelistview) - An Android List View implementation with support for secondary swipe-activated action menu

## References

* <http://developer.android.com/guide/topics/ui/menus.html>
* <http://mobile.tutsplus.com/tutorials/android/android-sdk-context-menus/>
* <http://gurushya.com/android-popup-menus/>
* <http://learnandroideasily.blogspot.com/2013/01/creating-context-menu-in-android.html>
* <http://www.javacodegeeks.com/2013/06/android-listview-context-menu-actionmode-callback.html>
* <http://mobile.dzone.com/news/context-menu-android-tutorial>
* <http://www.mikeplate.com/2010/01/21/show-a-context-menu-for-long-clicks-in-an-android-listview/>
* <http://htc-magic-android.gb-eu.com/131/accessing-listview-items-with-a-context-menu.html>
* <http://www.coderzheaven.com/2013/04/07/create-simple-popup-menu-android/>
* <http://www.techrepublic.com/blog/software-engineer/working-with-androids-popupmenu-class/>