

## 🔹 **What is WorkManager?**

* Part of **Android Jetpack (AndroidX)**.
* A library for scheduling **deferrable, guaranteed background work**.
* **Deferrable** = not time-critical (e.g., “sync logs sometime today”).
* **Guaranteed** = system will make sure it runs, even if:

  * The app is killed
  * The device restarts
  * Battery saver / Doze mode kicks in



## 🔹 **Why WorkManager?**

* **IntentService** = simple background tasks, but dies if app is killed → ❌ not reliable.
* **JobScheduler** = system-level jobs, but only on API 21+ → ❌ not backward compatible.
* **AlarmManager** = schedules alarms, but doesn’t care about constraints (like Wi-Fi, charging) → ❌ not optimized.

👉 WorkManager = **One API to rule them all** ✅

* On API 23+ → uses **JobScheduler** internally.
* On older devices → falls back to **AlarmManager** or **Firebase JobDispatcher**.
* For you → same API everywhere.



## 🔹 **Core Concepts**

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



## 🔹 **Code Example**

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



## 🔹 **WorkManager Features**

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



## 🔹 **When to Use WorkManager**

* Uploading logs, crash reports, analytics.
* Syncing local DB with server.
* Sending deferred push tokens.
* Scheduling periodic jobs (daily/weekly).
* Background file uploads/downloads (non real-time).



## 🔹 **When NOT to Use WorkManager**

* **Real-time tasks** → Use ForegroundService (e.g., music, navigation).
* **Very short tasks** → Use coroutines/threads inside Activity/Service.
* **Exact timing** → Use AlarmManager (e.g., reminder at 9:00 AM sharp).



## 🔹 **Why WorkManager > JobScheduler**

* Backward compatible (API 14+).
* Auto-threading (JobScheduler runs on main thread).
* Built-in retries, chaining, constraints.
* Guaranteed execution across reboots.
* Google officially recommends it in modern apps.



✅ **In short:**
WorkManager is the **modern, reliable, and recommended** way to run deferrable + guaranteed background tasks in Android.

Ah! You probably mean **“Expedited Work”** in **WorkManager**. Let me explain clearly.

---



# 🔹 **What is Expedited Work in WorkManager?**

Normally, **WorkManager schedules work as deferrable**, meaning it may **wait until constraints are met** or the system decides it’s a good time (battery-friendly, Doze mode, etc.).

**Expedited work** is different:

* It tells WorkManager: **“Run this work immediately, even if the device is idle.”**
* Intended for **high-priority, time-sensitive tasks**.
* Still **runs in the background** but **as soon as possible**, without waiting for normal batching.



## 🔹 **Key Points**

1. **Requires API 23+** if constraints need to bypass Doze mode.
2. **Still respects battery optimizations**, but tries to run immediately.
3. You can use **OneTimeWorkRequest** with expedited flag.
4. Ideal for:

   * Sending crash reports immediately
   * Uploading urgent analytics
   * Triggering high-priority notifications



## 🔹 **Code Example**

```kotlin
val workRequest = OneTimeWorkRequestBuilder<UploadLogsWorker>()
    .setExpedited(OutOfQuotaPolicy.RUN_AS_NON_EXPEDITED_WORK) 
    .build()

WorkManager.getInstance(context).enqueue(workRequest)
```

* `setExpedited()` → Marks the work as **high-priority**.
* `OutOfQuotaPolicy` → If system cannot run expedited work (battery/limits):

  * `RUN_AS_NON_EXPEDITED_WORK` → Run normally (still guaranteed).
  * `DROP_WORK` → Cancel the work.



## 🔹 **Difference from Normal Work**

| Feature               | Normal Work                     | Expedited Work                            |
| --------------------- | ------------------------------- | ----------------------------------------- |
| **Execution timing**  | Deferrable, system decides      | Immediate, high-priority                  |
| **Constraints**       | Waits until constraints are met | Can run ASAP, may ignore some constraints |
| **Use case**          | Sync logs, periodic work        | Urgent uploads, crash reports             |
| **Battery optimized** | Yes                             | Less optimized (priority)                 |



✅ **In short:**
**Expedited Work** = WorkManager’s way of saying:

> “This task is urgent — please run it immediately, bypassing normal deferral, but still safely.”

Let’s build a **complete, practical WorkManager example** that includes:

1. **Normal One-Time Work**
2. **Expedited Work**
3. **Constraints** (Wi-Fi + Charging)
4. **LiveData observation** for UI
5. **Chaining**
6. **Periodic work**


## **Step 1: Setup Project & Dependencies**

1. Open Android Studio → create a new project.
2. Add WorkManager dependency in `build.gradle`:

