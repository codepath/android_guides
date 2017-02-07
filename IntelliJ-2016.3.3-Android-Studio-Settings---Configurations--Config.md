IntelliJ does not download the Android SDK as plug and play type installation, similar to Android Studio. It uses the system provided Android SDK. You have to download the SDK JAR files to its libraries which allow IntelliJ to set-up the Android SDK files in the same fashion that the Android Studio utilizes them. After this initial set up, IntelliJ provides all of the features of the Android Studio including the SDK manager and AVD manager as well as features not available in Studio.

After the initial configuration, you will be able to manage all of your repositories, extras, SDK files through the SDK manager as you would with the Android Studio. 

1) Install latest Java JDK (8)

2) Install latest IntelliJ

3) Install latest Android SDK files to your local drive

4) (We have to describe the path for the environment variables) for Windows 8,10 click Windows +E, select System Properties from the toolbar on top.  Then select Advanced System Settings on the left.

5) Select Environment Variables

6) Under System Variables, select Path

7) Click on Edit (this is an easier way). When you are in it, click on new.  Then go to Browse and choose the location of your Java 1.8.0_102

8) Make sure You know the location of your LOCAL SDK FILES. You will start extracting the JAR files.

9) After the installing IntelliJ;
         Default Project Structure
         Project Settings
         Project 
         SDKs ...Select Android API 25, (java version "1.8.0_102")
         SDK Default (8, Lamdas type annotation etc.)

before you go to the libraries ( you can make your selections afterward), now go to SDK. 
         Select SDK\platorms\android-25\android.jar    as your class path

10) You will need to select the JAR files for IntelliJ to Set-up the Android SDKs.
Click the plus icon on the right, while the class path tab is selected. Locate your JAR files and select them

Start with SDK\tools then follow below respectively.
Most of these files located generally although not always under the BIN folders. Thus open folders and search for them.

Summary of folders are:
tools, tool\Proguard(without this proguard won't work), \Add-ons, \build-tools, patcher, platforms, \platform-tools, \extras, select the Google directory as a whole

Project Library
Create A library File

Goto \sdk\extras\android\
Select this one
Very important JAR files are located here. Gradle wrapper and layout pallet + adapters. 

11) You need to download "gradle.exe" separately to your local drive. Without it, your project (MAVEN and GRADLE )dependencies won't work like they do in Studio.

12) As an option, I would encourage ALL Android programmers to download the Node.JS if using IntelliJ. A short description is provided from wiki below. It's a very real-time ASYNC in JAVA.

Node.js is an open-source, cross-platform JavaScript runtime environment for developing a diverse variety of tools and applications. Although Node.js is not a JavaScript framework,[4] many of its basic modules are written in JavaScript, and developers can write new modules in JavaScript. The runtime environment interprets JavaScript using Google's V8 JavaScript engine.
Node.js has an event-driven architecture capable of asynchronous I/O. These design choices aim to optimize throughput and scalability in Web applications with many input/output operations, as well as for real-time Web applications (e.g., real-time communication programs and browser games).

After you Install them, all you have to do is to tell IntelliJ to how to locate them. The beauty is that after the installation, IntelliJ sorts all of the settings.

13) Node.Js; after installation go to default settings;
Languages and Frameworks
Node.js and NPM
and select the location wherever you have extracted the of the Node.exe as the node interpreter.

14) Gradle binaries;
Make sure the Gradle plug-in is enabled. This is enabled by IntelliJ by default.
Go to default settings 
      Build, Execution, Deployment
                Gradle      Service Directory  select the below.(wherever you have extracted your local gradle binaries) 
.gradle/wrapper/dists/gradle-3.1-bin/37qejo6a26ua35lyn7h1u9v2n/gradle-3.1












 

     
         
           






