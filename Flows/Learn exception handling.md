# Exception Handling in Kotlin Flow

In real-world applications, flows often perform operations that can fail, such as:

* Network requests
* Database queries
* File operations
* Parsing JSON
* Authentication

Without proper exception handling, an exception will **cancel the flow** and stop all further emissions.

Kotlin Flow provides operators to handle these situations gracefully:

* `catch`
* `retry`
* `retryWhen`
* `onCompletion`

---

# What Happens Without Exception Handling?

Consider this flow:

```kotlin
val numbers = flow {

    emit(1)

    emit(2)

    throw Exception("Something went wrong")

    emit(3)
}
```

Collecting

```kotlin
numbers.collect {
    println(it)
}
```

Output

```text
1
2

Exception: Something went wrong
```

Notice

```text
3
```

is never emitted because the exception immediately terminates the flow.

---

# Flow Execution

```text
Flow

↓

1

↓

2

↓

Exception

↓

Flow Cancelled
```

---

# Exception Propagation

Exceptions travel **downstream** through the flow pipeline.

Example

```kotlin
flow {

    emit(1)

    throw Exception()

}
.map {

    it * 10

}
.collect {

    println(it)

}
```

The exception reaches the collector unless handled.

---

# catch

## Purpose

`catch` intercepts exceptions that occur **upstream** (before the `catch` operator).

Syntax

```kotlin
flow
    .catch {

    }
```

---

# Basic Example

```kotlin
flow {

    emit(1)

    throw Exception("Network Error")

}
.catch {

    println("Caught: ${it.message}")

}
.collect {

    println(it)

}
```

Output

```text
1

Caught: Network Error
```

The application doesn't crash because the exception is handled.

---

# Emitting a Fallback Value

One of the most common uses of `catch` is to emit fallback data.

Example

```kotlin
flow {

    emit(apiCall())

}
.catch {

    emit("Error")

}
.collect {

    println(it)

}
```

If `apiCall()` succeeds

```text
User Data
```

If it fails

```text
Error
```

The collector still receives a value.

---

# Real Android Example

Repository

```kotlin
fun getUsers() = flow {

    emit(api.getUsers())

}
.catch {

    emit(emptyList())

}
```

UI

```kotlin
viewModel.users.collect {

    adapter.submitList(it)

}
```

If the API fails,

the UI receives

```kotlin
emptyList()
```

instead of crashing.

---

# catch Only Handles Upstream Exceptions

This is one of the most important Flow concepts.

Example

```kotlin
flow {

    emit(1)

}
.map {

    it * 10

}
.catch {

    println("Caught")

}
.collect {

    error("Collector Error")

}
```

Output

```text
10

Collector Error
```

Why?

Because

```text
collect()
```

is **downstream** of `catch`.

`catch` cannot catch exceptions thrown inside `collect`.

---

## Visualization

```text
flow

↓

map

↓

catch

↓

collect
```

`catch` handles exceptions from

* `flow`
* `map`

but **not** from `collect`.

---

# retry

## Purpose

Automatically retries the flow if it throws an exception.

Syntax

```kotlin
.retry(3)
```

means

```text
Try

↓

Fail

↓

Retry

↓

Retry

↓

Retry

↓

Give Up
```

---

# Example

```kotlin
var count = 0

flow {

    count++

    println("Attempt $count")

    if (count < 3)
        throw Exception()

    emit("Success")

}
.retry(2)

.collect {

    println(it)

}
```

Output

```text
Attempt 1

Attempt 2

Attempt 3

Success
```

The flow succeeds on the third attempt.

---

# retry Count

```kotlin
.retry(5)
```

means

```text
Original Attempt

+

5 Retries

=

Maximum 6 Attempts
```

---

# retry Example with API

```kotlin
flow {

    emit(api.getUsers())

}
.retry(3)
```

If the network is temporarily unavailable,

Flow automatically retries up to three times.

---

# retryWhen

`retry()` always retries based on a fixed count.

`retryWhen()` lets you decide **when** to retry.

Syntax

```kotlin
.retryWhen { cause, attempt ->

}
```

Parameters

```text
cause

↓

Exception
```

```text
attempt

↓

Retry Number
```

---

# Example

```kotlin
flow {

    emit(apiCall())

}
.retryWhen { cause, attempt ->

    cause is IOException && attempt < 3

}
.collect {

    println(it)

}
```

Meaning

```text
Network Error?

↓

Yes

↓

Retry

↓

Maximum 3 retries
```

If it's another type of exception,

don't retry.

---

# Example with Delay

```kotlin
flow {

    emit(apiCall())

}
.retryWhen { cause, attempt ->

    delay(1000)

    attempt < 3

}
.collect {

}
```

Timeline

```text
Attempt

↓

Wait 1 Second

↓

Retry

↓

Wait 1 Second

↓

Retry
```

This is useful for temporary network failures.

---

# retry vs retryWhen

| retry                  | retryWhen                             |
| ---------------------- | ------------------------------------- |
| Fixed retry count      | Custom retry logic                    |
| Simple                 | Flexible                              |
| No access to exception | Access to exception and retry attempt |
| Good for simple cases  | Good for production apps              |