```gradle
dependencies {
    def work_version = "2.9.2"
    implementation "androidx.work:work-runtime-ktx:$work_version"
}
```

3. Sync the project. ✅



## **Step 2: Create a Simple One-Time Worker**

**Goal:** Run a background task once, no constraints yet.

```kotlin
class SimpleWorker(appContext: Context, workerParams: WorkerParameters) 
    : Worker(appContext, workerParams) {

    override fun doWork(): Result {
        Log.d("WorkManager", "SimpleWorker is running!")
        Thread.sleep(1000) // simulate work
        return Result.success()
    }
}
```



### Step 2a: Enqueue the worker

```kotlin
val simpleWorkRequest = OneTimeWorkRequestBuilder<SimpleWorker>().build()

WorkManager.getInstance(context).enqueue(simpleWorkRequest)
```

**Practice Task:**

* Run this and check logcat → “SimpleWorker is running!” should appear.
* Try changing the `Thread.sleep()` to simulate longer work.



## **Step 3: Add Input & Output Data**

**Goal:** Send some data to the worker and return a result.

```kotlin
// Worker
class DataWorker(appContext: Context, workerParams: WorkerParameters) 
    : Worker(appContext, workerParams) {

    override fun doWork(): Result {
        val input = inputData.getString("input") ?: "No input"
        Log.d("WorkManager", "Received input: $input")
        val output = workDataOf("output" to "Processed: $input")
        return Result.success(output)
    }
}
```

```kotlin
// Enqueue
val inputData = workDataOf("input" to "Hello WorkManager")
val dataWorkRequest = OneTimeWorkRequestBuilder<DataWorker>()
    .setInputData(inputData)
    .build()

WorkManager.getInstance(context).enqueue(dataWorkRequest)
```

**Practice Task:**

* Observe the logs → “Received input: Hello WorkManager”.
* Later, observe **outputData** after completion.

---

## **Step 4: Add Constraints**

**Goal:** Run only when device is charging and on Wi-Fi.

```kotlin
val constraints = Constraints.Builder()
    .setRequiredNetworkType(NetworkType.UNMETERED) // Wi-Fi
    .setRequiresCharging(true)
    .build()

val constrainedWork = OneTimeWorkRequestBuilder<DataWorker>()
    .setConstraints(constraints)
    .setInputData(workDataOf("input" to "Constrained Work"))
    .build()

WorkManager.getInstance(context).enqueue(constrainedWork)
```

**Practice Task:**

* Run this on device/emulator without Wi-Fi or charging → work should not run.
* Enable Wi-Fi + charging → work executes.

---

## ✅ **Step 5: Observe Worker Status (LiveData)**

```kotlin
WorkManager.getInstance(context)
    .getWorkInfoByIdLiveData(constrainedWork.id)
    .observe(this) { info ->
        info?.let {
            Log.d("WorkManager", "State: ${info.state}")
            if (info.state.isFinished) {
                Log.d("WorkManager", "Output: ${info.outputData.getString("output")}")
            }
        }
    }
```

 Let’s create a **full Android Kotlin example** for **Step 1–5**:


## **Full Example: WorkManagerPractice**

### 1️⃣ **MainActivity.kt**

```kotlin
package com.example.workmanagerpractice

import android.os.Bundle
import android.util.Log
import androidx.appcompat.app.AppCompatActivity
import androidx.lifecycle.Observer
import androidx.work.*

class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // Step 1: Define constraints
        val constraints = Constraints.Builder()
            .setRequiredNetworkType(NetworkType.UNMETERED) // Wi-Fi
            .setRequiresCharging(true)                     // Charging
            .build()

        // Step 2: Input data
        val inputData = workDataOf("input" to "Hello WorkManager")

        // Step 3: Create OneTimeWorkRequest
        val constrainedWork = OneTimeWorkRequestBuilder<DataWorker>()
            .setConstraints(constraints)
            .setInputData(inputData)
            .build()

        // Step 4: Enqueue work
        WorkManager.getInstance(this).enqueue(constrainedWork)

        // Step 5: Observe status
        WorkManager.getInstance(this)
            .getWorkInfoByIdLiveData(constrainedWork.id)
            .observe(this, Observer { info ->
                info?.let {
                    Log.d("WorkManager", "State: ${info.state}")
                    if (info.state.isFinished) {
                        Log.d("WorkManager", "Output: ${info.outputData.getString("output")}")
                    }
                }
            })
    }
}
```



### 2️⃣ **DataWorker.kt**

