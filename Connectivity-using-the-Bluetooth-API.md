## Overview

The Android platform includes support for the Bluetooth network stack, which allows a device to wirelessly exchange data with other Bluetooth devices. The application framework provides access to the Bluetooth functionality through the Android Bluetooth APIs. These APIs let applications wirelessly connect to other Bluetooth devices, enabling point-to-point and multipoint wireless features.

Using the Bluetooth APIs, an Android application can perform the following:

 * Scan for other Bluetooth devices
 * Query the local Bluetooth adapter for paired Bluetooth devices
 * Connect to other devices through service discovery
 * Transfer data to and from other devices
 * Manage multiple connections

## Usage

In order to use Bluetooth features in your application, you must declare two permissions. The first of these is `BLUETOOTH`. You need this permission to perform any Bluetooth communication, such as requesting a connection, accepting a connection, and transferring data.
The other permission that you must declare is either `ACCESS_COARSE_LOCATION` or `ACCESS_FINE_LOCATION`. A location permission is required because Bluetooth scans can be used to gather information about the location of the user.

Declare the Bluetooth permission(s) in your application manifest file. For example:
```
<manifest ... >
  <uses-permission android:name="android.permission.BLUETOOTH" />
  <uses-permission android:name="android.permission.BLUETOOTH_ADMIN" />
  <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
  ...
</manifest>
```

### Set up bluetooth
1. Get the BluetoothAdapter.
```
BluetoothAdapter bluetoothAdapter = BluetoothAdapter.getDefaultAdapter();
if (bluetoothAdapter == null) {
    // Device doesn't support Bluetooth
}
``` 
2. Enable Bluetooth. 
```
if (!bluetoothAdapter.isEnabled()) {
    Intent enableBtIntent = new Intent(BluetoothAdapter.ACTION_REQUEST_ENABLE);
    startActivityForResult(enableBtIntent, REQUEST_ENABLE_BT);
}
```
A dialog appears requesting user permission to enable Bluetooth. If the user responds "Yes", the system begins to enable Bluetooth, and focus returns to your application once the process completes (or fails).

The REQUEST_ENABLE_BT constant passed to startActivityForResult() is a locally defined integer that must be greater than 0. The system passes this constant back to you in your onActivityResult() implementation as the requestCode parameter.

If enabling Bluetooth succeeds, your activity receives the RESULT_OK result code in the onActivityResult() callback. If Bluetooth was not enabled due to an error (or the user responded "No") then the result code is RESULT_CANCELED.

## Libraries

Check out these libraries for easy handling of Bluetooth in your apps:

 * [Android-BluetoothSPPLibrary](https://github.com/akexorcist/Android-BluetoothSPPLibrary) - Hardware developer? This is an easy way to use the Bluetooth Serial Port Profile
 * [RxAndroidBle](https://github.com/Polidea/RxAndroidBle) - Easily handle Bluetooth Low Energy

## References

* <http://developer.android.com/guide/topics/connectivity/bluetooth-le.html>
* <http://www.tutorialspoint.com/android/android_bluetooth.htm>
* <http://developer.android.com/guide/topics/connectivity/bluetooth.html>
* <http://code.tutsplus.com/tutorials/create-a-bluetooth-scanner-with-androids-bluetooth-api--cms-24084>
* <http://www.intorobotics.com/how-to-develop-simple-bluetooth-android-application-to-control-a-robot-remote/>
* <https://github.com/palaima/AndroidSmoothBluetooth>
* <https://github.com/DeveloperPaul123/SimpleBluetoothLibrary>