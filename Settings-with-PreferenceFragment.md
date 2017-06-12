## Overview

In Android apps, there are often settings pages that contain different options the user can tweak. The [`PreferenceFragment`](http://developer.android.com/reference/android/preference/PreferenceFragment.html) and [`PreferenceFragmentCompat`](https://developer.android.com/reference/android/support/v7/preference/PreferenceFragmentCompat.html) contains a hierarchy of preference objects displayed on screen in a list. These preferences will automatically save to SharedPreferences as the user interacts with them.

### What's your minimum API level?

**Lollipop and below**: The suggested way of handling settings is through the `PreferenceFragment` for API 11 (Honeycomb) and above. You can avoid the original [`PreferenceActivity`](http://developer.android.com/reference/android/preference/PreferenceActivity.html), which has many _deprecated_ methods. Note, however, that the `PreferenceFragment` is **NOT** compatible with Android support v4. For alternatives:


* [https://github.com/Machinarius/PreferenceFragment-Compat](https://github.com/Machinarius/PreferenceFragment-Compat)
* [https://github.com/kolavar/android-support-v4-preferencefragment](https://github.com/kolavar/android-support-v4-preferencefragment)

**Marshmallow and above**: The support v7 library introduced the [`PreferenceFragmentCompat`](https://developer.android.com/reference/android/support/v7/preference/PreferenceFragmentCompat.html). We'll be using this for the rest of the tutorial.

### Edit your gradle dependencies

Open your app's gradle file (`Your-Project/app/build.gradle`) and add the following to the dependencies:

```gradle
dependencies {
    // your other dependencies...
    compile 'com.takisoft.fix:preference-v7:25.3.0.0'
}
```

Note that due to a bug with using the Material design theme and the Preference Fragment compat, an open source project is used instead. See more details on this [StackOverflow post](http://stackoverflow.com/questions/32070670/preferencefragmentcompat-requires-preferencetheme-to-be-set/32325283).

### Defining the XML
First, define the preference object hierarchy by creating a new xml file in `res/xml`:

```xml
<PreferenceScreen xmlns:android="http://schemas.android.com/apk/res/android">
   
    <PreferenceCategory
        android:title="@string/title_first_section">
 
        <CheckBoxPreference
            android:key="checkbox_preference"
            android:title="@string/title_checkbox_preference"
            android:defaultValue="@string/default_checkbox_preference"/>
            
        <EditTextPreference
            android:key="edittext_preference"
            android:title="@string/title_edittext_preference"
            android:summary="@string/summary_edittext_preference"
            android:dialogTitle="@string/dialog_title_edittext_preference"
            android:dependency="checkbox_preference" />
 
        </PreferenceCategory>
 
    <PreferenceCategory
        android:title="@string/title_second_section">
 
        <ListPreference
            android:key="list_preference"
            android:title="@string/title_list_preference"
            android:dialogTitle="@string/dialog_title_list_preference"
            android:entries="@array/entries_list_preference"
            android:entryValues="@array/entryvalues_list_preference" />
 
        <Preference
            android:title="@string/title_intent_preference">
            <intent android:action="android.intent.action.VIEW"
                android:data="http://codepath.com/" />
        </Preference>
 
        </PreferenceCategory>
 
</PreferenceScreen>
```

All preferences are saved as key-value pairs in the default `SharedPreferences` with the key specified through the xml above. To retrieve an instance of those preferences, call the following with a context in the same package as the fragment:
```java
SharedPreferences preferences = PreferenceManager.getDefaultSharedPreferences(android.content.Context);
```

The root for the XML file must be a `<PreferenceScreen>`. Within this screen, you can either list all preferences or group with `<PreferenceCategory>`. The grouped preferences will appear together under the same section heading. The XML above creates the following setting screen:

![Settings Screen](https://i.imgur.com/qkE1W6H.png?1=200x)

#### Preference Types

There are several main types of preferences used for settings:
* [ListPreference](http://developer.android.com/reference/android/preference/ListPreference.html): opens dialog with a list of options (see below for defining options); persists a __string__
* [EditTextPreference](http://developer.android.com/reference/android/preference/EditTextPreference.html): opens a dialog with an EditText view for typed input; persists a __string__
*  [CheckBoxPreference](http://developer.android.com/reference/android/preference/CheckBoxPreference.html): uses a checkbox to persist a __boolean__ (true when checked, false when unchecked)
*  [SwitchPreference](http://developer.android.com/reference/android/preference/SwitchPreference.html) (min API 14): uses a switch to persist a __boolean__ (true when on, false when off)

Two of these preference types (`ListPreference` and `EditTextPreference`) descend from [`DialogPreference`](http://developer.android.com/reference/android/preference/DialogPreference.html) and can therefore define dialog-specific attributes in the XML (e.g. a `dialogTitle`).

Note that the `SwitchPreference` type was introduced in API 14 as a subclass of [TwoStatePreference](http://developer.android.com/reference/android/preference/TwoStatePreference.html). At this time,`CheckBoxPreference` was also [implemented to also be a direct subclass of it](https://android.googlesource.com/platform/frameworks/base/+/d54f3f41c4b41955b7b4382a08b97a356b31fde4%5E2..d54f3f41c4b41955b7b4382a08b97a356b31fde4/) rather directly from`Preference`. To use `SwitchPreference`, your app needs to use either min API level 14 or the Android support v4 library.

##### Defining the ListPreference Options
Two arrays must be specified when defining a list array. The first is under `android:entries`, which specifies  human-readable options to display to the user. Second is an array of the values for each corresponding option, which is defined by `android:entryValues`.

```xml
    <ListPreference
        android:key="list_preference"
        android:title="@string/title_list_preference"
        android:dialogTitle="@string/dialog_title_list_preference"
        android:entries="@array/entries_list_preference"
        android:entryValues="@array/entryvalues_list_preference" />
```

#### Intents in Preferences
Preferences can also hold intents which can open a new activity or perform other intent actions. No data is persisted.

```xml
    <Preference
        android:title="@string/title_intent_preference">
        <intent android:action="android.intent.action.VIEW"
            android:data="http://codepath.com/" />
    </Preference>
```
### Java Implementation

Use [`PreferenceFragmentCompat`](https://developer.android.com/reference/android/support/v7/preference/PreferenceFragmentCompat.html) to programatically handle preferences. To load the settings into the fragment, load the preferences in the `onCreatePreferences()` method. To get an instance of a preference, search for a preference using its key through the `PreferenceManager` within `onCreateView()`. You can use this instance to customize the preference, such as add a custom [OnPreferenceChangeListener](http://developer.android.com/reference/android/preference/Preference.OnPreferenceChangeListener.html).
```java
public class SettingsFragment extends PreferenceFragmentCompat {
    
    private ListPreference mListPreference;
    
    ...
    
    @Override
    public void onCreatePreferences(Bundle savedInstanceState, String rootKey) {
        // Indicate here the XML resource you created above that holds the preferences
        setPreferencesFromResource(R.xml.settings, rootKey);
    }
    
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        
        mListPreference = (ListPreference)  getPreferenceManager().findPreference("preference_key");
        mListPreference.setOnPreferenceChangeListener(new Preference.OnPreferenceChangeListener() {
            @Override
            public boolean onPreferenceChange(Preference preference, Object newValue) {
                // your code here
            }
        }
        
        return inflater.inflate(R.layout.fragment_settings, container, false);
    }
}
```

### Custom Preferences

If none of the previous preference types work for your needs, you can create a custom preference extending from [`DialogPreference`](http://developer.android.com/reference/android/preference/DialogPreference.html), [`TwoStatePreference`](http://developer.android.com/reference/android/preference/TwoStatePreference.html), or [`Preference`](http://developer.android.com/reference/android/preference/Preference.html) itself.

For a detailed guide on implementing a custom preference, refer to the [Android Developer: Settings](http://developer.android.com/guide/topics/ui/settings.html#Custom) API guide.

## References

 * <https://developer.android.com/reference/android/support/v7/preference/PreferenceFragmentCompat.html>
 * <http://developer.android.com/guide/topics/ui/settings.html>
 * <http://www.mysamplecode.com/2011/11/android-shared-preferences-example_12.html>
 * <http://mobile.tutsplus.com/tutorials/android/android-user-interface-design-building-application-preference-screens/>
 * <https://storiesandroid.wordpress.com/2015/10/06/android-settings-using-preference-fragments/>
 * <http://stackoverflow.com/questions/32070670/preferencefragmentcompat-requires-preferencetheme-to-be-set/32325283/>