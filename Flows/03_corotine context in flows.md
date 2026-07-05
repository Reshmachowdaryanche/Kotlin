# Coroutine Context in Kotlin Flow

One of the most important concepts in Kotlin Flow is **where the flow runs**.

When using flows, you'll often need to answer questions like:

* Should this code run on the **Main thread**?
* Should database work run on the **IO thread**?
* Should CPU-intensive work run on the **Default thread**?
* What happens if the coroutine is cancelled?
* How does Flow follow structured concurrency?

These concepts are controlled by the **Coroutine Context**.

---

# What is Coroutine Context?

A **Coroutine Context** defines the environment in which a coroutine executes.

It contains information like:

* Which thread (Dispatcher) to use
* Job (for cancellation)
* Coroutine name
* Exception handler

For Flow, the most important parts are:

```
Coroutine Context
│
├── Dispatcher
├── Job (Cancellation)
└── Coroutine Scope
```

---

# Understanding Dispatchers

A **Dispatcher** decides **which thread** executes your coroutine.

Imagine you have different workers.

```
Main Worker
↓

Updates UI
```

```
IO Worker
↓

Network
Database
File Reading
```

```
Default Worker
↓

Heavy CPU work

Sorting

Image Processing

Compression
```

---

## Dispatchers.Main

Runs on the **UI thread**.

Use for:

* Updating TextView
* Showing dialogs
* RecyclerView updates
* Compose UI

Example

```kotlin
launch(Dispatchers.Main) {
    textView.text = "Hello"
}
```

Never perform network/database operations here.

---

## Dispatchers.IO

Designed for

* API calls
* Room database
* File operations
* Reading storage

Example

```kotlin
launch(Dispatchers.IO) {
    val users = api.getUsers()
}
```

---

## Dispatchers.Default

Used for CPU-intensive work.

Examples

* Sorting
* Searching
* Image processing
* Encryption
* Parsing huge JSON

```kotlin
launch(Dispatchers.Default) {
    val result = hugeList.sorted()
}
```

---

## Dispatchers.Unconfined

Rarely used.

Execution starts in the current thread and may resume on a different thread after suspension.

Avoid using it in production unless you fully understand its behavior.

---

# Flow Execution

Consider this flow:

```kotlin
val numbers = flow {

    println(Thread.currentThread().name)

    emit(1)

    emit(2)
}
```

Collecting

```kotlin
runBlocking {

    numbers.collect {

        println(it)
    }
}
```

Output

```
main
1
2
```

The flow runs in the **same coroutine context** as the collector.

This is called **context preservation**.

---

# Context Preservation

A flow **inherits the collector's coroutine context** unless you explicitly change it.

Example

```kotlin
runBlocking {

    launch(Dispatchers.IO) {

        flow {
            println(Thread.currentThread().name)
            emit(10)
        }.collect {
            println(it)
        }

    }
}
```

Output

```
DefaultDispatcher-worker-1
10
```

The flow executes on the IO dispatcher because the collector is running there.

---

# Why flowOn?

Suppose your flow performs a network request.

```kotlin
flow {

    val users = api.getUsers()

    emit(users)
}
```

If collected on the Main thread:

```
Main Thread

↓

Network Call

↓

UI Freezes
```

That's bad.

We want:

```
Main Thread

↓

Collect

↓

IO Thread

↓

Network
```

This is what `flowOn()` does.

---

# flowOn()

`flowOn()` changes the **upstream coroutine context** (everything before it).

Example

```kotlin
flow {

    emit(fetchData())

}
.flowOn(Dispatchers.IO)
```

Now

```
Flow Builder

↓

IO Thread

↓

Collector

↓

Main Thread
```

The network work happens on IO, while the collector remains on the Main thread.

---

# Example

```kotlin
fun fetchData(): String {

    Thread.sleep(1000)

    return "Data"
}

flow {

    emit(fetchData())

}
.flowOn(Dispatchers.IO)
.collect {

    println(it)
}
```

`fetchData()` runs on the IO dispatcher.

---

# Important Rule

`flowOn()` affects only the operators **before** it.

Example

```kotlin
flow {

    emit(1)

}
.map {

    println("Map")

    it * 10

}
.flowOn(Dispatchers.IO)

.filter {

    println("Filter")

    true

}
.collect {

    println(it)

}
```

Execution

```
IO Thread

↓

flow

↓

map

────────────

Main Thread

↓

filter

↓

collect
```

Only `flow {}` and `map` run on IO.

`filter` and `collect` run in the collector's context.

---

# Multiple flowOn()

Example

```kotlin
flow {

    emit(1)

}
.map {

    it * 10

}
.flowOn(Dispatchers.Default)

.filter {

    true

}
.flowOn(Dispatchers.IO)

.collect {

}
```

Execution

```
Default

↓

flow

↓

map

----------------

IO

↓

filter

----------------

Collector Context

↓

collect
```

Each `flowOn()` changes the context for everything **upstream** from it until another `flowOn()` is encountered.

---

# Visualization

```
flow

↓

map

↓

flowOn(IO)

↓

filter

↓

collect
```

Execution

```
IO

↓

flow

↓

map

----------------

Main

↓

filter

↓

collect
```

---

# Cancellation

Flows fully support **cooperative cancellation**.

Suppose

