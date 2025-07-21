
https://docs.gradle.org/8.14.3/samples/sample_building_cpp_applications.html

Gradle build system is used to build Android applications.

Build system transforms source code->executable application (by using tools to analyze-compile-link-package your application or library).
Gradle uses a task based approach to do these commands.

Tasks encapsulate commands that translate their inputs into outputs.
Plugins define tasks and their configuration. Applying a plugin to your build registers its tasks, and wires them together using their inputs and outputs. 
For example, applying the Android Gradle Plugin (AGP) to your build file will register all the tasks necessary to build an APK, or an Android Library. 
The java-library plugin lets you build a jar from Java source code.
Similar plugins exist for Kotlin, and other languages, but other plugins are meant to extend plugins. 
For example, the protobuf plugin is meant to add protobuf support to existing plugins like AGP or java-library.



go to project level build.gradle file and set Android Plugin Version:
  buildscript {
    ..
    dependencies {
        classpath 'com.android.tools.build:gradle:8.7.3'
        }
  }
