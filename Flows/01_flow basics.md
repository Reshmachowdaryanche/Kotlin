Here's a structured guide to **Kotlin Flow** covering all the topics you listed.

# Kotlin Flow

**Flow** is a coroutine-based API used to handle **asynchronous streams of data**. It emits multiple values sequentially over time instead of returning a single value.

Unlike `suspend` functions that return only one result, a `Flow` can emit many values.

```kotlin
val numbers = flow {
    emit(1)
    emit(2)
    emit(3)
}
```

The values are not produced until someone starts collecting them.

---

# What is a Flow?

A **Flow** is:

* A stream of values.
* Built on Kotlin Coroutines.
* Asynchronous.
* Cold by default.
* Can emit zero, one, or many values.

Think of it like a sequence whose values arrive over time.

Example:

```kotlin
val flow = flow {
    emit("Apple")
    delay(1000)
    emit("Banana")
    delay(1000)
    emit("Orange")
}
```

Nothing happens until it is collected.

---

# Cold Streams vs Hot Streams

## Cold Flow

A cold flow starts producing values **only when collected**.

Every collector gets its own fresh execution.

```kotlin
val numbers = flow {
    println("Flow started")
    emit(1)
    emit(2)
}

numbers.collect()
numbers.collect()
```

Output:

```
Flow started
1
2

Flow started
1
2
```

The flow executes twice because each collector starts it from the beginning.

### Characteristics

* Starts on collect
* Each collector gets independent data
* No work is done if nobody collects

---

## Hot Flow

Hot streams continue producing values **even if nobody is collecting**.

Examples:

* StateFlow
* SharedFlow

```kotlin
val state = MutableStateFlow(0)

state.value = 10
state.value = 20
```

Collectors receive the current value immediately.

### Characteristics

* Always active
* Shared among collectors
* Doesn't restart for each collector

---

## Comparison

| Cold Flow              | Hot Flow                  |
| ---------------------- | ------------------------- |
| Starts when collected  | Always active             |
| Independent collectors | Shared collectors         |
| Re-executes each time  | Doesn't restart           |
| `Flow`                 | `StateFlow`, `SharedFlow` |

---

# Flow Lifecycle

A flow follows this lifecycle:

```
Create Flow
      ↓
Collect Flow
      ↓
Start Emitting
      ↓
Collector Receives Values
      ↓
Complete or Cancel
```

Example:

```kotlin
val numbers = flow {
    println("Started")

    emit(1)
    emit(2)
    emit(3)

    println("Finished")
}
```

When collected:

```kotlin
numbers.collect {
    println(it)
}
```

Output:

```
Started
1
2
3
Finished
```

If no one collects it, nothing is printed.

---

# Creating Flows

There are multiple ways to create flows.

---

## 1. `flow {}`

Most common way.

```kotlin
val flow = flow {
    emit(10)
    emit(20)
    emit(30)
}
```

You can use coroutine functions inside.

```kotlin
val flow = flow {
    delay(1000)
    emit("Hello")
}
```

---

## 2. `flowOf()`

Creates a flow from fixed values.

```kotlin
val numbers = flowOf(
    1,
    2,
    3,
    4,
    5
)
```

Equivalent to

```kotlin
flow {
    emit(1)
    emit(2)
    emit(3)
    emit(4)
    emit(5)
}
```

---

## 3. `asFlow()`

Converts collections into flows.

List

```kotlin
val flow = listOf(
    1,
    2,
    3
).asFlow()
```

Range

```kotlin
val flow = (1..5).asFlow()
```

Array

```kotlin
val flow = arrayOf(
    "A",
    "B",
    "C"
).asFlow()
```

---

# Collecting Values

Flows don't emit until collected.

```kotlin
val numbers = flowOf(1, 2, 3)

numbers.collect {
    println(it)
}
```

Output

```
1
2
3
```

---

# `collect`

`collect` receives **every emitted value**.

Example

```kotlin
flowOf(
    1,
    2,
    3
).collect {

    println(it)
}
```

Output

```
1
2
3
```

With delay

```kotlin
val numbers = flow {
    for (i in 1..5) {
        delay(1000)
        emit(i)
    }
}

numbers.collect {
    println(it)
}
```

Output

```
1
2
3
4
5
```

Every value is processed completely.

---

# `collectLatest`

`collectLatest` cancels processing of the previous value if a new one arrives before processing finishes.

Example

```kotlin
flow {
    emit(1)
    delay(100)
    emit(2)
}.collectLatest {

    println("Start $it")

    delay(300)

    println("Done $it")
}
```

Output

```
Start 1
Start 2
Done 2
```

Notice

```
Done 1
```

never prints because processing of `1` is cancelled.

---

### Difference

Using `collect`

```
Start 1
Done 1
Start 2
Done 2
```

Using `collectLatest`

```
Start 1
Start 2
Done 2
```

---

# `launchIn`

Instead of manually calling `collect`, you can launch collection inside a `CoroutineScope`.

Example

```kotlin
flowOf(
    1,
    2,
    3
)
.onEach {
    println(it)
}
.launchIn(scope)
```

Equivalent to

```kotlin
scope.launch {
    flow.collect {
        println(it)
    }
}
```

This is commonly used with operators like `onEach`.

Example in an Android `ViewModel`:

```kotlin
repository.dataFlow
    .onEach {
        println(it)
    }
    .launchIn(viewModelScope)
```

Advantages:

* Doesn't block the current coroutine.
* Collection is automatically tied to the provided scope.
* Commonly used with `viewModelScope` or `lifecycleScope`.

---

# Complete Example

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

fun main() = runBlocking {

    val numbers = flow {
        for (i in 1..5) {
            delay(500)
            emit(i)
        }
    }

    numbers.collect {
        println(it)
    }
}
```

Output

```
1
2
3
4
5
```

---

# Summary

| Topic             | Description                                                                               |
| ----------------- | ----------------------------------------------------------------------------------------- |
| `Flow`            | Asynchronous stream of multiple values                                                    |
| Cold Flow         | Starts only when collected; each collector gets a new execution                           |
| Hot Flow          | Emits regardless of collectors; shared among subscribers (`StateFlow`, `SharedFlow`)      |
| `flow {}`         | Builder for custom flows using `emit()`                                                   |
| `flowOf()`        | Creates a flow from a fixed set of values                                                 |
| `asFlow()`        | Converts collections, arrays, or ranges into a flow                                       |
| `collect`         | Processes every emitted value sequentially                                                |
| `collectLatest`   | Cancels processing of the previous value when a new value arrives                         |
| `launchIn(scope)` | Starts collecting a flow in a given `CoroutineScope` without explicitly calling `collect` |


---




