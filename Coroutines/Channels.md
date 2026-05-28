# Kotlin Channels — Complete Deep Dive Notes

# What are Channels?

Channels are used for **communication between coroutines**.

They allow one coroutine to send data and another coroutine to receive it safely.

Think of a channel like a **pipe** or **queue** between coroutines.

```text
Producer Coroutine ---> Channel ---> Consumer Coroutine
```

Official Kotlin definition:

> Deferred transfers a single value.
> Channel transfers a stream of values.

---

# Why Channels Exist

Before channels, communication between threads/coroutines usually happened using:

* shared mutable variables
* callbacks
* synchronized blocks
* locks/mutex
* blocking queues

These approaches can become difficult and error-prone.

Channels provide:

* safe communication
* structured concurrency
* suspension instead of thread blocking
* asynchronous message passing

---

# Real World Analogy

Imagine:

* Chef prepares food
* Waiter delivers food

They communicate using a food counter.

```text
Chef ---> Counter ---> Waiter
```

Counter = Channel

Chef places food → `send()`

Waiter picks food → `receive()`

Neither directly touches the other's internal state.

---

# Dependency

```kotlin
implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core:<latest-version>")
```

---

# Basic Channel Example

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.channels.Channel

fun main() = runBlocking {

    // Create a channel
    // This is a RENDEZVOUS channel by default
    // Capacity = 0
    val channel = Channel<Int>()

    // Producer Coroutine
    launch {

        println("Producer Started")

        for (i in 1..5) {

            println("Sending $i")

            // send() is a suspend function
            // Producer waits until receiver takes the value
            channel.send(i)

            println("$i sent successfully")
        }

        // Important!
        // Close channel after sending all values
        channel.close()

        println("Producer Finished")
    }

    // Consumer Coroutine
    launch {

        println("Consumer Started")

        // Loop automatically ends when channel closes
        for (item in channel) {

            println("Received $item")

            // Delay added just for visualization
            delay(500)
        }

        println("Consumer Finished")
    }

    println("Main Coroutine Waiting...")
}
```

---

# Output

```text
Main Coroutine Waiting...
Producer Started
Sending 1
Consumer Started
Received 1
1 sent successfully
Sending 2
Received 2
2 sent successfully
Sending 3
Received 3
3 sent successfully
Sending 4
Received 4
4 sent successfully
Sending 5
Received 5
5 sent successfully
Producer Finished
Consumer Finished
```

---

# Important Understanding

## send() is suspend

## receive() is suspend

This means:

* sender suspends if receiver not ready
* receiver suspends if no data available

NO thread blocking occurs.

---

# Internal Working

Internally channel maintains:

* sender queue
* receiver queue
* optional buffer

Depending on capacity:

* sender may suspend
* receiver may suspend
* values may be buffered

---

# Deferred vs Channel

Deferred values provide a convenient way to transfer a single value between coroutines. Channels provide a way to transfer a stream of values.

---

## Deferred

Transfers ONE value.

```kotlin
val deferred = async {
    fetchUser()
}

val user = deferred.await()
```

Flow:

```text
Coroutine ---> One Result
```

Used for:

* API result
* DB result
* one-time computation

---

## Channel

Transfers MANY values.

```kotlin
val channel = Channel<Int>()

