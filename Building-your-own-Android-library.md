## Overview

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
