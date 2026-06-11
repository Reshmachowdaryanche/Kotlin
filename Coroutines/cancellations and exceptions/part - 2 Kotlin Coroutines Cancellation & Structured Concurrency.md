Below are **interview-focused Markdown notes** on **Kotlin Coroutine Cancellation & Structured Concurrency**, organized from basics to advanced concepts with explanations, examples, and common interview questions.

# Kotlin Coroutines Cancellation & Structured Concurrency

---

# 1. Why Coroutine Cancellation Matters

## Problem

In applications, tasks may become unnecessary before they finish:

* User leaves a screen
* User cancels a request
* ViewModel is destroyed
* App lifecycle changes

If coroutines continue running:

* CPU is wasted
* Memory is wasted
* Battery consumption increases
* Resources remain occupied

Therefore, coroutines must be cancellable.

---

# 2. What is Structured Concurrency?

## Definition

Structured Concurrency means:

> Every coroutine should belong to a scope, and when the scope is cancelled, all child coroutines are cancelled automatically.

This ensures:

* No orphan coroutines
* Easy lifecycle management
* Automatic cleanup

---

## Example

```kotlin
val job1 = scope.launch {
    // work
}

val job2 = scope.launch {
    // work
}

scope.cancel()
```

### What happens?

```text
scope
 ├── job1
 └── job2
```

Cancelling the scope:

```text
scope (cancelled)
 ├── job1 (cancelled)
 └── job2 (cancelled)
```

All children are cancelled.

---

# 3. Cancelling a Specific Coroutine

Sometimes we want to cancel only one coroutine.

## Example

```kotlin
val job1 = scope.launch {
    // work
}

val job2 = scope.launch {
    // work
}

job1.cancel()
```

### Result

```text
job1 → Cancelled
job2 → Running
```

Sibling coroutines are not affected.

---

# 4. What Happens Internally During Cancellation?

Coroutine cancellation works through an exception:

```kotlin
CancellationException
```

When you call:

```kotlin
job.cancel()
```

Kotlin internally throws:

```kotlin
CancellationException
```

---

## Custom Cancellation Reason

```kotlin
job.cancel(
    CancellationException("User pressed cancel")
)
```

Useful for:

* Logging
* Debugging
* Tracking cancellation causes

---

# 5. Important Interview Point

## Question

### Does `cancel()` immediately stop coroutine execution?

### Answer

No.

`cancel()` only requests cancellation.

Coroutine code must cooperate with cancellation.

---

# 6. Why My Coroutine Doesn't Stop?

Example:

```kotlin
val job = launch {

    var i = 0

    while (i < 5) {
        println("Hello $i")
        i++
    }
}

job.cancel()
```

Even after cancellation:

```text
Hello 0
Hello 1
Hello 2
Hello 3
Hello 4
```

may still print.

Reason:

The loop never checks cancellation status.

---

# 7. Coroutine Cancellation is Cooperative

## Definition

Coroutines stop only when they check whether cancellation happened.

This is called:

> Cooperative Cancellation

---

# 8. Making Coroutine Code Cancellable

There are 3 common techniques:

1. isActive
2. ensureActive()
3. yield()

---

# 9. Using isActive

## Example

```kotlin
launch {

    var i = 0

    while (i < 100 && isActive) {
        println(i++)
    }
}
```

---

## How it Works

When:

```kotlin
job.cancel()
```

Then:

```kotlin
isActive == false
```

Loop exits immediately.

---

## Advantages

* Flexible
* Can perform custom actions

Example:

```kotlin
if (!isActive) {
    println("Cancelled")
}
```

---

# 10. Using ensureActive()

## Example

```kotlin
while (i < 100) {

    ensureActive()

    println(i++)
}
```

---

## Internal Implementation

```kotlin
fun Job.ensureActive() {
    if (!isActive) {
        throw getCancellationException()
    }
}
```

---

## Benefits

Less boilerplate.

Instead of:

```kotlin
if (!isActive) {
    throw CancellationException()
}
```

you write:

```kotlin
ensureActive()
```

---

# Interview Difference

| isActive            | ensureActive()   |
| ------------------- | ---------------- |
| Returns Boolean     | Throws exception |
| Flexible            | Cleaner          |
| Allows custom logic | Immediate exit   |

---

# 11. Using yield()

## Purpose

Allows:

* Cancellation check
* Other coroutines to run

---

## Example

```kotlin
while (true) {

    yield()

    doHeavyWork()
}
```

---

## Why Use It?

Suppose:

```kotlin
Dispatchers.Default
```

Thread pool has limited threads.

Heavy CPU work may block them.

`yield()`:

* Checks cancellation
* Gives thread to another coroutine

---

# Interview Answer

### When should yield() be used?

Use it in:

* CPU intensive loops
* Long-running computations
* Tight loops without suspension points

---

# 12. Suspend Functions Are Already Cancellable

Most coroutine suspend functions already check cancellation.

Examples:

```kotlin
delay()
```

```kotlin
withContext()
```

```kotlin
await()
```

```kotlin
join()
```

No manual cancellation check required.

---

# 13. Coroutine States During Cancellation

```text
New
 ↓
Active
 ↓
Cancelling
 ↓
Cancelled
```

---

## Active

Coroutine running normally.

---

## Cancelling

Cancellation requested.

```kotlin
job.cancel()
```

Coroutine cleaning up.

---

## Cancelled

