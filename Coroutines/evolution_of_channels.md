# Queue → Blocking Queue → Producer Consumer → Coroutine Channels (Kotlin)

# 1. Introduction

This document explains the evolution of concurrent communication patterns using Kotlin.

We will cover:

```text id="4s0l08"
Simple Queue
→ Producer Consumer Pattern
→ wait()/notify()
→ Blocking Queue
→ Coroutine Channels
```

including:

* race conditions
* synchronization
* notify/wait
* blocking
* thread coordination
* coroutine communication

All examples are complete runnable Kotlin programs.

---

# 2. Simple Queue

A queue is a FIFO (First In First Out) data structure.

```text id="vzjzod"
Producer → Queue → Consumer
```

Producer inserts items.

Consumer removes items.

---

# Simple Queue Example (Unsafe)

```kotlin id="2mjlwm"
import kotlin.concurrent.thread

val queue = mutableListOf<Int>()

fun main() {

    thread {
        producer()
    }

    thread {
        consumer()
    }
}

fun producer() {

    var value = 0

    while (true) {

        queue.add(value)

        println("[Producer] Produced: $value")

        value++

        Thread.sleep(1000)
    }
}

fun consumer() {

    while (true) {

        if (queue.isNotEmpty()) {

            val item = queue.removeAt(0)

            println("[Consumer] Consumed: $item")
        }

        Thread.sleep(500)
    }
}
```

---

# Problems with Simple Queue

---

# Problem 1 — Race Conditions

Multiple threads access shared queue simultaneously.

```text id="3h08vs"
Producer modifies queue
Consumer modifies queue
At same time
```

Possible issues:

* corrupted state
* crashes
* inconsistent data

---

# Problem 2 — Busy Waiting

Consumer repeatedly checks queue.

```kotlin id="91n02o"
while (true) {

    if (queue.isNotEmpty()) {
        consume()
    }
}
```

This wastes CPU cycles.

This is called:

* polling
* spinning
* busy waiting

---

# Problem 3 — Queue Overflow

Producer faster than consumer.

Queue grows infinitely.

Results:

* memory issues
* latency spikes
* crashes

---

# Problem 4 — Empty Queue

Consumer keeps checking empty queue.

Still wasting CPU.

---

# 3. Producer Consumer Pattern

Producer and consumer communicate through shared buffer.

---

# Architecture

```text id="4lf1yu"
Producer → Shared Queue → Consumer
```

---

# Example Without Proper Synchronization

```kotlin id="x6xgtg"
import kotlin.concurrent.thread

val queue = mutableListOf<Int>()

const val MAX_SIZE = 5

fun main() {

    thread {
        producer()
    }

    thread {
        consumer()
    }
}

fun producer() {

    var value = 0

    while (true) {

        if (queue.size < MAX_SIZE) {

            queue.add(value)

            println("[Producer] Produced: $value")

            value++
        }

        Thread.sleep(1000)
    }
}

fun consumer() {

    while (true) {

        if (queue.isNotEmpty()) {

            val item = queue.removeAt(0)

            println("[Consumer] Consumed: $item")
        }

        Thread.sleep(2000)
    }
}
```

---

# Problems Still Exist

Even now:

* busy waiting exists
* synchronization is manual
* race conditions still possible

Need thread coordination.

---

# 4. wait() and notify()

Instead of continuously polling:

* consumer sleeps when queue empty
* producer wakes consumer after adding item

---

# Concept

```text id="jlwmh7"
Queue Empty
    ↓
Consumer waits()

Producer adds item
    ↓
Producer notify()

Consumer wakes up
```

---

# Full wait()/notify() Example

