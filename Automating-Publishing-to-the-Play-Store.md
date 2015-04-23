The Developer Console for Google Play provides API support for you to be able to push to the store automatically.   This ability allows you to trigger builds on your continuous integration server (i.e. Jenkins) and have them uploaded the Play store for alpha or beta testing, as well as pushing to production directly.

## Initial Setup

1. Navigate to `Settings` -> `API Access`:

   <img src="http://imgur.com/0n7ihzM.png"/>

2. There should be a `Service Accounts` section where you need to click the `Create Service Account button`.  You will be asked to click on the first step to take you to the Google Developers Console.

   <img src="http://imgur.com/6TnR700.png"/>

3. Click on the `Create New Client ID` button.

   <img src="http://imgur.com/7VPlkHM.png"/>

4. Click to create new `Service Account`.  Make sure to also request the `.p12 Key File` instead of JSON:

   <img src="http://imgur.com/paTHMHK.png"/>

5. You will be prompted to download the .p12 file.  Save it somewhere.  

6. Note the service account email associated with this new account.  You should see it appear in the Google Developer Console:

   <img src="http://imgur.com/TVm6CLM.png"/>

7. Use a Gradle plugin such as the [Gradle Play Publisher](https://github.com/Triple-T/gradle-play-publisher).  Add the following to the top of your `app/build.gradle` file:

   ```gradle
       buildscript {

       repositories {
           mavenCentral()
       }

       dependencies {
           // ...
           classpath 'com.github.triplet.gradle:play-publisher:1.0.2'
       }
    }

   apply plugin: 'play'
   ```

8. Configure the plugin with the Google Service Account and p12 file saved in steps #5 and #6.

   ```gradle

      play {
         track = 'alpha'
         serviceAccountEmail = 'abcd@developer.gserviceaccount.com'
         pk12File = file('Google Play Android Developer-12345.p12')
      }
   ```

9. The plugin creates the following tasks for you:

| Command                     | Description                                                          |
|:---------------------------:|--------------------------------------------------------------------- |  |publishApkRelease            | Uploads the APK and the summary of recent changes.                   |
|publishListingRelease        | Uploads the descriptions and images for the Play Store listing.      |
|publishRelease               | Uploads everything.                                                  |
|bootstrapReleasePlayResources| Fetch data from the Play Store & bootstrap the required files/folders|

You can now type the following gradle commands such as the follwoing:

```bash
./gradlew publishApkRelease
```

