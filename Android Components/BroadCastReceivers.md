### 📡 Broadcast Receivers in Android

**Broadcast Receivers** are one of the core Android components used to **listen for system-wide or app-wide events** (called *broadcasts*) and respond to them.

---

## 🧠 What is a Broadcast?

A **broadcast** is a message sent by:

* The **Android system** (e.g., battery low, screen on/off)
* **Other apps**
* **Your own app**

---

## 📥 What is a Broadcast Receiver?

A **BroadcastReceiver** is a component that:

* Waits for specific broadcasts
* Executes code when the broadcast is received

---

## 🧩 Basic Syntax

```kotlin
class MyReceiver : BroadcastReceiver() {
    override fun onReceive(context: Context?, intent: Intent?) {
        // Handle broadcast here
        Log.d("MyReceiver", "Broadcast Received")
    }
}
```

---

## 🔔 Types of Broadcasts

### 1. **System Broadcasts**

Examples:

* `android.intent.action.BATTERY_LOW`
* `android.intent.action.AIRPLANE_MODE`
* `android.intent.action.BOOT_COMPLETED`

---

### 2. **Custom Broadcasts**

You can create your own:

```kotlin
val intent = Intent("com.example.MY_CUSTOM_ACTION")
sendBroadcast(intent)
```

---

## 📌 Registering Broadcast Receivers

### ✅ 1. Static Registration (Manifest)

```xml
<receiver android:name=".MyReceiver">
    <intent-filter>
        <action android:name="android.intent.action.BOOT_COMPLETED"/>
    </intent-filter>
</receiver>
```

👉 Used when:

* You want the receiver to work even if app is not running

---

### ✅ 2. Dynamic Registration (Code)

```kotlin
val receiver = MyReceiver()
val filter = IntentFilter(Intent.ACTION_AIRPLANE_MODE_CHANGED)

registerReceiver(receiver, filter)
```

👉 Used when:

* Receiver is needed only when app is active

---

## ⚠️ Important Notes

* `onReceive()` runs on **main thread** → keep it short
* For long tasks → use:

  * `WorkManager`
  * `Foreground Service`

---

## 🔐 Restrictions (Modern Android)

From Android 8 (Oreo):

* Many **implicit broadcasts are restricted**
* You must use:

  * **Dynamic receivers**
  * Or **explicit broadcasts**

---

## 🎯 Example Use Case

👉 Detect airplane mode change:

```kotlin
class AirplaneModeReceiver : BroadcastReceiver() {
    override fun onReceive(context: Context?, intent: Intent?) {
        val isOn = intent?.getBooleanExtra("state", false) ?: false
        Log.d("Receiver", "Airplane mode: $isOn")
    }
}
```

---

## 🧱 Real Android Use Cases

* Listening to network changes
* Receiving push notifications (via FCM internally)
* Alarm triggers
* Download complete events

---

## ⚖️ Static vs Dynamic (Quick Comparison)

| Feature               | Static (Manifest) | Dynamic (Code) |
| --------------------- | ----------------- | -------------- |
| Works when app closed | ✅ Yes             | ❌ No           |
| Lifecycle aware       | ❌ No              | ✅ Yes          |
| Android 8+ limits     | ⚠️ Restricted     | ✅ Preferred    |

---

## 💡 Pro Tips (Android Dev Interview)

* Prefer **dynamic registration**
* Avoid heavy work in `onReceive()`
* Use **explicit broadcasts for security**
* Use **LocalBroadcastManager (deprecated → now use LiveData/Flow instead)**

---

# 📡 Full Example of Broadcast Receiver in Android

Let’s build a real example:

👉 Detect when **Airplane Mode** is turned ON/OFF.

---

# 🧱 Step 1: Create BroadcastReceiver