```kotlin id="wl75ea"
import kotlin.concurrent.thread

val queue = mutableListOf<Int>()

const val MAX_SIZE = 5

fun main() {

    thread(name = "Producer") {
        producer()
    }

    thread(name = "Consumer") {
        consumer()
    }
}

fun producer() {

    var value = 0

    while (true) {

        synchronized(queue) {

            while (queue.size == MAX_SIZE) {

                println("[Producer] Queue full. Waiting...")

                queue.wait()
            }

            queue.add(value)

            println("[Producer] Produced: $value")

            value++

            queue.notify()

            Thread.sleep(1000)
        }
    }
}

fun consumer() {

    while (true) {

        synchronized(queue) {

            while (queue.isEmpty()) {

                println("[Consumer] Queue empty. Waiting...")

                queue.wait()
            }

            val item = queue.removeAt(0)

            println("[Consumer] Consumed: $item")

            queue.notify()

            Thread.sleep(2000)
        }
    }
}
```

---

# Important: Why while Instead of if?

Correct:

```kotlin id="9vv8m0"
while (queue.isEmpty()) {
    queue.wait()
}
```

Wrong:

```kotlin id="1l6q7q"
if (queue.isEmpty()) {
    queue.wait()
}
```

Because:

* spurious wakeups happen
* another thread may consume first
* condition must be rechecked

---

# Problems with wait()/notify()

---

# Problem 1 — Missed Notifications

Producer may notify before consumer waits.

Result:

```text id="bm6n0i"
Consumer sleeps forever
```

---

# Problem 2 — Deadlocks

Improper locking freezes program.

---

# Problem 3 — Complex Synchronization

Need careful handling of:

* synchronized
* locks
* monitors
* condition variables

Code becomes hard to maintain.

---

# 5. Blocking Queue

BlockingQueue solves synchronization automatically.

---

# Features

If queue empty:

```text id="nnj6z3"
Consumer blocks automatically
```

If queue full:

```text id="l0p4mj"
Producer blocks automatically
```

No polling.

No manual notify logic.

---

# Architecture

```text id="pkv4kc"
Producer → BlockingQueue → Consumer
```

---

# Full BlockingQueue Example

```kotlin id="5s48q0"
import java.util.concurrent.ArrayBlockingQueue
import kotlin.concurrent.thread

val queue = ArrayBlockingQueue<Int>(5)

fun main() {

    thread(name = "Producer") {
        producer()
    }

    thread(name = "Consumer") {
        consumer()
    }
}

fun producer() {

    var value = 0

    while (true) {

        queue.put(value)

        println("[Producer] Produced: $value")

        value++

        Thread.sleep(1000)
    }
}

fun consumer() {

    while (true) {

        val item = queue.take()

        println("[Consumer] Consumed: $item")

        Thread.sleep(2000)
    }
}
```

---

# Advantages of BlockingQueue

* thread-safe
* automatic blocking
* automatic wakeup
* no busy waiting
* simpler code

---

# Internal Behavior

## Consumer

```text id="6f5c1l"
take()
↓
If queue empty → thread sleeps
```

---

# Producer

```text id="q7j0m7"
put()
↓
If queue full → thread sleeps
```

---

# 6. Multiple Producers and Consumers

BlockingQueue scales naturally.

---

# Example

```kotlin id="h8ih74"
import java.util.concurrent.ArrayBlockingQueue
import kotlin.concurrent.thread

val queue = ArrayBlockingQueue<Int>(10)

fun main() {

    repeat(2) { id ->
        thread {
            producer(id)
        }
    }

    repeat(3) { id ->
        thread {
            consumer(id)
        }
    }
}

fun producer(id: Int) {

    var value = 0

    while (true) {

        queue.put(value)

        println("[Producer-$id] Produced: $value")

        value++

        Thread.sleep(1000)
    }
}

fun consumer(id: Int) {

    while (true) {

        val item = queue.take()

        println("[Consumer-$id] Consumed: $item")

        Thread.sleep(2000)
    }
}
```

---

# Limitation of BlockingQueue

Still tightly coupled.

```text id="a10zjv"
Producer knows queue
Consumer knows queue
```

Mostly useful inside single JVM/process.

---

# 7. Coroutine Channels

Coroutine channels are modern communication primitives.

Built for Kotlin coroutines.

---

# Philosophy

```text id="m0efzx"
Do not share mutable state.
Communicate using messages.
```

---

# Architecture

