# Kotlin Coroutines Exceptions & SupervisorJob – Interview Notes

> These notes cover **Exception Propagation, Job vs SupervisorJob, launch vs async exception handling, CoroutineExceptionHandler, and common interview questions**.

---

# 1. Why Exception Handling Matters

In real applications:

* Network requests can fail
* Database operations can fail
* API responses may be invalid
* User input may be incorrect

A good application should:

✅ Avoid crashes

✅ Show meaningful error messages

✅ Recover gracefully

✅ Keep unaffected parts of the app running

---

# 2. What Happens When a Coroutine Fails?

When a coroutine throws an exception:

```kotlin
throw Exception("Something went wrong")
```

The exception is propagated to its parent.

---

## Exception Propagation Flow

```text
Child Coroutine
        ↓
Parent Coroutine
        ↓
Parent Scope
        ↓
Root Coroutine
        ↓
Application
```

---

# Default Behavior with Job

Suppose we have:

```kotlin
val scope = CoroutineScope(Job())

scope.launch {
    launch {
        throw Exception("Failure")
    }
}
```

---

## What Happens?

### Step 1

Child throws exception.

```text
Child failed
```

### Step 2

Parent receives exception.

```text
Parent cancelled
```

### Step 3

All siblings cancelled.

```text
Sibling coroutines cancelled
```

### Step 4

Exception propagated upward.

```text
Scope cancelled
```

### Step 5

Root receives exception.

```text
Application crash (if uncaught)
```

---

# Interview Definition

### What happens when a coroutine throws an exception?

Answer:

> The exception is propagated to the parent. The parent cancels its children, cancels itself, and propagates the exception further up the coroutine hierarchy.

---

# 3. Why This Can Be a Problem?

Imagine:

```text
UI Scope
 ├── Load User
 ├── Load Posts
 └── Load Notifications
```

If:

```text
Load User fails
```

Then:

```text
Load Posts cancelled
Load Notifications cancelled
UI Scope cancelled
```

Entire UI becomes unusable.

---

# 4. SupervisorJob

SupervisorJob changes this behavior.

---

## Definition

A SupervisorJob allows child coroutines to fail independently.

Failure of one child:

❌ Does NOT cancel siblings

❌ Does NOT cancel parent

❌ Does NOT propagate automatically

---

## Example

```kotlin
val scope = CoroutineScope(
    SupervisorJob()
)

scope.launch {
    // Child 1
}

scope.launch {
    // Child 2
}
```

---

## If Child 1 Fails

```text
Child 1 → Failed
Child 2 → Running
Scope   → Running
```

---

## Hierarchy

```text
SupervisorJob
    ├── Child 1 (Failed)
    └── Child 2 (Running)
```

No cancellation propagation.

---

# Interview Definition

### What is SupervisorJob?

Answer:

> SupervisorJob allows child coroutines to fail independently without cancelling sibling coroutines or the parent scope.

---

# 5. Job vs SupervisorJob

| Feature                        | Job   | SupervisorJob |
| ------------------------------ | ----- | ------------- |
| Child failure cancels parent   | ✅ Yes | ❌ No          |
| Child failure cancels siblings | ✅ Yes | ❌ No          |
| Exception propagates upward    | ✅ Yes | ❌ No          |
| Independent child failures     | ❌ No  | ✅ Yes         |

---

# Visual Comparison

## Job

```text
Job
 ├── Child1 ❌
 ├── Child2 ❌
 └── Child3 ❌
```

Everything cancelled.

---

## SupervisorJob

```text
SupervisorJob
 ├── Child1 ❌
 ├── Child2 ✅
 └── Child3 ✅
```

Only failing child stops.

---

# 6. Creating a Supervisor Scope

## Method 1

```kotlin
val scope =
    CoroutineScope(SupervisorJob())
```

---

## Method 2

```kotlin
supervisorScope {

}
```

Creates temporary supervisor hierarchy.

---

# Example

```kotlin
supervisorScope {

    launch {
        throw Exception()
    }

    launch {
        println("Still running")
    }
}
```

