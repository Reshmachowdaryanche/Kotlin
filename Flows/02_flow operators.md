# Kotlin Flow Operators

Flow operators let you **modify, combine, transform, or consume** data emitted by a `Flow`.

Just like collections (`List`, `Set`) have functions like `map()` and `filter()`, `Flow` also has similar operators—but they work **asynchronously**.

Example:

```kotlin
flowOf(1, 2, 3, 4)
    .filter { it % 2 == 0 }
    .map { it * 10 }
    .collect {
        println(it)
    }
```

Output

```
20
40
```

---

# Types of Flow Operators

There are three major categories:

```
Flow Operators
│
├── Transformation
│     ├── map
│     ├── filter
│     ├── transform
│     ├── flatMapConcat
│     ├── flatMapLatest
│     └── flatMapMerge
│
├── Combination
│     ├── zip
│     └── combine
│
└── Terminal
      ├── collect
      ├── first
      ├── last
      ├── single
      └── toList
```

---

# Transformation Operators

These operators **change the emitted values**.

```
Flow
 ↓
Operator
 ↓
New Flow
```

---

# 1. map

## Purpose

Transforms every emitted value into another value.

Syntax

```kotlin
flow.map {
    // transform
}
```

Example

```kotlin
flowOf(1, 2, 3)
    .map {
        it * 10
    }
    .collect {
        println(it)
    }
```

Output

```
10
20
30
```

### String Example

```kotlin
flowOf(
    "apple",
    "banana"
)
.map {
    it.uppercase()
}
.collect {
    println(it)
}
```

Output

```
APPLE
BANANA
```

### Real Project Example

```kotlin
data class User(
    val name: String,
    val age: Int
)

flowOf(
    User("John", 25),
    User("Alice", 30)
)
.map {
    it.name
}
.collect {
    println(it)
}
```

Output

```
John
Alice
```

---

# 2. filter

## Purpose

Removes unwanted values.

Example

```kotlin
flowOf(1,2,3,4,5,6)
    .filter {
        it % 2 == 0
    }
    .collect {
        println(it)
    }
```

Output

```
2
4
6
```

Only values satisfying the condition continue through the flow.

### Real Example

```kotlin
flowOf(
    "Android",
    "",
    "Kotlin",
    ""
)
.filter {
    it.isNotEmpty()
}
.collect {
    println(it)
}
```

Output

```
Android
Kotlin
```

---

# map + filter Together

Very common in projects.

```kotlin
flowOf(1,2,3,4,5)
    .filter {
        it > 2
    }
    .map {
        it * 100
    }
    .collect {
        println(it)
    }
```

Output

```
300
400
500
```

Pipeline visualization:

```
1 2 3 4 5
      │
      ▼
Filter >2
      │
3 4 5
      │
      ▼
Map ×100
      │
300 400 500
```

---

# 3. transform

## Purpose

More flexible than `map`.

With `map`, **one input becomes one output**.

With `transform`, **one input can produce zero, one, or many outputs**.

Example

```kotlin
flowOf(1,2,3)
    .transform {

        emit(it)

        emit(it * 10)
    }
    .collect {
        println(it)
    }
```

Output

```
1
10
2
20
3
30
```

One value generated two outputs.

### Another Example

```kotlin
flowOf(1,2,3)
    .transform {

        if(it % 2 == 0)
            emit(it)
    }
    .collect {
        println(it)
    }
```

Output

```
2
```

It behaves like `filter`, but with greater flexibility.

---

# flatMap Operators

In **Kotlin Flows**, *flatMap operators* are used when each emitted value produces **another Flow**, and you want to **flatten those inner flows into a single stream**.

This is similar to `flatMap` in collections, but for asynchronous streams.

---

# 🔷 Why “flatMap” exists in Flow

Normally:

* `map` → 1 value → 1 transformed value
* but sometimes:

  * 1 value → a whole Flow (async stream)

So you need:

> “Take a Flow of Flows and flatten it into one Flow”

---

# 🔹 Core flatMap operators in Flow

There are 3 main ones:

## 1. `flatMapConcat`

## 2. `flatMapMerge`

## 3. `flatMapLatest`

They differ in **ordering and concurrency behavior**.

---

# 🔶 1. `flatMapConcat` (sequential)

### ✔ Behavior

* Runs inner flows **one after another**
* Waits for one to finish before starting next

### Example

```kotlin
flowOf(1, 2, 3)
    .flatMapConcat { value ->
        flow {
            emit("$value-A")
            delay(100)
            emit("$value-B")
        }
    }
    .collect { println(it) }
```

### Output order

```
1-A
1-B
2-A
2-B
3-A
3-B
```

### Use when:

* Order matters
* You want strict sequencing

---

# 🔶 2. `flatMapMerge` (concurrent)

### ✔ Behavior

* Starts all inner flows **at the same time**
* Merges results as they arrive

### Example

```kotlin
flowOf(1, 2, 3)
    .flatMapMerge { value ->
        flow {
            delay(100L * (4 - value))
            emit("$value")
        }
    }
    .collect { println(it) }
```

### Output (order may vary)

```
3
2
1
```

### Use when:

* You want parallel execution
* Order does NOT matter
* You want better performance

---

# 🔶 3. `flatMapLatest` (cancels previous)

### ✔ Behavior

* Cancels the previous inner flow
* Only keeps the **latest one active**

### Example

```kotlin
flowOf(1, 2, 3)
    .onEach { delay(50) }
    .flatMapLatest { value ->
        flow {
            repeat(3) {
                delay(100)
                emit("$value-$it")
            }
        }
    }
    .collect { println(it) }
```

