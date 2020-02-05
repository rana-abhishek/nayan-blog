---
title: App-Heartbeat
date: 2020-02-03 13:40:48
author: Diwakar Singh
category: Android
tags:
- android
- user interaction
- sessions
---

This is an era of mobile technology where everyone is a smartphone user. To be able to use a smartphone we need to ‘Interact’ with it. A simple touch with a finger to open an app is an example of this ‘interact’ and this phenomenon is called ‘User Interaction’.
Heart of application will continue beat, as long as user interact. Heartbeat is used to calculate sessions, for how much time a user is interacted with application. When a user is started using app, we create a session. A session has two values, start time and end time. For a new session both start and end time will same (current time of system). Every one minute, we check, is user interacted or not. If user is interacted then we update that session by changing its end time (now end time for that session will be current time of system). If user is not interacted then we create a new session. Reason behind to create a heartbeat of application , we will have at least 60 seconds lost.

{% asset_img heartbeat.jpeg Image_1 %}

## Let’s get started

### Create a model

In that data class, we will handle some sessions related tasks (check session is active or not, update session etc.). A session will have two values start time and end time.

```
data class Session(
    val startTime: Long,
    var endTime: Long
    )
```

In that we perform some operations -
#### - Is session active :

In that we check current session is active or not. If difference between System current time and end time of that session is less than a heartbeat( heartbeat duration + heartbeat buffer). In our case heartbeat duration is 60 seconds and heartbeat buffer is 2 seconds.

```
fun isActive(): Boolean {
        return System.currentTimeMillis() - endTime <  HEARTBEAT_DURATION + HEARTBEAT_BUFFER
    }
```

#### - Update current session:

If session is active then we update current session. For updating current session we will put system current time in end time of that session.

```
fun update(): Session {
        endTime = System.currentTimeMillis()
        return this
    }
```

### Create a manager

For handling all heartbeat operations, we will create a session manager. A session manager will manage all sessions activities like — add session, update session. When we open application in onResume() of activity we will start a handler and onPause(),will stop handler.

```
init {
    lifecycleOwner.lifecycle.addObserver(object : LifecycleObserver
           {
               @OnLifecycleEvent(Lifecycle.Event.ON_RESUME)
               fun startSession() {
               Timber.d("Starting Session for activity:${lifecycleOwner.javaClass.name}")
               handler.post(runnable)
               }

               @OnLifecycleEvent(Lifecycle.Event.ON_PAUSE)
               fun pauseSession() {
               Timber.d("Pausing Session for activity:    ${lifecycleOwner.javaClass.name}")
               handler.removeCallbacks(runnable)
               }
          })
    }
```

In handler we are checking heartbeat of application, that user is interacted with app for last 60 seconds or not. If user is interacted then update current session otherwise create new session. We store these sessions into shared preferences.

```
fun heartBeat() {
    if (hasUserInteracted) {
        val currentSession = getLastSession()
        if (currentSession != null && currentSession.isActive()) {
            updateCurrentSession()
        } else {
            createNewSession()
        }
    } else {
        handler.removeCallbacks(runnable)
        onSessionTimeoutListener.onTimeout()
    }

    hasUserInteracted = false
}
```

For more sample code , see the [`App-Heartbeat`](https://github.com/diwakarsinghdiwakar/App-Heartbeat "App-Heartbeat")

## And we’re done!