launch {
    repeat(100) {
        channel.send(it)
    }
}
```

Flow:

```text
Coroutine ---> Stream of Values
```

Used for:

* events
* queues
* pipelines
* continuous communication

---

# Mental Model

## Deferred = Future/Promise

## Channel = Async Queue

---

# Channel vs BlockingQueue
A Channel is conceptually very similar to BlockingQueue. One key difference is that instead of a blocking put operation it has a suspending send, and instead of a blocking take operation it has a suspending receive.
Kotlin documentation compares Channel with BlockingQueue.

---

| Feature          | Channel    | BlockingQueue  |
| ---------------- | ---------- | -------------- |
| Used with        | Coroutines | Threads        |
| Waiting          | Suspension | Blocking       |
| Thread efficient | Yes        | Less efficient |
| Async friendly   | Excellent  | Limited        |
| Closeable        | Yes        | No             |

---

# Major Difference

---

## BlockingQueue

```kotlin
queue.put(item)
```

If queue full:

```text
THREAD BLOCKS
```

Blocked thread cannot do anything else.

---

## Channel

```kotlin
channel.send(item)
```

If full:

```text
COROUTINE SUSPENDS
```

Thread becomes free for other work.

This is why coroutines scale much better.

---

# Channel Types

This is the MOST IMPORTANT topic.

---

## 1. RENDEZVOUS Channel (Default)
A **RENDEZVOUS channel** in Kotlin Coroutines is a channel with **zero buffer capacity**.

```kotlin
val channel = Channel<Int>()
```

or

```kotlin
val channel = Channel<Int>(Channel.RENDEZVOUS)
```

Both are the same because the default channel type is `RENDEZVOUS`. ([Kotlin][1])

---

### Core Idea

There is **no storage/buffer** inside the channel.

So:

* `send()` waits until someone calls `receive()`
* `receive()` waits until someone calls `send()`

The sender and receiver must **meet at the same time**.

That is why it is called **Rendezvous** (meeting point). ([Kotlin][1])

---

### Visual Understanding

```text
Sender ---- waits ----> Receiver
```

or

```text
Receiver ---- waits ----> Sender
```

No data is stored in between.

---

### Important Behavior

#### Sender suspends

```kotlin
channel.send(10)
```

If no receiver is ready → coroutine pauses.

---

###3 Receiver suspends

```kotlin
channel.receive()
```

If no sender is ready → coroutine pauses.

---

### Real Example

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.channels.Channel

fun main() = runBlocking {

    val channel = Channel<Int>()

    launch {
        println("Before send")
        channel.send(100)
        println("After send")
    }

    delay(2000)

    launch {
        println("Before receive")
        println("Received: ${channel.receive()}")
        println("After receive")
    }
}
```

---

### Output

```text
Before send
(2 seconds pause)
Before receive
Received: 100
After receive
After send
```

---

### Why "After send" comes later?

Because:

```kotlin
channel.send(100)
```

suspends until:

```kotlin
channel.receive()
```

happens.

So the sender is blocked (suspended) waiting for receiver.

---

### Internal Flow

```text
1. Sender tries to send
2. No receiver available
3. Sender suspends

Later...

4. Receiver starts receive()
5. Data transferred directly
6. Both continue
```

---

### Think of It Like This

Imagine handing over a paper directly to another person.

You cannot leave it on a table.

Both people must be present at the same time.

That is RENDEZVOUS channel.

---

### Capacity

RENDEZVOUS channel capacity:

Capacity = 0

---

### Why Use It?

Use when you want:

* strict synchronization
* producer and consumer coordination
* backpressure
* guaranteed handoff
* no buffering

---

### Backpressure Meaning

Producer cannot produce too fast.

Because sender waits for receiver.

This naturally controls speed.

---

### Compare with Buffered Channel

| Feature         | Rendezvous | Buffered       |
| --------------- | ---------- | -------------- |
| Buffer size     | 0          | > 0            |
| Sender waits?   | Yes        | Only when full |
| Data stored?    | No         | Yes            |
| Synchronization | Strict     | Loose          |
| Memory usage    | Minimal    | More           |

---

### Simple Producer-Consumer Example

```kotlin
fun main() = runBlocking {

    val channel = Channel<String>()

    launch {
        val items = listOf("A", "B", "C")

        for (item in items) {
            println("Sending $item")
            channel.send(item)
            println("Sent $item")
        }
    }

    launch {
        repeat(3) {
            delay(1000)
            val value = channel.receive()
            println("Received $value")
        }
    }
}
```

---

### Expected Flow

```text
Sending A
(wait)
Received A
Sent A

Sending B
(wait)
Received B
Sent B
```

Each send waits for receive.

---

### Important Interview Point

A RENDEZVOUS channel is similar to:

* a `BlockingQueue` of size `0`
* direct handoff communication

But unlike `BlockingQueue`,
Kotlin Channels use **suspension**, not thread blocking. ([Kotlin][2])

---

### One-Line Definition

> A RENDEZVOUS channel is a zero-buffer channel where sender and receiver must meet simultaneously to exchange data. ([Kotlin][1])

