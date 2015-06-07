If you are having trouble with Eclipse or the Emulator, check here for common problems and solutions for [Android development problems](http://www.vogella.com/articles/AndroidDevelopmentProblems/article.html). A few of the most common issues encountered are listed below.

## Setting Up Eclipse

### Project generation fails with "Errors running builder 'Android Resource Manager' on project"?

When generating a new project in Eclipse, the generation fails:

<img src="http://i.imgur.com/j0j986J.png" width="600" />

Here's a workaround that lets you keep Java 7 as the default but run ADT with Java 6 when you have both installed. Find the Eclipse app in `<YOUR_ADT_PATH>/eclipse/Eclipse.app` and right click and select "Show Package Contents" and then navigate to `/Contents/MacOS/eclipse.ini` in an editor.  Before the `-vmargs` line, insert these two lines:

```
-vm
/System/Library/Frameworks/JavaVM.framework/Versions/1.6/Commands/java
```

Read [comments about the workaround which launches ADT with Java 6 instead](https://code.google.com/p/android/issues/detail?id=68755#c18) in this bug report for this issue. 

### Generated Project Missing Activities?

This problem will cause new Android projects not to generate an initial Activity, even when that box was checked. `/src` and `/res/layout` were empty. 

Fixed by updating the Eclipse plugin: `Eclipse > Help > Install New Software` and enter the URL `http://dl-ssl.google.com/android/eclipse/` into "Work with", make sure "Developer Tools" is checked and hit `Finish`. Leave unchecked "Contact all update sites during install to find required software". Now regenerate the project.

<img src="http://i.imgur.com/Hg8QXDP.png" width="460" />

See [this stackoverflow post](http://stackoverflow.com/questions/22219392/eclipse-doesnt-create-main-activity-and-layout) and [this other one](http://stackoverflow.com/questions/6470802/what-to-do-about-eclipses-no-repository-found-containing-error-messages) for more details.

### Generated Project has Placeholder Fragment

**Note:** Easiest way to avoid placeholder fragments is to update to latest eclipse and select **Empty Activity** rather than **Blank Activity** when generating activities which does not include the fragment placeholder.

<img src="http://i.imgur.com/u3r3LPd.png" width="460" />

In recent versions of Eclipse, a "Blank Activity" is generated with a built-in placeholder fragment which can't be easily disabled. If you want to remove this placeholder fragment from a "Blank Activity" and work directly with the activity, you can follow these steps:

1. Copy contents of fragment XML at `res/layout/fragment_main.xml`
2. Replace entire contents of activity XML with the contents of the fragment XML
3. Delete the fragment XML at `res/layout/fragment_main.xml`
4. Remove the if statement in the `onCreate` in the Activity class file at `src/com.../MainActivity.java`
5. Remove the `PlaceholderFragment` inner class defined within `src/com.../MainActivity.java` 

See [this guide for a more detailed set of instructions](https://gist.github.com/nesquena/8de33f701e387be0a80d) with images.

### Getting "java.lang.IllegalStateException: You need to use a Theme.AppCompat theme..."

First, try simply running `Project => Clean...` from the top-menu on the project and then re-running the app. If it still doesn't work, it might be the issue explained in the next paragraph.

This happens when the Android project was generated with a minimum SDK of 10 or below. One fix is to simply generate new projects with a minimum SDK of 14 instead. However when generating with a lower minSDK, in order to maintain compatibility with older versions, you'll notice that the activity Java class (i.e src/.../MainActivity.java) extends from `AppCompatActivity` rather than the standard `Activity` class. When an activity extends from `AppCompatActivity`, this requires the app to use a "backwards compatible theme". Easiest fix is to change the theme for the application within the `AndroidManifest.xml` such that `application:theme` is set to `@style/Theme.AppCompat.Light.DarkActionBar` as shown below:

<img src="http://i.imgur.com/aIXHlQM.gif" width="750" />

See [this issue](http://stackoverflow.com/questions/18063395/actionbarcompat-java-lang-illegalstateexception-you-need-to-use-a-theme-appcom), [this issue](http://stackoverflow.com/questions/21814825/you-need-to-use-a-theme-appcompat-theme-or-descendant-with-this-activity) or [this issue](http://stackoverflow.com/questions/21331836/java-lang-illegalstateexception-you-need-to-use-a-theme-appcompat-theme-or-des) for more details on this issue.

### Problem with Autocompletion?

You should be able to use Autocompletion (Ctrl + Space) to complete words as you code. If you can't, try the following steps:

1. Try restarting eclipse, quit and re-open. Check to see if autocompletion has returned. 
2. Open eclipse and go to the following in the menu: `Preferences > Java > Editor > Content Assist > Advanced`. Select all checkboxes here (especially ones mentioning Java) and click OK.
3. Open eclipse and go to the following in the menu: `Preferences > General > Keys` and the find the Command "Content Assist" in the list and remap the "Binding" to a different set of keys and click OK.

<img src="http://i.imgur.com/FT2hAm4.png" width="440" />
<img src="http://i.imgur.com/fopm3WG.png" width="460" />


## Using Eclipse

### Getting "Cannot Be Resolved to a Type" Error 

See a red line under a class that should exist (i.e ArrayList, View)? - Hover over the class and select "import" from the list of suggestions or better use "Cmd + Shift + O" to auto-import all missing types.

![Red Line](http://i.imgur.com/ibxttW4.png)

### Getting "R cannot be resolved to a variable" errors?

This means that the project does not have an `R` file generated and is usually a sign of an **invalid file within res**. Try a series of steps to fix:

1. Look for errors in "Console" or "Problems" to resolve (usually invalid XML)
2. Try running `Project => Clean` to regenerate the `R` file
3. Try closing and relaunching Eclipse

<img src="http://i.imgur.com/ChMkx09.png" width="440" />&nbsp;
<img src="http://i.imgur.com/AXcdNlW.png" width="440" />

Be sure to check for these possibilities if that does not work, this is an indication there is an issue with your resources (within `res`):

 * Check the `res/menu` XML files **very carefully**. Do the `@drawable` and `@string` references resolve? Is the `showAsAction` specified as `app:`. If it is, try switching them to have an `android:` prefix.
 * Check the `res/drawable*` files and make sure they are named appropriately (only lowercase letters, numbers and underscores)

A more comprehensive guide to every possible way this error can crop up can be found in this [comprehensive troubleshooting guide](http://www.techrepublic.com/blog/software-engineer/a-comprehensive-troubleshooting-guide-for-androids-r-cannot-be-resolved-error/).

### Getting "R cannot be resolved" or "resource can not be resolved" errors?

This common issue is a beginner gotcha. In certain cases, you will accidentally import `android.R` which will shadow the local resources with the built-in Android resources.

<img src="http://i.imgur.com/n9WNqat.png" width="440" />&nbsp;
<img src="http://i.imgur.com/ZUnGWU1.png" width="440" />

The simple fix is to **make sure you don't import android.R**. Remove that line at the top of your source file. See [this StackOverflow post](http://stackoverflow.com/questions/885009/r-cannot-be-resolved-android-error) to learn more about this issue. A more comprehensive guide to every possible way this error can crop up can be found in this [comprehensive troubleshooting guide](http://www.techrepublic.com/blog/software-engineer/a-comprehensive-troubleshooting-guide-for-androids-r-cannot-be-resolved-error/).

### Getting "Error executing aapt: Return code 138" in "Problems"?

The first culprit of this error is simply an **invalid id specified** in one of your Android XML files. For example if one of your android layout or menu XML files has an "android:id" specified that is incorrect such as "btn_foo" or "id/btn_foo" instead of the correct "@+id/btn_foo", you may experience this issue for a project.

Another culprit might be having an `app:` prefix in your `res/menu` files. Check your menu files and verify that if you don't have the support-v7 library included that your menu xml files **do not contain** `app:showAsAction` but instead have `android:showAsAction`. Make sure to clean and rebuild once you've made this change.

Getting this error could also mean your Android project compilation is likely corrupted in some way. Typically this error will occur on projects in particular cases such as generating a new icon or adding a new xml file. If you see this error in your "Problems" window after a clean, try **closing and reopening Eclipse** first, then try doing a `Project => Clean`. 

### Getting "Unable to execute dex: Multiple dex files define" or "java.nio.BufferOverflowException"?

This problem prevents a project from being built, and instead displays the error "Unable to execute dex: Multiple dex files define" when compiling is attempted. The issue is that the bin directory becomes included in the project build path. Excluding the bin from the build path often resolves the problem. Right click on the project name, select `Build Path -> Configure Build Path` and in Java Build Path, go to the tab `Order and Export`. See the full details of the [solution here](http://stackoverflow.com/a/7884908). 

### Getting "gen already exists but is not a source folder"?

This problem can be fixed by following these steps:

1. Right click on the project and go to "Properties"
2. Select "Java Build Path" on the left
3. Open the "Source" tab
4. Click "Add Folder..."
5. Check the "gen" folder and click "Ok" and "Ok" again
6. Right click on the project and select `Android Tools => Fix Project Properties`

See this [stackoverflow post](http://stackoverflow.com/questions/6185600/android-fbreaderj-gen-already-exists-but-is-not-a-source-folder-convert-to-a-s) for the full details.

### Getting "Unable to instantiate activity ComponentInfo{...}"?

If this error contains information about the `java.lang.ClassNotFoundException` then this usually means one the following issues:

1. Please verify that every Activity is registered in your `AndroidManifest.xml`
2. Check that you're Activity is a public class and not protected or private
3. Ensure that any dependency jars are checked in the "Java Build Path" 

For more possibilities, check out this [stackoverflow post](http://stackoverflow.com/questions/4688277/java-lang-runtimeexception-unable-to-instantiate-activity-componentinfo) for the full details.

### Getting "java.lang.ClassCastException: android.widget.Foo cannot be cast to android.widget.Bar"?

There are several possible reasons for this exception. The most logical is that in the code you are literally casting a view to an incorrect type. For example, suppose we defined a layout XML with:

```xml
<EditText
    android:id="@+id/etNewItem"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content" />
```

Now suppose in the Activity code we write:

```java
ImageView etNewItem = (ImageView) findViewById(R.id.etNewItem);
```

This would generate the error above "java.lang.ClassCastException: android.widget.EditText cannot be cast to android.widget.ImageView". First, **check if you have improperly cast a view to the wrong type**.

If that doesn't work, and the view appears to be properly cast to the correct type, then the resource mapping may simply be confused. In that case, as [explained here](http://stackoverflow.com/a/20215613/313399), you need to **clean your project** by running `Project => Clean...` from the menu. Cleaning will regenerate the resource mappings and often fix this issue.

### Imported Project Won't Compile

If you have imported an existing Android project into the Eclipse workspace and the project gives an error such as "cannot resolve target android-17" or another obscure error, follow this checklist below.

**Select Build Target**

Right click on the project name in the package explorer on the left side and select "Properties". Select "Android" on the left side and then pick a sensible "Project Build Target" (i.e Android 4.4.2). 

<img src="http://i.imgur.com/UiMC3iN.png" width="450" />

**Fix Imported Libraries**

Right click on the project name in the package explorer on the left side and select "Properties". Select "Android" on the left side and verify there are no "missing" libraries. If there is a missing library project, remove that library and add the library into your project using the "Add" button. Keep in mind you need to have the necessary library projects loaded into eclipse for this to work. (See screenshot above)

**Fix Build Path**

Right click on the project name in the package explorer on the left side and select "Properties". Select "Java Build Path" on the left side and select the tab "Libraries". Expand all the groups displayed there. Ensure that there are no "missing" libraries or folders on the build path.

<img src="http://i.imgur.com/ZfZPZtS.png" width="450" />

**Fix Project Properties**

Right click on the project name in the package explorer on the left side and select `Android Tools => Fix Project Properties`.

<img src="http://i.imgur.com/tmXvUeX.png" width="450" />

**Clean the Project**

Select `Project => Clean` and then either select "Clean all projects" or ensure the broken project is selected in the list. Then hit OK. 

**Restart Eclipse**

If after all these steps and a clean the project is still not able to compile, try restarting eclipse and see if the project is able to compile successfully after a reboot.

## Emulator

### App running in emulator but no logs showing up in LogCat?

* Switch to DDMS mode in Eclipse
* Verify the emulator is listed and select the emulator
* Select the small down arrow and click "reset adb"
* Still having problems? Time to restart eclipse

![Reset ADB](http://i.imgur.com/5pj2rZA.png)

### Cannot access the internet 

In some cases, even after requesting Internet access in the Manifest.xml, you might experience problems when trying to access Internet.

<img src="http://i.imgur.com/aCuexMz.png" width="440" />

One fix for this is to manually specify DNS in `Configuration / Target / Emulator` launch parameters: `-dns-server 8.8.8.8` (that's a Google one). Make sure to **restart the emulator** after making this change to see the effect.

### Booting the emulator is slow?

* Open "Window => Android Virtual Device Manager"
* Click "Edit" on your Virtual Device and verify "Intel x86" is selected for CPU
* Make sure not to close your emulator once it is booted, leave it open and just re-run

![Intel HAXM](http://i.imgur.com/gU6KWNO.png)

### Started emulator either doesn't boot, freezes computer or looks glitchy

* Go to [Intel HAXM page](http://software.intel.com/en-us/articles/intel-hardware-accelerated-execution-manager) and **install the latest hotfixes** for your platform.
* Open "Window => Android Virtual Device Manager"
* Click "Edit" on your Virtual Device and toggle the "Use Host GPU" checkbox
* Verify the CPU has "Intel x86" selected rather than "ARM"
* Now fully restart the emulator and relaunch

![Host GPU](http://i.imgur.com/5pMO0PH.png)

### Emulator is crashing unexpectedly and my mac has dual-monitors

Check out [this post for quick fix](http://timvoet.com/2013/01/04/avd-emulator-crashes-on-mac/). In short, we have to edit a file specifying the X/Y coordinates for the virtual device to use to position itself on startup. If these coordinates are outside the normal bounds of the primary monitor, then the emulator will crash.
 * Edit the file `~/.android/avd/.avd/emulator-user.ini`
 * Reset the value of both `window.x` and `window.y` to 0

## Android

### I am getting an error "Application has stopped" in a dialog and my app closes

* This means your application had a runtime crash
* If you don't see LogCat, open with `Window => Show View => Other => Android => LogCat`
* Use LogCat to debug where the crash originated (or use the debugger)
* Find the error stacktrace and identify the line (in your code) that triggered the error.
* This is often a NullPointerException (accessing a null object) or other null references (such as an `android:onClick` handler referencing a non-existent method)

<img src="http://i.imgur.com/851bDxf.png" alt="Show LogCat" width="300">
<img src="http://i.imgur.com/lZlJ2z1.png" alt="LogCat" width="600">