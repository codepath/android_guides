[SugarORM](https://github.com/satyan/sugar) is a database persistence library that provides a simple and concise way to integrate your application models into SQLite. In contrast to [ActiveAndroid](https://github.com/pardom/ActiveAndroid), which is mature, powerful, and flexible, SugarORM is:

* Less verbose
* Quicker to set up
* More hands-free

## Install

SugarORM has no dependencies, so installation is as simple as downloading the `.jar` file and putting it in your `libs` folder. Maven central hosting is currently being [planned](https://github.com/satyan/sugar/issues/91), and this page will update with additional information whenever SugarORM becomes available on Maven.

The current stable release is [v1.2](https://github.com/satyan/sugar/blob/master/dist/sugar-1.2.jar?raw=true), but the beta release [v1.3_beta](https://github.com/satyan/sugar/blob/master/dist/sugar-1.3_beta.jar?raw=true) is highly recommended. Once you have the `.jar` file in your `libs` folder, finishing the installation requires just setting the `android:name` attribute of the `application` tag in your `AndroidManifest.xml`:

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example"
    android:versionCode="1"
    android:versionName="1.0" >

    <uses-sdk ... />
    <uses-permission ... />

    <application
        ...
        android:name="com.orm.SugarApp" >  <!--Use this attribute verbatim-->
        <meta-data android:name="DATABASE" android:value="example.db" />
        <meta-data android:name="VERSION" android:value="1" />
        <meta-data android:name="QUERY_LOG" android:value="true" />
        <meta-data android:name="DOMAIN_PACKAGE_NAME" android:value="com.example" />
        <activity ... />
        <activity ... />
    </application>
</manifest>
```

There are four additional parameters you can set. `DATABASE` is the name of the database file that will be created by SugarORM. `VERSION` is the version of the database schema. This is used for schema migrations, which are described in more detail below. `QUERY_LOG` can be set to `"true"` or `"false"` and determines whether to log debug messages for the queries made to the underlying SQLite database. Finally, `DOMAIN_PACKAGE_NAME` narrows down the packages that SugarORM scans for Classes to persist.

## Integrate

## Query

## References

* <https://github.com/satyan/sugar>
* <https://github.com/pardom/ActiveAndroid> 