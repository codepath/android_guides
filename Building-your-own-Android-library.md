## Overview

Building your own Android library enables other developers to take advantage of code that you've written for Android.  You can share existing activities, services, images, drawables, resource strings, and layout files that enable other people to leverage your work such as those documented in the [[must have libraries|Must-Have-Libraries]] guide.  Also, if your code base begins to take longer times to compile and/or run, creating a library also enables you to iterate faster by working on a more smaller component.  

### Creating a new Android Library

When you create a new Android project, a new application is always created.  You can use this application to test your library.  After creating the project, go to `New` -> `New Module`:

<img src="http://imgur.com/hbv3Eo4.png"/>

Select `Android Library`.  There is the option to choose `Java library`, but there is a major difference in that an Android library, which compiles to an [.AAR file](http://tools.android.com/tech-docs/new-build-system/aar-format), will include not only the Java classes but the resource files, image files, and Android manifest file normally associated with Android.

<img src="http://imgur.com/xDUBjYg.png"/>

You will prompted next to provide a name and the module name.  The name will simply be used to [label](http://developer.android.com/guide/topics/manifest/manifest-intro.html#iconlabel) the application in the Android Manifest file, while the module name will correspond to the directory to be created:

<img src="http://imgur.com/kNKP51f.png"/>

When you click Next, a directory with the module name will be generated along with other files including a resource and Java folder:

<img src="http://imgur.com/tllGHUh.png"/>

In addition, a `build.gradle` file will be created.  One major different is that  Android applications use the `com.android.application` plugin.  Android libraries will use the `com.android.library` plugin.  This signals to the Android Gradle plug-in to generate an `.aar` file instead of an `.apk` file normally installed on Android devices.

```gradle
// Android library
apply plugin: 'com.android.library'
```
### Setting up a private Amazon S3 Maven repository

#### Add a task for Maven publishing

We can use the Maven plugin to release snapshots or releases.

```gradle

apply plugin: 'maven'

def isReleaseBuild() {
    return VERSION_NAME.contains("SNAPSHOT") == false
}

def getOutputDir() {
  if (isReleaseBuild()) {
      return "${project.buildDir}/releases"
  } else {
      return "${project.buildDir}/snapshots"
  }
}

def getDestUrl() {
  if (isReleaseBuild()) {
      return "s3://yourmavenrepo-bucket/android/releases"
  } else {
      return "s3://yourmavenrepo-bucket/android/snapshots"
  }
}

uploadArchives {
    repositories {
        mavenDeployer {
          repository(url: "file:///" + getOutputDir())
        }
    }
}

task copyToS3(type: Exec) {
    commandLine 'aws', 's3', 'cp', '--recursive', getOutputDir(), getDestUrl()
}

copyToS3.dependsOn uploadArchives
```

Currently the Gradle/Amazon S3 integration does not support IAM roles.  You will need to provide an access key or secret.   To circumvent this issue, you can output the repository to a private repo and use the AWS command-line client to copy the snapshot dirs.

#### Reference the private Maven repository

For the app, add the entry to your `build.gradle` file:

```gradle
allprojects {
    repositories {
        jcenter()

        maven {
            url "s3://yourmavenrepo-bucket/android/snapshots"
            credentials(AwsCredentials) {
                accessKey "<ACCESS KEY>"
                secretKey "<SECRET KEY>"
            }
        }
    }
```

#### Handling `AWS authentication requires a valid Date or x-amz-date header` errors.

If you are trying to access a private Amazon S3 repository, you may see an `AWS authentication requires a valid Date or x-amz-date header` error.  It is a known issue with [Gradle](https://issues.gradle.org/browse/GRADLE-3338) and Java versions.  

To fix this issue, you will need to upgrade to Gradle v2.8 by editing your `gradle/wrapper.properties`:
```gradle
distributionUrl=https\://services.gradle.org/distributions/gradle-2.8-all.zip
```
