## Overview

ProgressBar is used to display the progress of an activity while the user is waiting. You can display an **indeterminate** progress (spinning wheel) or **result-based** progress.

![ProgressBars](http://i.imgur.com/1nUlHOq.png)

### Indeterminate

We can display an indeterminate progress bar which we show to indicate waiting:

```xml
<ProgressBar
  android:id="@+id/pbLoading"
  android:visibility="invisible" />
```

and then manage the visibility in the activity:

```java
// on some click or some loading we need to wait for...
ProgressBar pb = (ProgressBar) findViewById(R.id.pbLoading);
pb.setVisibility(ProgressBar.visible);
// run a background job and once complete
pb.setVisibility(ProgressBar.invisible);
```

### Result-based

ProgressBar can be used to report the progress of a long-running AsyncTask. In this case:

 * ProgressBar can report numerical results for a task.
 * Must specify horizontal style and result max value.
 * Must `publishProgress(value)` in your AsyncTask

```xml
<ProgressBar
  android:id="@+id/progressBar1"
  android:layout_width="wrap_content"
  android:layout_height="wrap_content"
  android:visibility="invisible"
  style="?android:attr/progressBarStyleHorizontal"
  android:max="100" />
```

and then within the AsyncTask:

```java
public class DelayTask extends AsyncTask<Void, Integer, String> {
	int count = 0;
	@Override
	protected void onPreExecute() {
	  pb.setVisibility(ProgressBar.VISIBLE);
	}

	@Override
	protected String doInBackground(Void... params) {
		while (count < 5) {
			SystemClock.sleep(1000); count++;
			publishProgress(count * 20);
		}
		return "Complete";
	}

	@Override
	protected void onProgressUpdate(Integer... values) {
		 pb.setProgress(values[0]);
	}
	
}
```

and using this pattern any background tasks can be reflected by an on-screen progress report.



## References

 * <http://developer.android.com/reference/android/widget/ProgressBar.html>
 * <http://www.mkyong.com/android/android-progress-bar-example/>