```kotlin id="n88vfx"
package com.example.broadcastdemo

import android.content.BroadcastReceiver
import android.content.Context
import android.content.Intent
import android.widget.Toast

class AirplaneModeReceiver : BroadcastReceiver() {

    override fun onReceive(context: Context?, intent: Intent?) {

        val isAirplaneModeOn =
            intent?.getBooleanExtra("state", false) ?: false

        if (isAirplaneModeOn) {
            Toast.makeText(
                context,
                "Airplane Mode ON",
                Toast.LENGTH_SHORT
            ).show()
        } else {
            Toast.makeText(
                context,
                "Airplane Mode OFF",
                Toast.LENGTH_SHORT
            ).show()
        }
    }
}
```

---

# 🧱 Step 2: Register Receiver in Manifest

```xml id="3fowvj"
<receiver
    android:name=".AirplaneModeReceiver"
    android:exported="true">

    <intent-filter>
        <action android:name="android.intent.action.AIRPLANE_MODE" />
    </intent-filter>

</receiver>
```

---

# 🧱 Step 3: Run App

Now:

* Open app
* Turn Airplane Mode ON/OFF

✅ Receiver gets triggered automatically.

---

# 🔄 Flow of Execution

```id="fwxt6n"
System Broadcast
       ↓
BroadcastReceiver listens
       ↓
onReceive() called
       ↓
Your code executes
```

---

# 🧠 Important Concepts in This Example

## ✅ BroadcastReceiver

```kotlin id="nq10ud"
class AirplaneModeReceiver : BroadcastReceiver()
```

Receiver class that listens for broadcasts.

---

## ✅ onReceive()

```kotlin id="vjlwm3"
override fun onReceive(context: Context?, intent: Intent?)
```

Called when broadcast is received.

---

## ✅ Intent

Contains broadcast data.

```kotlin id="qdxj8m"
intent?.getBooleanExtra("state", false)
```

Gets airplane mode state.

---

## ✅ Intent Filter

```xml id="i1m95j"
<action android:name="android.intent.action.AIRPLANE_MODE" />
```

Tells Android:
👉 “Call this receiver when airplane mode changes.”

---

# 📌 Dynamic Registration Example (Modern Preferred Way)

Instead of manifest:

## MainActivity

```kotlin id="i0tghh"
class MainActivity : AppCompatActivity() {

    private lateinit var receiver: AirplaneModeReceiver

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        receiver = AirplaneModeReceiver()

        val filter = IntentFilter(Intent.ACTION_AIRPLANE_MODE_CHANGED)

        registerReceiver(receiver, filter)
    }

    override fun onDestroy() {
        super.onDestroy()

        unregisterReceiver(receiver)
    }
}
```

---

# ⚠️ Why unregisterReceiver() Important?

If not removed:

* Memory leaks
* Receiver still active after Activity destroyed

---

# 🆚 Static vs Dynamic in This Example

| Type    | Registered Where  | Works when app closed |
| ------- | ----------------- | --------------------- |
| Static  | Manifest          | ✅ Yes                 |
| Dynamic | Activity/Fragment | ❌ No                  |

---

# 🎯 Real Project Examples

Broadcast Receivers are used for:

* Internet connectivity changes
* Download completed
* Push notifications
* Battery low alerts
* SMS received
* Boot completed

---

# 🔥 Interview Questions

## ❓ Why should onReceive() be fast?

Because:

* Runs on main thread
* ANR can happen

---

## ❓ Can we do network calls inside onReceive()?

❌ Not recommended

Use:

* `WorkManager`
* `ForegroundService`

---

## ❓ Difference between BroadcastReceiver and Service?

| BroadcastReceiver       | Service                  |
| ----------------------- | ------------------------ |
| Short-lived             | Long-running             |
| Triggered by broadcasts | Performs background work |

---

## ❓ Why dynamic receivers preferred now?

Android 8+ restricts many manifest receivers for battery optimization.

---

# 📡 Custom Broadcast Receiver Example in Android

A **Custom Broadcast** means:

👉 Your app sends its own broadcast
👉 Another component in your app receives it

---

# 🧠 Real Use Case

Suppose:

* User logs in
* You want multiple screens/components to know login success

You can send a custom broadcast:

```id="rxjygm"
"com.example.LOGIN_SUCCESS"
```

---

# 🧱 Full Example

We’ll create:

1. Custom Broadcast Action
2. BroadcastReceiver
3. Send Broadcast
4. Receive Broadcast

