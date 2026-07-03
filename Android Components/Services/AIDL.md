# AIDL in Android — Complete Interview Explanation

---

# What is AIDL?

AIDL stands for:

```text id="j2v8ma"
Android Interface Definition Language
```

It is used for:

```text id="r4n1pz"
IPC (Inter Process Communication)
```

between:

* different Android apps
* different processes

AIDL allows one process to call methods in another process like local method calls.

Android implements this using:

* Binder
* Parcel
* Stub
* Proxy

Android documents AIDL as the standard way to define IPC interfaces between processes.

---

# Why Do We Need AIDL?

Every Android app runs in:

* separate Linux process
* separate memory space

So:

❌ App A cannot directly access App B memory.

Android uses Binder IPC to solve this.

AIDL gives a structured way to:

* define remote methods
* send data
* receive results

---

# Real World Example

Suppose:

* Music app provides playback service
* Another app wants to:

  * play song
  * pause song
  * get current track

Apps are in different processes.

AIDL enables communication.

---

# When to Use AIDL?

Use AIDL when:

✅ Cross-process communication required
✅ Need multiple method calls
✅ Need parallel requests
✅ Need high-performance IPC

Examples:

* Music player service
* Payment SDK
* Download manager
* Android system services

---

# When NOT to Use AIDL?

| Situation              | Better Choice     |
| ---------------------- | ----------------- |
| Same process           | Normal Binder     |
| Simple message passing | Messenger         |
| Database sharing       | ContentProvider   |
| One-way notification   | BroadcastReceiver |

---

# Core Components of AIDL

---

# 1. AIDL Interface File

Defines IPC contract.

Example:

```aidl id="m8x5dw"
interface ICalculator {
    int add(int a, int b);
}
```

---

# 2. Stub

Server-side Binder implementation.

Receives IPC calls.

---

# 3. Proxy

Client-side remote object.

Sends IPC requests.

---

# 4. Parcel

Serialization container.

Transfers data between processes.

---

# 5. Binder Driver

Kernel-level IPC mechanism.

Transfers Parcel between processes.

Binder is Android’s core IPC architecture.

---

# COMPLETE WORKING EXAMPLE

We will create:

## Server App

Provides calculator service.

## Client App

Calls:

```kotlin id="h6q2zt"
calculator.add(10,20)
```

Returns:

```text id="f3w9lv"
30
```

---

# STEP 1 — Create AIDL File

Inside server app:

```text id="u1k4rc"
app/src/main/aidl/com/example/aidl/ICalculator.aidl
```

Create:

```aidl id="q7m1yx"
package com.example.aidl;

interface ICalculator {
    int add(int a, int b);
}
```

VERY IMPORTANT:

* package must match
* filename must match

---

# STEP 2 — Create Service

```kotlin id="d5p8na"
class CalculatorService : Service() {

    private val binder =
        object : ICalculator.Stub() {

            override fun add(a: Int, b: Int): Int {
                return a + b
            }
        }

    override fun onBind(intent: Intent?): IBinder {
        return binder
    }
}
```

---

# What Happens Here?

```kotlin id="b9v2qe"
ICalculator.Stub()
```

extends generated Stub class.

This Stub:

* receives Binder requests
* invokes `add()`

---

# STEP 3 — Register Service

```xml id="x4z7kc"
<service
    android:name=".CalculatorService"
    android:exported="true">

    <intent-filter>
        <action android:name="calculator_service"/>
    </intent-filter>
</service>
```

`exported=true`
because another app accesses it.

---

# STEP 4 — Copy AIDL File to Client App

Client app ALSO needs same AIDL file.

```text id="t8n5sm"
app/src/main/aidl/com/example/aidl/ICalculator.aidl
```

Why?

Because both apps must understand same IPC contract.

---

# STEP 5 — Bind Service from Client

```kotlin id="v1r6jf"
class MainActivity : AppCompatActivity() {

    private var calculator: ICalculator? = null

    private val connection =
        object : ServiceConnection {

            override fun onServiceConnected(
                name: ComponentName?,
                service: IBinder?
            ) {

                calculator =
                    ICalculator.Stub.asInterface(service)

                val result =
                    calculator?.add(10,20)

                Log.d("RESULT", "$result")
            }

            override fun onServiceDisconnected(
                name: ComponentName?
            ) {
                calculator = null
            }
        }

    override fun onStart() {
        super.onStart()

        val intent = Intent("calculator_service")
        intent.setPackage("com.example.server")

        bindService(
            intent,
            connection,
            Context.BIND_AUTO_CREATE
        )
    }

    override fun onStop() {
        super.onStop()
        unbindService(connection)
    }
}
```

