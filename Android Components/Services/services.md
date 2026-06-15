# Android Services

We’ll learn step by step with:

* Theory
* Lifecycle
* Why needed
* Real-world usage
* Full working examples
* Interview questions
* Best practices

---

# PART 1 — What is a Service?

A **Service** is an Android component used to perform:

* long-running tasks
* background operations
* work without UI

Examples:

* Music playback
* File downloading
* GPS tracking
* Socket connection
* Uploading logs

---

# Important Interview Point

A Service:

* does NOT have UI
* runs in background
* BUT runs on **Main Thread by default**

Many candidates fail here.

---

# Why Not Use Activity?

Suppose:

```text id="8qu8cm"
User starts song in Activity
↓
User presses Home button
↓
Activity goes to background
↓
Song should still continue
```

Activity lifecycle is UI dependent.

Service lifecycle is independent.

So we use Service.

---

# PART 2 — Service Lifecycle

---

# Started Service Lifecycle

```text id="5stx9q"
startService()
    ↓
onCreate()          (only once)
    ↓
onStartCommand()    (every start)
    ↓
Service Running
    ↓
stopSelf()/stopService()
    ↓
onDestroy()
```

---

# Interview Question

## Q: Difference between onCreate() and onStartCommand()?

| onCreate       | onStartCommand              |
| -------------- | --------------------------- |
| Called once    | Called every startService() |
| Initialization | Actual work                 |

---

# PART 3 — Started Service (Most Important)

Used when:

* service should run independently
* even if activity closes

Example:

* music app
* upload manager

---

# FULL PRACTICE EXAMPLE — Started Service

---

# Step 1 — Create Service

```kotlin id="1b5xvn"
class DownloadService : Service() {

    override fun onCreate() {
        super.onCreate()

        Log.d("SERVICE", "onCreate called")
    }

    override fun onStartCommand(
        intent: Intent?,
        flags: Int,
        startId: Int
    ): Int {

        Log.d("SERVICE", "onStartCommand called")

        CoroutineScope(Dispatchers.IO).launch {

            for (i in 1..10) {

                delay(1000)

                Log.d(
                    "SERVICE",
                    "Downloading $i"
                )
            }

            stopSelf()
        }

        return START_STICKY
    }

    override fun onDestroy() {
        super.onDestroy()

        Log.d("SERVICE", "onDestroy called")
    }

    override fun onBind(intent: Intent?): IBinder? {
        return null
    }
}
```

---

# Step 2 — Manifest

```xml id="sv5f0f"
<service
    android:name=".DownloadService"
    android:exported="false"/>
```

---

# Step 3 — Start Service

```kotlin id="4u5n7d"
val intent = Intent(this, DownloadService::class.java)

startService(intent)
```

---

# Step 4 — Stop Service

```kotlin id="x02yrj"
stopService(Intent(this, DownloadService::class.java))
```

---

# Important Explanation

---

## Why return START_STICKY?

```kotlin id="c1lt9x"
return START_STICKY
```

Means:

* if system kills service
* recreate service automatically

Used in:

* music apps
* location tracking

---

# START_STICKY vs START_NOT_STICKY vs START_REDELIVER_INTENT

---

# 1. START_STICKY

```kotlin id="w2w75q"
return START_STICKY
```

System recreates service.

Intent may become null.

---

## Real Example

Music player.

Music should continue.

---

# 2. START_NOT_STICKY

```kotlin id="xumwji"
return START_NOT_STICKY
```

If killed:

* do NOT recreate.

---

## Real Example

Temporary upload task.

---

# 3. START_REDELIVER_INTENT

```kotlin id="4yl8gx"
return START_REDELIVER_INTENT
```

Recreate service AND resend intent.

---

## Real Example

Downloading a specific file.

Need same URL again.

---

# Interview Trick Question

## Q: Multiple startService() calls create multiple services?

NO.

Only:

* one service instance

But:

```kotlin id="jwm7qt"
onStartCommand()
```

called multiple times.

---

# PART 4 — Foreground Service

VERY IMPORTANT.

Android 8+ heavily asks this.

---

# Problem

Android kills background services aggressively.

Solution:

* Foreground Service

---

# Foreground Service Features

* Higher priority
* Less likely killed
* Persistent notification mandatory

---

# Real Examples

* Google Maps navigation
* Spotify
* Fitness tracker

---