---

# ✅ Step 1: Create Receiver

```kotlin id="qkv9gg"
package com.example.broadcastdemo

import android.content.BroadcastReceiver
import android.content.Context
import android.content.Intent
import android.widget.Toast

class LoginReceiver : BroadcastReceiver() {

    override fun onReceive(context: Context?, intent: Intent?) {

        val username = intent?.getStringExtra("username")

        Toast.makeText(
            context,
            "Login Success: $username",
            Toast.LENGTH_SHORT
        ).show()
    }
}
```

---

# ✅ Step 2: Register Receiver

## AndroidManifest.xml

```xml id="6j1fsd"
<receiver
    android:name=".LoginReceiver"
    android:exported="false">

    <intent-filter>
        <action android:name="com.example.LOGIN_SUCCESS"/>
    </intent-filter>

</receiver>
```

---

# ✅ Step 3: Send Custom Broadcast

## MainActivity.kt

```kotlin id="42ov7j"
val intent = Intent("com.example.LOGIN_SUCCESS")

intent.putExtra("username", "Reshma")

sendBroadcast(intent)
```

---

# 🔄 Full Flow

```id="j3b4vh"
MainActivity
    ↓
sendBroadcast()
    ↓
Android System
    ↓
LoginReceiver receives
    ↓
onReceive() executes
```

---

# 📦 How Matching Happens

Android checks:

```id="3vruqv"
Intent Action == Intent Filter Action
```

If both match:

```id="7w8qk5"
"com.example.LOGIN_SUCCESS"
```

then receiver is triggered.

---

# 🎯 Output

```id="8o7k2j"
Login Success: Reshma
```

Toast appears.

---

# 🔥 Dynamic Registration Version (Modern Way)

Instead of manifest:

---

## MainActivity.kt

```kotlin id="wq3m0y"
class MainActivity : AppCompatActivity() {

    private lateinit var receiver: LoginReceiver

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        receiver = LoginReceiver()

        val filter = IntentFilter("com.example.LOGIN_SUCCESS")

        registerReceiver(receiver, filter)

        // SEND BROADCAST
        val intent = Intent("com.example.LOGIN_SUCCESS")
        intent.putExtra("username", "Reshma")

        sendBroadcast(intent)
    }

    override fun onDestroy() {
        super.onDestroy()

        unregisterReceiver(receiver)
    }
}
```

---

# 🔐 Security Important

## ❌ Problem

Any app can send same broadcast:

```id="qek5f9"
"com.example.LOGIN_SUCCESS"
```

---

## ✅ Better Approach

Use:

* Explicit broadcasts
* Permissions
* Internal app communication

---

# 🚫 LocalBroadcastManager?

Previously used for app-internal broadcasts.

Now:

* Deprecated ❌

Modern replacements:

* StateFlow
* SharedFlow
* LiveData

---

# 🆚 Custom Broadcast vs SharedFlow

| Broadcast             | SharedFlow                 |
| --------------------- | -------------------------- |
| System-level          | App-level                  |
| Loose coupling        | Better modern architecture |
| Older Android pattern | Recommended now            |

---

# 🎯 Real Project Uses

Custom broadcasts used for:

* Logout event
* Session expired
* Download finished
* Payment success
* Game state updates

---

# 🔥 Interview Questions

## ❓ Why use custom broadcast?

To communicate between loosely coupled components.

---

## ❓ Difference between explicit and implicit broadcast?

| Explicit          | Implicit              |
| ----------------- | --------------------- |
| Specific receiver | Any matching receiver |
| More secure       | Less secure           |

---

## ❓ Why LocalBroadcastManager deprecated?

Because:

* Lifecycle-unaware
* Limited
* Flow/LiveData better

---
# 📡 Explicit Broadcast Receiver Example in Android

An **Explicit Broadcast** means:

👉 You directly specify **which BroadcastReceiver should receive the broadcast**.

Unlike implicit broadcast:

* No intent-filter matching needed
* More secure
* Faster
* Only targeted receiver gets it

---

# 🧠 Simple Understanding