---

# onCompletion

## Purpose

Runs when the flow completes or is cancelled.

Think of it like

```text
finally
```

in a `try-catch-finally` block.

---

# Example

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

```text
1

2

Flow Finished
```

---

# onCompletion with Error

```kotlin
flow {

    emit(1)

    throw Exception()

}
.onCompletion {

    println("Completed")

}
.catch {

    println("Caught")

}
.collect {

}
```

Output

```text
Completed

Caught
```

`onCompletion` still executes.

---

# Completion Cause

`onCompletion` receives a parameter called `cause`.

Example

```kotlin
flow {

    emit(1)

}
.onCompletion { cause ->

    if(cause == null)

        println("Success")

    else

        println("Failed")

}
.collect()
```

If everything succeeds

```text
Success
```

If cancelled

```text
Failed
```

If an exception occurs

```text
Failed
```

The `cause` contains the exception (or a `CancellationException` if the flow was cancelled), allowing you to distinguish why the flow ended.

---

# Combining Operators

Very common in production.

```kotlin
flow {

    emit(api.getUsers())

}
.retry(3)

.catch {

    emit(emptyList())

}

.onCompletion {

    println("Done")

}

.collect {

    println(it)

}
```

Execution

```text
Flow

↓

API

↓

Retry

↓

Catch

↓

Completion

↓

Collect
```

---

# Real Project Example

Repository

```kotlin
fun users() = flow {

    emit(api.getUsers())

}
.retryWhen { cause, attempt ->

    cause is IOException &&
    attempt < 3

}
.catch {

    emit(emptyList())

}
```

ViewModel

```kotlin
viewModelScope.launch {

    repository.users()

        .collect {

            _state.value = it

        }

}
```

Behavior

```text
Network Error

↓

Retry

↓

Retry

↓

Retry

↓

Still Failed

↓

Emit Empty List

↓

UI Updates
```

The app continues working without crashing.

---

# Order Matters

Consider this example:

```kotlin
flow {

    emit(1)

    throw Exception()

}
.catch {

    println("Caught")

}
.map {

    it * 10

}
.collect {

    println(it)

}
```

`catch` only handles exceptions **before** it.

If `map` throws an exception instead:

```kotlin
flow {

    emit(1)

}
.map {

    throw Exception("Map Error")

}
.catch {

    println("Caught: ${it.message}")

}
.collect()
```

Output

```text
Caught: Map Error
```

But if `catch` is placed before `map`:

```kotlin
flow {

    emit(1)

}
.catch {

    println("Caught")

}
.map {

    throw Exception("Map Error")

}
.collect()
```

The exception is **not** caught because it occurs downstream of `catch`.

---

# Visual Explanation

```text
flow

↓

map

↓

retry

↓

catch

↓

onCompletion

↓

collect
```

### Exceptions thrown in

```text
flow

map
```

are handled by

```text
retry

↓

catch
```

Exceptions thrown in

```text
collect
```

are **not** handled by `catch`.

---

# Complete Example

```kotlin
fun apiCall(): String {

    if ((1..2).random() == 1)
        throw IOException("No Internet")

    return "Success"
}

fun main() = runBlocking {

    flow {

        emit(apiCall())

    }
    .retryWhen { cause, attempt ->

        println("Retry: ${attempt + 1}")

        cause is IOException && attempt < 2

    }
    .catch {

        emit("Fallback Data")

    }
    .onCompletion { cause ->

        if (cause == null)
            println("Flow Completed Successfully")
        else
            println("Flow Completed with Error: ${cause.message}")
    }
    .collect {

        println(it)

    }

}
```

Possible output (if all retries fail)

```text
Retry: 1
Retry: 2
Retry: 3
Fallback Data
Flow Completed Successfully
```

**Why does `onCompletion` report success here?** Because `catch` handled the exception and emitted fallback data, so the flow completed normally. If the exception were not caught, `cause` would contain that exception.

---

# Summary

| Operator       | Purpose                                                                               | Common Use                                              |
| -------------- | ------------------------------------------------------------------------------------- | ------------------------------------------------------- |
| `catch`        | Handles upstream exceptions and can emit fallback values                              | Return cached or default data when an API fails         |
| `retry`        | Retries a failed flow a fixed number of times                                         | Retry transient network failures                        |
| `retryWhen`    | Retries based on custom conditions (exception type, attempt count, delays, etc.)      | Retry only `IOException`, implement exponential backoff |
| `onCompletion` | Executes when the flow completes or is cancelled, with access to the completion cause | Cleanup, logging, hide loading indicators               |

## Best Practices

* Use **`catch`** to convert failures into user-friendly fallback values.
* Use **`retryWhen`** instead of `retry` when you need conditions or delays between retries.
* Place **`catch` after the operators whose exceptions you want to handle**.
* Use **`onCompletion`** for cleanup or final actions, not for handling errors.
* Remember that **exceptions thrown inside `collect` are not caught by `catch`**; wrap `collect` in a `try-catch` if you need to handle those.
