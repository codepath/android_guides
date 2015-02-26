## Overview

If you want to deliver a web application (or just a web page) as a part of a client application, you can do it using [WebView](http://developer.android.com/reference/android/webkit/WebView.html). The `WebView` class is an extension of Android's `View` class that allows you to display web pages as a part of your activity layout.

This document shows you how to get started with WebView and how to do some additional things, such as handle page navigation and bind JavaScript from your web page to client-side code in your Android application. See [the official WebView](http://developer.android.com/guide/webapps/webview.html) docs for a more detailed look.

## Usage

### Load External Pages

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
    android:layout_width="match_parent"
    android:layout_height="match_parent"
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

### Loading Local Pages

In case you want to store a copy of a webpage locally to be loaded into a `WebView`, you can put it in the android `assets` folder. If you do not find one under your `main/` directory, then you can create one. Place the html, css, js, etc in this folder. 

For example, say I wanted to load a page entitled `index.html`. I would create my file under `{ProjectName}/app/src/main/assets/index.html`

then, in your activity or fragment you can use the code

```java
public class MainActivity extends Activity {
   private WebView myWebView;

   @Override		
   protected void onCreate(Bundle savedInstanceState) {
      super.onCreate(savedInstanceState);
      setContentView(R.layout.activity_main);
      myWebView = (WebView) findViewById(R.id.webview);
      myWebView.getSettings().setJavaScriptEnabled(true);
      myWebView.setWebViewClient(new WebViewClient());
      String path = Uri.parse("file:///android_asset/index.html").toString();
      myWebView.loadUrl(path);
   }

}
```


## References

* <http://developer.android.com/guide/webapps/webview.html>
* <http://developer.android.com/guide/webapps/best-practices.html>
* <http://www.javacodegeeks.com/2013/06/fragment-in-android-tutorial-with-example-using-webview.html>
* <http://www.tutorialspoint.com/android/android_webview_layout.htm>
* <http://www.mkyong.com/android/android-webview-example/>