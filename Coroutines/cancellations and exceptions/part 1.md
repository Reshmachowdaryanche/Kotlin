# Kotlin Coroutines: CoroutineScope, Job, and CoroutineContext

## Overview

When working with Kotlin Coroutines, three concepts form the foundation of structured concurrency:

1. **CoroutineScope** → Defines the lifecycle of coroutines.
2. **Job** → Represents and manages a coroutine.
3. **CoroutineContext** → Defines how a coroutine behaves.

Understanding how these three work together is essential for writing safe and maintainable asynchronous code.

---

# 1. CoroutineScope

A `CoroutineScope` is responsible for creating and managing coroutines.

Every coroutine launched using `launch` or `async` belongs to a scope.

```kotlin
val scope = CoroutineScope(Job() + Dispatchers.Main)
```

### What does a scope do?

* Keeps track of all coroutines launched within it.
* Allows cancellation of all child coroutines.
* Defines the lifecycle boundary of coroutines.

```kotlin
val scope = CoroutineScope(Job())

scope.launch {
    println("Coroutine 1")
}

scope.launch {
    println("Coroutine 2")
}

scope.cancel()
```

When `scope.cancel()` is called:

```text
Coroutine 1 -> Cancelled
Coroutine 2 -> Cancelled
```

All child coroutines stop automatically.

---

## Real-World Example

### Android ViewModel

```kotlin
class UserViewModel : ViewModel() {

    fun loadUser() {
        viewModelScope.launch {
            repository.fetchUser()
        }
    }
}
```

`viewModelScope` is automatically cancelled when the ViewModel is cleared.

---

# 2. Job

A `Job` represents a coroutine and controls its lifecycle.

Every coroutine created using `launch` or `async` gets its own Job.

```kotlin
val job = scope.launch {
    delay(5000)
    println("Finished")
}
```

The returned Job can be used to:

```kotlin
job.cancel()
job.join()
job.isActive
job.isCompleted
job.isCancelled
```

---

## Job Example

```kotlin
val job = scope.launch {
    repeat(10) {
        delay(1000)
        println(it)
    }
}

delay(3000)

job.cancel()
```

Output:

```text
0
1
2
Cancelled
```

The coroutine stops after cancellation.

---

# Job Lifecycle

A Job moves through several states.

```text
New
 ↓
Active
 ↓
Completing
 ↓
Completed
```

Or when cancelled:

```text
Active
 ↓
Cancelling
 ↓
Cancelled
```

---

## Job State Properties

Instead of checking states directly, Kotlin provides:

```kotlin
job.isActive
job.isCompleted
job.isCancelled
```

### Active

```text
isActive = true
isCompleted = false
isCancelled = false
```

### Cancelling

```text
isActive = false
isCompleted = false
isCancelled = true
```

### Cancelled

```text
isActive = false
isCompleted = true
isCancelled = true
```

### Completed Normally

```text
isActive = false
isCompleted = true
isCancelled = false
```

---

# 3. CoroutineContext

A CoroutineContext defines how a coroutine behaves.

It is a collection of elements.

```kotlin
CoroutineScope(
    Job() +
    Dispatchers.Main +
    CoroutineName("UserLoader")
)
```

---

## Main Elements

### Job

Controls lifecycle and cancellation.

```kotlin
Job()
```

---

### Dispatcher

Determines which thread executes the coroutine.

```kotlin
Dispatchers.Main
Dispatchers.IO
Dispatchers.Default
```

Example:

```kotlin
launch(Dispatchers.IO) {
    // Database or Network work
}
```

---

### CoroutineName

Useful for debugging.

```kotlin
launch(CoroutineName("FetchUser")) {
    // work
}
```

---

### CoroutineExceptionHandler

Handles uncaught exceptions.

```kotlin
val handler = CoroutineExceptionHandler { _, exception ->
    println(exception.message)
}
```

---

# Parent CoroutineContext Explained

Think of `CoroutineContext` as a **bag of information** attached to a coroutine.

It can contain:

* `CoroutineDispatcher` (`Dispatchers.IO`, `Dispatchers.Main`, etc.)
* `Job`
* `CoroutineName`
* `CoroutineExceptionHandler`

