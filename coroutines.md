# Kotlin Coroutines & Concurrency

> Based on Kotlin official coroutine concepts and concurrency fundamentals.

---

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

| Feature | Benefit |
|---|---|
| Lightweight | Millions of coroutines possible |
| Non-blocking | Better performance |
| Structured concurrency | Safe lifecycle handling |
| Sequential syntax | Easy to read |
| Dispatcher support | Flexible threading |

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

To create a coroutine, you need:

| Requirement | Purpose |
|---|---|
| Suspend function | Async operation |
| CoroutineScope | Lifecycle management |
| Coroutine Builder | Launch coroutine |
| Dispatcher | Thread selection |

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

Coroutine builders create and manage coroutines.

| Builder | Purpose |
|---|---|
| `launch()` | Fire-and-forget task |
| `async()` | Returns result |
| `withContext()` | Switch dispatcher |
| `coroutineScope()` | Structured concurrency |
| `runBlocking()` | Bridge blocking code |

---

# launch()

Used when result is not required.

## Example

```kotlin
launch {
    delay(100)
    println("Background work")
}
```

## Characteristics

- Non-blocking
- Returns `Job`
- Used for background operations

---

# async()

Used when coroutine returns a value.

## Example

```kotlin
val result = async {
    delay(100)
    "Data"
}

println(result.await())
```

## Characteristics

- Returns `Deferred`
- Use `await()` for result

---

# withContext()

Switches coroutine execution context.

## Example

```kotlin
withContext(Dispatchers.IO) {
    fetchData()
}
```

## Common Usage

| Dispatcher | Use Case |
|---|---|
| `Dispatchers.IO` | Network / DB |
| `Dispatchers.Default` | CPU intensive |
| `Dispatchers.Main` | UI updates |

---

# coroutineScope()

Creates a new coroutine scope and waits for all child coroutines to complete.

## Example

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

---

# Structured Concurrency

Structured concurrency ensures:

- Parent waits for children
- Child cancellation propagates
- Safe lifecycle management

## Hierarchy

```text
Parent Coroutine
 ├── Child Coroutine 1
 ├── Child Coroutine 2
 └── Nested Coroutine
```

---

# Coroutine Scope

A `CoroutineScope` manages coroutine lifecycle and context.

## Responsibilities

- Controls coroutine lifecycle
- Provides dispatcher
- Handles cancellation

---

# Dispatchers

Dispatchers determine which thread or thread pool executes a coroutine.

## Types of Dispatchers

| Dispatcher | Purpose |
|---|---|
| `Dispatchers.Main` | UI thread |
| `Dispatchers.IO` | File/Network |
| `Dispatchers.Default` | CPU work |
| `Dispatchers.Unconfined` | Advanced usage |

---

# Example Using Dispatchers

```kotlin
launch(Dispatchers.IO) {
    fetchApi()
}
```

---

# runBlocking()

Blocks current thread until coroutine completes.

## Example

```kotlin
runBlocking {
    delay(1000)
}
```

## Use Cases

- Testing
- Legacy integration
- Main function bridging

> Avoid using in production UI code.

---

# Coroutine vs Thread

| Feature | Coroutine | Thread |
|---|---|
| Lightweight | Yes | No |
| Memory Usage | Low | High |
| Blocking | Non-blocking | Blocking |
| Scalability | Very High | Limited |
| OS Managed | No | Yes |

---

# Example Comparison

## Coroutine

```kotlin
repeat(50_000) {
    launch {
        delay(5000)
    }
}
```

## Thread

```kotlin
repeat(50_000) {
    thread {
        Thread.sleep(5000)
    }
}
```

Coroutines use significantly less memory.

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

# Android Best Practices

## Recommended Architecture

```text
ViewModel
   ↓
Repository
   ↓
Data Source
```

Use:
- Coroutines
- Flow
- StateFlow
- Structured concurrency

---

# Android Coroutine Example

```kotlin
viewModelScope.launch {
    val data = withContext(Dispatchers.IO) {
        repository.fetchData()
    }

    _uiState.value = data
}
```

---

# Best Practices

## DO

- Use structured concurrency
- Use `Dispatchers.IO` for network/database
- Cancel unused coroutines
- Use `viewModelScope`

## DON'T

- Use `GlobalScope`
- Block main thread
- Overuse `runBlocking`
- Launch unmanaged coroutines

---

# Common Problems

| Problem | Description |
|---|---|
| Race Condition | Multiple threads modify same data |
| Deadlock | Threads waiting forever |
| Starvation | Thread never gets resources |
| Memory Leak | Coroutine not cancelled |

---

# Modern Android Concurrency Stack

Most modern Android apps use:

- Kotlin Coroutines
- Flow
- StateFlow
- WorkManager

---

# Recommended Learning Path

1. Threads Basics
2. Coroutines
3. Suspend Functions
4. launch & async
5. Dispatchers
6. Structured Concurrency
7. Flow & StateFlow
8. Channels
9. Mutex
10. Advanced Coroutine Context

---

# Summary

Kotlin Coroutines provide:
- Lightweight concurrency
- Non-blocking execution
- Structured lifecycle handling
- Better readability
- High scalability

They are the standard approach for modern Android and Kotlin asynchronous programming.
