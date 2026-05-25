

# 🔹 **What is WorkManager?**

* Part of **Android Jetpack (AndroidX)**.
* A library for scheduling **deferrable, guaranteed background work**.
* **Deferrable** = not time-critical (e.g., “sync logs sometime today”).
* **Guaranteed** = system will make sure it runs, even if:

  * The app is killed
  * The device restarts
  * Battery saver / Doze mode kicks in



# 🔹 **Why WorkManager?**

* **IntentService** = simple background tasks, but dies if app is killed → ❌ not reliable.
* **JobScheduler** = system-level jobs, but only on API 21+ → ❌ not backward compatible.
* **AlarmManager** = schedules alarms, but doesn’t care about constraints (like Wi-Fi, charging) → ❌ not optimized.

👉 WorkManager = **One API to rule them all** ✅

* On API 23+ → uses **JobScheduler** internally.
* On older devices → falls back to **AlarmManager** or **Firebase JobDispatcher**.
* For you → same API everywhere.

---

# 🔹 **Core Concepts**

1. **Worker** → The unit of work you define.

   * Extend `Worker` or `CoroutineWorker`.
   * Override `doWork()` → background code runs here.

2. **WorkRequest** → Represents a request to run work.

   * `OneTimeWorkRequest` → Run once.
   * `PeriodicWorkRequest` → Run repeatedly (e.g., every 24 hrs).

3. **Constraints** → Define when work should run.

   * Network type (Wi-Fi, any, etc.).
   * Device charging state.
   * Device idle state.
   * Battery level not low.

4. **WorkManager** → The orchestrator.

   * Queues, schedules, runs, and retries work.
   * Persists jobs in an **SQLite database** internally.

---

# 🔹 **Code Example**

### 1. Define a Worker

```kotlin
class UploadLogsWorker(appContext: Context, params: WorkerParameters)
    : Worker(appContext, params) {

    override fun doWork(): Result {
        return try {
            // Simulate background task
            val logs = inputData.getString("logs") ?: "No logs"
            Log.d("WorkManager", "Uploading logs: $logs")

            // ✅ Return success
            Result.success()
        } catch (e: Exception) {
            // ❌ Retry automatically with backoff
            Result.retry()
        }
    }
}
```

---

### 2. Enqueue Work

```kotlin
val input = workDataOf("logs" to "User activity logs")

val request = OneTimeWorkRequestBuilder<UploadLogsWorker>()
    .setConstraints(
        Constraints.Builder()
            .setRequiredNetworkType(NetworkType.UNMETERED) // Wi-Fi only
            .setRequiresCharging(true)                     // Only when charging
            .build()
    )
    .setInputData(input)
    .build()

WorkManager.getInstance(context).enqueue(request)
```

---

# 🔹 **WorkManager Features**

### ✅ Constraints

* Network (Wi-Fi, any, metered).
* Device charging.
* Device idle.
* Battery not low.
  👉 Work runs **only when constraints are satisfied**.

### ✅ Guaranteed Execution

* Tasks survive app restarts & device reboots.
* Stored in **internal SQLite DB** until completion.

### ✅ Chaining

```kotlin
val workA = OneTimeWorkRequestBuilder<WorkerA>().build()
val workB = OneTimeWorkRequestBuilder<WorkerB>().build()
val workC = OneTimeWorkRequestBuilder<WorkerC>().build()

WorkManager.getInstance(context)
    .beginWith(workA)  // A
    .then(workB)       // → B
    .then(workC)       // → C
    .enqueue()
```

### ✅ Periodic Work

```kotlin
val periodicRequest =
    PeriodicWorkRequestBuilder<SyncWorker>(24, TimeUnit.HOURS)
        .build()

WorkManager.getInstance(context).enqueue(periodicRequest)
```

### ✅ Retry & Backoff

```kotlin
val request = OneTimeWorkRequestBuilder<MyWorker>()
    .setBackoffCriteria(
        BackoffPolicy.EXPONENTIAL,
        10, TimeUnit.SECONDS
    )
    .build()
```

### ✅ Unique Work

* Prevent duplicate work.

```kotlin
WorkManager.getInstance(context).enqueueUniqueWork(
    "uploadLogs",
    ExistingWorkPolicy.REPLACE,  // or KEEP, APPEND
    request
)
```

### ✅ Observing Work

```kotlin
WorkManager.getInstance(context).getWorkInfoByIdLiveData(request.id)
    .observe(this) { info ->
        if (info != null && info.state == WorkInfo.State.SUCCEEDED) {
            Log.d("WorkManager", "Work finished successfully")
        }
    }
```

---

# 🔹 **When to Use WorkManager**

* Uploading logs, crash reports, analytics.
* Syncing local DB with server.
* Sending deferred push tokens.
* Scheduling periodic jobs (daily/weekly).
* Background file uploads/downloads (non real-time).

---

# 🔹 **When NOT to Use WorkManager**

* **Real-time tasks** → Use ForegroundService (e.g., music, navigation).
* **Very short tasks** → Use coroutines/threads inside Activity/Service.
* **Exact timing** → Use AlarmManager (e.g., reminder at 9:00 AM sharp).

---

# 🔹 **Why WorkManager > JobScheduler**

* Backward compatible (API 14+).
* Auto-threading (JobScheduler runs on main thread).
* Built-in retries, chaining, constraints.
* Guaranteed execution across reboots.
* Google officially recommends it in modern apps.

---

✅ **In short:**
WorkManager is the **modern, reliable, and recommended** way to run deferrable + guaranteed background tasks in Android.

---

👉 Do you want me to also show you how **WorkManager internally chooses between JobScheduler, AlarmManager, and FirebaseJobDispatcher** depending on API level?
