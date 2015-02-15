## Overview

AsyncTask is a mechanism for executing operations in a background thread without having to manually handle thread creation or execution. AsyncTasks were designed to be used for short operations (a few seconds at the most) and you might want to use a [[Service|Starting-Background-Services]] and/or [Executor](http://developer.android.com/reference/java/util/concurrent/Executor.html) for very long running tasks.

This is typically used for long running tasks that cannot be done on UIThread, such as downloading network data from an API or indexing data from elsewhere on the device.

See [Displaying Remote Images the Hard Way](http://guides.codepath.com/android/Sending-and-Managing-Network-Requests#displaying-remote-images-the-hard-way) for an example of how AsyncTask can be used for retrieving a remote image.

### Defining an AsyncTask

Creating an AsyncTask is as simple as defining a class that extends from `AsyncTask` such as: 

```java
// The types specified here are the input data type, the progress type, and the result type
private class MyAsyncTask extends AsyncTask<String, Void, Bitmap> {
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

private void downloadImageAsync() {
    // Now we can execute the long-running task at any time.
    new MyAsyncTask().execute("http://images.com/image.jpg");
}
```

AsyncTask accept three generic types to inform the background work being done:

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

## References

 * <http://developer.android.com/reference/android/os/AsyncTask.html>
 * <http://www.vogella.com/articles/AndroidBackgroundProcessing/article.html>
 * <http://androidresearch.wordpress.com/2012/03/17/understanding-asynctask-once-and-forever/>