```text id="xx9k6x"
Coroutine Sender → Channel → Coroutine Receiver
```

---

# Advantages

* lightweight
* structured concurrency
* cancellation support
* safer communication
* no manual locks

---

# Dependency

```kotlin id="dd6gko"
implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core:1.8.1")
```

---

# Full Coroutine Channel Example

```kotlin id="4cl70k"
import kotlinx.coroutines.*
import kotlinx.coroutines.channels.Channel

fun main() = runBlocking {

    val channel = Channel<Int>()

    launch {
        producer(channel)
    }

    launch {
        consumer(channel)
    }

    delay(20000)
}

suspend fun producer(channel: Channel<Int>) {

    var value = 0

    while (true) {

        println("[Producer] Sending: $value")

        channel.send(value)

        value++

        delay(1000)
    }
}

suspend fun consumer(channel: Channel<Int>) {

    while (true) {

        val item = channel.receive()

        println("[Consumer] Received: $item")

        delay(2000)
    }
}
```

---

# Important Behavior

---

# send()

```text id="0j6h14"
If channel full
↓
Coroutine suspends
```

NOT thread blocking.

---

# receive()

```text id="jlwmx3"
If channel empty
↓
Coroutine suspends
```

Again:

NOT thread blocking.

---

# Difference Between Thread Blocking and Coroutine Suspension

---

# Thread Blocking

```text id="uxq4os"
OS thread becomes inactive
```

Heavyweight.

Consumes resources.

Example:

```kotlin id="m9jls0"
Thread.sleep(1000)
```

---

# Coroutine Suspension

```text id="h5c58m"
Coroutine pauses
Thread reused by others
```

Very lightweight.

Example:

```kotlin id="w3q55x"
delay(1000)
```

---

# Buffered Channels

Buffered channels behave similar to BlockingQueue.

---

# Example

```kotlin id="wz0sw6"
import kotlinx.coroutines.*
import kotlinx.coroutines.channels.Channel

fun main() = runBlocking {

    val channel = Channel<Int>(5)

    launch {

        for (i in 0..20) {

            println("[Producer] Sending: $i")

            channel.send(i)
        }

        channel.close()
    }

    launch {

        for (item in channel) {

            println("[Consumer] Received: $item")

            delay(2000)
        }
    }
}
```

---

# Unbuffered vs Buffered Channels

| Type       | Behavior                           |
| ---------- | ---------------------------------- |
| Unbuffered | sender waits for receiver          |
| Buffered   | sender waits only when buffer full |

---

# 8. Evolution Summary

```text id="7g5m81"
Simple Queue
    ↓
Problems:
- race conditions
- busy waiting

wait()/notify()
    ↓
Solved:
- sleeping/wakeup

Problems:
- deadlocks
- complexity

BlockingQueue
    ↓
Solved:
- synchronization
- automatic blocking

Problems:
- tight coupling

Coroutine Channels
    ↓
Solved:
- shared mutable state
- structured concurrency
- lightweight communication
```

---

# 9. Comparison Table

| Feature            | Queue  | wait/notify | BlockingQueue | Channels  |
| ------------------ | ------ | ----------- | ------------- | --------- |
| Thread Safe        | No     | Manual      | Yes           | Yes       |
| Busy Waiting       | Yes    | No          | No            | No        |
| Blocking           | Manual | Manual      | Automatic     | Automatic |
| Shared Memory      | Yes    | Yes         | Yes           | Minimal   |
| Locks Required     | Yes    | Yes         | Hidden        | No        |
| Lightweight        | No     | No          | Moderate      | Yes       |
| Coroutine Friendly | No     | No          | Limited       | Excellent |

---

# 10. Key Takeaways

Simple queues are unsafe in concurrent systems.

wait()/notify() introduced sleeping and waking.

BlockingQueue simplified synchronization.

Coroutine channels modernized concurrency using lightweight suspension instead of blocking threads.

Modern Kotlin applications prefer:

* coroutines
* channels
* flows
* structured concurrency

over manual thread synchronization.
