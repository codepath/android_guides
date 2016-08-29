## Overview

Android for Work enables your mobile devices to be managed centrally by enterprises as well to help mitigate issues with lost or stolen devices.  You can leverage Google's own enterprise mobile management (EMM) system, or you can use a third-party [approved vendor list](https://www.google.com/work/android/partners/).  Using Android for Work, you can also leverage [managed app configurations](https://developer.android.com/work/managed-configurations.html), which allow a way to push custom key/value pairs to apps for large scale deployments.

## Setup

In order to use Android for Work, you either need to 1) signup for a Google Apps for Work (Gmail, Calendar, Drive) or 2) use a personal Gmail account to provision for an Android for Work account.  If you want to use the Google Apps for Work option, you need to upgrade to a paid subscription service but you can upgrade to a 30-day free trial service to evaluate the solution first.   Also, if you choose not to use this paid offering, you can easily downgrade without any cost. 

For the 2nd option, you need to fill out this [registration link](https://www.google.com/a/signup/?enterprise_product=ANDROID_WORK) to manage Android for Work accounts.  You can only use this option if you are not currently using Google Apps to manage your domain.   You will also need at least a personal Gmail account that will serve as the administrator.  Note that **Android for Work accounts do NOT have any access to paid products such as Gmail, Google Calendar, and Drive**.  You will mostly use it in conjunction with a third-party EMM system.

See this [reference guide](https://support.google.com/work/android/answer/6371476?hl=en&ref_topic=6151012) for more details about the differences between using a Google Apps for Work domain vs. using an Android for Work account with a persona Gmail account.

### Google Apps for Work Setup

If you are using Google Apps for Work, inside the admin console, make sure to click on the `Android for Work` option:

<img src="http://i.imgur.com/Ccg7pdi.png"/>

Once this option is selected, you will need to either rely on Google to manage the devices, or you can use a third-party solution.  

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
