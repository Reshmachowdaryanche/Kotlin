# Service

A **Service** is an Android component that can perform long-running operations in the background.

- Runs on the **main thread (UI thread)** by default → if you do heavy work here, it will block UI and cause **ANR (App Not Responding)**.
- If you need background threading, you must manually create a new thread inside the Service  
  (e.g., using `Thread`, `ExecutorService`, or `Coroutine`).

## Lifecycle Methods

### `onCreate()`
Called once when the service is created.

### `onStartCommand()`
Called whenever the service is started.

### `onDestroy()`
Called when the service is destroyed.

## Use Case

👉 Use when you want a service that runs indefinitely.

Examples:
- Music player
- Location tracking
- Real-time chat

---

# IntentService

> **Deprecated after Android 11**  
> Replaced by `JobIntentService` or `WorkManager`.

`IntentService` is a subclass of `Service` designed to handle asynchronous requests (`Intent`s) on a worker thread automatically.

- Each incoming intent is queued and processed sequentially in a background thread.
- After finishing all work, the service stops itself automatically.

## Lifecycle Methods

### `onHandleIntent(Intent?)`
Runs on a background thread, so no need to manage threading yourself.

---

## Use Case

👉 Use for short background tasks that should finish automatically.

Examples:
- Downloading a file
- Uploading logs
- Syncing data

---

# Key Differences

| Feature | Service | IntentService |
|---|---|---|
| Threading | Runs on main thread by default | Runs on worker thread automatically |
| Work Handling | You must manage threads manually | Handles intent queue automatically |
| Multiple Requests | You need to handle manually (threads) | Queues requests sequentially |
| Lifecycle | Runs until explicitly stopped | Stops automatically after work is done |
| Best For | Long-running tasks (music, tracking, chat) | Short background tasks (download, upload, sync) |
| Status | Still used | Deprecated → replaced by `JobIntentService` / `WorkManager` |

---

# Quick Example

## Service

```kotlin
class MyService : Service() {

    override fun onStartCommand(
        intent: Intent?,
        flags: Int,
        startId: Int
    ): Int {

        Thread {
            // Do background work manually
        }.start()

        return START_STICKY
    }

    override fun onBind(intent: Intent?): IBinder? = null
}
```

## Intent Service
```
class MyIntentService : IntentService("MyIntentService") {
    override fun onHandleIntent(intent: Intent?) {
        // Automatically runs on background thread
        val data = intent?.getStringExtra("data")
        // Do work here
    }
}
```
# IntentService vs JobScheduler vs WorkManager

# 1. IntentService

## What it is
A special type of `Service` that runs tasks on a background thread sequentially.

## How it works
You send `Intent`s → it queues them → executes one by one → stops itself when done.

## Threading
Automatically runs in a worker thread  
(no need to create your own thread).

## Persistence
Task dies if the app is killed.

## Best For
Short background tasks like:
- Logging
- Uploading
- Small syncs

## Status
❌ Deprecated after Android 11.

---

## ✅ Pros

- Simple to use
- Handles threading for you
- Stops automatically

---

## ❌ Cons

- No guaranteed execution if app is killed
- Not suitable for long-running tasks
- Deprecated → replaced by `WorkManager` / `JobIntentService`

---

# 2. JobScheduler

## What it is
A framework (`API 21+`) to schedule jobs (background tasks) that the system runs when conditions are met.

## How it works
You define:
- `JobService`
- `JobInfo`

with conditions like:
- WiFi
- Charging
- Device idle

The system batches and runs jobs efficiently.

## Threading
Runs on the main thread, so you must manually create background threads.

## Persistence
Survives app restarts and device reboots (if configured).

## Best For
Periodic tasks and system-managed jobs.

Examples:
- Sync data only on WiFi
- Run tasks while charging
- Background maintenance work

---

## ✅ Pros

- System-optimized (battery + Doze mode aware)
- Supports constraints:
  - WiFi
  - Charging
  - Idle mode
- Persists across device restarts

---

## ❌ Cons

- Available only on API 21+ (`Lollipop`)
- Manual thread management required
- Tasks may be delayed because system decides execution timing

