---
title: Android Testing Strategy
date: 2019-11-29 17:07:05
author: Puneet Kashyap
category: Android
tags: 
- Instrumentation Test
- Unit Test
- Android Testing
---

#Android Testing Strategy
Testing android application is quite hard. There is no set guidelines for us to follow. When ever I started thinking of testing my application I always get confused where to start, should I write unit tests or instrumentation tests and should I start with integration and End to end tests. There is also a lot of confusion on frameworks available for android testing.
This article has been written in a sense to address this confusion and show us as developers what kind of testing is most preferred and what frameworks are available.

##Kinds of Tests 
###Unit Test 
This is often referred to as local tests and  doesn't require a device or emulator for running them. These can be classified broadly into categories
..*Local/ Pure Unit tests - tests which can run on JVM, mainly written for testing business logic. On android JUnit, Mockito, the mockable Android JARs give us a nice combination of tools to run fast unit-style tests in the local JVM.
*Android  Unit Tests -  tests which  requires the Android system (ART) and for this we need to replace android dependencies using Roboelectric
Guidelines
..*If you have dependencies on the Android framework, particularly those that create complex interactions with the framework, it's better to include framework dependencies using Robolectric.
..*If your tests have minimal dependencies on the Android framework, or if the tests depend only on your own objects, it's fine to include mock dependencies using a mocking framework like Mockito and PowerMock.

Reference Url : https://developer.android.com/training/testing/unit-testing/local-unit-tests

###Instrumentation Tests 
These tests requires device or an emulator for running. This is mostly used for UI testing but it can be used to test none UI logic as well.
..*This is useful when you need to test code that has a dependency on context. 
..*UI tests can be an essential component of any testing strategy since they can uncover issues related to UI, hardware, firmware, and backwards compatibility
 Reference Url: https://developer.android.com/training/testing/unit-testing/instrumented-unit-tests

##Testing Frameworks
..*###Android X Test Framework**
 It is testing framework and APIs provided  by android team for writing unit tests.
 
..*###Roboelectric 
It is a framework that brings fast and reliable unit tests to android. Runs inside JVM or your workstation in seconds. This is usually used to Integration testing. Integration tests validate how your code interacts with other parts of the system but without the added complexity of a UI framework.

```app/src/test/java - for any unit test which can run on the JVM```

**Question** : How does it work?
**Answer** : Unlike traditional emulators-based androids tests, it tests run inside a sandbox which allows the android environment to be precisely configured to the desired conditions for each test. It lets you run your tests on your workstation, or on your continuous integration environment in a regular JVM, without an emulator. It handles inflation of views, resource loading, and lots of other stuff that's implemented in native C code on Android devices. This allows tests to do most things you could do on a real device. It's easy to provide  our own implementation for specific SDK methods too, so you could simulate error conditions or real-world sensor behaviour. It allows a test style that is closer to black box testing, making the tests more effective for refactoring and allowing the tests to focus on the behaviour of the application instead of the implementation of Android

**Question** : Why should be prefer this?
**Answer** : In order for this to run tests it needs regular JVM,  Because of this, the dexing, packaging, and installing-on-the emulator steps aren't necessary, reducing test cycles from minutes to seconds so you can iterate quickly and refactor your code with confidence. Robolectric executes your code against real (not mock) Android JARs in the local JVM.

..*###Espresso 
 Use it to write concise, beautiful, and reliable Android UI tests. These tests are called Instrumentation tests and unlike unit tests takes more time to run them.

```app/src/androidTest/java - for any instrumentation test which should run on an Android```

**Question** : How does it work?
**Answer** : it requires an emulator or a real device to run tests. At the time of execution along with the main application, A testing application is also installed in the device which controls main application automatically.

..*###UI Automator 
It allows us to write cross application functional tests ( End to End) . Example, Sharing messages via Text intent or sending email via locally installed email clients.

..*###Monkey Runner CL
 Monkey is a command line tool which sends pseudo random events to your device. You can restrict Monkey to run only for a certain package and therefore instruct Monkey to test only your application. it can be used for Stress testing for android.

Reference Url : https://developer.android.com/studio/test/monkey			

##Recommandations
..*Creating test groups - @SmallTest. @MediumTest and @LargeTest annotation allows us to classify tests. Allows you to run, for example, only short running tests for development cycle. You may run your long running tests on a continuous integration server.
This can be easily configured this via InstrumentationTestRunner in user build.gradle	(app)

..*We can use a three tiered approach
 **Pure Unit tests : **   these can be written for our business logic which are completely android independent of API and can run on JVM. These can be written using Junit Framework. Roboelectric Unit tests: where code has only small dependencies on android APIs and can be easily mocked with Roborelectric.
**Android Instrumentation tests :** where code heavily interact with device hardware, sensors and android APIs. These tests will usually take most time to run.