```kotlin
package com.example.workmanagerpractice

import android.content.Context
import android.util.Log
import androidx.work.Worker
import androidx.work.WorkerParameters
import androidx.work.workDataOf

class DataWorker(appContext: Context, workerParams: WorkerParameters)
    : Worker(appContext, workerParams) {

    override fun doWork(): Result {
        return try {
            // Step 1: Get input
            val input = inputData.getString("input") ?: "No input"
            Log.d("WorkManager", "Received input: $input")

            // Step 2: Simulate work
            Thread.sleep(2000)

            // Step 3: Return output
            val output = workDataOf("output" to "Processed: $input")
            Log.d("WorkManager", "Work completed: $output")
            Result.success(output)
        } catch (e: Exception) {
            Log.e("WorkManager", "Work failed", e)
            Result.retry()
        }
    }
}
```



### 3️⃣ **AndroidManifest.xml**

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.workmanagerpractice">

    <application
        android:allowBackup="true"
        android:label="WorkManagerPractice"
        android:theme="@style/Theme.AppCompat.Light.NoActionBar">
        <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN"/>
                <category android:name="android.intent.category.LAUNCHER"/>
            </intent-filter>
        </activity>
    </application>

</manifest>
```



### ✅ **How to Test**

1. Run the app on device/emulator.
2. Make sure **Wi-Fi + charging** conditions are met.
3. Open **Logcat** → filter `WorkManager` → you should see:

```
Received input: Hello WorkManager
Work completed: {output=Processed: Hello WorkManager}
State: ENQUEUED → RUNNING → SUCCEEDED
```

4. Change constraints to see how work waits until conditions are met.


Perfect! Let’s extend your WorkManager practice to **Step 6: Periodic Work**.

Periodic work is useful for tasks that need to **run repeatedly**, like syncing logs, refreshing data, or uploading analytics.

---

# **Step 6: Periodic Work**

### 🔹 **Goal**

* Run a worker **every 15 minutes** (minimum interval allowed by WorkManager).
* Respect constraints (Wi-Fi + Charging).
* Observe status in UI/logs.



### 1️⃣ **Create a Worker**

We can reuse `DataWorker.kt` from the previous example. Optionally, log something different to distinguish periodic runs.

```kotlin
class PeriodicWorker(appContext: Context, workerParams: WorkerParameters)
    : Worker(appContext, workerParams) {

    override fun doWork(): Result {
        return try {
            Log.d("WorkManager", "Periodic work running!")
            Thread.sleep(1000) // simulate work
            Result.success()
        } catch (e: Exception) {
            Result.retry()
        }
    }
}
```



### 2️⃣ **Create Periodic WorkRequest**

```kotlin
import java.util.concurrent.TimeUnit

val periodicWorkRequest = PeriodicWorkRequestBuilder<PeriodicWorker>(15, TimeUnit.MINUTES)
    .setConstraints(
        Constraints.Builder()
            .setRequiredNetworkType(NetworkType.UNMETERED)
            .setRequiresCharging(true)
            .build()
    )
    .build()
```

* `15 minutes` → Minimum allowed interval.
* Constraints ensure it only runs on **Wi-Fi + Charging**.



### 3️⃣ **Enqueue Periodic Work**

```kotlin
WorkManager.getInstance(context).enqueueUniquePeriodicWork(
    "periodicLogUpload",                    // Unique name
    ExistingPeriodicWorkPolicy.KEEP,       // KEEP = don’t replace existing work
    periodicWorkRequest
)
```

* Using `enqueueUniquePeriodicWork` ensures **no duplicate periodic work**.
* If already scheduled, it **won’t enqueue again** (with `KEEP`).



### 4️⃣ **Observe Periodic Work Status**

```kotlin
WorkManager.getInstance(context)
    .getWorkInfoByIdLiveData(periodicWorkRequest.id)
    .observe(this) { info ->
        info?.let {
            Log.d("WorkManager", "Periodic Work State: ${info.state}")
        }
    }
```

* You can see `ENQUEUED → RUNNING → SUCCEEDED` repeatedly in logs.



### ✅ **Practice Tasks**

1. Run the app → verify **periodic logs appear every 15 minutes**.
2. Toggle **Wi-Fi / Charging** → see how constraints delay execution.
3. Change interval to **1 day** → see how scheduling behaves.
4. Try **ExistingPeriodicWorkPolicy.REPLACE** → replace old periodic work with new one.

---


# **Step 7: Chaining Work**

### 🔹 **Goal**

* Run multiple workers **one after another**.
* Each worker executes **only after the previous one succeeds**.
* Observe state for each worker.



### 1️⃣ **Create Workers**

We already have `DataWorker` for uploading. Let’s create a **CleanLogsWorker**.

```kotlin
class CleanLogsWorker(appContext: Context, workerParams: WorkerParameters)
    : Worker(appContext, workerParams) {

    override fun doWork(): Result {
        return try {
            Log.d("WorkManager", "Cleaning local logs...")
            Thread.sleep(1000) // simulate cleaning
            Result.success()
        } catch (e: Exception) {
            Result.retry()
        }
    }
}
```

Optionally, you can add **NotifyServerWorker**:

```kotlin
class NotifyServerWorker(appContext: Context, workerParams: WorkerParameters)
    : Worker(appContext, workerParams) {

    override fun doWork(): Result {
        return try {
            Log.d("WorkManager", "Notifying server...")
            Thread.sleep(1000)
            Result.success()
        } catch (e: Exception) {
            Result.retry()
        }
    }
}
```



### 2️⃣ **Create WorkRequests**

```kotlin
val uploadWork = OneTimeWorkRequestBuilder<DataWorker>()
    .setInputData(workDataOf("input" to "Upload logs for chain"))
    .build()