Every coroutine carries this information, which determines **where it runs**, **how it is cancelled**, **what it is called**, and **how exceptions are handled**.

---

## How Kotlin Calculates a Parent CoroutineContext

When a coroutine is created, Kotlin first calculates its **Parent CoroutineContext**.

Formula:

```kotlin
Parent CoroutineContext =
Defaults + Inherited CoroutineContext + Arguments
```

Where:

### Defaults

Kotlin provides default values when certain context elements are missing.

Examples:

```kotlin
Dispatchers.Default
CoroutineName("coroutine")
```

---

### Inherited CoroutineContext

The coroutine inherits context elements from its parent:

* Parent `CoroutineScope`
* Parent coroutine

---

### Arguments

Any context elements passed to the coroutine builder (`launch`, `async`, etc.) override inherited elements.

Example:

```kotlin
launch(Dispatchers.IO) {

}
```

The provided dispatcher overrides the inherited dispatcher.

---

# Step-by-Step Example

Suppose we create a scope:

```kotlin
val scope = CoroutineScope(
    Dispatchers.Main +
    CoroutineName("MyScope") +
    Job()
)
```

The scope contains:

```text
Dispatcher     -> Main
CoroutineName  -> MyScope
Job            -> scopeJob
```

---

## Creating a Coroutine

```kotlin
scope.launch(Dispatchers.IO) {

}
```

Let's calculate its context.

---

### Step 1: Start With Defaults

Kotlin internally starts with default values.

```text
Dispatcher     -> Default
CoroutineName  -> coroutine
```

Since the scope already provides values, these defaults will be replaced.

---

### Step 2: Inherit Parent Scope Context

The coroutine inherits:

```text
Dispatchers.Main
CoroutineName("MyScope")
Job(scopeJob)
```

Current parent context:

```text
Main Dispatcher
MyScope
scopeJob
```

---

### Step 3: Apply launch() Arguments

We passed:

```kotlin
Dispatchers.IO
```

Context elements of the same type are replaced.

```kotlin
Dispatchers.Main + Dispatchers.IO
```

Result:

```kotlin
Dispatchers.IO
```

Therefore:

```text
IO Dispatcher
MyScope
scopeJob
```

---

# Final Parent CoroutineContext

The calculated parent context becomes:

```kotlin
Dispatchers.IO +
CoroutineName("MyScope") +
Job(scopeJob)
```

Visual representation:

```text
[IO Dispatcher]
[CoroutineName = MyScope]
[Parent Job = scopeJob]
```

---

# Actual CoroutineContext

After calculating the parent context, Kotlin creates a **new Job** for the new coroutine.

Formula:

```kotlin
Actual CoroutineContext =
Parent CoroutineContext + New Job()
```

Therefore:

```kotlin
Dispatchers.IO +
CoroutineName("MyScope") +
childJob
```

Visual representation:

```text
[IO Dispatcher]
[CoroutineName = MyScope]
[Child Job]
```

---

# Parent Job vs Child Job

Important:

The child coroutine NEVER uses the same Job instance as its parent.

Instead:

```text
scopeJob
    │
    └── childJob
```

The child Job is linked to the parent Job.

This relationship creates the coroutine hierarchy.

---

# Why Does Kotlin Create a New Job?

Every coroutine requires its own:

* Lifecycle
* Cancellation state
* Completion state
* Error state

Example:

```kotlin
val job1 = scope.launch {

}

val job2 = scope.launch {

}
```

Hierarchy:

```text
scopeJob
   ├── job1
   └── job2
```

Each coroutine must be managed independently.

---

# Context Overriding Example

Contexts are combined using the `+` operator.

```kotlin
val context1 =
    Dispatchers.Main +
    CoroutineName("A")

val context2 =
    Dispatchers.IO

val result = context1 + context2
```

Result:

```kotlin
Dispatchers.IO +
CoroutineName("A")
```

Explanation:

* `Dispatchers.IO` replaces `Dispatchers.Main`
* `CoroutineName("A")` remains unchanged

Visual:

```text
Before:
[Main] + [Name=A]

Add:
[IO]

Result:
[IO] + [Name=A]
```

---

