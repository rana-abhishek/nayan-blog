---
title: App-Heartbeat
date: 2020-02-03 13:40:48
author: Diwakar Singh
tags:
- android
- user interaction
- sessions
---

This is an era of mobile technology where everyone is a smartphone user. To be able to use a smartphone we need to ‘Interact’ with it. A simple touch with a finger to open an app is an example of this ‘interact’ and this phenomenon is called ‘User Interaction’. If a user does not interact for a while, then heartbeat of application will stop.

{% asset_img heartbeat.jpg Image_1 %}

## Let’s get started

### Step -1: Add dependencies

```
// Timber for logging
implementation "com.jakewharton.timber:timber:4.7.1"

//Gson
implementation 'com.squareup.retrofit2:converter-gson:2.6.2'
```

### Step -2: Create a layout for interaction

In layout, we are adding a button which will change background colour.

### activity_session.xml

```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/layout"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="#33000000"
    tools:context=".SessionActivity">

    <Button
        android:id="@+id/btn"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentBottom="true"
        android:layout_centerHorizontal="true"
        android:layout_marginBottom="24dp"
        android:background="@color/colorPrimary"
        android:paddingStart="16dp"
        android:paddingEnd="16dp"
        android:text="@string/click"
        android:textColor="@android:color/white" />

</RelativeLayout>
```

### Step-3: Create a model class for sessions

In that data class, we will perform some sessions related tasks (check session is active or not, update session etc.)

```
data class Session(
    val startTime: Long,
    var endTime: Long) {

    override fun toString(): String {
        return "[$startTime,$endTime]"
    }

    fun isActive(): Boolean {
        return System.currentTimeMillis() - endTime <  HEARTBEAT_DURATION + HEARTBEAT_BUFFER
    }

    fun update(): Session {
        endTime = System.currentTimeMillis()
        return this
    }

    companion object {

        const val HEARTBEAT_DURATION = 60_000L
        private const val HEARTBEAT_BUFFER = 2_000L

        fun fromString(str: String): Session {
            val (startTime, endTime) = str.subSequence(1, str.length  - 1)
                .split(",").map { it.toLong() }
            return Session(startTime, endTime)
        }

        fun newSession() = Session(
            System.currentTimeMillis(),
            System.currentTimeMillis()
        )
    }
}
```

### Step-4: Create session manager

Now we create a session manager, which will manage all sessions activities like — add session, update session. When we open application in onResume() of activity we start a handler and onPause(), stop handler.

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

For more sample code , see the [Draw-Line](https://github.com/diwakarsinghdiwakar/App-Heartbeat "App-Heartbeat")

## And we’re done!