[1]: https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.channels/-channel/-factory/-r-e-n-d-e-z-v-o-u-s.html?utm_source=chatgpt.com "RENDEZVOUS - kotlinx.coroutines.channels"
[2]: https://kotlinlang.org/docs/channels.html?utm_source=chatgpt.com "Channels | Kotlin Documentation"


---
## 2. Buffered Channel
A **Buffered Channel** in Kotlin Coroutines is a channel that has a **buffer/storage space**.

It allows the sender to send values **without waiting immediately** for the receiver.

---

### Basic Syntax

```kotlin
val channel = Channel<Int>(3)
```

This means:

* buffer size = 3
* up to 3 items can be stored inside the channel

---

### Core Idea

Unlike RENDEZVOUS channel:

```text
RENDEZVOUS:
Sender ↔ Receiver must meet immediately
```

Buffered channel:

```text
Sender → Buffer → Receiver
```

The sender can continue until the buffer becomes full.

---

### Visual Understanding

#### Buffer Size = 3

```text
[10][20][30]
```

If buffer still has space:

```kotlin
channel.send(value)
```

does NOT suspend.

---

### When Does Sender Suspend?

Sender suspends only when:

```text
Buffer is FULL
AND
No receiver is consuming
```

---

### Example

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.channels.Channel

fun main() = runBlocking {

    val channel = Channel<Int>(2)

    launch {
        println("Sending 1")
        channel.send(1)
        println("Sent 1")

        println("Sending 2")
        channel.send(2)
        println("Sent 2")

        println("Sending 3")
        channel.send(3)
        println("Sent 3")
    }

    delay(3000)

    launch {
        repeat(3) {
            println("Received: ${channel.receive()}")
        }
    }
}
```

---

### What Happens Internally?

Buffer capacity:

Capacity = 2

---

### Step-by-step

#### Send 1

```text
Buffer: [1]
```

No suspension.

---

#### Send 2

```text
Buffer: [1][2]
```

Still fine.

---

#### Send 3

Now buffer is full.

```text
Buffer: [1][2]
```

So:

```kotlin
channel.send(3)
```

suspends.

---

#### Receiver Starts

Receiver consumes:

```text
Receive 1
Buffer becomes: [2]
```

Now space is available.

So sender resumes and inserts 3.

```text
Buffer: [2][3]
```

---

### Expected Output

```text
Sending 1
Sent 1

Sending 2
Sent 2

Sending 3
(waiting...)

Received: 1

Sent 3

Received: 2
Received: 3
```

---

### Main Difference from RENDEZVOUS

| Feature                   | Rendezvous   | Buffered      |
| ------------------------- | ------------ | ------------- |
| Capacity                  | 0            | > 0           |
| Buffer exists?            | No           | Yes           |
| Sender waits immediately? | Yes          | No            |
| Sender suspends when      | No receiver  | Buffer full   |
| Speed                     | Synchronized | More flexible |

---

### Real-Life Analogy

#### RENDEZVOUS

Direct handover.

```text
Person A → Person B
```

Both must be present.

---

#### Buffered Channel

Mailbox system.

```text
Person A → Mailbox → Person B
```

Sender can drop messages and leave.

Receiver reads later.

---

### Why Buffered Channels Are Useful

Useful when:

* producer is faster than consumer
* temporary storage needed
* smoother async communication
* reduce suspension frequency

---

### Producer Faster Than Consumer Example

```kotlin
fun main() = runBlocking {

    val channel = Channel<Int>(5)

    launch {
        repeat(10) {
            println("Producing $it")
            channel.send(it)
        }
    }

    launch {
        repeat(10) {
            delay(1000)
            println("Consuming ${channel.receive()}")
        }
    }
}
```

---

### What Happens?

Producer quickly fills:

```text
[0][1][2][3][4]
```

Then waits when full.

Consumer slowly removes items.

---

### Important Interview Point

Buffered channels provide:

* asynchronous communication
* temporary queueing
* controlled backpressure

But:

* larger buffer = more memory usage
* too much buffering can hide slow consumers

---

### Special Buffered Constants

Kotlin provides predefined capacities:

| Type                 | Meaning                 |
| -------------------- | ----------------------- |
| `Channel.RENDEZVOUS` | Capacity 0              |
| `Channel.CONFLATED`  | Keeps latest value only |
| `Channel.UNLIMITED`  | Infinite buffer         |
| `Channel.BUFFERED`   | Default buffered size   |

---

### Example of Default Buffered

```kotlin
val channel = Channel<Int>(Channel.BUFFERED)
```

Uses default internal buffer size.

---

### One-Line Definition

> A Buffered Channel is a channel with storage capacity that allows senders to continue sending until the buffer becomes full.

---

## 3. UNLIMITED Channel

An **UNLIMITED channel** in Kotlin Coroutines is a channel with an **unlimited buffer size**.

```kotlin id="my31zx"
val channel = Channel<Int>(Channel.UNLIMITED)
```

This means:

* sender almost never suspends
* values keep getting stored in memory
* receiver can consume later

---

### Core Idea

```text id="pd4bgq"
Sender → Infinite Buffer → Receiver
```

The producer can keep sending data continuously.

---

### Capacity

Unlimited channel capacity:

Capacity = \infty

(Not truly infinite internally, but practically unbounded until memory runs out.)

---

### Important Behavior

#### send() does NOT suspend

```kotlin id="og0xiz"
channel.send(value)
```

usually continues immediately.

Because buffer can always grow.

---

### Example

```kotlin id="k2n50c"
import kotlinx.coroutines.*
import kotlinx.coroutines.channels.Channel