---

# 3. WorkManager

## What it is
An AndroidX library and the modern replacement for:
- `IntentService`
- `JobScheduler`

## How it works
You define a `Worker` (background task).

`WorkManager` ensures it runs while respecting constraints such as:
- WiFi
- Charging
- Battery state

## Threading
Runs on a background thread automatically.

## Persistence
Guaranteed to run even if:
- App is killed
- Device restarts  
(if configured properly)

## Best For
Deferrable and guaranteed background tasks.

Examples:
- Upload logs
- Sync data
- Retry failed network calls
- Backup tasks

---

## ✅ Pros

- Backward compatible (`API 14+`)
- Guaranteed execution
- Supports constraints:
  - Network
  - Charging
  - Battery state
- Supports chained tasks:
  - A → B → C
- Internally uses:
  - `JobScheduler`
  - `AlarmManager`
  - `FirebaseJobDispatcher`

---

## ❌ Cons

- Not suitable for real-time tasks:
  - Music playback
  - Live location updates
- Slightly heavier setup than `IntentService`

---

# Comparison Table

| Feature | IntentService | JobScheduler | WorkManager |
|---|---|---|---|
| Threading | Background thread automatically | Main thread (manual threading needed) | Background thread automatically |
| Task Type | One-off short tasks | Periodic / scheduled tasks | One-off + periodic tasks |
| Persistence | Dies if app is killed | Survives restart (if configured) | Guaranteed execution + survives reboot |
| Constraints | ❌ None | ✅ WiFi, charging, idle | ✅ Network, charging, battery, etc. |
| API Level | Any (deprecated) | 21+ | 14+ |
| Stops Automatically | ✅ Yes | ❌ System controlled | ✅ Yes |
| Status | Deprecated | Low-level API | ✅ Recommended by Google |

---

# When to Use What?

## IntentService
👉 Legacy apps only.  
Not recommended for modern Android development.

---

## JobScheduler
👉 Use when:
- You need direct system-level scheduling
- App supports only API 21+
- You do not want AndroidX dependency

(Rare in modern apps)

---

## WorkManager
👉 ✅ Recommended modern solution.

Use for:
- Syncing
- Uploading
- Logging
- Retrying failed work
- Any guaranteed background task

---

# Rule of Thumb

## Foreground Service
👉 Use for:
- Real-time work
- Long-running operations

Examples:
- Music player
- Navigation
- Fitness tracking
- Active calls

---

## WorkManager
👉 Use for:
- Deferrable work
- Guaranteed execution

Examples:
- Sync data
- Upload logs
- Backup files
- Retry network requests

---

## JobScheduler
👉 Mostly used in:
- Older apps
- Low-level system scheduling use cases


# What does system-level scheduling mean?

It means:  
👉 You give the job request (what to do + conditions like Wi-Fi, charging, idle, etc.) to the Android OS, and then the system itself decides when and how to run it.

So instead of your app trying to run background tasks whenever it wants (which can waste battery and violate Doze mode restrictions), the Android system batches, delays, and optimizes jobs from all apps together.

---

# Example

Suppose your app needs to sync contacts with server:

## With IntentService
You’d just start it right away, even if the phone has:
- 1% battery
- no internet
- device is in Doze mode

→ drains battery, may get killed.

---

## With JobScheduler
You’d schedule:
- Run only when charging
- Run only on Wi-Fi
- Run after 2 hours
- Repeat every 12 hours

Then the OS says:

> “Okay, Gmail also has a sync job and WhatsApp has a backup job. I’ll batch all three at 2 AM when the phone is charging + idle + on Wi-Fi.”

This way, battery and performance are optimized.

---

# Why is this called "system-level"?

Because the decision is not in your app’s hands.

You only:
- request the work
- set constraints

The Android system (`JobScheduler` service) actually controls:
- When the job runs
- How often it runs
- Whether it should be postponed (due to Doze / battery saver)

Your app doesn’t run continuously in background → the system wakes it only when needed.

---

# Visual Analogy

## IntentService
👉 You wake up whenever you want → no control, may disturb battery.

---

