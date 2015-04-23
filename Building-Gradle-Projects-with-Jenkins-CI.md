Once you've begun to use Gradle to build and test projects (see [[Getting Started with Gradle|Getting-Started-with-Gradle]]), you might like to build automatically, run tests on every push to your repo, and push your builds to a deployment system.
 
You may be able to adapt this guide if you are using [Travis CI](http://docs.travis-ci.com/user/languages/java/) or [CircleCI](https://circleci.com/docs/android/).

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

### Install the Android SDK 

Follow the instructions for [[Installing Android SDK Tools]].  The guide for taking care of missing SDK dependencies can be greatly simplified by following the automated way.  The directions for doing it manually is also included, except that you need to login to your Jenkins build server, sudo as `ciandroid`, and download the packages on the server.

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
`cd` into the project directory on your build machine. Make sure you see your `build.gradle` file, and run `./gradlew assemble`.

Hopefully, this is what you'll get:

![Successful build output](https://dl.dropboxusercontent.com/u/10808663/gradle_jenkins_android/build_successful.png)
    
### Configure Jenkins
Now you are ready to set up Jenkins to build for you.

Pre-existing settings on your build machine may cause your setup to vary from these directions. You can skip to [Create the Build Job](#create-the-build-job) if you didn't set up an Android user environment, and instead you are building all Jenkins jobs within the same user environment.

### Set Up SSH Keys
If you are using SSH key authentication to connect Jenkins, your `ciandroid` user will need its own SSH key pair. In my work environment, I already had a key pair ready to drop into `/Users/ciandroid/.ssh`. When you set up SSH keys for your build environment, make certain that the `.ssh/` directory and contents are owned by the build environment user (`ciandroid`) and not by any other user. Use `chown` on the  `.ssh/` directory to fix it if necessary.

### Create the Build Node
Using the menus in the Jenkins UI, go to **Jenkins** -> **Manage Jenkins** -> **Manage Nodes** -> **New Node**. Select "Dumb Slave" and give it the name `ciandroid`.

![Creating a build node](https://dl.dropboxusercontent.com/u/10808663/gradle_jenkins_android/create_android_build_node.png)

You'll need to provide your own values for your **Host** and **Credentials** settings, tailored to your own systems. But make sure to set **Remote FS root** with the `ci/` path you created earlier. Also ensure that **Environment Variables** is checked under the **Node Properties** section, and fill in values for `ANDROID_HOME` and `PATH`.

When you set this up, a Jenkins jarfile called `slave.jar` will be downloaded into your node environment `ci/` directory. 

One more detail: My build machine uses SSH on a nonstandard port. If this sounds familiar, you may need to hit the "Advanced" button on the **Host** section and provide the following extra values in the hidden fields:

    Port: [your machine's SSH port number]
    JVM Options: -Djava.awt.headless=true
   
### Create the Build Job
Now you have a Jenkins node that's capable of running jobs. You're ready to create the Jenkins job that actually runs the build.

![Creating a new job](https://dl.dropboxusercontent.com/u/10808663/gradle_jenkins_android/create_new_build.png)

Go back to the Jenkins main page and hit **New Item**. Give the job a name and select "Build a free-style software project." In the job configuration screen that follows, you will do all the setup to tell Jenkins about your source.

Jenkins build configurations can have many steps. The key fields you want to look out for are:
 * `Restrict where this project can be run`: give it the name of the build node you created
 * Source code integration for your version control system
 * Build Triggers
 * Build
 
Also, look for an "Advanced" button just below the `Restrict where this project can be run` setting. This is where Jenkins hides the setting for the source code path on your build node. The `workspace` part of this path is like a variable name for the **Remote FS root** path you added when you set up the Jenkins node. So, `workspace/` stands in for `/Users/ciandroid/ci/`. 

Under the **Build** heading, add shell execute statements with the "Add build step" tool:
* `./gradlew clean`
* `./gradlew assemble`
  
Make sure you hit "Save" after all this work!

All of the following screenshots are intended as a starting point only.

![Screenshot 1](https://dl.dropboxusercontent.com/u/10808663/gradle_jenkins_android/create_job1.png)

![Screenshot 2](https://dl.dropboxusercontent.com/u/10808663/gradle_jenkins_android/create_job2.png)

![Setting the node source location](https://dl.dropboxusercontent.com/u/10808663/gradle_jenkins_android/create_job_node_source.png)

![Screenshot 3](https://dl.dropboxusercontent.com/u/10808663/gradle_jenkins_android/create_job3.png)

![Screenshot 4](https://dl.dropboxusercontent.com/u/10808663/gradle_jenkins_android/create_job4.png)

### Run Your Build
Once your Jenkins job has been created, give it a go. Click **Build Now** on the job page. If the build didn't succeed for any reason, **Console Output** under the page for the failed build will help you debug it. Don't get discouraged!

When the build has succeeded, Gradle will put your new APK in `/Users/ciandroid/ci/your_project_name/build/` on the build machine. 

From here, you can configure Jenkins to build on push, run tests automatically, and upload APKs to a deployment system such as [Hockey](hockeyapp.net).

Congratulations on building your Android project! 

## Automated Publishing

If you are interested in automating APK distribution to the Google Play Store, check out this [guide]
(Automating-Publishing-to-the-Play-Store#setting-up-jenkins-for-automating-ci-builds).

## References
* http://ingorichter.blogspot.com/2012/02/jenkins-change-workspaces-and-build.html
* http://www.ericrgon.com/android-with-circle-ci/ - extra tips on building with CircleCI