---

# Internal Working of AIDL

MOST IMPORTANT INTERVIEW PART.

---

# Step-by-Step IPC Flow

---

# STEP 1 — Client Calls Method

```kotlin id="m3k8qy"
calculator.add(10,20)
```

Looks like local call.

BUT:

```text id="n7v4he"
calculator
```

is actually Proxy object.

---

# STEP 2 — Proxy Converts Data into Parcel

Proxy internally does:

```java id="p5w1fc"
data.writeInt(10)
data.writeInt(20)
```

Parcel stores serialized data.

---

# STEP 3 — Binder Driver Transfers Data

Proxy calls:

```java id="q2x9td"
remote.transact(...)
```

Binder driver:

* kernel-level IPC
* transfers Parcel to server process

---

# STEP 4 — Stub Receives Request

Server-side Stub receives transaction.

Internally:

```java id="y6m3rn"
onTransact()
```

gets called.

Stub extracts data:

```java id="n4j8kp"
int a = data.readInt()
int b = data.readInt()
```

---

# STEP 5 — Actual Method Executes

```kotlin id="e8s5vl"
override fun add(a: Int, b: Int): Int {
    return a + b
}
```

Returns:

```text id="q1t7wc"
30
```

---

# STEP 6 — Result Returns Back

Stub writes result into Parcel:

```java id="g5r2mx"
reply.writeInt(result)
```

Binder sends Parcel back.

Proxy reads:

```java id="w9n4za"
reply.readInt()
```

Client gets result.

---

# FULL INTERNAL FLOW

```text id="a3f6jd"
CLIENT PROCESS
--------------------------------

calculator.add(10,20)
       |
       v
Proxy.add()
       |
Parcel.writeInt()
       |
remote.transact()
       |
================================
BINDER DRIVER (Kernel)
================================
       |
       v
Stub.onTransact()
       |
Parcel.readInt()
       |
Actual add()
       |
reply.writeInt(30)
       |
================================
Back to Client
================================
       |
reply.readInt()
       |
returns 30
```

---

# What Stub Actually Looks Like

Simplified generated code:

```java id="r8q1mv"
abstract class Stub extends Binder
        implements ICalculator {

    @Override
    protected boolean onTransact(
            int code,
            Parcel data,
            Parcel reply,
            int flags) {

        int a = data.readInt();
        int b = data.readInt();

        int result = add(a,b);

        reply.writeInt(result);

        return true;
    }
}
```

---

# What Proxy Actually Looks Like

```java id="c7x5nh"
class Proxy implements ICalculator {

    private IBinder remote;

    @Override
    public int add(int a, int b) {

        Parcel data = Parcel.obtain();
        Parcel reply = Parcel.obtain();

        data.writeInt(a);
        data.writeInt(b);

        remote.transact(
            TRANSACTION_add,
            data,
            reply,
            0
        );

        return reply.readInt();
    }
}
```

---

# Interview Understanding

| Component     | Responsibility       |
| ------------- | -------------------- |
| Proxy         | Method call → Parcel |
| Binder Driver | IPC transport        |
| Stub          | Parcel → method call |
| Parcel        | Serialized data      |
| Binder Thread | Executes IPC         |

---

# What Thread Does AIDL Run On?

IMPORTANT.

AIDL methods execute on:

* Binder thread pool
* NOT main thread

So:

* methods must be thread-safe
* avoid UI operations directly

Android docs warn that incoming calls can occur on multiple Binder threads concurrently.

---

# Supported Data Types

| Type       | Supported |
| ---------- | --------- |
| int        | ✅         |
| long       | ✅         |
| boolean    | ✅         |
| String     | ✅         |
| List       | ✅         |
| Map        | ✅         |
| Parcelable | ✅         |

---

# Passing Custom Objects

Need Parcelable.

Example:

```kotlin id="x2m8zt"
@Parcelize
data class User(
    val id: Int,
    val name: String
) : Parcelable
```

AIDL:

```aidl id="v9k4qe"
parcelable User;
```

---

# `in`, `out`, `inout`

```aidl id="u5n7la"
void updateUser(in User user);
```

| Keyword | Meaning         |
| ------- | --------------- |
| in      | client → server |
| out     | server → client |
| inout   | both            |

---

# Security in AIDL

Protect exported services.

Example:

```xml id="j4r8ps"
<service
    android:permission="com.example.PERMISSION"/>
```

Otherwise any app can bind.

---

# Important Interview Questions

---

# Q1. Why AIDL?

For IPC between apps/processes using Binder.

---

# Q2. Why not normal interface?

Normal interfaces work only within same process memory.

---

# Q3. Why same AIDL file in both apps?

Both apps must understand same transaction contract.

---

# Q4. Is AIDL synchronous?

Yes by default.

Client waits for response.

---

# Q5. Why Parcelable instead of Serializable?

Parcelable:

* faster
* optimized
* less memory
* no reflection

---

# Q6. What is `onTransact()`?

Low-level Binder callback handling IPC transactions.

---

# Q7. What is `asInterface()`?

Converts `IBinder` into:

* local object if same process
* Proxy if remote process

---

# Q8. What happens if service crashes?

```kotlin id="n1s6xd"
onServiceDisconnected()
```

gets called.

---

# Q9. Why must AIDL methods be thread-safe?

Multiple Binder threads may invoke methods simultaneously.

---

# Q10. What is Binder transaction limit?

Large objects may cause:

```text id="k8m2rw"
TransactionTooLargeException
```

Android documents Binder transaction buffer limits around IPC payload size.

---

# Difference Between AIDL and Messenger

| AIDL           | Messenger     |
| -------------- | ------------- |
| Parallel calls | Sequential    |
| Faster         | Simpler       |
| Multithreaded  | Single thread |
| Complex IPC    | Simple IPC    |

---

# Difference Between AIDL and ContentProvider

| AIDL                  | ContentProvider |
| --------------------- | --------------- |
| Method calls          | CRUD            |
| RPC style             | URI style       |
| Real-time interaction | Data sharing    |

---

# Real Android Services Using Binder

Android framework heavily uses Binder IPC:

* ActivityManagerService
* PackageManagerService
* LocationManager
* MediaSessionService

Almost entire Android framework depends on Binder architecture.

---

# One-Line Interview Definition

> “AIDL is Android’s IPC framework that allows remote method invocation between processes using Binder.”


 The Binder Thread Pool is a pool of threads that Android creates to handle incoming Binder IPC (Inter-Process Communication) requests.

Why is it needed?

When one process calls a service in another process (using Binder), the receiving process needs threads to handle those requests. Instead of creating a new thread for every request, Android maintains a Binder thread pool.

How it works

Process A (Client)
       |
       | Binder IPC
       v
-------------------------
Process B (Service)
-------------------------
Binder Driver (Kernel)
       |
       v
+-----------------------+
| Binder Thread Pool    |
|  Thread 1             | --> Handles Request 1
|  Thread 2             | --> Handles Request 2
|  Thread 3             | --> Handles Request 3
+-----------------------+

1. A client makes a Binder call.


2. The Binder driver in the kernel delivers the request.


3. An available thread from the Binder thread pool in the target process executes the service method.


4. The result is returned to the caller.



Why not use the main thread?

If Binder requests were handled on the main thread:

The UI could freeze.

Multiple clients couldn't be served concurrently.


The Binder thread pool allows multiple IPC requests to be processed in parallel.

Android interview example

class MyService : IMyService.Stub() {
    override fun getData(): String {
        // Runs on a Binder thread, NOT the main thread
        return "Hello"
    }
}

getData() executes on one of the Binder threads.

Important interview points

Binder thread pool is used for incoming IPC requests.

It belongs to the server process, not the client.

Threads are reused, avoiding the cost of creating new threads for every request.

Multiple Binder calls can run simultaneously on different Binder threads.

Binder threads are not the UI thread.

Long-running work should not be done directly on a Binder thread. Instead, hand it off to a coroutine, executor, or background thread if it might block, so other IPC requests aren't delayed.


Interview question

Q: On which thread does an AIDL method execute?

A: By default, it executes on a Binder thread from the Binder thread pool, not on the main thread.

This is one of the most common Android IPC interview questions.