fun main() = runBlocking {

    val channel = Channel<Int>(Channel.UNLIMITED)

    launch {
        repeat(5) {
            println("Sending $it")
            channel.send(it)
        }
    }

    delay(3000)

    launch {
        repeat(5) {
            println("Received ${channel.receive()}")
        }
    }
}
```

---

### What Happens?

Producer sends all items immediately:

```text id="cl2js9"
Buffer:
[0][1][2][3][4]
```

Even though receiver starts later.

---

### Expected Output

```text id="aw2d5p"
Sending 0
Sending 1
Sending 2
Sending 3
Sending 4

(3 seconds later)

Received 0
Received 1
Received 2
Received 3
Received 4
```

Notice:

* sender never waits
* all items stored in buffer

---

### Internal Working

#### Producer Side

```text id="h1lnrb"
send(1) → store
send(2) → store
send(3) → store
```

No suspension.

---

#### Consumer Side

Later:

```text id="ntp5a6"
receive() → gets oldest item
```

FIFO order maintained.

---

### Real-Life Analogy

Think of:

```text id="s0wd8n"
Producer → Huge warehouse → Consumer
```

Producer keeps dumping packages.

Consumer processes whenever ready.

---

### Danger of Unlimited Channels

Very important.

If producer is much faster than consumer:

```text id="2r80k7"
Messages keep accumulating
```

which can cause:

* huge memory usage
* OutOfMemoryError
* app slowdown

---

### Dangerous Example

```kotlin id="1xv0x8"
val channel = Channel<Int>(Channel.UNLIMITED)

launch {
    while (true) {
        channel.send(Random.nextInt())
    }
}
```

If consumer is slow or absent:

```text id="x3bwy9"
Memory keeps growing forever
```

---

### Comparison with Other Channels

| Type        | Capacity       | Sender Suspends |
| ----------- | -------------- | --------------- |
| RENDEZVOUS  | 0              | Immediately     |
| Buffered(3) | 3              | When full       |
| UNLIMITED   | Infinite       | Almost never    |
| CONFLATED   | 1(latest only) | Never           |

---

### Unlimited vs Buffered

#### Buffered

```kotlin id="c2xjsv"
Channel<Int>(5)
```

Limited queue.

---

#### Unlimited

```kotlin id="brm7n9"
Channel<Int>(Channel.UNLIMITED)
```

No practical limit.

---

### When Should You Use It?

Use only when:

* data volume is controlled
* consumer will definitely catch up
* temporary bursts expected
* message loss unacceptable

---

### When NOT to Use

Avoid when:

* producer speed is unpredictable
* infinite streams exist
* memory usage matters
* consumer may become slow

---

### Better Alternatives Sometimes

Instead of unlimited channels:

* bounded buffered channels
* Flow
* SharedFlow
* StateFlow
* conflated channels

depending on use case.

---

### Important Interview Point

Unlimited channels remove backpressure.

Meaning:

```text id="tv22ov"
Producer never slows down
```

This can be risky in real systems.

---

### One-Line Definition

> An UNLIMITED channel is a channel with an unbounded buffer where send operations usually never suspend because values are continuously accumulated in memory.

---

## 4. CONFLATED Channel

A **Conflated Channel** in Kotlin Coroutines is a channel that keeps **only the latest value**.

Older values are automatically discarded if a new value arrives before the receiver consumes the previous one.

---

### Syntax

```kotlin id="c1j8pp"
val channel = Channel<Int>(Channel.CONFLATED)
```

---

### Core Idea

```text id="d9hm5o"
Old values are replaced by latest value
```

The receiver may miss intermediate values.

Only the newest/latest value matters.

---

### Capacity Behavior

Conflated channel behaves like:

Capacity = 1

BUT with special replacement behavior.

---

### Important Rule

When a new value comes:

```text id="g9x0cz"
Old buffered value → removed
New value → stored
```

So buffer always contains only ONE latest value.

---

### Example

```kotlin id="3qpkv5"
import kotlinx.coroutines.*
import kotlinx.coroutines.channels.Channel

