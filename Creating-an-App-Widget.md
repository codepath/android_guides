## Overview
This guide covers how to create an app Widget.
A widget is a small gadget or control of your android application placed on the home screen. Widgets can be very handy as they allow you to put your favourite applications on your home screen in order to quickly access them. You have probably seen some common widgets, such as music widget, weather widget, clock widget e.t.c

Widgets could be of many types such as information widgets, collection widgets, control widgets and hybrid widgets. Android provides us a complete framework to develop our own widgets.
## Basics
To create an App Widget, you need the following:
* `AppWidgetProviderInfo` **object**
Describes the metadata for an App Widget, such as the App Widget's layout, update frequency, and the AppWidgetProvider class. This should be defined in XML.
* `AppWidgetProvider` **class implementation**
Defines the basic methods that allow you to programmatically interface with the App Widget, based on broadcast events. Through it, you will receive broadcasts when the App Widget is updated, enabled, disabled and deleted.
* **View layout** 
Defines the initial layout for the App Widget, defined in XML.

### Declaring an App Widget in the Manifest
First, declare the AppWidgetProvider class in your application's AndroidManifest.xml file. For example:
```xml 
<receiver android:name="ExampleAppWidgetProvider" >
    <intent-filter>
        <action android:name="android.appwidget.action.APPWIDGET_UPDATE" />
    </intent-filter>
    <meta-data android:name="android.appwidget.provider"
               android:resource="@xml/example_appwidget_info" />
</receiver>
```
### Widget - XML file
In order to create an application widget, the first thing you need is the `AppWidgetProviderInfo` object, which you will define in a separate widget XML file. In order to do that, right-click on your project and create a new folder called XML. Now right-click on the newly created folder and create a new XML file. The resource type of the XML file should be set to AppWidgetProvider. In the XML file, define some properties which are as follows −
```xml
<appwidget-provider 
   xmlns:android="http://schemas.android.com/apk/res/android" 
   android:minWidth="146dp" 
   android:updatePeriodMillis="0" 
   android:minHeight="146dp" 
   android:initialLayout="@layout/activity_main">
</appwidget-provider>
```
### Widget - Layout file
Now you have to define the layout of your widget in your default XML file. You can drag components to generate auto XML.
### Widget - Java file
After defining the layout, now create a new JAVA file or use an existing one, and extend it with the `AppWidgetProvider` class and override its update method as follows.
```java
PendingIntent pending = PendingIntent.getActivity(context, 0, intent, 0);
RemoteViews views = new RemoteViews(context.getPackageName(), R.layout.activity_main);
```
In the end, you have to call an update method updateAppWidget() of the AppWidgetManager class. Its syntax is −
```java 
appWidgetManager.updateAppWidget(currentWidgetId,views); 
```
A part from the updateAppWidget method, there are other methods defined in this class to manipulate widgets. They are as follows −
| Sr.No | Method & Description                                                                                              |
|:-----:|-------------------------------------------------------------------------------------------------------------------|
| 1     | `onDeleted(Context context, int[] appWidgetIds)` This is called when an instance of AppWidgetProvider is deleted. |
| 2     | `onDisabled(Context context)` This is called when the last instance of AppWidgetProvider is deleted               |
| 3     | `onEnabled(Context context)` This is called when an instance of AppWidgetProvider is created.                     |
| 4     | `onReceive(Context context, Intent intent)` It is used to dispatch calls to the various methods of the class      |

## Example
Following is the content of the modified `MainActivity.java`.
```java
import android.app.PendingIntent;
import android.appwidget.AppWidgetManager;
import android.appwidget.AppWidgetProvider;
import android.content.Context;
import android.content.Intent;
import android.net.Uri;
import android.widget.RemoteViews;
import android.widget.Toast;

public class MainActivity extends AppWidgetProvider{
   public void onUpdate(Context context, AppWidgetManager appWidgetManager,int[] appWidgetIds) {
      for(int i=0; i<appWidgetIds.length; i++){
         int currentWidgetId = appWidgetIds[i];
         String url = "http://www.tutorialspoint.com";
         
         Intent intent = new Intent(Intent.ACTION_VIEW);
         intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
         intent.setData(Uri.parse(url));
         
         PendingIntent pending = PendingIntent.getActivity(context, 0,intent, 0);
         RemoteViews views = new RemoteViews(context.getPackageName(),R.layout.activity_main);
         
         views.setOnClickPendingIntent(R.id.button, pending);
         appWidgetManager.updateAppWidget(currentWidgetId,views);
         Toast.makeText(context, "widget added", Toast.LENGTH_SHORT).show();
      }
   }
}
```

Following is the modified content of the xml `res/layout/activity_main.xml`.
```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
   xmlns:tools="http://schemas.android.com/tools" android:layout_width="match_parent"
   android:layout_height="match_parent" android:paddingLeft="@dimen/activity_horizontal_margin"
   android:paddingRight="@dimen/activity_horizontal_margin"
   android:paddingTop="@dimen/activity_vertical_margin"
   android:paddingBottom="@dimen/activity_vertical_margin"
   tools:context=".MainActivity"
   android:transitionGroup="true">
   
   <TextView
      android:layout_width="wrap_content"
      android:layout_height="wrap_content"
      android:text="Tutorials point"
      android:id="@+id/textView"
      android:layout_centerHorizontal="true"
      android:textColor="#ff3412ff"
      android:textSize="35dp" />
      
   <Button
      android:layout_width="wrap_content"
      android:layout_height="wrap_content"
      android:text="Widget"
      android:id="@+id/button"
      android:layout_centerHorizontal="true"
      android:layout_marginTop="61dp"
      android:layout_below="@+id/textView" />

</RelativeLayout>
```

Following is the content of the `res/xml/mywidget.xml`.
```xml
<?xml version="1.0" encoding="utf-8"?>
<appwidget-provider 
   xmlns:android="http://schemas.android.com/apk/res/android" 
   android:minWidth="146dp" 
   android:updatePeriodMillis="0" 
   android:minHeight="146dp" 
   android:initialLayout="@layout/activity_main">
</appwidget-provider>
```

Following is the content of the `res/values/string.xml`.
```xml
<resources>
   <string name="app_name">My Application</string>
</resources>
```

Following is the content of the `AndroidManifest.xml` file.
```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
   package="com.example.sairamkrishna.myapplication" >
   
   <application
      android:allowBackup="true"
      android:icon="@mipmap/ic_launcher"
      android:label="@string/app_name"
      android:theme="@style/AppTheme" >
      <receiver android:name=".MainActivity">
      
      <intent-filter>
         <action android:name="android.appwidget.action.APPWIDGET_UPDATE"></action>
      </intent-filter>
      
      <meta-data android:name="android.appwidget.provider"
         android:resource="@xml/mywidget"></meta-data>
      
      </receiver>
   
   </application>
</manifest>
```
**NOTE**:- By just changing the URL in the java file, your widget will open your desired website in the browser.

## Reference
* <https://www.tutorialspoint.com/android/android_widgets.htm>
* <https://developer.android.com/guide/topics/appwidgets/index.html>