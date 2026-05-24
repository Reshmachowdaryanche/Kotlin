# Kotlin Coroutines & Concurrency


# Overview

Concurrency allows an application to perform multiple tasks simultaneously without blocking the main execution flow.

In Kotlin, concurrency is primarily handled using:

- Coroutines
- Suspending Functions
- Coroutine Scopes
- Dispatchers
- Structured Concurrency

Coroutines provide a lightweight and efficient way to write asynchronous and concurrent code.

---

# What is a Coroutine?

A **Coroutine** is a lightweight, suspendable computation.

Unlike threads:
- Coroutines are not tied to a single thread
- They can pause and resume later
- Multiple coroutines can share the same thread
- They consume significantly less memory

---

# Key Advantages of Coroutines

- **Lightweight**  
  Coroutines are much lighter than traditional threads. Thousands or even millions of coroutines can run efficiently without consuming huge amounts of memory.

- **Non-blocking**  
  Coroutines can suspend their execution without blocking the thread. This improves application performance and keeps the UI responsive.

- **Structured Concurrency**  
  Coroutines follow a parent-child hierarchy, making lifecycle management, cancellation, and error handling safer and more predictable.

- **Sequential Syntax**  
  Asynchronous code written with coroutines looks like normal sequential code, making it easier to read, write, and maintain.

- **Dispatcher Support**  
  Coroutines use dispatchers to control thread execution, allowing flexible switching between background threads, IO operations, and the main UI thread.

---

# Suspending Functions

A suspending function can pause execution and resume later without blocking a thread.

## Syntax

```kotlin
suspend fun greet() {
    println("Hello World")
}
```

## Important Rules

- Must use `suspend` keyword
- Can only be called from:
  - Another suspend function
  - A coroutine scope

---

# Example

```kotlin
suspend fun main() {
    showUserInfo()
}

suspend fun showUserInfo() {
    println("Loading user...")
    greet()
    println("User Loaded")
}

suspend fun greet() {
    println("Hello World")
}
```

---

# Adding Coroutine Dependency

## Gradle

```kotlin
dependencies {
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core:1.10.2")
}
```

---

# Creating Coroutines

To create and run a coroutine in Kotlin, you need a few important components:

- **Suspend Function**  
  A suspending function contains asynchronous or long-running operations that can pause and resume without blocking the thread.

- **CoroutineScope**  
  A `CoroutineScope` manages the lifecycle of coroutines and ensures proper structured concurrency.

- **Coroutine Builder**  
  Coroutine builders such as `launch()` or `async()` are used to start new coroutines.

- **Dispatcher**  
  A dispatcher decides which thread or thread pool the coroutine should run on, such as `Dispatchers.IO` or `Dispatchers.Default`.

---

# Basic Coroutine Example

```kotlin
import kotlinx.coroutines.*

suspend fun greet() {
    println("Running on ${Thread.currentThread().name}")
    delay(1000L)
}

suspend fun main() {
    withContext(Dispatchers.Default) {
        launch {
            greet()
        }

        launch {
            println("Another coroutine")
            delay(1000L)
        }

        println("Main execution")
    }
}
```

---

# Coroutine Builders

Coroutine builders are special functions used to create and manage coroutines in Kotlin.

They define how a coroutine starts, executes, and handles its result.

---

## launch()

The `launch()` coroutine builder is used when you want to execute a task asynchronously without returning any result.

It is commonly used for:
- Background operations
- Fire-and-forget tasks
- UI-related asynchronous work

The `launch()` function starts a new coroutine and immediately continues execution without blocking the current thread.

### Example

```kotlin
launch {
    delay(100)
    println("Background work")
}
```

### Characteristics

- Non-blocking execution
- Returns a `Job`
- Suitable for background operations
- Does not return a result

---

## async()

The `async()` coroutine builder is used when a coroutine needs to return a value.

Unlike `launch()`, it returns a `Deferred<T>` object, which represents a future result.

You can retrieve the result using the `await()` function.

It is commonly used for:
- Parallel API calls
- Concurrent calculations
- Data fetching operations

### Example

```kotlin
val result = async {
    delay(100)
    "Data"
}

println(result.await())
```

### Characteristics

- Returns `Deferred<T>`
- Supports parallel execution
- Result is accessed using `await()`
- Non-blocking

---

## withContext()

The `withContext()` function is used to switch coroutine execution from one dispatcher or thread context to another.

It is mainly used to:
- Perform network calls
- Execute database operations
- Move heavy tasks off the main thread

### Example

```kotlin
withContext(Dispatchers.IO) {
    fetchData()
}
```

### Common Dispatcher Usage

- `Dispatchers.IO`  
  Used for network requests, database operations, and file handling.

- `Dispatchers.Default`  
  Used for CPU-intensive tasks such as sorting, calculations, or data processing.

- `Dispatchers.Main`  
  Used for updating UI components on the main thread.

---

## coroutineScope()

The `coroutineScope()` function creates a new coroutine scope and ensures that all child coroutines complete before the parent coroutine finishes.

It supports structured concurrency and proper lifecycle management.

### Example

```kotlin
coroutineScope {
    launch {
        delay(1000)
    }

    launch {
        delay(500)
    }
}
```

### Key Points

