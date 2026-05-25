

## 🔹 **What is AlarmManager?**

* A **system service** that lets you schedule your app to run **at a specific time in the future**, even if your app is not running.
* Think of it as an “alarm clock” ⏰ for your app → wakes up your app at the requested time.



## 🔹 **When to use AlarmManager?**

* When you need **time-accurate execution** (e.g., 7:00 AM reminder).
* When the task **must run at an exact clock time**, regardless of battery optimization.
* Use cases:

  * Calendar reminders
  * Medicine/alarm notifications
  * Daily 9:00 AM push
  * Scheduled IoT communication

👉 If **timing is flexible** (e.g., “sync logs sometime today”), use **WorkManager** instead.



## 🔹 **Types of Alarms**

AlarmManager gives flexibility depending on **when** and **how accurately** you need the task.

1. **RTC (Real Time Clock)**

   * Based on **wall-clock time** (system time).
   * Example: Trigger at **9:00 AM real-world time**.
   * If the device sleeps, won’t wake up unless you use `RTC_WAKEUP`.

   Variants:

   * `RTC_WAKEUP` → Wake device if asleep.
   * `RTC` → Don’t wake device, trigger only if awake.



2. **ELAPSED_REALTIME**

   * Based on **time since boot** (ignores clock changes).
   * Example: Trigger **30 mins after device boot**.
   * Used for **relative delays**.

   Variants:

   * `ELAPSED_REALTIME_WAKEUP` → Wakes device.
   * `ELAPSED_REALTIME` → Doesn’t wake device.



3. **Exact vs Inexact**

* `setExact()` → Runs at the **exact millisecond**.
* `setInexactRepeating()` → System may adjust/batch the timing for efficiency.
* Since API 19 (KitKat), **all repeating alarms became inexact** unless you use `setExactAndAllowWhileIdle()`.



4. **Doze Mode & Idle Restrictions**

* From API 23+ (Marshmallow), Doze mode limits alarms.
* For critical alarms (like alarms, medication reminders):

  * Use `setExactAndAllowWhileIdle()`



## 🔹 **Code Examples**

### 1. One-time exact alarm at 9:00 AM

```kotlin
val alarmManager = getSystemService(Context.ALARM_SERVICE) as AlarmManager

val intent = Intent(this, AlarmReceiver::class.java)
val pendingIntent = PendingIntent.getBroadcast(this, 0, intent, PendingIntent.FLAG_IMMUTABLE)

val calendar = Calendar.getInstance().apply {
    set(Calendar.HOUR_OF_DAY, 9)
    set(Calendar.MINUTE, 0)
    set(Calendar.SECOND, 0)
}

alarmManager.setExact(
    AlarmManager.RTC_WAKEUP,
    calendar.timeInMillis,
    pendingIntent
)
```



### 2. Repeating alarm (every day at 7:00 AM)

```kotlin
alarmManager.setRepeating(
    AlarmManager.RTC_WAKEUP,
    calendar.timeInMillis,
    AlarmManager.INTERVAL_DAY,
    pendingIntent
)
```



### 3. Allow during Doze mode

```kotlin
alarmManager.setExactAndAllowWhileIdle(
    AlarmManager.RTC_WAKEUP,
    calendar.timeInMillis,
    pendingIntent
)
```



### 4. Cancel alarm

```kotlin
alarmManager.cancel(pendingIntent)
```



## 🔹 **Alarm Delivery**

* **AlarmManager itself does NOT execute work**.
* It **delivers a PendingIntent** → usually a **BroadcastReceiver**.
* That receiver then starts a `Service` or shows a `Notification`.

Example receiver:

```kotlin
class AlarmReceiver : BroadcastReceiver() {
    override fun onReceive(context: Context, intent: Intent) {
        Log.d("Alarm", "Alarm triggered!")
        // Start a service, show notification, etc.
    }
}
```



## 🔹 **AlarmManager vs Others**

* **AlarmManager** = Best for **time-precise alarms/reminders**.
* **JobScheduler** = Best for **deferrable jobs with constraints** (Wi-Fi, charging).
* **WorkManager** = Best for **guaranteed, persistent background work** (sync, uploads).

👉 Rule of thumb:

* **Exact time** (alarm at 7:00 AM) → AlarmManager.
* **Best-effort / deferred** → WorkManager.
* **System-level batching** (API 21+) → JobScheduler.



## 🔹 Summary

✅ AlarmManager = clock-based scheduling API.
✅ Supports **exact & repeating alarms**.
✅ Works even if app is not running.
✅ Needs **PendingIntent + BroadcastReceiver**.
✅ Use `setExactAndAllowWhileIdle()` for critical alarms in Doze mode.
❌ Not good for flexible or battery-optimized work → that’s WorkManager’s job.


## 🔹 Does AlarmManager survive device shutdown?

❌ **No.**

* If the **device is switched off**, all scheduled alarms are lost.
* AlarmManager does not persist across device reboots.



## 🔹 What happens after reboot?

* When the device powers back on, **your alarms are gone** unless you **reschedule them**.
* That’s why Android gives you the **`BOOT_COMPLETED` broadcast**.
* You can listen for it and **reschedule alarms** when the device restarts.

Example:

```kotlin
class BootReceiver : BroadcastReceiver() {
    override fun onReceive(context: Context, intent: Intent) {
        if (intent.action == Intent.ACTION_BOOT_COMPLETED) {
            // Re-schedule your alarms here
        }
    }
}
```

And register in `AndroidManifest.xml`:

```xml
<receiver android:name=".BootReceiver"
          android:enabled="true"
          android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.BOOT_COMPLETED"/>
    </intent-filter>
</receiver>

<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED"/>
```



## 🔹 How to make alarms "survive" reboot?

* By default → they don’t.
* To make them survive → you must handle `BOOT_COMPLETED` and **recreate them manually**.
* Alternative: If you need guaranteed persistence across reboots without extra effort → **WorkManager** is better, because it automatically persists jobs in its internal DB.



✅ So summary:

* If device is off → AlarmManager alarms won’t trigger.
* After reboot → You must re-register alarms yourself using `BOOT_COMPLETED`.