fun main() = runBlocking {

    val channel = Channel<Int>(Channel.CONFLATED)

    launch {
        repeat(5) {
            println("Sending $it")
            channel.send(it)
            delay(100)
        }
    }

    launch {
        delay(500)

        repeat(2) {
            println("Received ${channel.receive()}")
        }
    }
}
```

---

### What Happens Internally?

Producer sends quickly:

```text id="c8wnx7"
Send 0 → buffer = [0]
Send 1 → replace 0
Send 2 → replace 1
Send 3 → replace 2
Send 4 → replace 3
```

Final buffer:

```text id="m6lk9z"
[4]
```

Receiver starts later and gets only latest value.

---

### Expected Output

```text id="of6y32"
Sending 0
Sending 1
Sending 2
Sending 3
Sending 4

Received 4
```

The receiver does NOT get:

```text id="57s89o"
0,1,2,3
```

because they were replaced.

---

### Key Feature

#### send() never suspends

Because:

```text id="f50n3w"
Old value is overwritten
```

instead of waiting for space.

---

### Real-Life Analogy

Think of a:

```text id="0we0be"
Live GPS location tracker
```

You only care about:

```text id="wckm42"
latest location
```

not every intermediate movement.

---

### Great Use Cases

Conflated channels are useful for:

* UI state updates
* latest sensor values
* progress updates
* location tracking
* stock price updates
* live data streams

Where:

```text id="twxj17"
latest value matters more than history
```

---

### Example: Progress Updates

```kotlin id="09nyne"
launch {
    for (i in 1..100) {
        channel.send(i)
    }
}
```

Receiver may only see:

```text id="rbliv7"
40
75
100
```

instead of all values.

This reduces unnecessary processing.

---

### Difference from Buffered Channel

| Feature                   | Buffered | Conflated |
| ------------------------- | -------- | --------- |
| Stores multiple values    | Yes      | No        |
| Keeps history             | Yes      | No        |
| Old values preserved      | Yes      | Replaced  |
| Latest value only         | No       | Yes       |
| send() suspends when full | Yes      | No        |

---

### Difference from StateFlow

Very important interview point.

#### Conflated Channel

* one consumer usually
* hot stream
* values can be lost
* receive-based API

---

#### StateFlow

* multiple collectors
* always has current state
* replay latest automatically
* designed for UI state

Today, many StateFlow use cases replaced conflated channels.

---

### Internal Visualization

```text id="kv9c5d"
Send 1 → [1]
Send 2 → [2]
Send 3 → [3]
```

Receiver gets:

```text id="rpnr5h"
3
```

Only latest survives.

---

### Important Warning

Do NOT use conflated channels when:

* every event matters
* order/history important
* no data loss allowed

Bad examples:

* banking transactions
* chat messages
* payment processing

---

### Interview One-Liner

> A Conflated Channel keeps only the latest value and automatically drops older unconsumed values.


---

# Channel Types Comparison

| Type       | Buffer      | Sender Suspends? | Keeps All Values? |
| ---------- | ----------- | ---------------- | ----------------- |
| RENDEZVOUS | 0           | Immediately      | Yes               |
| BUFFERED   | Fixed       | When full        | Yes               |
| UNLIMITED  | Infinite    | Never            | Yes               |
| CONFLATED  | Latest only | Rarely           | No                |

---

# Sending Data

```kotlin
channel.send(value)
```

Suspends safely.

---

# Receiving Data

```kotlin
val value = channel.receive()
```

Suspends until value available.

---

# Iterating Over Channel

```kotlin
for(item in channel){
    println(item)
}
```

Loop automatically ends when channel closes.

---

# Why close() is Important

Without closing:

```kotlin
for(item in channel)
```

never ends.

Receiver waits forever.

---

# Example Problem

```kotlin
launch {
    repeat(5){
        channel.send(it)
    }
}
```

Receiver hangs forever because channel not closed.

---

# Correct Version

```kotlin
launch {
    repeat(5){
        channel.send(it)
    }

    channel.close()
}
```

---

# What close() Actually Does

Conceptually:

```text
[1][2][3][CLOSE]
```

Receiver consumes:

```text
1 -> 2 -> 3 -> CLOSE -> stop
```

---

# Sending After close()

```kotlin
channel.send(10)
```

Throws exception.

---

# trySend()

Non-suspending send.

```kotlin
channel.trySend(10)
```

Returns success/failure.

Useful for:

* callbacks
* UI listeners
* non-suspending contexts

---

# tryReceive()

Non-suspending receive.

```kotlin
val result = channel.tryReceive()
```

---

# Producer Consumer Pattern

MOST COMMON interview topic.

---

# Producer

Produces data.

```kotlin
launch {
    repeat(5){
        channel.send(it)
    }

    channel.close()
}
```

---

# Consumer

Consumes data.

```kotlin
launch {
    for(item in channel){
        println(item)
    }
}
```

---

# Real Android Examples

Producer:

* socket listener
* API polling
* DB observer

Consumer:

* UI
* analytics worker
* background sync

---

# produce Builder

Creates producer coroutine easily.

---

# Normal Way

```kotlin
val channel = Channel<Int>()