- Waits for all child coroutines to complete
- Maintains structured concurrency
- Handles cancellation safely
- Useful for grouping related coroutines

---

# Structured Concurrency

Structured concurrency is a coroutine design principle that organizes coroutines in a parent-child hierarchy.

This ensures:
- Parent coroutine waits for child coroutines
- Child coroutine cancellation propagates automatically
- Safer lifecycle handling
- Better error management

## Coroutine Hierarchy

```text
Parent Coroutine
 ├── Child Coroutine 1
 ├── Child Coroutine 2
 └── Nested Coroutine
```

Structured concurrency helps prevent memory leaks and unmanaged background tasks.

---

# Coroutine Scope

A `CoroutineScope` is responsible for managing coroutine lifecycle and execution context.

Every coroutine must run inside a scope.

## Responsibilities of CoroutineScope

- Controls coroutine lifecycle
- Provides coroutine context
- Manages cancellation
- Defines dispatcher configuration

### Common Coroutine Scopes in Android

- `viewModelScope`
- `lifecycleScope`
- `GlobalScope` (not recommended)

---

# Dispatchers

Dispatchers determine which thread or thread pool a coroutine runs on.

They help separate:
- UI operations
- Network operations
- Background processing

---

## Types of Dispatchers

### `Dispatchers.Main`

Runs coroutines on the Android main/UI thread.

Used for:
- UI updates
- User interaction handling

---

### `Dispatchers.IO`

Optimized for IO operations.

Used for:
- API calls
- Database access
- File operations

---

### `Dispatchers.Default`

Optimized for CPU-intensive work.

Used for:
- Data processing
- Sorting
- Heavy calculations

---

### `Dispatchers.Unconfined`

Starts coroutine in the current thread but may resume in another thread after suspension.

Mostly used for advanced coroutine scenarios and debugging.

---

# Example Using Dispatchers

```kotlin
launch(Dispatchers.IO) {
    fetchApi()
}
```

This launches the coroutine on the IO dispatcher for background execution.

---

# runBlocking()

The `runBlocking()` function creates a coroutine scope and blocks the current thread until all coroutines inside it complete.

It acts as a bridge between blocking and suspending code.

## Example

```kotlin
runBlocking {
    delay(1000)
}
```

## Common Use Cases

- Unit testing
- Main function entry points
- Legacy synchronous code integration

## Important Note

Avoid using `runBlocking()` in Android production UI code because it blocks the main thread and can freeze the application.

---

# Coroutine vs Thread

Coroutines and threads are both used for concurrent programming, but they work differently internally.

A thread is managed directly by the operating system and requires its own memory stack. Creating too many threads increases memory usage and system overhead.

A coroutine, on the other hand, is lightweight and managed by Kotlin. Multiple coroutines can run on a small number of threads, making them much more efficient for asynchronous programming.

---

## Key Differences

### Coroutines

- Lightweight
- Consume less memory
- Non-blocking
- Highly scalable
- Managed by Kotlin runtime
- Can suspend and resume without blocking threads

### Threads

- Heavyweight
- Higher memory consumption
- Blocking by nature
- Limited scalability
- Managed by the operating system
- Each thread requires a separate memory stack

---

# Example Comparison

## Coroutine Example

The following example launches 50,000 coroutines efficiently using very little memory.

```kotlin
repeat(50_000) {
    launch {
        delay(5000)
    }
}
```

### How it works

- `launch` creates lightweight coroutines
- `delay()` suspends without blocking the thread
- Threads remain free to execute other coroutines
- Suitable for large-scale concurrent operations

---

## Thread Example

The following example creates 50,000 JVM threads.

```kotlin
repeat(50_000) {
    thread {
        Thread.sleep(5000)
    }
}
```

### How it works

- Each thread allocates its own memory stack
- `Thread.sleep()` blocks the thread completely
- Creating thousands of threads consumes huge memory
- Can lead to performance issues or OutOfMemory errors

---

# Memory Comparison

Coroutines are significantly more memory-efficient than threads.

Approximate comparison:

- 50,000 coroutines → around a few hundred MB
- 50,000 threads → can consume tens of GB of memory

Because of this efficiency, coroutines are preferred for modern Android and backend asynchronous programming.

---

# Why Coroutines are Preferred in Android

Coroutines help Android applications by:

- Keeping the UI responsive
- Avoiding ANR (Application Not Responding)
- Reducing memory consumption
- Simplifying asynchronous code
- Improving scalability

Modern Android development heavily relies on:
- Kotlin Coroutines
- Flow
- StateFlow
- Structured Concurrency

---

# Concurrency Concepts

## Parallelism

Multiple tasks running simultaneously on multiple cores.

## Concurrency

Multiple tasks making progress together.

---

# Common Concurrency Patterns

## Producer–Consumer

```text
Producer -> Queue -> Consumer
```

Used in:
- Logging systems
- Messaging queues

---

## Thread Pool

Reusable worker threads for executing tasks efficiently.

---

## Future/Promise Pattern

Represents a value available later.

In Kotlin:
- `Deferred`
- `async/await`

---

# Delay vs Thread.sleep()

| delay() | Thread.sleep() |
|---|---|
| Non-blocking | Blocking |
| Coroutine-friendly | Thread blocking |
| Efficient | Expensive |

---