## JobScheduler
👉 You give your tasks to a system “task manager” → it wakes you only when the right time comes (efficient scheduling).

---

## WorkManager
👉 Google’s modern wrapper → it uses `JobScheduler` internally on API 23+, but falls back to:
- `AlarmManager`
- `FirebaseJobDispatcher`

on older devices.

---

# Final Understanding

✅ System-level scheduling = Android OS takes responsibility for when + how jobs run, instead of your app forcing it.



# JobScheduler

Introduced: API 21 (Lollipop).

What it does: Lets you schedule background jobs with constraints (charging, Wi-Fi, idle, etc.).

## How it works

- You create a `JobInfo` with constraints.
- Submit it to the system via `JobScheduler`.
- The system decides when to run it, batching jobs across apps.

## Threading

Runs on the main thread → you must manually start a worker thread.

## Persistence

Jobs can persist across reboots if configured with `setPersisted(true)`.

## Limitations

Only available on API 21+ and above.

---

## ✅ Pros

- Battery optimized (system batches jobs).
- Supports constraints (Wi-Fi, charging, idle).
- Can survive reboots.

---

## ❌ Cons

- API 21+ only (no support for KitKat or below).
- No built-in retries, chaining, or guaranteed execution if app is force-stopped.
- Threading must be managed manually.

---

# WorkManager

Introduced: AndroidX (Jetpack), modern replacement for `IntentService` & `JobScheduler`.

What it does: Runs deferrable + guaranteed background work.

## How it works

- You define a `Worker` with your task.
- Submit it to `WorkManager`.
- `WorkManager` ensures execution, choosing the best backend:
  - `JobScheduler` (API 23+)
  - `AlarmManager` or `FirebaseJobDispatcher` (older devices)

## Threading

Runs on a background thread automatically.

## Persistence

Guaranteed execution, even after app restart or device reboot.

## Extras

Supports:
- chaining
- retries
- unique work policies
- constraints

---

## ✅ Pros

- Backward compatible (API 14+).
- Guaranteed execution, even if app restarts.
- Automatic background thread (no manual thread handling).
- Supports constraints (Wi-Fi, charging, idle).
- Can chain tasks (A → B → C).
- Built-in retries with exponential backoff.

---

## ❌ Cons

- Heavier API than `JobScheduler` (slightly more setup).
- Not for real-time tasks (slight delay since system batches jobs).

---

# Comparison Table

| Feature | JobScheduler | WorkManager |
|---|---|---|
| API Support | 21+ only (Lollipop+) | 14+ (backward compatible via AndroidX) |
| Threading | Runs on main thread (need manual threading) | Runs on background thread automatically |
| Persistence | Can survive reboots (`setPersisted`) | Guaranteed execution, survives reboots |
| Constraints | Yes (network, charging, idle) | Yes (same + more flexible) |
| Retries | No built-in retry | Built-in retries with backoff policy |
| Chaining | ❌ Not supported | ✅ Supported (A → B → C) |
| Guarantee | May not run if app is killed | Guaranteed to run (system ensures) |
| Best For | Periodic jobs on modern devices | Deferrable + guaranteed background work (all devices) |

---

# Quick Example

## JobScheduler

```kotlin
val component = ComponentName(this, MyJobService::class.java)

val jobInfo = JobInfo.Builder(123, component)
    .setRequiredNetworkType(JobInfo.NETWORK_TYPE_UNMETERED) // WiFi only
    .setRequiresCharging(true)
    .build()

val scheduler = getSystemService(Context.JOB_SCHEDULER_SERVICE) as JobScheduler

scheduler.schedule(jobInfo)
```

## WorkManager
```
val workRequest = OneTimeWorkRequestBuilder<MyWorker>()
    .setConstraints(
        Constraints.Builder()
            .setRequiredNetworkType(NetworkType.UNMETERED) // WiFi only
            .setRequiresCharging(true)
            .build()
    )
    .build()

WorkManager.getInstance(context).enqueue(workRequest)

```

## Rule of Thumb

JobScheduler → System job management for API 21+ only, lightweight but limited.

WorkManager → ✅ Recommended for all background tasks that must run reliably, works on all devices, modern, flexible.