Finished completely.

---

# 14. Job.join()

## Purpose

Wait until coroutine completes.

---

## Example

```kotlin
job.cancel()

job.join()
```

---

### Meaning

```text
Cancel coroutine
Wait until cancellation finishes
Continue execution
```

---

# Interview Question

### Difference between cancel() and join()?

Answer:

```kotlin
cancel()
```

Requests cancellation.

```kotlin
join()
```

Waits until completion.

---

# 15. Deferred.await()

Used with:

```kotlin
async
```

---

## Example

```kotlin
val deferred = async {
    100
}
```

Retrieve result:

```kotlin
val result = deferred.await()
```

---

# 16. await() After Cancellation

Example:

```kotlin
val deferred = async {
    100
}

deferred.cancel()

deferred.await()
```

---

### Result

```text
JobCancellationException
```

---

## Why?

Cancelled coroutine cannot produce a result.

Therefore:

```kotlin
await()
```

throws exception.

---

# Interview Difference

| Job                  | Deferred         |
| -------------------- | ---------------- |
| launch               | async            |
| join()               | await()          |
| No return value      | Returns value    |
| Waits for completion | Waits for result |

---

# 17. Handling Cancellation Side Effects

When coroutine is cancelled, we may need:

* Close files
* Close sockets
* Release resources
* Save logs

This is called cleanup.

---

# Method 1: Using isActive

```kotlin
while (isActive) {
    work()
}

println("Cleanup")
```

---

# Method 2: try-catch-finally

Most common interview answer.

---

## Example

```kotlin
launch {

    try {

        work()

    } catch (e: CancellationException) {

        println("Cancelled")

    } finally {

        println("Cleanup")

    }
}
```

---

## Flow

```text
work()
 ↓
cancel()
 ↓
CancellationException
 ↓
catch
 ↓
finally
```

---

# 18. Problem with Suspending Cleanup

Example:

```kotlin
finally {

    delay(1000)
}
```

This fails.

Why?

Coroutine is already in:

```text
Cancelling state
```

and cannot suspend anymore.

---

# 19. NonCancellable Context

Solution:

```kotlin
finally {

    withContext(NonCancellable) {

        delay(1000)

        println("Cleanup done")
    }
}
```

---

## Why It Works

`NonCancellable` temporarily ignores cancellation.

Allows:

```kotlin
delay()
```

or other suspend functions.

---

# Interview Question

### Why use NonCancellable?

Answer:

To execute suspending cleanup code after coroutine cancellation.

---

# 20. suspendCancellableCoroutine

Used when converting callback APIs to coroutines.

---

## Bad

```kotlin
suspendCoroutine {}
```

No cancellation support.

---

## Good

```kotlin
suspendCancellableCoroutine {}
```

Supports cancellation.

---

# 21. invokeOnCancellation()

Used for cleanup when cancellation occurs.

---

## Example

```kotlin
suspend fun work() {

    suspendCancellableCoroutine<Unit> {

        continuation ->

        continuation.invokeOnCancellation {

            println("Cleanup")
        }
    }
}
```

---

# Real-world Use Cases

```kotlin
Socket.close()
```

```kotlin
Listener.remove()
```

```kotlin
File.close()
```

```kotlin
Database transaction rollback
```

---

# 22. Android Best Practices

Use lifecycle-aware scopes.

---

## ViewModel

```kotlin
viewModelScope.launch {

}
```

Automatically cancelled when:

```text
ViewModel.onCleared()
```

runs.

---

## Activity / Fragment

```kotlin
lifecycleScope.launch {

}
```

Automatically cancelled when lifecycle ends.

---

# Interview Questions & Answers

## Q1: What exception is used for coroutine cancellation?

```text
CancellationException
```

---

## Q2: Does cancel() stop execution immediately?

```text
No.
Cancellation is cooperative.
```

---

## Q3: How can you make a coroutine cancellable?

```text
1. isActive
2. ensureActive()
3. yield()
4. Use cancellable suspend functions
```

---

## Q4: Difference between join() and await()?

```text
join() → waits for completion

await() → waits for result
```

---

## Q5: What happens if await() is called after cancellation?

```text
JobCancellationException
```

---

## Q6: Why use NonCancellable?

```text
To execute suspending cleanup code inside finally block.
```

---

## Q7: Difference between suspendCoroutine and suspendCancellableCoroutine?

```text
suspendCoroutine
→ no cancellation support

suspendCancellableCoroutine
→ cancellation support
```

---

# Quick Revision Sheet

```text
cancel()
    ↓
CancellationException
    ↓
Coroutine enters Cancelling state
    ↓
Cleanup executes
    ↓
Cancelled state
```

### Cancellation Tools

```kotlin
isActive
ensureActive()
yield()
```

### Cleanup

```kotlin
try
catch(CancellationException)
finally
```

### Suspending Cleanup

```kotlin
withContext(NonCancellable)
```

### Waiting

```kotlin
job.join()
deferred.await()
```

### Lifecycle Scopes

```kotlin
viewModelScope
lifecycleScope
```

### Callback Conversion

```kotlin
suspendCancellableCoroutine
invokeOnCancellation
```

**Interview one-liner:**

> Coroutine cancellation in Kotlin is cooperative, implemented through `CancellationException`, managed by structured concurrency, and should always include proper cancellation checks and cleanup logic to avoid unnecessary work and resource leaks.
