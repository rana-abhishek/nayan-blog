---
title: Applying Proguard in an android application
date: 2020-05-25 18:40:48
author: Diwakar Singh
category: Android
tags:
- android
- proguard
---

To make an application is not good enough, but it also needs to make secure and optimize. It’s the basic needs of an application.

To make your app as small as possible, you should enable shrinking in your release build to remove unused code and resources. When enabling shrinking, you also benefit from obfuscation, which shortens the names of your app’s classes and members, and optimization, which applies more aggressive strategies to further reduce the size of your app.

ProGuard is a tool used to do the following in an Android application:

## Minify the code
Detects and safely removes unused classes, fields, methods, and attributes from your app and its library dependencies

## Obfuscate the code
Shortens the name of classes and members, which results in reduced DEX file sizes.

## Optimize the code
Inspects and rewrites your code to further reduce the size of your app’s DEX files.

{% asset_img flow_diagram.jpeg Image_1 %}

To enable shrinking, obfuscation, and optimization, include the following in your project-level build.gradle file.

In that data class, we will handle some sessions related tasks (check session is active or not, update session etc.). A session will have two values start time and end time.

```
android {
    buildTypes {
        release {
            shrinkResources true
            minifyEnabled true
            proguardFiles getDefaultProguardFile(
              'proguard-android-optimize.txt'),
              'proguard-rules.pro'
        }
      }
    ...
}
```
While enabling proguard in your application there are some rules , that should be considered. Do not forget to add the Proguard rules in proguard-rules.pro file for any library that you have included in your project.

Like this for classes, also can apply for members and fields

```
-keepclassmembers class <className with pakage>.** { *; }
```

For Warning : You need to take a look on stacktrace to find which classes gives those warnings

```
-dontwarn <classes_name>
-dontwarn java.nio.file.*
```

## Some stats related to APK size

{% asset_img stats.png Image_2 %}

## References

https://developer.android.com/studio/build/shrink-code

## Some good reads you may like:-

1. [App-Heartbeat](https://nayan.co/blog/Android/App-Heartbeat/)