This page will compile common issues experienced with Android Studio 1.0 or above as they are experienced and recorded. 

**Needs Attention**

## Android Studio Issues

* If you decide to rename any of your ID tags in your XML files, you may get "No resource found that matches given name."   You will need to do a Clean Project so that the entire resource files can be generated and the build/ directories are removed.  

* One of the issues in the new Gradle build system is that you can often get "Multiple dex files define" issues.  If one dependency library already includes an identical set of libraries, then you may have to make changes to your Gradle configurations to avoid this conflict.  For instance, including the Butterknife library with the Parceler library causes multiple declarations of javax.annotation.processing.Processor.  In this case, you have to exclude this conflict:

```
   packagingOptions {

        exclude 'META-INF/services/javax.annotation.processing.Processor'  // butterknife
    }
```

Another error if you attempt to include a library that is a subset of another library.  For instance, suppose we included the Google play-services library but thought we also needed to include it with the play-services-map library.:

```
dependencies {

    compile 'com.google.android.gms:play-services:6.5.+'
    compile 'com.google.android.gms:play-services-maps:6.5.+'
}
```

It turns out that having both is redundant and will cause errors.  It is necessary in this case to remove one or the other, depending on your need to use other Google API libraries.

## Eclipse ADT Issues

For common issues experienced with Eclipse, check the [[Troubleshooting Eclipse Issues]] page instead for a detailed list of common problems.