```kotlin
val numbers = flow {

    for(i in 1..100){

        delay(500)

        emit(i)
    }
}
```

Collect

```kotlin
val job = launch {

    numbers.collect {

        println(it)

    }

}

delay(2000)

job.cancel()
```

Output

```
1
2
3
```

Collection stops immediately after cancellation.

---

# Why?

Functions like

* `delay()`
* `emit()`
* `collect()`

are suspension points.

They check whether the coroutine is cancelled.

---

# Cancellation Example

```kotlin
flow {

    emit(1)

    delay(1000)

    emit(2)

    delay(1000)

    emit(3)

}
.collect {

    println(it)

}
```

If the coroutine is cancelled after emitting `2`:

Output

```
1

2
```

`3` is never emitted.

---

# onCompletion()

Useful for cleanup.

```kotlin
flow {

    emit(1)

    emit(2)

}
.onCompletion {

    println("Flow Finished")

}
.collect {

    println(it)

}
```

Output

```
1

2

Flow Finished
```

`onCompletion` is called whether the flow completes normally or is cancelled (the `cause` parameter can tell you why).

---

# Structured Concurrency

One of Kotlin Coroutines' biggest advantages is **structured concurrency**.

### What is it?

Child coroutines are tied to their parent.

```
Parent Coroutine

│

├── Child 1

├── Child 2

└── Child 3
```

If the parent is cancelled,

```
Parent Cancelled

↓

Child 1 Cancelled

↓

Child 2 Cancelled

↓

Child 3 Cancelled
```

Everything stops together.

---

# Flow and Structured Concurrency

Example

```kotlin
runBlocking {

    launch {

        flow {

            emit(1)

            delay(1000)

            emit(2)

        }.collect {

            println(it)

        }

    }

}
```

Here

```
runBlocking

↓

launch

↓

Flow

↓

collect
```

If `runBlocking` ends,

everything underneath is cancelled automatically.

---

# Android Example

Inside a `ViewModel`

```kotlin
viewModelScope.launch {

    repository.users

        .collect {

            _state.value = it

        }

}
```

When the `ViewModel` is cleared,

```
ViewModel Destroyed

↓

viewModelScope Cancelled

↓

Flow Cancelled

↓

Network Cancelled
```

You don't need to manually stop the flow.

---

# Lifecycle Example

```
Activity

↓

lifecycleScope

↓

Collect Flow

↓

User leaves Activity

↓

lifecycleScope cancelled

↓

Flow cancelled
```

This prevents memory leaks and unnecessary work.

---

# Best Practices

### Use `flowOn(Dispatchers.IO)` for

* Network calls
* Room database
* File operations

```kotlin
repository.users
    .flowOn(Dispatchers.IO)
```

---

### Use `Dispatchers.Default` for

* Sorting
* Parsing
* Calculations

```kotlin
flow {
    emit(processLargeData())
}.flowOn(Dispatchers.Default)
```

---

### Collect on Main

UI updates should happen on the Main thread.

```kotlin
lifecycleScope.launch {

    repository.users.collect {

        adapter.submitList(it)

    }

}
```

No `flowOn()` is needed here if the upstream work is already on the appropriate dispatcher.

---

### Avoid blocking calls

Don't do this:

```kotlin
flow {

    Thread.sleep(3000)

    emit(1)

}
```

Use suspend functions instead:

```kotlin
flow {

    delay(3000)

    emit(1)

}
```

Blocking calls tie up a thread, while suspension frees it for other coroutines.

---

# Complete Example

```kotlin
fun getUsers() = flow {

    println("Fetching on: ${Thread.currentThread().name}")

    delay(1000) // Simulate network call

    emit(listOf("Alice", "Bob", "Charlie"))

}
.flowOn(Dispatchers.IO)

fun main() = runBlocking {

    getUsers().collect { users ->

        println("Collected on: ${Thread.currentThread().name}")

        println(users)

    }

}
```

Typical output (thread names may vary):

```
Fetching on: DefaultDispatcher-worker-1
Collected on: main
[Alice, Bob, Charlie]
```

The flow builder executes on the **IO dispatcher** because of `flowOn(Dispatchers.IO)`, while `collect` runs in the coroutine that called it (`runBlocking` on the main thread in this example).

---

# Summary

| Concept                | Purpose                                                                            | Example                                                                     |
| ---------------------- | ---------------------------------------------------------------------------------- | --------------------------------------------------------------------------- |
| `Dispatchers.Main`     | UI work                                                                            | Update TextView, Compose state                                              |
| `Dispatchers.IO`       | Blocking I/O operations                                                            | Network, Room, file access                                                  |
| `Dispatchers.Default`  | CPU-intensive work                                                                 | Sorting, parsing, image processing                                          |
| `flowOn()`             | Changes the context of upstream flow operators                                     | `flow { ... }.flowOn(Dispatchers.IO)`                                       |
| Context preservation   | Flow runs in the collector's context unless changed                                | Collector on IO → flow runs on IO                                           |
| Cancellation           | Stops flow cooperatively when its coroutine is cancelled                           | `job.cancel()` stops `collect`                                              |
| Structured concurrency | Child coroutines (including flow collection) are cancelled with their parent scope | `viewModelScope`, `lifecycleScope` automatically cancel ongoing collections |
