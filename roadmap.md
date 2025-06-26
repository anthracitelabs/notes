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
