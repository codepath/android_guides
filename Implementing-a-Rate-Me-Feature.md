When you have a published app on the Google Play Store, it is important that you get feedback from your users. Unless your users absolutely love or absolutely hate your app, they're not likely to go out of their way to rate your app on the Play Store. Since high ratings are indicative and predictive of the success of your app, it is critical that you nudge users into rating your app. This guide will go over two different methods for implementing a Rate Me feature:

 * Manually attaching a Rate Me intent to a `Button`
 * Automatically prompt the user to rate your app based on predefined conditions

## Manual Rate Me `Button`

This method is simpler to implement and is less annoying to users because it requires them to first, find your Rate Me `Button`, and second, actually click it and follow through. That said, it is also less likely to get users to actually rate your app. As with any feature that does not directly add to the user experience, you must weigh the negative impact of bugging the user to rate your app with the benefit that ratings have on your app's success.

To implement this method, you just need a `Button` or `View` with an `onClick` handler. For instance, let's say you have the xml-defined `Button`:

```xml
<Button
    android:id="@+id/btnRate"
    android:text="Rate Me!"
    android:onClick="rateMe" >
</Button>
```

The `onClick` handler points to a method called `rateMe`, so in the `Activity` file where this `Button` lives, you must define:

```java
public void rateMe(View view) {
    try {
        startActivity(new Intent(Intent.ACTION_VIEW,
                                 Uri.parse("market://details?id=" + this.getPackageName())));
    } catch (android.content.ActivityNotFoundException e) {
        startActivity(new Intent(Intent.ACTION_VIEW,
                                 Uri.parse("http://play.google.com/store/apps/details?id=" + this.getPackageName())));
    }
}
```

The `try/catch` block is necessary because the user may not have the Play Store installed, in which case we direct the user to a browser version of the Play Store. In either case, notice that this code block can actually be placed anywhere in your app. It's wisest, however, to play nice with the user and only launch this intent when they most expect it (like when they click on a Rate Me `Button`).

## Automatic Rate Me Prompts

For this method, we make use of the open source library [AppRate](https://github.com/TimotheeJeannin/AppRate). Download this [jar](https://github.com/TimotheeJeannin/AppRate/raw/master/AppRateDownloads/AppRate_1.1.jar) and place it in your `libs` folder. In the `onCreate` method of your `MAIN` activity, place the codeblock to launch the prompt:

```java
new AppRate(this).init();
```

The key difference between this method and the manual method is that this one launches a prompt, and only when the user clicks on the positive action in that prompt will they be sent to the Play Store to rate your app.

The above codeblock will launch the prompt as soon as they open your app, which is generally not the desired behavior. You'll most likely want to customize when your prompt occurs. Thankfully, most of the public methods for the [AppRate](https://github.com/TimotheeJeannin/AppRate) object return itself, so you can chain the methods together to easily customize the prompt. The following setup is recommended:

```java
new AppRate(this)
    .setMinDaysUntilPrompt(7)
    .setMinLaunchesUntilPrompt(20)
    .setShowIfAppHasCrashed(false)
    .init()
```

This will not launch the prompt until 7 days have passed and the user has launched the app a minimum of 20 times. Additionally, if the app has crashed at any point before then, the prompt will not launch, even if the other two criteria are met. More information on how to set up the prompt can be found in this [README](https://github.com/TimotheeJeannin/AppRate/blob/master/README.md).

## References

 * <https://github.com/TimotheeJeannin/AppRate>
 * <http://stackoverflow.com/questions/11753000/how-to-open-the-google-play-store-directly-from-my-android-application>