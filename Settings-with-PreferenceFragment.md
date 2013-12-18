## Overview

In Android apps, there are often settings pages that contain different options the user can tweak. The [PreferenceFragment](http://developer.android.com/reference/android/preference/PreferenceFragment.html) contains a hierarchy of preference objects displayed on screen in a list. These preferences will automatically save to SharedPreferences as the user interacts with them.

To retrieve an instance of `SharedPreferences` that the preference hierarchy in this fragment will use, call `PreferenceManager.getDefaultSharedPreferences(android.content.Context)` with a context in the same package as this fragment.

## Needs attention

## References

 * <http://developer.android.com/reference/android/preference/PreferenceFragment.html>
 * <http://developer.android.com/guide/topics/ui/settings.html>
 * <http://www.mysamplecode.com/2011/11/android-shared-preferences-example_12.html>
 * <http://mobile.tutsplus.com/tutorials/android/android-user-interface-design-building-application-preference-screens/>
 * <http://gmariotti.blogspot.com/2013/01/preferenceactivity-preferencefragment.html>