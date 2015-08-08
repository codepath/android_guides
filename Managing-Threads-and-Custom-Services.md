## Overview

This guide focuses on defining custom services including the various mechanisms for sending messages and managing threads to do background work. Android has many different abstractions related to messages and thread management which need to be summarized and understood. These threading abstractions are often related to [defining custom services](http://developer.android.com/guide/components/services.html) because a service by default still runs in the application's main thread and background work must be delegated to threads.

**TODO: Needs Attention**

### Services and Threading

[Services](http://developer.android.com/guide/components/services.html) and [thread management](http://developer.android.com/training/multiple-threads/index.html) are closely related but also distinct topics. A service is simply a component that can run in the background **even when the user is not interacting with your app**. You should only create a service if you **need to perform work while your app isn't open**. 

Thread management is important to understand because a custom service **still runs in your application's main thread by default**. If you create a custom `Service`, then you will still **need to manage the background threads** manually unless the [[IntentService|Starting-Background-Services]] is leveraged.

### Background Services with IntentService 

In 90% of cases when you need a background service doing a task when your app is closed, you will [[leverage the IntentService|Starting-Background-Services]] as your first tool for the job. However, `IntentService` does have a few limitations. The biggest limitation is that the `IntentService` uses a **single worker thread** to handle start requests **one at a time**. However, as long as you don't require that your service handle multiple requests simultaneously, the `IntentService` is typically the easiest tool for the job.

However, in certain specialized cases where you do need background tasks to be processed in parallel using a concurrent thread pool and as such you cannot use `IntentService` and must extend from [Service](http://developer.android.com/reference/android/app/Service.html) directly. The rest of this guide is focused on that particular use case.

## References

* <http://developer.android.com/guide/components/services.html>
* <http://developer.android.com/guide/components/processes-and-threads.html#Threads>
* <http://developer.android.com/training/multiple-threads/index.html>
* <http://developer.android.com/training/multiple-threads/define-runnable.html>
* <http://developer.android.com/training/multiple-threads/create-threadpool.html>
* <http://developer.android.com/training/multiple-threads/communicate-ui.html>
* <http://www.techotopia.com/index.php/A_Basic_Overview_of_Android_Threads_and_Thread_handlers>
* <http://codetheory.in/android-handlers-runnables-loopers-messagequeue-handlerthread/>
* <http://stackoverflow.com/questions/7597742/what-is-the-purpose-of-looper-and-how-to-use-it>
* <http://developer.android.com/reference/java/util/concurrent/ExecutorService.html>
* <http://edisontus.blogspot.com/2014/01/simple-guide-to-executorservice.html>
* <http://codetheory.in/android-java-executor-framework/>
* <http://thegreyblog.blogspot.com/2011/12/using-threadpoolexecutor-to-parallelize.html>
* <http://developer.android.com/guide/components/bound-services.html>
* <http://stackoverflow.com/questions/25246185/what-is-more-efficient-broadcast-receiver-or-handler>
* <http://codetheory.in/android-interprocess-communication-ipc-messenger-remote-bound-services/>
* <http://www.intertech.com/Blog/using-localbroadcastmanager-in-service-to-activity-communications/>