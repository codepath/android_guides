## Overview

Android for Work enables your mobile devices to be managed centrally by enterprises as well to help mitigate issues with lost or stolen devices.  You can leverage Google's own enterprise mobile management (EMM) system, or you can use a third-party [approved vendor list](https://www.google.com/work/android/partners/).  Using Android for Work, you can also leverage [managed app configurations](https://developer.android.com/work/managed-configurations.html), which allow a way to push settings to apps for large scale deployments.

## Setup

In order to use Android for Work, you must have a Google Apps domain setup.  In addition, you must also convert your Google Apps domain to a Google Apps for Work, which is a paid subscription service.  Standard Google Apps domains cannot be used, but you can upgrade to a 30-day free trial service to evaluate the service.    If you choose not to use it, you can easily downgrade too.

Next, make sure to click on the `Android for Work` option:

<img src="http://i.imgur.com/Ccg7pdi.png"/>

Once this option is selected, you will need to either rely on Google to manage the devices, or you can use a third-party solution.  If you intend to use a third-party solution, you will need to generate a token that can be used.  If you wish to try to use Google's, you can always unbind the selection.

<img src="http://i.imgur.com/EyiIQS1.png">

### Using a Third-Party Solution

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