Output:

```text
Still running
```

Second coroutine survives.

---

# 7. coroutineScope vs supervisorScope

---

## coroutineScope

```kotlin
coroutineScope {

    launch {
        throw Exception()
    }

    launch {
        println("Won't execute")
    }
}
```

Result:

```text
All children cancelled
```

---

## supervisorScope

```kotlin
supervisorScope {

    launch {
        throw Exception()
    }

    launch {
        println("Runs normally")
    }
}
```

Result:

```text
Only failing child cancelled
```

---

# Interview Question

### Difference between coroutineScope and supervisorScope?

Answer:

| coroutineScope     | supervisorScope         |
| ------------------ | ----------------------- |
| Uses Job           | Uses SupervisorJob      |
| Failure propagates | Failure isolated        |
| Cancels siblings   | Doesn't cancel siblings |

---

# 8. Common Interview Trap

Consider:

```kotlin
val scope = CoroutineScope(Job())

scope.launch(
    SupervisorJob()
) {

    launch {
        // Child 1
    }

    launch {
        // Child 2
    }
}
```

---

## Question

What is parent of Child 1?

Many developers answer:

```text
SupervisorJob
```

Wrong ❌

---

## Correct Answer

Parent is:

```text
Job
```

---

### Why?

Every coroutine builder creates a new Job.

```kotlin
launch {}
```

creates:

```text
New Job
```

Therefore:

```text
SupervisorJob
        ↓
launch coroutine
        ↓
Job
     /     \
Child1    Child2
```

Child1 and Child2 belong to Job.

SupervisorJob does not protect them.

---

# Interview Question

### Why doesn't passing SupervisorJob to launch work?

Answer:

> Because launch creates its own Job. Supervisor behavior works only when SupervisorJob is part of a CoroutineScope or created using supervisorScope.

---

# 9. Under the Hood

Normal Job:

```text
Child fails
      ↓
Parent notified
      ↓
Parent cancelled
```

---

SupervisorJob:

```text
Child fails
      ↓
Parent notified
      ↓
Ignore cancellation
```

Internally:

```kotlin
childCancelled()
```

returns:

```kotlin
false
```

meaning:

```text
Don't cancel me
```

---

# 10. Handling Exceptions

Coroutines use standard Kotlin exception handling.

---

## try/catch

```kotlin
try {

} catch(e: Exception) {

}
```

---

## runCatching

```kotlin
runCatching {
    riskyCode()
}
```

---

# 11. Exception Handling in launch

---

## Example

```kotlin
scope.launch {

    try {

        riskyCode()

    } catch (e: Exception) {

        println("Handled")
    }
}
```

---

## Behavior

launch throws exceptions immediately.

```text
Exception occurs
        ↓
catch executes immediately
```

---

# Interview Question

### When are exceptions thrown in launch?

Answer:

> Immediately when they occur.

---

# 12. Exception Handling in async

---

## Example

```kotlin
val deferred = async {

    riskyCode()
}
```

No exception thrown yet.

---

## Exception Appears Here

```kotlin
deferred.await()
```

---

Example:

```kotlin
try {

    deferred.await()

} catch(e: Exception) {

}
```

---

# Behavior

```text
async starts
      ↓
Exception stored internally
      ↓
await() called
      ↓
Exception thrown
```

---

# Interview Question

### When are exceptions thrown in async?

Answer:

> Exceptions are deferred and thrown when await() is called.

---

# launch vs async

| launch             | async            |
| ------------------ | ---------------- |
| Throws immediately | Throws on await  |
| Returns Job        | Returns Deferred |
| Fire-and-forget    | Returns result   |

---

# 13. Important Exception Rule

Root async:

```kotlin
supervisorScope {

    val deferred = async {
        throw Exception()
    }

    deferred.await()
}
```

Exception handled by await.

---

Nested async:

```kotlin
scope.launch {

    async {
        throw Exception()
    }
}
```

---

Result:

```text
launch receives exception immediately
```

