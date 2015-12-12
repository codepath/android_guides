## Overview

Building your own Android library (an [`.aar` file](http://tools.android.com/tech-docs/new-build-system/aar-format)) enables other developers to take advantage of code that you've written.  You can share existing activities, services, images, drawables, resource strings, and layout files that enable other people to leverage your work such as those documented in the [[must have libraries|Must-Have-Libraries]] guide.  Also, if your code base begins to take longer times to compile and/or run, creating a library also enables you to iterate faster by working on a more smaller component.  

### Creating a new Android Library

When you create a new Android project, a new application is always created.  You can use this application to test your library.  After creating the project, go to `New` -> `New Module`:

<img src="http://imgur.com/hbv3Eo4.png"/>

Select `Android Library`.  There is the option to choose `Java library`, but there is a major difference in that an Android library will include not only the Java classes but the resource files, image files, and Android manifest file normally associated with Android.  

<img src="http://imgur.com/xDUBjYg.png"/>

You will prompted next to provide a name and the module name.  The name will simply be used to [label](http://developer.android.com/guide/topics/manifest/manifest-intro.html#iconlabel) the application in the Android Manifest file, while the module name will correspond to the directory to be created:

<img src="http://imgur.com/kNKP51f.png"/>

When you click Next, a directory with the module name will be generated along with other files including a resource and Java folder:

<img src="http://imgur.com/tllGHUh.png"/>

In addition, a `build.gradle` file will be created.  One major difference is that Android applications use the `com.android.application` plugin.  Android libraries will use the `com.android.library` plugin.  This statement at the top signals to the [Android Gradle plug-in](http://developer.android.com/tools/building/plugin-for-gradle.html) to generate an `.aar` file instead of an `.apk` file normally installed on Android devices.

```gradle
// Android library
apply plugin: 'com.android.library'
```

### Compiling a Library

Android applications usually have a build and debug variation.  The `buildTypes` parameter designates the settings for each type:

```gradle
android {
  buildTypes {
    release {
        minifyEnabled true
    } 
    debug {
        minifyEnabled false         
    }
}
```

You can compile the library with Android Studio, or type `./gradlew build` at the command line.  The output will be stored under the library's subdirectory under `build/outputs/aar`.   Unlike Android applications in which `debug` or `release` versions can be generated, only release versions by default are published as documented [here](http://tools.android.com/tech-docs/new-build-system/user-guide#TOC-Referencing-a-Library).  

If you wish to build multiple variations, you will need to add this statement to your library `build.gradle` file:

```gradle
android {
     publishNonDefault true
}
```

When using this statement, different `.aar` packages are generated for each build type specified.  To reference them once they are published, see [[this section|Building-your-own-Android-library#add-the-gradle-dependency]].

If you wish to reference the library from your demo application within the same Android project, you will need to explicitly specify which library to use with the `configuration` parameter.    You need to add this statement to your `app/build.gradle`:

```gradle
dependencies {
    // colon needs to prefixed with the library path
    debugCompile project(path: ':mylibrary', configuration: 'debug')
    releaseCompile project(path: ':mylibrary', configuration: 'release')
}
```

#### Using with ButterKnife

If you intend use the library with [ButterKnife](https://github.com/JakeWharton/butterknife/issues/45), you will find that it does not work with Android libraries.  There are specific reasons for not supporting it as stated [here](https://github.com/JakeWharton/butterknife/issues/100).  For any existing code, you will have to convert your `@InjectView` annotations back to `findViewById()` calls.  

### Publishing

To publish your library, you can either make it available to a public or private repository.  jCenter and Maven Central are the most popular ones, though jCenter has become the default one used in Android Studio.  For more instructions about how to distribute through jCenter or Maven Central, see this [blog link](http://inthecheesefactory.com/blog/how-to-upload-library-to-jcenter-maven-central-as-dependency/en).

#### Setting up a private Amazon S3 Maven repository

Another approach is to setup a private Maven repository, which also be done through Amazon's Web Services (AWS) and the Simple Storage Service [(S3)](https://aws.amazon.com/s3/).  Gradle supports the ability to access private S3 repositories with a secret access key and ID used to authenticate with Amazon:

#### Adding the private Maven repository 

To add the S3 repository to the list, you will need to add the credentials to access the S3 bucket to your root `build.gradle` file:

```gradle
allprojects {
    repositories {
        jcenter()

        maven {
            url "s3://yourmavenrepo-bucket/android/snapshots"
            credentials(AwsCredentials) {
                accessKey AWS_ACCESS_KEY
                secretKey AWS_SECRET_KEY
            }
        }
    }
```

Instead of adding the keys directly, it is recommended that you add it to your `local.properties` to your local machine:

```gradle
AWS_ACCESS_KEY=<my_aws_access_key>
AWS_SECRET_KEY=<my_super_secret_key>
```

In order to publish the plugin, we need to create a separate Gradle file that can be use in our library configuration.  Create a file called `gradle/gradle-mvn-push.gradle`, which will apply the Gradle-Maven plugin and specify the location of the S3 bucket when using the `./gradlew uploadArchives` command:

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
          // repository(url: "file:///" + getOutputDir()) can also be used to copy to local file
          repository(url: getDestUrl()) // for copying directly to S3

          pom.groupId = GROUP
          pom.artifactId = POM_ARTIFACT_ID
          pom.version = VERSION_NAME

          pom.project {
             name POM_NAME
             packaging POM_PACKAGING
             description POM_DESCRIPTION
          }
        }
    }
}

```

We will then apply the statements in this file by applying it within the library's Gradle file (i.e. `mylibrary/build.gradle`):

```gradle
apply from: rootProject.file('gradle/gradle-mvn-push.gradle')
```

Edit the root project's `gradle.properties` too:

```gradle
VERSION_NAME=0.4-SNAPSHOT
VERSION_CODE=0.4
GROUP=com.codepath
```

Finally, we need to setup the metadata necessary for publishing.  Edit the library's `gradle.properties` and set the values:

```gradle
POM_NAME=My Library
POM_ARTIFACT_ID=library
POM_PACKAGING=aar
```

#### Support for Amazon IAM Roles

Currently Gradle's Amazon S3 integration only supports access keys and does not support [Identity Access Management (IAM)](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html) roles.  To take advantage of a client that does, you can output the repository to a local file and use the AWS command-line client to copy the snapshot dirs:  

```gradle

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

To publish and execute the task to copy the build to S3, the command to enter is `./gradlew copyToS3`.

#### Add the Gradle dependency

Once the private S3 repository has been added to the list, you can simply add this line to the Gradle dependency list.  The Android Gradle plug-in will search through all possible repositories searching for a match.  Add this line to your `app/build.gradle` file:
 
```gradle
dependencies {
  compile 'com.codepath:mylibrary:0.4-SNAPSHOT'
}
```

If you published multiple versions of your package as described [[here|Building-your-own-Android-library#building-different-versions]], you will need to specify the build type (i.e. `release` or `debug`).  Gradle may assume that if you specify the build type to search for a `.jar` file, so you must also specify the `@aar`.  Using this syntax doesn't follow the dependencies included in the package, so `transitive=true` must also be included as described in this [Google discussion](https://groups.google.com/forum/#!msg/adt-dev/Ll2JcCfgBsQ/eHjJ8EcZI5MJ).

```gradle
dependencies {
  releaseCompile('com.codepath:mylibrary:0.4:release@aar') {
    transitive=true
  }
  debugCompile('com.codepath:mylibrary:0.4-SNAPSHOT:debug@aar') {
    transitive=true
  }
}
```

#### Prevent caching

If you are making constant changes to your snapshot versions and wish to have the latest updates pulled each time, you can mark the dependency as a **changing module** in Gradle:

```gradle
dependencies {
  debugCompile('com.codepath:mylibrary:0.4-SNAPSHOT:debug@aar') {
    transitive=true
    changing=true
  }
}
```
  
Gradle will normally cache the module for 24 hours for those marked as changing, but you can lower this setting:

```gradle
configurations.all {
    // check for updates every build
    resolutionStrategy.cacheChangingModulesFor 0, 'seconds'
}
```

#### Issues with JDK 8u60 

If you are trying to access a private Amazon S3 repository, you may see an `AWS authentication requires a valid Date or x-amz-date header` error.  It is a known issue with [Gradle](https://issues.gradle.org/browse/GRADLE-3338) and Java versions.  

To fix this issue, you will need to upgrade to Gradle v2.8 by editing your `gradle/wrapper.properties`:
```gradle
distributionUrl=https\://services.gradle.org/distributions/gradle-2.8-all.zip
```

Even though the default Gradle version used in Android projects is 2.4, the build should compile without needing to make any changes.

### Using with ProGuard

If you intend to export your release with ProGuard to help reduce the total number of methods exported to avoid the [64K method limit](http://developer.android.com/tools/building/multidex.html), modify your library `build.gradle` file to enable ProGuard.  You will need to enable `minifyEnabled` and specify the configuration file for ProGuard to use:

```gradle
buildTypes {
         release {
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
         }
```

If you use the default configuration, ProGuard will obfuscate and alter the name of your library classes, making it impossible for Android projects to reference them.  The most basic example of ensuring your library classes are exported is shown below:

```
-dontobfuscate
# See https://speakerdeck.com/chalup/proguard 
-optimizations !code/allocation/variable

-keep public class * {
    public protected *;
}
```

See the [ProGuard documentation](http://proguard.sourceforge.net/manual/usage.html) for more information about the syntax of this file.  See [this example](https://stuff.mit.edu/afs/sipb/project/android/sdk/android-sdk-linux/tools/proguard/docs/index.html#manual/examples.html) of an Android library definition.

### References

* <http://ryanharter.com/blog/2015/06/18/hosting-a-private-maven-repo-on-amazon-s3/>
* <http://chris.banes.me/2013/08/27/pushing-aars-to-maven-central/>