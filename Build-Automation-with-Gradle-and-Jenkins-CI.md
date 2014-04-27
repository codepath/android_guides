Once you've begun to use Gradle to build and test projects (see [Getting Started with Gradle](https://github.com/thecodepath/android_guides/wiki/Getting-Started-with-Gradle)), you might like to build automatically, run tests on every push to your repo, and push your builds to a deployment system.
 
This tutorial reflects the author's workplace build environment, with [Jenkins CI](http://jenkins-ci.org/) running on a build server on the local network. You may be able to adapt this guide if you are using [Travis CI](http://docs.travis-ci.com/user/languages/java/) or [CircleCI](https://circleci.com/docs/android/).

## Build Your Project Locally
Use your local machine as a testbed to keep track of your environment configuration: 
your `PATH` environment variable, and the other environment variables you set in order to get Gradle to run. This step is ready as soon as you can run `gradle assemble` in your project and see `BUILD SUCCESSFUL`.

### Confirm Your Jenkins Configuration
This tutorial presumes that you already have a Jenkins installation running.

Go to your Jenkins environment through the web UI and determine whether it's running all jobs as the same user, or is configured to use separate user accounts for different jobs. A basic Jenkins configuration will use only one user account on its host server. If you currently have no jobs configured or the existing jobs are all sharing the Jenkins default environment, you can skip ahead to [Install the Android SDK](#install-the-android-sdk), but do consider migrating to the build node approach with a separate user!

In my workspace configuration, Jenkins was set up with multiple node environments because the same machine is used to build iOS and Django projects.

Here's how to determine whether Jenkins is already set up to sandbox projects in their own environments. 

1. On the build machine, are there already other user accounts corresponding to other projects?
2. On Jenkins UI, when you go to **Manage Jenkins** -> **Configure Jenkins**, do you see multiple nodes?