# Context Inheritance Rules

When a child coroutine is created:

| Context Element  | Inherited? |
| ---------------- | ---------- |
| Dispatcher       | ✅ Yes      |
| CoroutineName    | ✅ Yes      |
| ExceptionHandler | ✅ Yes      |
| Job Instance     | ❌ No       |

A new Job is always created.

---

# Structured Concurrency Connection

Because every child Job is linked to a parent Job:

```text
Parent Job
    │
    ├── Child Job A
    ├── Child Job B
    └── Child Job C
```

Cancelling the parent:

```kotlin
scope.cancel()
```

Cancels all children automatically.

```text
Parent Cancelled
       ↓
All Child Jobs Cancelled
```

This behavior is the foundation of **Structured Concurrency**.

---

# Interview Answer

A coroutine inherits its parent's `CoroutineContext`, but context elements passed to the coroutine builder override inherited elements. Kotlin then creates a new `Job` for the coroutine and links it to the parent job. This creates a parent-child hierarchy that enables structured concurrency, lifecycle management, and cancellation propagation.

---

# Coroutine Hierarchy

Coroutines create a parent-child relationship.

Example:

```kotlin
val scope = CoroutineScope(Job())

scope.launch {

    async {

    }.await()
}
```

Hierarchy:

```text
CoroutineScope
    │
    └── launch
            │
            └── async
```

---

# Nested Coroutine Example

```kotlin
scope.launch {

    launch {

        launch {

        }

    }

}
```

Hierarchy:

```text
Scope
 │
 └── Launch A
      │
      └── Launch B
           │
           └── Launch C
```

Each coroutine becomes the parent of the coroutine it creates.

---

# Structured Concurrency

One of Kotlin Coroutines' biggest features.

If a parent is cancelled:

```kotlin
scope.cancel()
```

Every child coroutine is automatically cancelled.

Example:

```kotlin
val scope = CoroutineScope(Job())

scope.launch {

    launch {

        launch {

            delay(10000)

        }

    }

}

scope.cancel()
```

Result:

```text
Scope Cancelled
    ↓
Launch A Cancelled
    ↓
Launch B Cancelled
    ↓
Launch C Cancelled
```

This prevents memory leaks and orphaned tasks.

---

# Parent vs Child Job

Important rule:

```text
Every coroutine gets a NEW Job
```

Even though context is inherited:

```text
Dispatcher -> inherited
Name -> inherited
ExceptionHandler -> inherited
Job -> NEW INSTANCE
```

Example:

```kotlin
scope.launch {

}
```

```text
Scope Job  = Job#1
Child Job  = Job#2
```

The child Job references the parent Job but is not the same object.

---

# SupervisorJob

Normally:

```text
Child Failure
     ↓
Parent Cancels
     ↓
All Siblings Cancel
```

With SupervisorJob:

```text
Child Failure
     ↓
Only Failed Child Cancels
```

Example:

```kotlin
val scope = CoroutineScope(
    SupervisorJob() +
    Dispatchers.Main
)
```

Useful when one child failing should not cancel other children.

Commonly used in:

* ViewModels
* UI Layers
* Independent background tasks

---

# Interview Questions

### What is CoroutineScope?

A lifecycle-aware container that creates and manages coroutines.

---

### What is Job?

A handle to a coroutine that controls its lifecycle and cancellation.

---

### What is CoroutineContext?

A set of elements that define coroutine behavior such as Job, Dispatcher, Name, and ExceptionHandler.

---

### Does a child coroutine inherit its parent's Job?

No.

A child receives a new Job instance linked to the parent Job.

---

### What gets inherited from the parent context?

* Dispatcher
* CoroutineName
* ExceptionHandler

But not the same Job instance.

---

### Why is Structured Concurrency important?

It guarantees that child coroutines are automatically managed and cancelled with their parent, preventing leaks and orphaned tasks.

---

# Quick Revision

```text
CoroutineScope
    ↓
Creates Coroutines
    ↓
Each Coroutine Gets
    ↓
New Job
    +
Inherited Context
    ↓
Forms Parent-Child Hierarchy
    ↓
Cancellation Propagates Downward
    ↓
Structured Concurrency
```
