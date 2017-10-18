## Overview

While setting up breakpoints in your code is usually the first place to check, it often can be useful to see the network traffic exchanged between your Android device and a server.  There are often things you want to check:

* Am I using the correct API key?
* Am I passing in the correct HTTP GET or POST parameters?
* Do I have the right URL pattern?
* Did I properly format the parameters being sent?
* What is the server error code?
* What is the server response?

You should first try to verify whether your API calls are correct by visiting [https://dhc.restlet.com/](https://dhc.restlet.com/).   You can also install a Chrome browser extension [Dev HTTP Client](https://chrome.google.com/webstore/detail/dhc-rest-client/aejoelaoggembcahagimdiliamlcdmfm) or any other available REST-based client tester. 

<img src="https://imgur.com/GfVK80o.png" width="600"/>

In other cases, it is often useful to monitor the network traffic to help diagnose these issues.  Monitoring network traffic depends on the networking library you use.   There are two general approaches: using the Stetho library or setting up a proxy.  The former is the simplest but requires the [[OkHttp|Using-OkHttp]] or [[Retrofit|Consuming-APIs-with-Retrofit]] networking library to be used.

### Using Stetho

If you are using OkHttp or Retrofit, you can take advantage of Facebook's [Stetho](http://facebook.github.io/stetho) project that will allow you to leverage the Chrome network traffic inspector.  You can walk through [[this guide|Debugging with Stetho]] for more details.

<img src="http://facebook.github.io/stetho/static/images/inspector-network.png"/>

### Setting up a proxy

If you are using [[Android Async Http Client|Using-Android-Async-Http-Client]] or any other networking library that relies on Android's legacy Apache HttpClient implementation, you cannot leverage the Stetho project and Chrome project as noted in this [issue](https://github.com/facebook/stetho/issues/116).  However, you can still setup an HTTP proxy that can intercept the network requests. 

The process requires two parts: one on the PC that will act as the proxy, and the other on the Android device.  It will also walk you through installing a root SSL certificate that can be used by the proxy to inspect SSL network requests.

**Note**: one drawback of this approach is that if you are using a network where the IP address of your PC changes, you will need to re-update the proxy settings on the device each time.

***On your PC***:

1. Download a 30-day trial of [Charles Proxy](https://www.charlesproxy.com/download/).  There are versions available for Windows, Mac OS, and Linux.

2. You will need to insert a special root certificate in order to inspect SSL traffic.  Run Charles Proxy and navigate to `Help` -> `SSL Proxying` -> `Install Charles Root Certificate on a Mobile Device or Remote Browser`.

     <img src="https://imgur.com/Ac5QR0x.png"/>

     You should see `Configure your device to use Charles as its HTTP proxy on xxxx:x:x:x:xxxx:xxx:xxxx:xxxxxx:8888, then browse to chls.pro/ssl to download and install the certificate.`    Take note of the URL that you should be using (i.e. chls.pro/ssl).

3. SSL sites that must be inspected must be explicitly declared by going to `Proxy` -> `SSL Proxying Settings`.  Type the domain name and port 443 assuming the site is connecting to a standard SSL port.

     <img src="https://imgur.com/YXTqq93.png"/>

4. Disable the proxy from being used on your desktop web browser by simply going to `Proxy` -> `Proxy Settings` and uncheck the `Enable macOS proxy` option:

     <img src="https://imgur.com/zzWkuEX.png"/>

5. Make sure to ascertain your IP address of the machine with the proxy installed.  You can go to the `Help` -> `Local IP Address` to find it:

<img src="https://imgur.com/AwbbEwA.png"/>

***On your phone:***

1. Open your Android device and go to [http://chls.pro/ssl](http://chls.pro/ssl).  Download and install the root certificate on the phone.   If you are using a Genymotion emulator, it may be easier to install Chrome first to download this cert, which requires [[setting up Google Play Services|Genymotion-2.0-Emulators-with-Google-Play-support#setup-google-play-services]] first.

2. Go to your WiFi settings and modify the existing network:

     <img src="https://imgur.com/DXqpvWl.png"/>

3. Select on advanced options, set the proxy settings to manual, and enter the IP address and port 8888.  
     
     <img src="https://imgur.com/AclSz0z.png"/> 

4. Connect to any web site.  You need to go back to the PC to grant authorization for the device to connect:

     <img src="https://imgur.com/yuRmGRC.png">

5. If you are using AsyncHttpClient, you have to set the proxy settings manually (see [this GitHub issue](https://github.com/loopj/android-async-http/issues/971) for more details):

     ```java
     AsyncHttpClient client = new AsyncHttpClient();
     client.setProxy(System.getProperty("http.proxyHost"), Integer.parseInt(System.getProperty("http.proxyPort")));
     ```

6. If you are targeting API 24 or above, make sure that your app allows the self-signed Charles Proxy certificate to be used.  Add this file to your `res/xml/network_security_config.xml`:

     ```xml
     <?xml version="1.0" encoding="utf-8"?>
     <network-security-config>
        <debug-overrides>
           <trust-anchors>
               <!-- Trust user added CAs while debuggable only -->
               <certificates src="user" />
           </trust-anchors>
       </debug-overrides>
     </network-security-config>
    ```

    Inside your `AndroidManifest.xml` file, make sure to include this `networkSecurityConfig` parameter:

    ```xml
       <application
           android:name=".RestApplication"
           android:networkSecurityConfig="@xml/network_security_config"
    ```

    If you forget this step, a proxied SSL connection will not work for any devices targeted for API 24 (Nougat) and above.  For more information, see [this link](https://developer.android.com/training/articles/security-config.html).

6. Go back to Charles Proxy and start recording network traffic:

     <img src="https://imgur.com/c0q6j2j.png"/>

For more details, take a look at this [blog](https://jaanus.com/debugging-http-on-an-android-phone-or-tablet-with-charles-proxy-for-fun-and-profit/) for using Charles Proxy.
