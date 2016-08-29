## Overview

Android for Work enables your mobile devices to be managed centrally by enterprises as well to help mitigate issues with lost or stolen devices.  You can leverage Google's own enterprise mobile management (EMM) system, or you can use a third-party [approved vendor list](https://www.google.com/work/android/partners/).  Using Android for Work, you can also leverage [managed app configurations](https://developer.android.com/work/managed-configurations.html), which allow a way to push custom key/value pairs to apps for large scale deployments.

## Setup

In order to use Android for Work, you either need to do the following:

1) **Signup for a Google Apps for Work (Gmail, Calendar, Drive)** through [apps.google.com](http://apps.google.com).

If you want to use this option, you need to upgrade to a paid subscription service but you can upgrade to a 30-day free trial service to evaluate the solution first.   Also, if you choose not to use this paid offering, you can easily downgrade without any cost. 

Once you have upgraded to a paid subscription plan, make sure to click on the `Android for Work` option inside the admin console:

<img src="http://i.imgur.com/Ccg7pdi.png" width="450"/>

Once this option is selected, you will need to either rely on Google to manage the devices, or you can use a third-party solution.  

2) **Signup for Android for Work using a personal Gmail account** using this [registration link](https://www.google.com/a/signup/?enterprise_product=ANDROID_WORK).

You can only use this option if you are not currently using Google Apps to manage your domain.   You will also need at least a personal Gmail account that will serve as the administrator.  Note that **Android for Work accounts do NOT have any access to paid products such as Gmail, Google Calendar, and Drive**.  You can only use this option in conjunction with a third-party EMM system.

Upon signing up, your screenshot will look like:

<img src="http://imgur.com/tODDzBG.png" width="450"/>

You will need to validate that you own the domain name (DNS).  The account and password that you used in the signup page can be used to relogin to [http://admin.google.com](http://admin.google.com) similar to other Google logins.  The one difference is that this account has no access to other paid products such as Gmail, Google Calendar, and Drive.  See this [reference guide](https://support.google.com/work/android/answer/6371476?hl=en&ref_topic=6151012) for more details.

### EMM Tokens

If you are using Android for Work accounts or use a third-party solution instead of Google's (only applicable with using with Google Apps for Work), you will need to generate a token that can be used.  If you wish to try to use Google's, you can always unbind the selection.

<img src="http://i.imgur.com/EyiIQS1.png">

### Using a Third-Party EMM Provider

If you use a third-party solution from the approved vendor list, you are likely to need to grant access to the Google Play EMM API.  

1. Visit [https://console.developers.google.com](https://console.developers.google.com) with the Google Apps domain.  

2. Create a new project or use an existing one.

3. Make sure to enable EMM API

   <img src="http://i.imgur.com/GyMQzmX.png"/>

4. Go the `Credentials` section and create a service account.  Save the credential file to JSON format.

5. Upload this JSON data along with the EMM token you generated to the vendor system you've chosen.   You may need to follow other steps as provided by the vendor to complete the necessary steps.

## References

* <https://support.google.com/work/android>
* <https://developer.android.com/work/managed-configurations.html>