### Output

Only results from the **latest emission survive**, e.g.:

```
3-0
3-1
3-2
```

### Use when:

* Search suggestions
* Typing updates
* Live UI updates

---

# 🔥 Quick comparison

| Operator        | Concurrency | Order | Cancels old? | Best for           |
| --------------- | ----------- | ----- | ------------ | ------------------ |
| `flatMapConcat` | No          | Yes   | No           | Sequential tasks   |
| `flatMapMerge`  | Yes         | No    | No           | Parallel tasks     |
| `flatMapLatest` | Yes         | No    | Yes          | Latest result only |

---

# 🧠 Simple mental model

* **Concat** → queue
* **Merge** → parallel workers
* **Latest** → “only care about newest request”



---

# Combination Operators

Sometimes you need to combine multiple flows.

---

# 1. zip

Pairs values by their positions.

Example

```kotlin
val numbers = flowOf(1,2,3)

val letters = flowOf(
    "A",
    "B",
    "C"
)

numbers.zip(letters) { n, l ->
    "$n$l"
}
.collect {
    println(it)
}
```

Output

```
1A
2B
3C
```

Visualization

```
1 2 3

A B C

↓

1A
2B
3C
```

If one flow finishes earlier, `zip` also finishes.

Example

```
1 2

A B C D
```

Output

```
1A
2B
```

---

# 2. combine

Combines the **latest values** from both flows.

Example

```kotlin
val flow1 = flow {
    emit(1)
    delay(200)
    emit(2)
}

val flow2 = flow {
    delay(100)
    emit("A")
    delay(200)
    emit("B")
}

flow1.combine(flow2) { n, l ->
    "$n$l"
}
.collect {
    println(it)
}
```

Possible output

```
1A
2A
2B
```

Visualization

```
Flow1

1 -------- 2

Flow2

----- A -------- B

Combine Latest

↓

1A

↓

2A

↓

2B
```

Unlike `zip`, `combine` emits whenever **either flow emits**, using the latest value from the other flow.

### Real Project Example

Imagine a login button should be enabled only when both username and password are valid:

```kotlin
usernameFlow.combine(passwordFlow) { username, password ->
    username.isNotBlank() && password.length >= 8
}.collect { isEnabled ->
    println("Button enabled: $isEnabled")
}
```

Whenever either field changes, the combined result is recalculated.

---

# Terminal Operators

These operators **end the flow** and produce a result or perform an action.

---

# 1. collect

Receives every value.

```kotlin
flowOf(1,2,3)
    .collect {
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

# 2. first

Returns the first emitted value.

```kotlin
val value = flowOf(5,6,7).first()

println(value)
```

Output

```
5
```

It stops collecting after the first value.

---

# 3. last

Returns the last emitted value.

```kotlin
val value = flowOf(5,6,7).last()

println(value)
```

Output

```
7
```

The flow is collected until completion.

---

# 4. single

Returns the only emitted value.

```kotlin
flowOf(10)
    .single()
```

Output

```
10
```

If the flow emits:

```kotlin
flowOf(1,2)
```

Then

```kotlin
.single()
```

throws an exception because there is more than one element.

---

# 5. toList

Collects every emitted value into a `List`.

```kotlin
val list = flowOf(1,2,3)
    .toList()

println(list)
```

Output

```
[1, 2, 3]
```

This is useful when you need all emitted values at once.

---

# Complete Example

```kotlin
flowOf(1,2,3,4,5)

    .filter {
        it % 2 == 1
    }

    .map {
        it * 10
    }

    .collect {
        println(it)
    }
```

Output

```
10
30
50
```

Pipeline

```
1 2 3 4 5

↓

Filter Odd

↓

1 3 5

↓

Multiply ×10

↓

10 30 50
```

---

# Summary Table

| Operator        | Purpose                                                    | Example Output            |
| --------------- | ---------------------------------------------------------- | ------------------------- |
| `map`           | Transform each value                                       | `1 → 10`                  |
| `filter`        | Keep matching values                                       | `2, 4, 6`                 |
| `transform`     | Emit zero, one, or many values per input                   | `1 → 1, 10`               |
| `flatMapConcat` | Run inner flows sequentially                               | `1A, 1B, 2A, 2B`          |
| `flatMapLatest` | Cancel previous inner flow when a new value arrives        | `Start A, Start B, End B` |
| `flatMapMerge`  | Run inner flows concurrently                               | Interleaved results       |
| `zip`           | Pair values by index                                       | `1A, 2B, 3C`              |
| `combine`       | Combine the latest values from each flow                   | `1A, 2A, 2B`              |
| `collect`       | Consume every emitted value                                | Prints all values         |
| `first`         | Return the first value                                     | `5`                       |
| `last`          | Return the last value                                      | `7`                       |
| `single`        | Return the only value; fails if there are 0 or more than 1 | `10`                      |
| `toList`        | Collect all values into a list                             | `[1, 2, 3]`               |

## Which operators are most common in real Android projects?

You'll see these very frequently:

* **State transformation:** `map`, `filter`
* **Combining UI state:** `combine`
* **Search and autocomplete:** `flatMapLatest`
* **Collecting UI updates:** `collect`, `collectLatest`
* **Testing flows:** `first`, `toList`

The others (`transform`, `flatMapConcat`, `flatMapMerge`, `zip`, `last`, `single`) are useful in specific scenarios but appear less often in day-to-day Android application code.