## Implicit Broadcast

```id="5ycbn0"
Intent("com.example.LOGIN")
```

Android searches:
👉 “Who can handle this action?”

---

## Explicit Broadcast

```kotlin id="m6v4du"
Intent(this, LoginReceiver::class.java)
```

Android already knows:
👉 “Send directly to LoginReceiver”

---

# 🧱 Full Example

We’ll:

1. Create Receiver
2. Send explicit broadcast
3. Receive it

---

# ✅ Step 1: Create BroadcastReceiver

```kotlin id="v0mq4d"
package com.example.broadcastdemo

import android.content.BroadcastReceiver
import android.content.Context
import android.content.Intent
import android.widget.Toast

class LoginReceiver : BroadcastReceiver() {

    override fun onReceive(context: Context?, intent: Intent?) {

        val username = intent?.getStringExtra("username")

        Toast.makeText(
            context,
            "Explicit Broadcast: $username",
            Toast.LENGTH_SHORT
        ).show()
    }
}
```

---

# ✅ Step 2: Register in Manifest

```xml id="8aef6u"
<receiver
    android:name=".LoginReceiver"
    android:exported="false"/>
```

⚠️ Notice:

* No intent-filter needed
* Because receiver is directly targeted

---

# ✅ Step 3: Send Explicit Broadcast

## MainActivity.kt

```kotlin id="rr5m7s"
val intent = Intent(this, LoginReceiver::class.java)

intent.putExtra("username", "Reshma")

sendBroadcast(intent)
```

---

# 🔄 Flow

```id="7dbjlwm"
MainActivity
    ↓
Explicit Intent
    ↓
sendBroadcast()
    ↓
LoginReceiver directly called
```

---

# 🎯 Output

```id="x17d3x"
Explicit Broadcast: Reshma
```

---

# 🧠 Why Explicit Broadcasts Are Better?

## ✅ More Secure

Only intended receiver gets it.

---

## ✅ Faster

Android doesn’t need to search all apps.

---

## ✅ No Intent Filter Needed

Because class already known.

---

# 🆚 Explicit vs Implicit Broadcast

| Feature                     | Explicit        | Implicit      |
| --------------------------- | --------------- | ------------- |
| Receiver specified directly | ✅ Yes           | ❌ No          |
| Uses intent-filter          | ❌ No            | ✅ Yes         |
| Secure                      | ✅ More          | ❌ Less        |
| Android 8 restrictions      | ✅ Less affected | ⚠️ Restricted |

---

# 🔥 Real Project Example

Suppose:

* DownloadService completes file download
* Notify only your app’s receiver

```kotlin id="0ujq9z"
Intent(this, DownloadReceiver::class.java)
```

---

# 📌 Another Example

## Send Broadcast From Service

```kotlin id="39qz7g"
val intent = Intent(this, PaymentReceiver::class.java)

intent.putExtra("status", "SUCCESS")

sendBroadcast(intent)
```

---

## Receiver

```kotlin id="5bnvml"
class PaymentReceiver : BroadcastReceiver() {

    override fun onReceive(context: Context?, intent: Intent?) {

        val status = intent?.getStringExtra("status")

        Log.d("Payment", status ?: "")
    }
}
```

---

# ⚠️ Important Interview Point

## Explicit Broadcast ≠ Explicit Activity Intent

Both are explicit intents, but used differently:

| Component | Example                                    |
| --------- | ------------------------------------------ |
| Activity  | `Intent(this, SecondActivity::class.java)` |
| Broadcast | `Intent(this, MyReceiver::class.java)`     |

---

# 🔥 Interview Questions

## ❓ Why explicit broadcast preferred?

* Secure
* Efficient
* Avoids unnecessary receivers

---

## ❓ Can explicit broadcast work without manifest?

✅ Yes, if dynamically registered receiver exists.

---

## ❓ Does explicit broadcast need action string?

❌ No

Because target class already specified.

---

# 🧠 How Can Sender App Know Receiver App's BroadcastReceiver?

The sender app must know one of these:

1. ✅ Receiver app's **action string** (implicit broadcast)
   OR