launch {
    send data
}
```

---

# Cleaner Way

```kotlin
fun CoroutineScope.produceNumbers() = produce<Int> {
    repeat(5){
        send(it)
    }
}
```

Usage:

```kotlin
val numbers = produceNumbers()
```

Returns:

```kotlin
ReceiveChannel<Int>
```

---

# Why produce is Useful

Encapsulates producer logic cleanly.

Like:

```text
Function returns async stream
```

---

# Pipelines

[Coroutine Pipeline](Coroutines/coroutine_pipeline.md)
---

# Fan-Out

One producer.

Multiple consumers.

---

# Visual

```text
          ---> Worker 1
Producer ---> Worker 2
          ---> Worker 3
```

---

# Important Rule

Each item goes to ONLY ONE consumer.

NOT broadcast.

---

# Example

```kotlin
fun CoroutineScope.launchProcessor(
    id: Int,
    channel: ReceiveChannel<Int>
) = launch {

    for(msg in channel){
        println("Processor $id received $msg")
    }
}
```

Usage:

```kotlin
val producer = produceNumbers()

repeat(5){
    launchProcessor(it, producer)
}
```

---

# Real Example

Download manager.

Producer:

```text
download requests
```

Workers:

```text
parallel download workers
```

Each request handled once.

---

# Fan-In

Multiple producers.

One consumer.

---

# Visual

```text
Producer 1 ---
Producer 2 ----> Channel ---> Consumer
Producer 3 ---
```

---

# Example

```kotlin
suspend fun sendString(
    channel: SendChannel<String>,
    s: String,
    time: Long
){
    while(true){
        delay(time)
        channel.send(s)
    }
}
```

Usage:

```kotlin
val channel = Channel<String>()

launch {
    sendString(channel, "foo", 200)
}

launch {
    sendString(channel, "BAR!", 500)
}

