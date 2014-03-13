If you are having trouble with Android, Eclipse, or the Emulator, check here for common problems and solutions for [Android development problems](http://www.vogella.com/articles/AndroidDevelopmentProblems/article.html). A few of the most common ones encountered are listed below.

## Eclipse

### Generated Project Missing Activities?

This problem will cause new Android projects not to generate an initial Activity, even when that box was checked. `/src` and `/res/layout` were empty. 

Fixed by updating the Eclipse plugin: Eclipse --> Help --> Install New Software --> Pasted URL `http://dl-ssl.google.com/android/eclipse/`, check "Developer Tools" and hit Finiah. Leave unchecked "Contact all update sites during install to find required software". Now regenerate the project.

<img src="http://i.imgur.com/Hg8QXDP.png" width="460" />

See [this stackoverflow post](http://stackoverflow.com/questions/22219392/eclipse-doesnt-create-main-activity-and-layout) and [this other one](http://stackoverflow.com/questions/6470802/what-to-do-about-eclipses-no-repository-found-containing-error-messages) for more details.

### Problem with Autocompletion?

You should be able to use Autocompletion (Ctrl + Space) to complete words as you code. If you can't, try the following steps:

1. Try restarting eclipse, quit and re-open. Check to see if autocompletion has returned. 
2. Open eclipse and go to the following in the menu: `Preferences > Java > Editor > Content Assist > Advanced`. Select all checkboxes here (especially ones mentioning Java) and click OK.
3. Open eclipse and go to the following in the menu: `Preferences > General > Keys` and the find the Command "Content Assist" in the list and remap the "Binding" to a different set of keys and click OK.

<img src="http://i.imgur.com/FT2hAm4.png" width="440" />
<img src="http://i.imgur.com/fopm3WG.png" width="460" />

### Getting "Cannot Be Resolved to a Type" Error 

See a red line under a class that should exist (i.e ArrayList, View)? - Hover over the class and select "import" from the list of suggestions or better use "Cmd + Shift + O" to auto-import all missing types.

![Red Line](http://i.imgur.com/ibxttW4.png)

### Getting "R cannot be resolved to a variable" errors?

This means that the project does not have an `R` file generated and is usually a sign of an **invalid file within res**. Try a seres of steps to fix:

1. Look for errors in "Console" or "Problems" to resolve (usually invalid XML)
2. Try running `Project => Clean` to regenerate the `R` file
3. Try closing and relaunching Eclipse

<img src="http://i.imgur.com/ChMkx09.png" width="440" />&nbsp;
<img src="http://i.imgur.com/AXcdNlW.png" width="440" />

### Getting "Error executing aapt: Return code 138" in "Problems"?

Getting this error means your Android installation is likely corrupted in some way. Typically this error will occur on projects in particular cases such as generating a new icon or adding a new xml file. If you see this error in your "Problems" window after a clean, try **closing and reopening Eclipse** first, then try doing a `Project => Clean`. 

If the same message persists, you may need to do a **complete reinstallation** of the [ADT Bundle](http://developer.android.com/sdk/index.html) which includes eclipse. You should delete the entire existing ADT bundle from your computer (including eclipse and SDK folder) and re-download the bundle, extract the contents and re-setup from scratch. Users rarely experience this error after a total reinstall. 

### Getting "Unable to execute dex: Multiple dex files define"?

This problem prevents a project from being built, and instead displays the error "Unable to execute dex: Multiple dex files define" when compiling is attempted. The issue is that the bin directory becomes included in the project build path. Excluding the bin from the build path often resolves the problem. Right click on the project name, select `Build Path -> Configure Build Path` and in Java Build Path, go to the tab `Order and Export`. See the full details of the [solution here](http://stackoverflow.com/a/7884908). 

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