2. ✅ Receiver app's **package + receiver class name** (explicit broadcast)

---

# 📡 Case 1: Implicit Broadcast (Most Common Between Apps)

## Receiver App

### Manifest

```xml id="9c1m3f"
<receiver
    android:name=".PaymentReceiver"
    android:exported="true">

    <intent-filter>
        <action android:name="com.bank.PAYMENT_SUCCESS"/>
    </intent-filter>

</receiver>
```

---

## Sender App

```kotlin id="u4r8f8"
val intent = Intent("com.bank.PAYMENT_SUCCESS")

intent.putExtra("amount", "500")

sendBroadcast(intent)
```

---

# 🔄 How Android Delivers?

Android:

1. Reads action:

   ```id="6s8k1g"
   com.bank.PAYMENT_SUCCESS
   ```
2. Searches installed apps
3. Finds matching receiver
4. Delivers broadcast

---

# 🧠 So Sender App Only Needs:

```id="tk6vsh"
Action String
```

NOT receiver class name.

---

# 📡 Case 2: Explicit Broadcast Between Apps

Here sender knows:

* Package name
* Receiver class name

---

## Receiver App

```xml id="tb3v5n"
<receiver
    android:name=".PaymentReceiver"
    android:exported="true"/>
```

---

## Sender App

```kotlin id="mjlwm2"
val intent = Intent()

intent.component = ComponentName(
    "com.bank.receiverapp",
    "com.bank.receiverapp.PaymentReceiver"
)

sendBroadcast(intent)
```

---

# 🔄 Internally

Android directly sends broadcast to:

```id="xps6zx"
com.bank.receiverapp.PaymentReceiver
```

No intent-filter matching.

---

# ⚠️ Important Requirement

Receiver MUST be:

```xml id="2k0y4r"
android:exported="true"
```

Otherwise other apps cannot access it.

---

# 🔐 Why Android Restricts This?

Imagine malicious apps sending fake broadcasts.

So Android added:

* `exported`
* Permissions
* Android 8+ restrictions

---

# 🧠 Real-World Example

Suppose:

* Paytm app broadcasts payment result
* Your app listens

Paytm documentation gives:

```id="2hhp3g"
Action: com.paytm.PAYMENT_RESULT
```

Your app registers receiver.

---

# 🎯 Important Concepts

| Concept            | Meaning                |
| ------------------ | ---------------------- |
| exported=true      | Allow other apps       |
| implicit broadcast | Based on action        |
| explicit broadcast | Based on component     |
| intent-filter      | Used only for implicit |

---

# 🔥 Interview-Level Answer

## ❓ How does sender app know receiver app?

### For implicit broadcasts:

Sender app only needs the agreed action string.

### For explicit broadcasts:

Sender app must know:

* Package name
* Receiver class name

Android then routes the broadcast appropriately.

---

# ⚠️ Security Best Practice

Avoid public broadcasts unless necessary.

Use:

* Explicit broadcasts
* Permissions
* Bound services
* Deep links
* Content providers

for safer communication.

---

# 🧱 Full Flow Diagram

## Implicit

```id="cw3k8d"
Sender App
   ↓
Action String
   ↓
Android searches matching intent-filters
   ↓
Receiver App gets broadcast
```

---

## Explicit

```id="xmw5ev"
Sender App
   ↓
ComponentName(package, class)
   ↓
Android directly invokes receiver
```
--- 

# 📱 BroadcastReceiver with Service in Android

This is a very common Android pattern.

👉 BroadcastReceiver receives an event
👉 Starts a Service to perform long-running/background work

Because:

⚠️ `BroadcastReceiver.onReceive()` must finish quickly.

So heavy work is moved to:

* Service
* ForegroundService
* WorkManager

---

# 🧠 Real Example

Suppose:

* Internet becomes available
* Receiver detects it
* Starts sync service

---

# 🔄 Flow

```id="5m3pf2"
Broadcast Received
        ↓
BroadcastReceiver
        ↓
Start Service
        ↓
Service does long work
```

---

# 🧱 Full Example

We’ll create:

