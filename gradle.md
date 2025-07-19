
https://docs.gradle.org/8.14.3/samples/sample_building_cpp_applications.html

go to project level build.gradle file and set Android Plugin Version:
  buildscript {
    ..
    dependencies {
        classpath 'com.android.tools.build:gradle:8.7.3'
        }
  }
