start by configuring your build:
  https://developer.android.com/build

walkthroughs hello-jni, native activity and teapot
  https://developer.android.com/ndk/samples/walkthroughs

take a look at android game development libraries:
  https://developer.android.com/games/develop/overview

I have jdk-21.0.7 installed and JAVA_HOME defined, also jdk bin folder added to PATH variable.
Gradle downloaded

Android command line tools only downloaded.
  https://developer.android.com/studio/index.html#command-line-tools-only

Unzip android command line tools into ANDROID SDK folder. 
List installed and available packages:
sdkmanager --list [options] \
           [--channel=channel_id] // Channels: 0 (stable), 1 (beta), 2 (dev), or 3 (canary)
Execute the following to list stable packages:
sdkmanager --list --channel=0

To install packages:
sdkmanager packages [options]
The packages argument is an SDK-style path, as shown with the --list command, wrapped in quotes. For example, "build-tools;34.0.0" or "platforms;android-33".
To install  Android SDK Platform 36, Android SDK Build-Tools 36 and Android SDK Platform-Tools
  sdkmanager "platforms;android-36" "build-tools;36.0.0" "platform-tools"

SDK folder structure:
android_sdk/cmdline-tools/version/bin/
android_sdk/build-tools/version/
android_sdk/platform-tools/
android_sdk/emulator/

Recommended settings for environment: (env variables reference : https://developer.android.com/tools/variables#set)
ANDROID_HOME - the path to the SDK installation directory
ANDROID_USER_HOME - the path to the user preferences directory for tools that are part of the Android SDK. Defaults to $HOME/.android/
add the following to PATH
  ANDROID_HOME/build-tools/version
  ANDROID_HOME/cmdline-tools/version/bin
  ANDROID_HOME/platform-tools 

Android project folder structure:

.gradle/                  Gradle project cache directory
                          Managed by Gradle and contains the downloaded Gradle distribution, project cache, and configuration files.

build.gradle(.kts)        Root build file. Should only contain plugin declarations to set up a common plugin classpath across subprojects.
                          Other code should reside in the settings or nested-project-level build files.

gradle.properties         Gradle execution configuration
                          Contains Gradle properties, controlling Gradle build environment aspects such as heap size, caching and parallel execution.

gradlew (linux, Mac)      Gradle wrapper file
gradlew.bat (Windows)     Bootstraps your build by downloading a Gradle distribution and then forwarding commands to it. This lets you run builds without having to preinstall Gradle.

local.properties          Local machine configuration
                          Contains properties related to the local machine, such as the location of the Android SDK.
                          Exclude this file from source control!

settings.gradle(.kts)     Gradle build initialization
                          Contains global build information for Gradle initialization and project configuration, such as
                                          project name
                                          list of subprojects to include in this build
                                          repository specifications to locate plugins and dependencies
                                          external Version Catalog imports.
gradle/                   includes wrapper folder and libs.versions.toml file (version catalog file; 
                          Defines variables for dependencies and plugins used inside your build. You specify which versions you want to use here, ensuring consistency across all subprojects in your project.)

app/                      Subproject directory. Any directory can be a subproject, and must contain at least a build.gradle(.kts) file, and be included in the build using settings.gradle(.kts).
  build.gradle(.kts)      Subproject-level build file
                          Declares how to build this subproject. Each subproject requires a separate build file, and should contain
                                      plugins used to build this subproject
                                      configuration blocks required by plugins
                                      dependencies (libraries and platforms) included when building this subproject
  src/                    Subproject source files
                          Groups source files (application code and resources) into source sets. The main source set contains source files that are common to all variants, while other source sets contain source files unique to a variant.
      main/                   Main source set
                              Source code and resources that are common across all build variants. This source acts as the base for all builds, and other, more specific source sets add to or override this source.
          java/           The java directory can contain mixed Java and Kotlin source code. If this subproject contains only Kotlin code, you can rename this directory kotlin.
          kotlin/         Android is a Kotlin-first platform. Java source is supported, but new APIs target the Kotlin language.
          res/            Android resource files
                          Contains application resources, such as XML files and images. 
                          All applications use some basic resources, such as launcher icons, but many of these resources, such as layouts and menus, are only used in view-based applications. 
                          Compose applications use String resources defined under this directory.
          AndroidManifest.xml       Android application metadata
                                    Read by Android package manager to tell the system
                                              components defined by your application
                                              necessary permissions
                                              device compatibility
                                              Android platform restrictions
      androidTest/          Device test source set
      test/                 Host test source set
  proguard-rules.pro        R8 configuration rules
                            Defines rules to control application shrinking, optimization, and obfuscation. 
                            R8 removes unneeded code and resources, optimizes runtime performance and further minimizes your code by renaming identifiers.

Lets try initializing a java application project using gradle:
    gradle init --type java-application  --dsl kotlin

Folder structure is created like below:
      settings.gradle.kts
      gradlew.bat
      gradlew
      gradle.properties
      gradle/
          wrapper/
          libs.versions.toml
      app/
          build.gradle.kts
          src/
              main/
                  java/
                      org/
                          example/
                              App.java
                  resources/                  
              test/
                  java/
                      org/
                          example/
                              AppTest.java
                  resources/
      
to use the same version of gradle for this project on different machines: 
  gradle wrapper

edit build.gradle file:
        buildscript {
          repositories {
              jcenter()
          }
          dependencies {
              classpath 'com.android.tools.build:gradle:2.3.3'
          }
        }
      
        apply plugin: 'com.android.application'
      
        android {
          compileSdkVersion 23
          buildToolsVersion '26.0.1'
        }

