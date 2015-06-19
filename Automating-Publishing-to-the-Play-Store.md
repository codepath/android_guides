The Developer Console for Google Play provides API support for you to be able to push to the store automatically.   This ability allows you to trigger builds on your continuous integration server (i.e. Jenkins) and have them uploaded the Play store for alpha or beta testing, as well as pushing to production directly.   

This guide will show you how you can setup publishing APK's directly through the command-line, or how you configure a continuous integrations server such as Jenkins to do the same.   Regardless of which approach, you will need to setup Google API access.

## Setup for Google API access

1. Inside the Google Play Store for your project, navigate to `Settings` -> `API Access`:

       <img src="http://imgur.com/0n7ihzM.png"/>
   
2. There should be a `Service Accounts` section where you need to click the `Create Service Account button`.  Click on the link shown on the first step to visit the Google Developers Console.  

       <img src="http://imgur.com/6TnR700.png"/>

3. Click on the `Create New Client ID` button.

       <img src="http://imgur.com/7VPlkHM.png"/>

4. Click to create new `Service Account`.  Make sure to also request the `.p12 Key File` instead of JSON:

       <img src="http://imgur.com/paTHMHK.png"/>

5. You will be prompted to download the .p12 file.  Save it somewhere.  

6. Note the service account email associated with this new account.  You should see it appear in the Google Developer Console:

       <img src="http://imgur.com/TVm6CLM.png"/>

7. Once you are done, go back to the Google Play Developer Console and navigate to the `Settings` -> `API Access`.  Make sure the checkboxes for `Edit store listing, pricing & distribution`, `Manage Production APKs`, and `Manage Alpha & Beta APKs` are checked for the Google Service account used.  (If you intend to upload an alpha or beta SDK through a Google service account, apparently these permissions must be checked according to this [discussion](http://echelog.com/logs/browse/jenkins/1409263200).

       <img src="http://i.imgur.com/QBF0Vmp.png"/>

## Setting Up Gradle Plugin (for command-line APK publishing)

If you want to be push your APKs directly through Gradle, you can install a plugin such as the [Gradle Play Publisher](https://github.com/Triple-T/gradle-play-publisher).

1. Add the following to the top of your `app/build.gradle` file:

       ```gradle
           buildscript {

       repositories {
           jcenter()
       }

       dependencies {
           // ...
           classpath 'com.github.triplet.gradle:play-publisher:1.0.2'
       }
    }

       apply plugin: 'play'
       ```

2. Configure the plugin with the Google Service Account and p12 file saved in steps #5 and #6.

       ```gradle

      play {
         track = 'alpha'
         serviceAccountEmail = 'abcd@developer.gserviceaccount.com'
         pk12File = file('Google Play Android Developer-12345.p12')
      }
   ```

3. The plugin creates the following tasks for you:

| Command                     | Description                                                          |
|:---------------------------:|--------------------------------------------------------------------- |  |publishApkRelease            | Uploads the APK and the summary of recent changes.                   |
|publishListingRelease        | Uploads the descriptions and images for the Play Store listing.      |
|publishRelease               | Uploads everything.                                                  |
|bootstrapReleasePlayResources| Fetch data from the Play Store & bootstrap the required files/folders|

You can now type the following gradle commands such as the following:

```bash
./gradlew publishApkRelease
```

## Setting Up Jenkins (for automating CI builds)

This section mainly shows how to setup the [Google Play Android Publisher plugin](https://wiki.jenkins-ci.org/display/JENKINS/Google+Play+Android+Publisher+Plugin) with Jenkins.  You can most likely adapt the same steps for other services that enable beta distribution and have a related plugin, such as the one for [Hockey](https://wiki.jenkins-ci.org/display/JENKINS/HockeyApp+Plugin).

1. Make sure you have already gone through the process of [Building Gradle Projects with Jenkins CI](Building-Gradle-Projects-with-Jenkins-CI) and already have a Jenkins job correctly running.   You will only need to install a Jenkins plugin that will allow you to create a build step that will enable the APK generated to be published to the Google Play store directly.

2. Verify that you've the followed the guide about how to [configure Google API access](#setup-for-google-api-access).

3. Inside Jenkins, go to `Manage Jenkins` -> `Manage Plugins`.  Assuming the plugin has not yet been installed, select the `Available` tab and search for the `Google Play Android Publisher Plugin`.    

4. Navigate to the `Credentials` section in Jenkins and load the `.p12` key file downloaded during the initial setup process of setting up Google API access.  A [basic walkthrough video](https://www.youtube.com/watch?v=txdPSJF94RM&list=PLhF0STyfNdUk1R3taEmgFR30yzp41yuRK) also demonstrates how to do this step.

       <img src="http://i.imgur.com/xxs8qlD.png"/>

5. Add a post-build step to your existing Jenkins project.  

       <a href="http://i.imgur.com/nfc4xDA.png"><img src="http://i.imgur.com/nfc4xDA.png"></a>

       a. Make sure to choose the credential name from the drop-down list.  It should belong to the Google Play account that manages the app.

       b. Enter path and/or an [Ant-style](http://stackoverflow.com/questions/69835/how-do-i-use-nant-ant-naming-patterns) wildcard pattern for the APK.  For instance, the example in the screenshot expects the APK to be generated inside `**/build/outputs/apk/yourappname*.apk`.

       c. Choose what track to which the APKs should be deployed (Alpha, Beta, Production).

       d. You can create release notes before you start the build.  If you forget to do this step or your automated process pushes the build, you can edit them later directly on the Google Play Developer Console.

### GitHub / Jenkins integration

**Do not follow these steps if you intend for your Jenkins jobs to push directly to Production tracks.** You should always verify your builds before rolling out to production users.

1. Setup a `Build Trigger` in your Jenkins job to allow builds to be [triggered via remote API commands](https://wiki.jenkins-ci.org/display/JENKINS/Remote+access+API).  You can use the URL shown below the `Authentication token` (i.e. http://ci.mycompany.com/view/All/job/AndroidBuild/build?token=TOKEN_NAME) to trigger this Jenkins job.  

       <img src="http://i.imgur.com/QfzhhQM.png"/>

2. Go to your GitHub repository `Settings` page and visit the `Webhooks & Services` section.  Enter the `Payload URL` for your Jenkins build job.   

       <img src="http://i.imgur.com/iONpTHh.png"/>

3. You can also control which GitHub events should fire these Jenkins build jobs:

       <img src="http://i.imgur.com/JpwMRTn.png/">