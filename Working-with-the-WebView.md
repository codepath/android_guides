## Overview

If you want to deliver a web application (or just a web page) as a part of a client application, you can do it using [WebView](http://developer.android.com/reference/android/webkit/WebView.html). The `WebView` class is an extension of Android's `View` class that allows you to display web pages as a part of your activity layout.

This document shows you how to get started with WebView and how to do some additional things, such as handle page navigation and bind JavaScript from your web page to client-side code in your Android application. See [the official WebView](http://developer.android.com/guide/webapps/webview.html) docs for a more detailed look.

## Usage

To get Internet access, request the INTERNET permission in your manifest file. For example:

```xml
<manifest ... >
    <uses-permission android:name="android.permission.INTERNET" />
    ...
</manifest>
```

To add a `WebView` to your Application, simply include the `<WebView>` element in your activity layout:

```xml
<?xml version="1.0" encoding="utf-8"?>
<WebView  xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/webview"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent"
/>
```

First, we need to configure the `WebView` to behave with reasonable defaults using [WebSettings](http://developer.android.com/reference/android/webkit/WebSettings.html) and creating a [WebViewClient](http://developer.android.com/reference/android/webkit/WebViewClient.html):

```java
public class MainActivity extends Activity {
   private WebView myWebView;

   @Override		
   protected void onCreate(Bundle savedInstanceState) {
      super.onCreate(savedInstanceState);
      setContentView(R.layout.activity_main);
      myWebView = (WebView) findViewById(R.id.webview);
      // Configure related browser settings
      myWebView.getSettings().setLoadsImagesAutomatically(true);
      myWebView.getSettings().setJavaScriptEnabled(true);
      myWebView.setScrollBarStyle(View.SCROLLBARS_INSIDE_OVERLAY);
      // Configure the client to use when opening URLs
      myWebView.setWebViewClient(new MyBrowser());
      // Load the initial URL
      myWebView.loadUrl("http://www.example.com");
   }
   
   // Manages the behavior when URLs are loaded
   private class MyBrowser extends WebViewClient {
      @Override
      public boolean shouldOverrideUrlLoading(WebView view, String url) {
         view.loadUrl(url);
         return true;
      }
   }
}
```

You can attach javascript functions and use them within the mobile webpages as described [here in the official docs](http://developer.android.com/guide/webapps/webview.html#UsingJavaScript).

## References

* <http://developer.android.com/guide/webapps/webview.html>
* <http://developer.android.com/guide/webapps/best-practices.html>
* <http://www.javacodegeeks.com/2013/06/fragment-in-android-tutorial-with-example-using-webview.html>
* <http://www.tutorialspoint.com/android/android_webview_layout.htm>
* <http://www.mkyong.com/android/android-webview-example/>