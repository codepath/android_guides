If you are having trouble with Android, Eclipse, or the Emulator, check here for common problems and solutions for [Android development problems](http://www.vogella.com/articles/AndroidDevelopmentProblems/article.html). A few of the most common ones encountered are listed below.

## Eclipse

**Problem with Autocompletion?** - You should be able to use Autocompletion (Ctrl + Space) to complete words as you code. If you can't, try the following steps:

1. Try restarting eclipse, quit and re-open. Check to see if autocompletion has returned. 
2. Open eclipse and go to the following in the menu: `Preferences > Java > Editor > Content Assist > Advanced`. Select all checkboxes here (especially ones mentioning Java) and click OK.
3. Open eclipse and go to the following in the menu: `Preferences > General > Keys` and the find the Command "Content Assist" in the list and remap the "Binding" to a different set of keys and click OK.

<img src="http://i.imgur.com/FT2hAm4.png" width="450" />
<img src="http://i.imgur.com/fopm3WG.png" width="470" />

**See a red line under a class that should exist (i.e ArrayList, View)?** - Hover over the class and select "import" from the list of suggestions or better use "Cmd + Shift + O" to auto-import all missing types.

![Red Line](http://i.imgur.com/ibxttW4.png)

## Emulator

**App running in emulator but no logs showing up in LogCat?**

* Switch to DDMS mode in Eclipse
* Verify the emulator is listed and select the emulator
* Select the small down arrow and click "reset adb"
* Still having problems? Time to restart eclipse

![Reset ADB](http://i.imgur.com/5pj2rZA.png)

**Booting the emulator is slow?**

* Open "Window => Android Virtual Device Manager"
* Click "Edit" on your Virtual Device and verify "Intel x86" is selected for CPU
* Make sure not to close your emulator once it is booted, leave it open and just re-run

![Intel HAXM](http://i.imgur.com/gU6KWNO.png)

**Started emulator either doesn't boot, freezes computer or looks glitchy**

* Go to [Intel HAXM page](http://software.intel.com/en-us/articles/intel-hardware-accelerated-execution-manager) and **install the latest hotfixes** for your platform.
* Open "Window => Android Virtual Device Manager"
* Click "Edit" on your Virtual Device and toggle the "Use Host GPU" checkbox
* Verify the CPU has "Intel x86" selected rather than "ARM"
* Now fully restart the emulator and relaunch

![Host GPU](http://i.imgur.com/5pMO0PH.png)

## Android

**I am getting an error "Application has stopped" in a dialog and my app closes**

* This means your application had a runtime crash
* If you don't see LogCat, open with `Window => Show View => Other => Android => LogCat`
* Use LogCat to debug where the crash originated (or use the debugger)
* Find the error stacktrace and identify the line (in your code) that triggered the error.
* This is often a NullPointerException (accessing a null object) or other null references (such as an `android:onClick` handler referencing a non-existent method)

![Show LogCat](http://i.imgur.com/851bDxf.png)
![LogCat](http://i.imgur.com/lZlJ2z1.png)