repeat(6){
    println(channel.receive())
}
```

---

# Fairness

Channels are FIFO fair.

First waiter gets served first.

---

# Example

```text
Coroutine A waiting
Coroutine B waiting
```

If data arrives:

```text
A receives first
```

---

# Why Important

Prevents starvation.

---

# Ticker Channels

Special channel producing periodic events.

---

# Example

```kotlin
val tickerChannel = ticker(delayMillis = 1000)
```

Every second:

```text
tick
tick
tick
```

---

# Use Cases

* periodic sync
* polling
* retry systems
* debounce
* timeout logic

---

# Actor Pattern

Actor = Coroutine + Mailbox(Channel)

Actors safely manage state.

---

# Problem Without Actor

```text
Multiple coroutines modify same variable
```

Leads to:

* race conditions
* synchronization problems

---

# Actor Solution

All updates go through ONE channel.

---

# Example

```kotlin
sealed class CounterMsg

object Increment : CounterMsg()

object GetCount : CounterMsg()

fun CoroutineScope.counterActor() = actor<CounterMsg> {

    var count = 0

    for(msg in channel){

        when(msg){

            is Increment -> count++

            is GetCount -> println(count)
        }
    }
}
```

---

# Why Actors Matter

Eliminates shared mutable state.

Very safe concurrency model.

---

# Backpressure

Backpressure means:

```text
Producer faster than consumer
```

Channels solve this using:

* suspension
* buffering
* conflation

---

# Channel Cancellation

Channels follow coroutine cancellation.

```kotlin
job.cancel()
```

Suspended send/receive operations cancel too.

---

# Structured Concurrency

Channels work best inside:

```kotlin
coroutineScope { }
```

Avoid:

```kotlin
GlobalScope.launch
```

because it may cause:

* leaks
* orphan coroutines
* unmanaged workers

---

# receiveAsFlow()

Very important Android API.

```kotlin
channel.receiveAsFlow()
```

Converts channel into Flow.

Useful for ViewModel events.

---

# Android Example

```kotlin
private val _events = Channel<UiEvent>()

val events = _events.receiveAsFlow()
```

Send:

```kotlin
_events.send(ShowToast)
```

Collect:

```kotlin
viewModel.events.collect {
    // handle event
}
```

---

# Channel vs Flow

Most important modern Android interview question.

---

| Feature             | Channel                     | Flow              |
| ------------------- | --------------------------- | ----------------- |
| Type                | Hot communication primitive | Cold async stream |
| Multiple collectors | Difficult                   | Easy              |
| State management    | Weak                        | Strong            |
| One-time events     | Excellent                   | Moderate          |
| Backpressure        | Manual                      | Built-in          |
| Lifecycle aware     | Less                        | Better            |

---

# StateFlow vs Channel

| Feature             | Channel | StateFlow |
| ------------------- | ------- | --------- |
| Holds current state | No      | Yes       |
| Stream of events    | Yes     | Limited   |
| UI state            | Poor    | Excellent |
| Latest value only   | No      | Yes       |

---

# SharedFlow vs Channel

| Feature             | Channel | SharedFlow |
| ------------------- | ------- | ---------- |
| Consumable once     | Yes     | No         |
| Broadcast           | No      | Yes        |
| Multiple collectors | Hard    | Easy       |
| Replay              | No      | Yes        |

---

# When To Use Channels

Use channels for:

* producer-consumer
* work queues
* pipelines
* coroutine communication
* one-time events
* actors

---

# When NOT To Use Channels

Avoid channels for:

* UI state
* observable app state
* continuous state updates

Use:

* StateFlow
* SharedFlow

instead.

---

# Common Mistakes

---

# 1. Forgetting close()

Receiver hangs forever.

---

# 2. Using UNLIMITED carelessly

Can cause memory explosion.

---

# 3. Using channels for UI state

Prefer StateFlow.

---

# 4. Using GlobalScope

Causes unmanaged coroutines.

---

# Interview Questions

---

# Why use channels?

Safe communication between coroutines without shared mutable state.

---

# Difference between Channel and Flow?

Channel = hot communication primitive.

Flow = cold async stream.

---

# Why send() suspends?

To handle backpressure safely.

---

# Difference between Buffered and Conflated?

Buffered keeps all values.

Conflated keeps latest value only.

---

# Why close channels?

To notify consumers that no more data exists.

---

# Mental Model

## Channel = Suspendable Async Queue

Instead of:

```text
sharing memory
```

channels encourage:

```text
sharing messages
```

---

# One-Line Summary

Kotlin Channels provide safe asynchronous communication between coroutines using suspending send and receive operations.
