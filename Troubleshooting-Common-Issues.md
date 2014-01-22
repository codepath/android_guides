If you are having trouble with Android, Eclipse, or the Emulator, check here for common problems and solutions for [Android development problems](http://www.vogella.com/articles/AndroidDevelopmentProblems/article.html). A few of the most common ones encountered are listed below.

## Eclipse

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

This means that the project does not have an `R` file generated and is usually a sign of a bad state. Try a seres of steps to fix:

1. Try running `Project => Clean` to regenerate the file
2. Look for errors in "Console" or "Problems" to resolve
2. Try relaunching Eclipse

<img src="http://i.imgur.com/ChMkx09.png" width="440" />&nbsp;
<img src="http://i.imgur.com/AXcdNlW.png" width="440" />

### Getting "Error executing aapt: Return code 138" in "Problems"?

Getting this error means your Android installation is likely corrupted in some way. Typically this error will occur on projects in particular cases such as generating a new icon or adding a new xml file. If you see this error in your "Problems" window after a clean, try **closing and reopening Eclipse** first, then try doing a `Project => Clean`. 

If the same message persists, you may need to do a **complete reinstallation** of the [ADT Bundle](http://developer.android.com/sdk/index.html) which includes eclipse. You should delete the entire existing ADT bundle from your computer (including eclipse and SDK folder) and re-download the bundle, extract the contents and re-setup from scratch. Users rarely experience this error after a total reinstall. 

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

Check out [this post for quick fix](http://timvoet.com/2013/01/04/avd-emulator-crashes-on-mac/).In short, we have to edit a file specifying the X/Y coordinates for the virtual device to use to position itself on startup. If these coordinates are outside the normal bounds of the primary monitor, then the emulator will crash.
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