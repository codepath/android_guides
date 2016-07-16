In the course of building your app, you will need many dependencies. Most can either be located as gradle dependencies and others as `jar`s, but sometimes none of them are reliable. In these cases you might want to build it from source along with your app. Here's the steps to get you going :

I'll be taking HeinrichReimer's amazing [material-intro](https://github.com/HeimrichReimer/material-intro) library as an example. I modified it a bit, reverting the commits that updated it to Android N and broke M compatibility, hence the need to build from source.

First up, move the source to a directory in your root dir, and add this to your settings.gradle

```
include ':app', ':material-intro'
```

Add the part after the ','

Then go to your app/build.gradle and add this : 

```
dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    testCompile 'junit:junit:4.12'
    compile project(':material-intro')
    compile 'com.android.support:appcompat-v7:23.4.0'
    compile 'com.android.support:design:23.4.0'
    compile 'com.android.support:support-v4:23.4.0'
 }
```
Add the `compile project(':material-intro')` line into the dependencies block.


Now run a gradle sync and you should be good to go!



Cheers to [florent37](https://github.com/florent37) from whose MaterialViewPager repo I saw how this functions :)