1. BroadcastReceiver
2. Service
3. Register receiver
4. Start service from receiver

---

# ✅ Step 1: Create Service

```kotlin id="9o5iqn"
package com.example.broadcastdemo

import android.app.Service
import android.content.Intent
import android.os.IBinder
import android.util.Log

class SyncService : Service() {

    override fun onStartCommand(
        intent: Intent?,
        flags: Int,
        startId: Int
    ): Int {

        Log.d("SyncService", "Sync Started")

        // Simulate long-running task
        Thread {
            Thread.sleep(5000)

            Log.d("SyncService", "Sync Completed")

            stopSelf()
        }.start()

        return START_NOT_STICKY
    }

    override fun onBind(intent: Intent?): IBinder? {
        return null
    }
}
```

---

# ✅ Step 2: Create BroadcastReceiver

```kotlin id="9fg2m1"
package com.example.broadcastdemo

import android.content.BroadcastReceiver
import android.content.Context
import android.content.Intent
import android.util.Log

class NetworkReceiver : BroadcastReceiver() {

    override fun onReceive(context: Context?, intent: Intent?) {

        Log.d("Receiver", "Broadcast Received")

        val serviceIntent = Intent(
            context,
            SyncService::class.java
        )

        context?.startService(serviceIntent)
    }
}
```

---

# ✅ Step 3: Register in Manifest

```xml id="s5ejyy"
<application>

    <receiver
        android:name=".NetworkReceiver"
        android:exported="true">

        <intent-filter>
            <action android:name="android.net.conn.CONNECTIVITY_CHANGE"/>
        </intent-filter>

    </receiver>

    <service
        android:name=".SyncService"
        android:exported="false"/>

</application>
```

---

# 🔄 What Happens?

## When internet changes:

Android sends broadcast:

```id="mqxlna"
CONNECTIVITY_CHANGE
```

↓

Receiver gets triggered

↓

Receiver starts service

↓

Service performs background work

---

# 🎯 Why Not Do Work Inside Receiver?

Because:

```kotlin id="if50nc"
onReceive()
```

has limited execution time.

If work takes long:

* ANR
* Receiver killed
* App crashes

---

# ⚠️ Android 8+ Important Change

Background service restrictions added.

So modern apps should use:

* `WorkManager` ✅ (recommended)
* `ForegroundService`
* `JobIntentService` (older)

---

# 🧠 Modern Version Using WorkManager

Today instead of:

```kotlin id="jq5ow4"
startService()
```

Google recommends:

```kotlin id="mq5q94"
WorkManager.enqueue()
```

Because:

* Lifecycle aware
* Battery optimized
* Guaranteed execution

---

# 🆚 BroadcastReceiver vs Service

| BroadcastReceiver | Service            |
| ----------------- | ------------------ |
| Short-lived       | Long-running       |
| Receives events   | Performs work      |
| Main thread       | Background capable |

---

# 🔥 Real Project Examples

| Broadcast          | Service Work   |
| ------------------ | -------------- |
| BOOT_COMPLETED     | Restore alarms |
| INTERNET AVAILABLE | Sync data      |
| DOWNLOAD_COMPLETE  | Process file   |
| BATTERY_OKAY       | Resume uploads |

---

# 🔥 Interview Questions

## ❓ Why start service from BroadcastReceiver?

Because receiver cannot do long-running work safely.

---

## ❓ Why onReceive() should be quick?

Runs on main thread and has timeout.

---

## ❓ What is modern replacement?

✅ WorkManager

---

# 🧱 Modern Real Architecture

```id="x7t1ut"
BroadcastReceiver
      ↓
WorkManager
      ↓
Repository
      ↓
API / DB
```

---

# ⚠️ Extra Important

Some broadcasts no longer work in manifest from Android 8+.

Example:

```id="pm5ql7"
CONNECTIVITY_CHANGE
```

needs dynamic registration.

---

# 📌 Interview-Level Final Summary

BroadcastReceiver is used to listen for events, but since it has a short lifecycle and runs on the main thread, heavy or long-running tasks should be delegated to a Service or WorkManager.
