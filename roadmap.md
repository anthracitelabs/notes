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
      
to use the same version of gradle for this project on different machines create gradle script by executing the following command: 
  gradle wrapper

Now back to gradle. It is a BUILD SYSTEM. A build system transforms source code into an executable application. Builds often involve tools to analyze, compile, link, package your application or library.
Gradle uses a TASK based approach to organize and run these commands.
  - Tasks encapsulate commands that translate their inputs into outputs. 
  - Plugins define tasks and their configuration.
Here is the Android developer docs explanation :
"Applying a plugin to your build registers its tasks, and wires them together using their inputs and outputs. For example, applying the Android Gradle Plugin (AGP) to your build file will register all the tasks necessary to build an APK, or an Android Library. The java-library plugin lets you build a jar from Java source code. Similar plugins exist for Kotlin, and other languages, but other plugins are meant to extend plugins. For example, the protobuf plugin is meant to add protobuf support to existing plugins like AGP or java-library.
Gradle prefers convention over configuration so plugins will come with good default values out of the box, but you can further configure the build through a declarative Domain-Specific Language (DSL). The DSL is designed so you can specify what to build, rather than how to build it.The logic in the plugins manages the "how". That configuration is specified across several build files in your project (and subprojects).
Task inputs can be files and directories as well as other information encoded as Java types (integer, strings, or custom classes). Outputs can only be directory or files as they have to be written on disk. Wiring a task output into another task input, links the tasks together so that one has to run before the other."

Gradle uses a Domain-Specific Language (DSL) to configure builds. For example, configuring the Android part of your build in build.gradle.kts file might look like:
android {
    namespace = "com.example.app"
    compileSdk = 34
    // ...

    defaultConfig {
        applicationId = "com.example.app"
        minSdk = 34
        // ...
    }
}

The Maven build system introduced a dependency specification, storage and management system. Libraries are stored in repositories (servers or directories), with metadata including their version and dependencies on other libraries. You specify which repositories to search, versions of the dependencies you want to use, and the build system downloads them during the build.
Maven Artifacts are identified by group name (company, developer, etc), artifact name (the name of the library) and version of that artifact. This is typically represented as group:artifact:version.
You can also modularize your project into subprojects (also known as "modules" in Android Studio), which can also be used as dependencies. 

Variants contain different code or are built with different options, and are composed of build types and product flavors.
Build types vary declared build options. By default, AGP sets up "release" and "debug" build types, but you can adjust them and add more (perhaps for staging or internal testing).
A debug build marks the application as "debuggable", signing it with a generic debug key and enabling access to the installed application files on the device.
A release build optimizes the application, signs it with your release key, and protects the installed application files.
Using product flavors, you can change the included source and dependency variants for the application. For example, you may want to create "demo" and "full" flavors for your application, or perhaps "free" and "paid" flavors. You write your common source in a "main" source set directory, and override or add source in a source set named after the flavor.
AGP creates variants for each combination of build type and product flavor.  If you don't define flavors, the variants are named after the build types. 
If you define both, the variant is named <flavor><Buildtype>. For example, with build types release and debug, and flavors demo and full, AGP will create variants:
    demoRelease
    demoDebug
    fullRelease
    fullDebug

Gradle groups library dependencies into different scopes (called configurations in the Gradle API), allowing you to specify different sets of library dependencies to be used in different parts of your build. 
For example, AGP defines implementation and api scopes, your way of specifying whether a dependency should be exposed to users of your subproject.

// In a module-level build script
// explicit dependency strings ("group:artifact:version")
dependencies {
    implementation("com.example:library1:1.2.3")
    api("com.example:library2:1.1.1")
}

Note that gradle automatically chooses newest version of a library if two different libraries depend on different versions of this library (LibA depends on LibC 2.1.1 and LibB depends on LibC 3.2.2. Gradle chooses LibC 3.2.2 for both libraries). You can use the Gradle dependencies task by running ./gradlew app:dependencies to display a tree of all dependencies used by your app module.
Whenever you see -> in your dependencies report, a requestor (your application or another library) uses a version of that dependency that it isn't expecting. In many cases, this doesn't cause any issues, as most libraries are written for backward compatibility. However, some libraries may make incompatible changes, and this report can help you determine where new issues with your application's behavior are coming from.

* You can specify requested versions directly, in a version catalog, or in a Bill of Materials (BOM).

In the module-level build file:
The compileSdk determines which Android and Java APIs are available when compiling your source code. 
The minSdk specifies the lowest version of Android that you want your app to support. Setting minSdk restricts which devices can install your app.
The targetSdk serves two purposes:
    - It sets runtime behavior of your application.
    - It attests which version of Android you've tested against.

compileSdk gives you access to new APIs
targetSdk sets the runtime behavior of your app
targetSdk must be less than or equal to compileSdk

For example, when API 23 introduced the runtime permissions model, not all apps were ready to immediately adopt it. By setting targetSdk to 22, those apps could run on API 23 devices without using runtime permissions, and could use features included in the latest compileSdk version.



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