# FULL PRACTICE EXAMPLE — Foreground Service

---

# Step 1 — Notification Channel

```kotlin id="l8nq0j"
fun createNotificationChannel(context: Context) {

    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {

        val channel = NotificationChannel(
            "music_channel",
            "Music",
            NotificationManager.IMPORTANCE_LOW
        )

        val manager =
            context.getSystemService(
                NotificationManager::class.java
            )

        manager.createNotificationChannel(channel)
    }
}
```

---

# Step 2 — Foreground Service

```kotlin id="uz4h8r"
class MusicService : Service() {

    override fun onStartCommand(
        intent: Intent?,
        flags: Int,
        startId: Int
    ): Int {

        val notification =
            NotificationCompat.Builder(
                this,
                "music_channel"
            )
                .setContentTitle("Music Playing")
                .setContentText("Song is running")
                .setSmallIcon(R.drawable.ic_launcher_foreground)
                .build()

        startForeground(1, notification)

        CoroutineScope(Dispatchers.IO).launch {

            while (true) {

                delay(1000)

                Log.d("SERVICE", "Music running")
            }
        }

        return START_STICKY
    }

    override fun onBind(intent: Intent?): IBinder? {
        return null
    }
}
```

---

# Step 3 — Manifest

```xml id="3v56xj"
<uses-permission
    android:name="android.permission.FOREGROUND_SERVICE"/>

<service
    android:name=".MusicService"
    android:foregroundServiceType="mediaPlayback"/>
```

---

# Step 4 — Start Foreground Service

Android 8+:

```kotlin id="vt97x0"
val intent = Intent(this, MusicService::class.java)

ContextCompat.startForegroundService(this, intent)
```

---

# IMPORTANT Interview Question

## Q: Why startForegroundService() introduced?

Because Android 8 restricted:

* background execution

Apps abused battery.

---

# VERY IMPORTANT

After:

```kotlin id="0ow69w"
startForegroundService()
```

You MUST call:

```kotlin id="1amcm2"
startForeground()
```

within:

* 5 seconds

Otherwise:

* app crashes

---

# PART 5 — Bound Service

### Bound Service — Complete Explanation with Full Example

A **Bound Service** is a Service that allows other components (Activity, Fragment, another Service) to **bind to it and interact with it directly**.

Think of it as:

```text
Started Service
    ↓
"Do this work"

Bound Service
    ↓
"Let me talk to you while you're running"
```

---

### Real World Example

Imagine a music app.

```text
MusicService
    ↓
Playing song in background
```

Your Activity needs:

```text
Play
Pause
Next
Current Position
Current Song
```

The Activity must communicate with the Service.

A Bound Service makes this possible.

---

### How Bound Service Works

```text
Activity
   |
bindService()
   |
   v
MusicService
   |
returns Binder
   |
   v
Activity gets Service reference
   |
   v
Calls service methods directly
```

---

### Lifecycle

```text
bindService()
      ↓
onCreate()
      ↓
onBind()
      ↓
Client gets Binder
      ↓
Communication
      ↓
unbindService()
      ↓
onUnbind()
      ↓
onDestroy()
```

---

### Step 1: Create the Service

Suppose we want a service that generates random numbers.

#### RandomNumberService.kt

```kotlin
class RandomNumberService : Service() {

    /**
     * Binder object returned to clients
     */
    private val binder = LocalBinder()

    /**
     * Binder implementation
     */
    inner class LocalBinder : Binder() {

        fun getService(): RandomNumberService {
            return this@RandomNumberService
        }
    }

    override fun onCreate() {
        super.onCreate()

        Log.d("SERVICE", "onCreate")
    }

    override fun onBind(intent: Intent?): IBinder {

        Log.d("SERVICE", "onBind")

        return binder
    }

    override fun onUnbind(intent: Intent?): Boolean {

        Log.d("SERVICE", "onUnbind")

        return super.onUnbind(intent)
    }

    override fun onDestroy() {
        super.onDestroy()

        Log.d("SERVICE", "onDestroy")
    }

    fun getRandomNumber(): Int {

        return (1..100).random()
    }
}
```

---

#### Understanding the Binder

This is the most important part:

```kotlin
inner class LocalBinder : Binder() {

    fun getService(): RandomNumberService {
        return this@RandomNumberService
    }
}
```

When the Activity binds:

