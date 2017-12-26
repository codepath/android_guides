## Overview

AsyncTask is a mechanism for executing operations in a background thread without having to manually handle thread creation or execution. AsyncTasks were designed to be used for short operations (a few seconds at the most) and you might want to use a [[Service|Starting-Background-Services]] and/or [Executor](http://developer.android.com/reference/java/util/concurrent/Executor.html) for very long running tasks.

This is typically used for long running tasks that cannot be done on UIThread, such as downloading network data from an API or indexing data from elsewhere on the device. An `AsyncTask` streamlines the following common background process:

1. **Pre** - Execute code on the UI thread before starting a task (e.g show ProgressBar)
2. **Task** - Run a background task on a thread given certain inputs (e.g fetch data)
3. **Updates** - Display progress updates during the task (optional)
4. **Post** - Execute code on UI thread following completion of the task (e.g show data)

See [[Displaying Remote Images the Hard Way|Sending-and-Managing-Network-Requests#displaying-remote-images-the-hard-way]] for an example of how AsyncTask can be used for retrieving a remote image.

### Defining an AsyncTask

Creating an AsyncTask is as simple as defining a class that extends from `AsyncTask` such as: 

```java
// The types specified here are the input data type, the progress type, and the result type
private class MyAsyncTask extends AsyncTask<String, Progress, Bitmap> {
     protected void onPreExecute() {
         // Runs on the UI thread before doInBackground
         // Good for toggling visibility of a progress indicator
         progressBar.setVisibility(ProgressBar.VISIBLE);
     }

     protected Bitmap doInBackground(String... strings) {
         // Some long-running task like downloading an image.
         Bitmap = downloadImageFromUrl(strings[0]);
         return someBitmap;
     }

     protected void onProgressUpdate(Progress... values) {
        // Executes whenever publishProgress is called from doInBackground
        // Used to update the progress indicator
        progressBar.setProgress(values[0]);
     }  

     protected void onPostExecute(Bitmap result) {
         // This method is executed in the UIThread
         // with access to the result of the long running task
         imageView.setImageBitmap(result);
         // Hide the progress bar
         progressBar.setVisibility(ProgressBar.INVISIBLE);
     }
}
```

### Executing the AsyncTask

The worker once defined can be started anytime by creating an instance of the class and then invoke `.execute` to start the task:

```java
public void onCreate(Bundle b) {
   // ...
   // Initiate the background task
   downloadImageAsync();
}

private void downloadImageAsync() {
    // Now we can execute the long-running task at any time.
    new MyAsyncTask().execute("http://images.com/image.jpg");
}
```

### Understanding the AsyncTask

AsyncTask accepts three generic types to inform the background work being done:

* `AsyncTask<Params, Progress, Result>`
  * `Params` - the type that is passed into the execute() method.
  * `Progress` - the type that is used within the task to track progress.
  * `Result` - the type that is returned by doInBackground().

For example `AsyncTask<String, Void, Bitmap>` means that the task requires a string input to execute, does not record progress and returns a Bitmap after the task is complete.

AsyncTask has multiple events that can be overridden to control different behavior:

 * `onPreExecute` - executed in the main thread to do things like create the initial progress bar view.
 * `doInBackground` - executed in the background thread to do things like network downloads.
 * `onProgressUpdate` - executed in the main thread when `publishProgress` is called from doInBackground.
 * `onPostExecute` - executed in the main thread to do things like set image views.

### Limitations

An `AsyncTask` is tightly bound to a particular `Activity`. In other words, if the `Activity` is destroyed or the configuration changes then the `AsyncTask` will not be able to update the UI on completion. As a result, for short one-off background tasks **tightly coupled to updating an Activity**, we should consider using an `AsyncTask` as outlined above. A good example is for a several second network request that will populate data into a `ListView` within an activity.

### Custom Thread Management

Using the `AsyncTask` is the easiest and most convenient way to manage background tasks from within an Activity. However, in cases where tasks need to be processed in parallel with more control, or the tasks **need to continue executing even when the activity leaves the screen**, you'll need to create a [[background service|Starting-Background-Services]] or [[manage threaded operations|Managing-Threads-and-Custom-Services]] more manually.

## References

 * <http://developer.android.com/reference/android/os/AsyncTask.html>
 * <http://www.vogella.com/articles/AndroidBackgroundProcessing/article.html>
 * <http://androidresearch.wordpress.com/2012/03/17/understanding-asynctask-once-and-forever/>