val cleanWork = OneTimeWorkRequestBuilder<CleanLogsWorker>().build()
val notifyWork = OneTimeWorkRequestBuilder<NotifyServerWorker>().build()
```



### 3️⃣ **Chain the Workers**

```kotlin
WorkManager.getInstance(context)
    .beginWith(uploadWork)   // Step 1: Upload
    .then(cleanWork)         // Step 2: Clean logs
    .then(notifyWork)        // Step 3: Notify server
    .enqueue()
```

* Each worker **starts only after the previous one succeeds**.
* If any worker fails and retries, the chain waits.



### 4️⃣ **Observe Chained Work**

```kotlin
WorkManager.getInstance(context).getWorkInfoByIdLiveData(uploadWork.id)
    .observe(this) { info ->
        info?.let { Log.d("WorkManager", "UploadWorker: ${info.state}") }
    }

WorkManager.getInstance(context).getWorkInfoByIdLiveData(cleanWork.id)
    .observe(this) { info ->
        info?.let { Log.d("WorkManager", "CleanWorker: ${info.state}") }
    }

WorkManager.getInstance(context).getWorkInfoByIdLiveData(notifyWork.id)
    .observe(this) { info ->
        info?.let { Log.d("WorkManager", "NotifyWorker: ${info.state}") }
    }
```



### ✅ **Practice Tasks**

1. Run the chain → check logs → should see:

```
Upload logs for chain
Cleaning local logs...
Notifying server...
```

in order.

2. Force **DataWorker to fail** → see how WorkManager **retries** before continuing.

3. Add **constraints** to one of the workers → check how chain **waits until constraints are satisfied**.

Great timing 😎 — let’s now dive into **CoroutineWorker**.

So far, we used the regular `Worker` (which runs on a background thread automatically). But sometimes we need to use **suspend functions** (e.g., network calls with Retrofit, Room DB queries, Firebase, etc.). For that, WorkManager gives us **CoroutineWorker**.

---

# **Step 8: CoroutineWorker**

### 🔹 Why CoroutineWorker?

* `Worker` → blocking work (uses `doWork(): Result`)
* `CoroutineWorker` → suspending work (`doWork(): suspend Result`)
* Lets you call `suspend` functions directly without blocking threads.



### 1️⃣ **Create CoroutineWorker**

```kotlin
class MyCoroutineWorker(
    appContext: Context,
    workerParams: WorkerParameters
) : CoroutineWorker(appContext, workerParams) {

    override suspend fun doWork(): Result {
        return try {
            // Simulate API call
            delay(2000) 
            val input = inputData.getString("input") ?: "No input"
            Log.d("WorkManager", "CoroutineWorker received: $input")

            val output = workDataOf("output" to "Processed by CoroutineWorker")
            Result.success(output)
        } catch (e: Exception) {
            Log.e("WorkManager", "Error in CoroutineWorker", e)
            Result.retry()
        }
    }
}
```



### 2️⃣ **Enqueue the Work**

```kotlin
val inputData = workDataOf("input" to "Hello from CoroutineWorker")

val coroutineWorkRequest = OneTimeWorkRequestBuilder<MyCoroutineWorker>()
    .setInputData(inputData)
    .build()

WorkManager.getInstance(this).enqueue(coroutineWorkRequest)
```



### 3️⃣ **Observe Output**

```kotlin
WorkManager.getInstance(this)
    .getWorkInfoByIdLiveData(coroutineWorkRequest.id)
    .observe(this) { info ->
        info?.let {
            Log.d("WorkManager", "CoroutineWorker state: ${info.state}")
            if (info.state.isFinished) {
                Log.d("WorkManager", "CoroutineWorker output: ${info.outputData.getString("output")}")
            }
        }
    }
```



### ✅ **Practice Tasks**

1. Replace `delay(2000)` with a **real suspend call** (e.g., Retrofit API, Room query).
2. Test retry logic by throwing an exception.
3. Chain a `CoroutineWorker` with a normal `Worker` → see how both run together.