```text
Activity
    ↓
asks Service
    ↓
Service returns Binder
    ↓
Binder returns Service instance
```

Now Activity has:

```kotlin
RandomNumberService
```

reference.

---

### Step 2: Register Service

#### AndroidManifest.xml

```xml
<service
    android:name=".RandomNumberService"
    android:exported="false"/>
```

---

### Step 3: Bind from Activity

#### MainActivity.kt

```kotlin
class MainActivity : AppCompatActivity() {

    private var randomService: RandomNumberService? = null

    private var isBound = false

    private val serviceConnection =
        object : ServiceConnection {

            override fun onServiceConnected(
                name: ComponentName?,
                service: IBinder?
            ) {

                val binder =
                    service as RandomNumberService.LocalBinder

                randomService =
                    binder.getService()

                isBound = true

                Log.d(
                    "ACTIVITY",
                    "Connected"
                )

                val random =
                    randomService?.getRandomNumber()

                Log.d(
                    "ACTIVITY",
                    "Random = $random"
                )
            }

            override fun onServiceDisconnected(
                name: ComponentName?
            ) {

                isBound = false

                randomService = null

                Log.d(
                    "ACTIVITY",
                    "Disconnected"
                )
            }
        }

    override fun onStart() {
        super.onStart()

        Intent(
            this,
            RandomNumberService::class.java
        ).also {

            bindService(
                it,
                serviceConnection,
                BIND_AUTO_CREATE
            )
        }
    }

    override fun onStop() {
        super.onStop()

        if (isBound) {

            unbindService(serviceConnection)

            isBound = false
        }
    }
}
```

---

### What Happens Internally?

#### Activity Starts

```text
MainActivity.onStart()
```

Calls:

```kotlin
bindService(...)
```

---

#### Android Creates Service

```text
RandomNumberService.onCreate()
```

Called once.

---

#### Android Calls

```text
RandomNumberService.onBind()
```

Returns:

```kotlin
binder
```

---

#### Activity Receives Binder

```kotlin
onServiceConnected()
```

gets:

```kotlin
IBinder
```

Convert:

```kotlin
val binder =
    service as RandomNumberService.LocalBinder
```

Get service:

```kotlin
randomService =
    binder.getService()
```

Now Activity has direct access.

---

#### Calling Service Methods

Now you can do:

```kotlin
val number =
    randomService?.getRandomNumber()
```

Exactly like a normal object:

```kotlin
randomService?.getRandomNumber()
```

No Intent.

No Broadcast.

No Messenger.

Direct function call.

---

### Lifecycle Log Output

#### App Opens

```text
onCreate
onBind

Connected
Random = 42
```

#### App Closes

```text
onUnbind
onDestroy
```

---

### Multiple Activities Binding

Possible.

```text
Activity A
       \
        \
         Service
        /
       /
Activity B
```

Both can bind.

Service remains alive while at least one client is bound.

---

### When Does Service Destroy?

Suppose:

```text
Activity A bound
Activity B bound
```

Service alive.

Now:

```text
Activity A unbinds
```

Service still alive.

Because:

```text
Activity B still bound
```

Only when:

```text
No clients bound
```

Android calls:

```text
onDestroy()
```

---

### Bound + Started Service Together

A service can be both.

```kotlin
startService(intent)

bindService(...)
```

Example:

```text
Music playback
```

Service should:

```text
Continue playing
```

even if Activity disappears.

So:

```text
Started Service
```

And Activity also needs controls:

```text
Play
Pause
Seek
```

So:

```text
Bound Service
```

Both at same time.

---

### Real Spotify Example

```text
MusicService
      ↑
      |
   Binder
      |
      ↓
Now Playing Activity
```

Activity calls:

```kotlin
service.play()

service.pause()

service.seekTo(12000)

service.getCurrentPosition()
```

through Binder.

---

### Interview Questions

#### Q: Why use a Bound Service?

To allow components to communicate directly with a Service and invoke its methods.

---

#### Q: What does bindService() return?

It doesn't return the Service directly.

Android eventually calls:

```kotlin
onServiceConnected()
```

with an:

```kotlin
IBinder
```

object.

---

##### Q: What is Binder?

Binder is Android's communication mechanism.

For a local bound service, Binder simply provides access to the Service instance.

---

#### Q: Does a Bound Service run in a separate thread?

No.

Like any Service:

```text
Runs on Main Thread by default
```

