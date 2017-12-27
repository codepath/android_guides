## Overview

As of API 23 (Marshmallow), the permission model for Android [[has changed significantly|Understanding-App-Permissions]]. Now, rather than all being setup at install-time, certain [dangerous permissions](http://developer.android.com/guide/topics/security/permissions.html#normal-dangerous) must be checked and activated at runtime instead. 

For a full breakdown of how runtime permissions work, check out the [[understanding app permissions|Understanding-App-Permissions#permission-updates-in-marshmallow]] guide. This guide is focused on the practical approach for managing runtime permissions, requesting access to a feature, and managing error cases where the permission is denied.

The easiest way to manage runtime permissions is by using third-party libraries. In this guide, we will be taking a look at the [PermissionsDispatcher](https://github.com/hotchemi/PermissionsDispatcher) library. The library is 100% reflection-free and as such does not cause significant performance penalties. 

### Dangerous Permissions

First, we need to recognize the [dangerous permissions](http://developer.android.com/guide/topics/security/permissions.html#normal-dangerous) that require us to request runtime permissions. This includes but is not limited to the following common permissions:

| Name                                         | Description              |
| --------------------------------             | --------------------     |
| `Manifest.permission.READ_CALENDAR`          | Read calendar events     |
| `Manifest.permission.WRITE_CALENDAR`         | Write calendar events    |
| `Manifest.permission.CAMERA`                 | Access camera object     | 
| `Manifest.permission.READ_CONTACTS`          | Read phone contacts      |
| `Manifest.permission.WRITE_CONTACTS`         | Write phone contacts     |
| `Manifest.permission.ACCESS_FINE_LOCATION`   | Access precise location  |
| `Manifest.permission.ACCESS_COARSE_LOCATION` | Access general location  |
| `Manifest.permission.RECORD_AUDIO`           | Record with microphone   | 
| `Manifest.permission.CALL_PHONE`             | Call using the dialer    |
| `Manifest.permission.READ_EXTERNAL_STORAGE`  | Read external or SD      |
| `Manifest.permission.WRITE_EXTERNAL_STORAGE` | Write to external or SD  |

The full list of [dangerous permissions](http://developer.android.com/guide/topics/security/permissions.html#normal-dangerous) contains all permissions that require runtime management. Please note that permissions in the [normal permission group](http://developer.android.com/guide/topics/security/normal-permissions.html) **do not require run-time checks** as [[outlined here|Understanding-App-Permissions#normal-permissions]].

### Installation

Make sure to [[upgrade|Getting-Started-with-Gradle#upgrading-gradle]] to the latest Gradle version. 

And on your app module in `app/build.gradle`:

```gradle
dependencies {
  compile 'com.github.hotchemi:permissionsdispatcher:3.1.0'
  annotationProcessor 'com.github.hotchemi:permissionsdispatcher-processor:3.1.0'
}
```

Be sure to use the latest version: [![Download](https://api.bintray.com/packages/hotchemi/maven/permissionsdispatcher/images/download.svg)](https://bintray.com/hotchemi/maven/permissionsdispatcher/_latestVersion) available.

### Usage

Suppose we wanted to record be able to call a number using the phone's dialer. Since this is a dangerous permission, we need to ask the user for the permission at runtime with the `Manifest.permission.CALL_PHONE` permission. This requires us to do the following:

1. Annotate the activity or fragment with `@RuntimePermissions`
2. Annotate the method that requires the permission with `@NeedsPermission`
3. Delegate the permissions events to a compiled helper class
4. Invoke the helper class in order to trigger the action with permission request
5. **Optional:** Annotate a method which explains the reasoning with a dialog
6. **Optional:** Annotate a method which fires if the permission is denied
7. **Optional:** Annotate a method which fires if the permission will never be asked for again.

First, we need to annotate the activity or fragment with `@RuntimePermissions`:

```java
@RuntimePermissions
public class MainActivity extends AppCompatActivity {
  // ...
}
```

Next, we need to annotate the method that requires the permission with `@NeedsPermission` tag:

```java
@RuntimePermissions
public class MainActivity extends AppCompatActivity {
    @NeedsPermission(Manifest.permission.CALL_PHONE)
    void callPhone() {
        // Trigger the calling of a number here
    }
}
```

After compiling the project, we need to delegate the permission events to the generated helper class ([Activity Name] + PermissionsDispatcher):

```java
@RuntimePermissions
public class MainActivity extends AppCompatActivity {
    // ...
    @Override
    public void onRequestPermissionsResult(int requestCode, String[] permissions, int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        // NOTE: delegate the permission handling to generated method
        MainActivityPermissionsDispatcher.onRequestPermissionsResult(this, requestCode, grantResults);
    }
}
```

In the code base, we can trigger the call with the appropriate runtime permission checks using the generated methods suffixed with `WithCheck` on the helper class:

```java
// NOTE: delegate the permission handling to generated method
MainActivityPermissionsDispatcher.callPhoneWithPermissionCheck(this);
```

This will invoke the `callPhone` method wrapped with the appropriate permission checks. 

### Permission Event Handling

We can also optionally configure the rationale dialog, handle the denial of a permission or manage when the user requests never to be asked again:

```java
@RuntimePermissions
public class MainActivity extends AppCompatActivity {

    // ...

    // Annotate a method which explains why the permission/s is/are needed. 
    // It passes in a `PermissionRequest` object which can continue or abort the current permission
    @OnShowRationale(Manifest.permission.CALL_PHONE)
    void showRationaleForPhoneCall(PermissionRequest request) {
        new AlertDialog.Builder(this)
            .setMessage(R.string.permission_phone_rationale)
            .setPositiveButton(R.string.button_allow, (dialog, button) -> request.proceed())
            .setNegativeButton(R.string.button_deny, (dialog, button) -> request.cancel())
            .show();
    }

    // Annotate a method which is invoked if the user doesn't grant the permissions
    @OnPermissionDenied(Manifest.permission.CALL_PHONE)
    void showDeniedForPhoneCall() {
        Toast.makeText(this, R.string.permission_call_denied, Toast.LENGTH_SHORT).show();
    }

    // Annotates a method which is invoked if the user 
    // chose to have the device "never ask again" about a permission
    @OnNeverAskAgain(Manifest.permission.CALL_PHONE)
    void showNeverAskForPhoneCall() {
        Toast.makeText(this, R.string.permission_call_neverask, Toast.LENGTH_SHORT).show();
    }
}
```

With that we can easily handle all of our runtime permission needs.

## Alternatives

In addition the the [PermissionsDispatcher](https://github.com/hotchemi/PermissionsDispatcher) outlined above, there are many other popular permissions libraries with various APIs and alternate designs including the following: 

* [Dexter](https://github.com/Karumi/Dexter)
* [easypermissions](https://github.com/googlesamples/easypermissions)
* [let](https://github.com/canelmas/let)
* [AndroidPermissions](https://github.com/ZeroBrain/AndroidPermissions)
* [Grant](https://github.com/anthonycr/Grant)
* [Assent](https://github.com/afollestad/assent)
* [Ask](https://github.com/00ec454/Ask)

A full list of permissions libraries [can be found here](https://gist.github.com/dlew/2a21b06ee8715e0f7338).

## References

* <https://github.com/hotchemi/PermissionsDispatcher>