Even without:

```kotlin
await()
```

---

Why?

Because async propagates exception to its parent.

Parent is launch.

launch throws immediately.

---

# Interview Question

### Can async crash without await()?

Answer:

> Yes, if async is not a root coroutine and its parent propagates the exception.

---

# 14. CoroutineExceptionHandler

---

## Definition

A special handler for uncaught coroutine exceptions.

---

## Creation

```kotlin
val handler =
    CoroutineExceptionHandler {

        context,
        exception ->

        println(exception)
    }
```

---

# Usage

```kotlin
scope.launch(handler) {

    throw Exception("Failed")
}
```

Output:

```text
Caught Failed
```

---

# When Does It Work?

Two conditions:

---

## Condition 1

Coroutine must automatically throw exceptions.

Works with:

```kotlin
launch
```

---

Does NOT work with:

```kotlin
async
```

because async stores exceptions.

---

## Condition 2

Must be installed on:

```text
Root coroutine
OR
CoroutineScope
```

---

# Works

```kotlin
scope.launch(handler) {

}
```

---

# Doesn't Work

```kotlin
scope.launch {

    launch(handler) {

        throw Exception()
    }
}
```

---

Why?

Exception already propagates to parent.

Parent knows nothing about handler.

---

# Interview Question

### Does CoroutineExceptionHandler catch async exceptions?

Answer:

```text
No.
```

Because async exposes exceptions through await().

---

# Exception Handling Strategy

## launch

```kotlin
launch {

    try {

    } catch(e: Exception) {

    }
}
```

---

## async

```kotlin
val deferred = async {

}

try {

    deferred.await()

} catch(e: Exception) {

}
```

---

## Global Fallback

```kotlin
CoroutineExceptionHandler
```

---

# Real Android Usage

---

## ViewModel Scope

```kotlin
viewModelScope.launch {

    try {

        repository.getUsers()

    } catch(e: Exception) {

        _uiState.value = Error
    }
}
```

---

## Independent UI Tasks

```kotlin
val scope =
    CoroutineScope(
        SupervisorJob() +
        Dispatchers.Main
    )
```

If one request fails:

```text
Load User ❌
Load Posts ✅
Load Notifications ✅
```

---

# Common Interview Questions

---

## Q1. What happens when a child coroutine throws an exception?

```text
Exception propagates to parent,
parent cancels children,
cancels itself,
propagates upward.
```

---

## Q2. What is SupervisorJob?

```text
A Job that isolates child failures.
```

---

## Q3. Difference between Job and SupervisorJob?

```text
Job → Failure propagates.

SupervisorJob → Failure isolated.
```

---

## Q4. Difference between coroutineScope and supervisorScope?

```text
coroutineScope → Job

supervisorScope → SupervisorJob
```

---

## Q5. When does launch throw exceptions?

```text
Immediately.
```

---

## Q6. When does async throw exceptions?

```text
When await() is called.
```

---

## Q7. Does CoroutineExceptionHandler work with async?

```text
No.
```

---

## Q8. Can one child failure cancel all siblings?

```text
Yes with Job.

No with SupervisorJob.
```

---

# Quick Revision Sheet

```text
Exception
    ↓
Parent
    ↓
Scope
    ↓
Root
```

### Isolation

```kotlin
SupervisorJob()
supervisorScope {}
```

### Exception Handling

```kotlin
try/catch
runCatching
CoroutineExceptionHandler
```

### launch

```kotlin
launch {
}
```

Exception:

```text
Thrown immediately
```

### async

```kotlin
async {
}
```

Exception:

```text
Thrown on await()
```

### Scope Types

```kotlin
Job
→ Failure propagates

SupervisorJob
→ Failure isolated
```

### Golden Interview One-Liner

> In Kotlin Coroutines, exceptions normally propagate up the coroutine hierarchy and cancel parent and sibling coroutines. Using `SupervisorJob` or `supervisorScope` isolates failures, while `launch` throws exceptions immediately and `async` defers them until `await()` is called.