Heavy work should still use:

```kotlin
Dispatchers.IO
```

or another worker thread.

---

#### Interview Answer (Short)

> A Bound Service allows components such as Activities to bind to a Service and communicate with it directly. The Service returns an IBinder object in `onBind()`, which provides access to the Service instance. Clients can then call the Service's methods directly. Common use cases include music controls, download progress updates, and chat connections. Unlike a Started Service, a Bound Service exists only while one or more clients are bound to it.



---

# PART 6 — Service vs Thread

VERY COMMON.

---

| Service                 | Thread               |
| ----------------------- | -------------------- |
| Android component       | Execution unit       |
| Lifecycle aware         | No lifecycle         |
| Runs on Main Thread     | Separate thread      |
| Background task manager | Concurrent execution |

---

# IMPORTANT INTERVIEW ANSWER

Service is NOT a thread.

Heavy work inside service still blocks UI.

---

# WRONG WAY

```kotlin id="0l8tpo"
override fun onStartCommand(...): Int {

    Thread.sleep(10000)

    return START_STICKY
}
```

Causes:

* ANR
* app freeze

---

# Correct Way

```kotlin id="d4sl8c"
CoroutineScope(Dispatchers.IO).launch {

}
```

---

# PART 7 — IntentService

Deprecated but interviewers ask.

---

# What is IntentService?

Special service that:

* creates worker thread automatically
* processes queue sequentially
* stops itself automatically

---

# Old Example

```kotlin id="a6fz1r"
class MyIntentService : IntentService("MyIntentService") {

    override fun onHandleIntent(intent: Intent?) {

        Log.d("TAG", "Background task")
    }
}
```

---

# Why Deprecated?

Replaced by:

* WorkManager
* Coroutine + Service

---

# PART 8 — Service vs WorkManager

SUPER IMPORTANT MODERN QUESTION.

---

| Service        | WorkManager          |
| -------------- | -------------------- |
| Immediate work | Deferred work        |
| User-visible   | Background scheduled |
| Long-running   | Guaranteed execution |
| Music          | Sync data            |

---

# Use Service When

* user actively aware
* ongoing task
* immediate execution

Examples:

* music
* navigation
* calls

---

# Use WorkManager When

* retry needed
* background sync
* upload logs

---

# PART 9 — Real Interview Scenarios

---

# Scenario 1

## Why Spotify uses Foreground Service?

Because:

* music continues after app close
* Android kills background tasks
* foreground service gets higher priority

---

# Scenario 2

## Why Service alone dangerous?

Because:

* service runs on main thread

Heavy work causes:

* ANR

---

# Scenario 3

## Can Service survive app kill?

Depends.

If:

```kotlin id="t7y2rk"
START_STICKY
```

system may recreate.

But:

* force stop kills everything

---

# Scenario 4

## Can multiple activities bind same service?

YES.

Multiple clients possible.

---

# PART 10 — Modern Best Practice

Today companies use:

* WorkManager
* Coroutines
* Lifecycle-aware APIs

More than classic services.

---

# PART 11 — Most Asked Interview Questions

---

# Q1

## Does Service run on separate thread?

NO.

Runs on Main Thread.

---

# Q2

## Why foreground service mandatory?

To:

* avoid battery abuse
* inform user

---

# Q3

## Difference between bindService and startService?

| bindService        | startService      |
| ------------------ | ----------------- |
| Client-server      | Independent       |
| Stops after unbind | Continues running |

---

# Q4

## What if startForeground() not called?

App crashes.

---

# Q5

## Can service update UI?

Directly NO.

Use:

* Broadcast
* LiveData
* Flow
* Binder callback

---

# PART 12 — Best Interview Explanation

If interviewer says:

## “Explain Android Services”

You can answer:

> Service is an Android component used for long-running background tasks without UI.
> By default it runs on the main thread, so heavy work should be moved to coroutines or worker threads.
> There are Started Services, Bound Services, and Foreground Services.
> Foreground services are used for user-visible tasks like music and navigation because they show persistent notifications and have higher system priority.

---

# Best Practice Exercises

Practice these yourself:

1. Music player foreground service
2. File download service
3. Timer service
4. Bound service for random number
5. Service + BroadcastReceiver communication
6. Service + Notification action buttons
7. Service + CoroutineScope
8. Service restart after reboot

These cover almost all interview questions.