![List of all nodes on this Jenkins host](https://dl.dropboxusercontent.com/u/10808663/gradle_jenkins_android/nodes_list.png)

If this is the case, you should set up a node environment just for your Android build tasks. 

You'll be creating a new user account on the Jenkins server, linking this account to a new virtual node configuration that runs on that user account, and then configuring your Jenkins job to run on the new node. 

SCREENSHOT: how to do all of that 

## Configure Your Build Machine
Whether you are using the default environment or working with a build node, your machine is ready to be customized for Android and Gradle. You'll be re-creating many features of your local development environment.

You'll need `wget`. Check that you've got it:

`which wget`

If it's not installed, [go get it](http://osxdaily.com/2012/05/22/install-wget-mac-os-x/). (My build machine is a Mac.)

Let's create a build node. So, your work will be separate from any other activities on the build machine, but you need to remember to make these changes as the build node user.

I have found it helpful to have two accounts on the build server, my own account `ari` with superuser privileges, and then the `ciandroid` account with normal user privileges. If you've created an Android user as I recommend, but you logged in as your superuser, remember to `sudo su ciandroid` (using the password of your own account, _not_ the `ciandroid` password). Install Gradle and the Android SDK within the `ciandroid` home directory.

SSH to your build server.

    $ ssh ari@ci.mydomain.com:9222

Go to your build environment's home directory.

    $ sudo su ciandroid
    $ cd

### Install the Android SDK
`cd` to your home directory (`/Users/ciandroid`), or your preferred folder for file downloads.

Now download the Android SDK without Eclipse bundled. Go to [Android SDK](http://developer.android.com/sdk/index.html) and copy the URL for the **SDK Tools Only** download that's appropriate for your build machine OS.

![List of Android SDK downloads from developers.android.com](https://dl.dropboxusercontent.com/u/10808663/gradle_jenkins_android/sdk_downloads.png)

On your build machine, `wget` the correct SDK URL:

    $ wget http://dl.google.com/android/android-sdk_r22.6.2-macosx.zip

Also download the Gradle binary. Remember, you want Gradle 1.6.

    $ wget https://services.gradle.org/distributions/gradle-1.6-bin.zip'

Unzip both of these directories and place the contents in their own locations within your home directory. The directory names can be anything you like, but I like the following setup. The Gradle binary is in `/Users/ciandroid/bin/` and the Android SDK is in `/Users/ciandroid/src/`.

 ![Directory structure on the build server](https://dl.dropboxusercontent.com/u/10808663/gradle_jenkins_android/directories_on_build_server.png)

Now it's time to set your build environment's `PATH` variable and other variables that Jenkins will use to locate Android and Gradle.

`cd` to your CI environment's home directory (`ciandroid` home dir) and edit your `.bash_profile` file. If you're not using bash, edit the right config file for your environment.

Make your `.bash_profile` look like the following, replacing paths as needed:

    # Android 
    export ANDROID_HOME=/Users/ciandroid/android-sdk-macosx
    export PATH=$PATH:$ANDROID_HOME/tools

    # Gradle
    export GRADLE_HOME=/Users/ciandroid/gradle-1.6
    export PATH=$PATH:GRADLE_HOME/bin

Save the file and quit your editor. Reload `.bash_profile`:

    $ source ~./bash_profile
 
### Install Android SDK Packages
For this step, it's especially helpful to have GUI access to the build server. Installing particular Android SDK packages from the command line is tricky. So if you have not already done so, use VNC to connect to your build machine and open a terminal there.

![Android SDK manager on build machine](https://dl.dropboxusercontent.com/u/10808663/gradle_jenkins_android/android_sdk_manager.png)

At the prompt, type `android` and hit Enter to launch the Android SDK Manager in a window. If this doesn't work, your `PATH` variable has not been set up with the Android SDK location.   

You will want to install the same Android SDK packages on your build machine as you did to get Gradle running locally. Before you begin, take a look at the `build.gradle` file in your project.

Here are the SDK packages names you'll definitely need:

#### Tools
* Android SDK Tools (latest version)
* Android SDK Platform-tools (latest version)
* Android SDK Build-tools (latest version)
* Android SDK Build-tools for the version of Android that you listed in the `build.gradle` file as the `android: buildToolsVersion` target. If your `build.gradle` says 

    android {
        buildToolsVersion '17'
        ...
    }
    
then make sure to download that API version in the Android SDK Manager. 

#### Android API
Download the complete SDK package set for the API levels that you named in the `android: compileSdkVersion` section of your `build.gradle` file.

#### Extras
* Android Support Repository
* Android Support Library

### Load your Test Project on the Build Server
Your environment should be ready to go! You can type `gradle` at a prompt to see that it's installed. 

Now you need to get your project onto the build server so that Jenkins can find it.

Again, if you are using separate environments for different projects (as suggested), make sure you are the `ciandroid` user in your build environment and that you are in the home directory for that user.

In your build environment, create a directory for Android project source code, and check out your project from version control into that location. I put my projects in `/Users/ciandroid/ci/`.

    $ whoami
    ciandroid
    $ cd ~/ci; pwd
    /Users/ciandroid/ci
    $ git clone git@bitbucket.org/my_repo/project.git
    Cloning into 'project'...

### Try Running Gradle
`cd` into the project directory on your build machine. Make sure you see your `build.gradle` file, and run `gradle assemble`.

Hopefully, this is what you'll get:

![Successful build output](https://dl.dropboxusercontent.com/u/10808663/gradle_jenkins_android/build_successful.png)
    
### Configure Jenkins
Now you are ready to set up Jenkins to build for you.

Pre-existing settings on your build machine may cause your setup to vary from these directions. Since I was already using a separate Jenkins node for other projects, I created a node for the Android project. You can skip to [Create the Build Job](#create-the-build-job) if, instead, you are building all Jenkins jobs within the same user environment.

SCREENSHOT: JENKINS CONFIGURATION

### Set Up SSH Keys
If you are using SSH key authentication to connect Jenkins, your `ciandroid` user will need its own SSH key pair. In my work environment, I already had a key pair ready to drop into a `/Users/ciandroid/.ssh` directory. When you set up SSH keys for your build environment, make certain that the `.ssh/` directory and contents are owned by the build environment user (`ciandroid`) and not by any other user. Use `chown` on the  `.ssh/` directory to fix it if necessary. 

### Create the Build Node
Using the menus in the Jenkins UI, go to **Jenkins** -> **Manage Jenkins** -> **Manage Nodes** -> **New Node**. Select "Dumb Slave" and give it the name `android`.

![Creating a build node](https://dl.dropboxusercontent.com/u/10808663/gradle_jenkins_android/create_android_build_node.png)

You'll need to provide your own values for your **Host** and **Credentials** settings, tailored to your own systems. But make sure to set **Remote FS root** with the `ci/` path you created earlier. Also ensure that **Environment Variables** is checked under the **Node Properties** section, and fill in values for `ANDROID_HOME` and `PATH`.

When you set this up, a Jenkins jarfile called `slave.jar` will be downloaded into your node environment `ci/` directory.

### Create the Build Job
Now you have a Jenkins node that's capable of running jobs. You're ready to create the Jenkins job that actually runs the build.

![Creating a new job](https://dl.dropboxusercontent.com/u/10808663/gradle_jenkins_android/create_new_build.png)

Go back to the Jenkins main page and hit **New Item**. Give the job a name and select "Build a free-style software project." In the job configuration screen that follows, you will do all the setup to tell Jenkins about your source.

Jenkins build configurations can have many steps. The key fields you want to look out for are:
 * `Restrict where this project can be run`: give it the name of the build node you created
 * Source code integration for your version control system
 * Build Triggers
 * Build
 
Under the **Build** heading, add shell execute statements with the "Add build step" tool:
* `gradle clean`
* `gradle assemble`
  
Make sure you hit "Save" after all this work!

All of the following screenshots are intended as a starting point only.

SCREENSHOT PARTY

### Run Your Build
Once your Jenkins job has been created, give it a go. Click **Build Now** on the job page. If the build didn't succeed for any reason, **Console Output** under the page for the failed build will help you debug it. Don't get discouraged!

When the build has succeeded, Gradle will put your build APK in a `build/` directory in your project's location in `ci/` on the build machine. 

Congratulations on building your Android project automatically!

## References
* https://ingorichter.blogspot.com/2012/02/jenkins-change-workspaces
* http://www.ericrgon.com/android-with-circle-ci/ - extra tips on building with CircleCI