# StateFlow and SharedFlow in Kotlin

`StateFlow` and `SharedFlow` are **hot flows** introduced in Kotlin Coroutines to share data between multiple collectors.

Many beginners get confused because both can emit values to multiple collectors. The biggest difference is:

* **StateFlow = Holds the latest state**
* **SharedFlow = Shares events/messages**

A simple analogy:

* **StateFlow** → A digital clock. Whenever you look at it, you immediately see the current time.
* **SharedFlow** → A radio station. You only hear broadcasts that happen after you start listening.

---

# Why StateFlow and SharedFlow?

Normal `Flow` is **cold**.

Example:

```kotlin
val numbers = flow {
    println("Flow Started")
    emit(1)
    emit(2)
}
```

If two collectors collect it:

```kotlin
numbers.collect { println("Collector 1: $it") }

numbers.collect { println("Collector 2: $it") }
```

Output

```
Flow Started
1
2

Flow Started
1
2
```

The flow executes twice.

Sometimes we don't want that.

Suppose you're downloading stock prices.

```
Server
   ↓
Flow
```

If 100 collectors subscribe,

```
Server
   ↓
Flow
Collector 1

Server
   ↓
Flow
Collector 2

Server
   ↓
Flow
Collector 3
```

The API is called multiple times.

Instead, we want

```
Server
    ↓
Shared Stream
   ↓ ↓ ↓ ↓
C1 C2 C3 C4
```

This is exactly what **StateFlow** and **SharedFlow** provide.

---

# Hot Flow

Both are **hot flows**.

This means

* They exist independently of collectors.
* They don't restart for each collector.
* Multiple collectors receive data from the same stream.

---

# StateFlow

## Definition

A `StateFlow` is a hot flow that **always contains one current value**.

It represents **state**.

Examples of state:

* Current user
* Current theme
* Current screen
* Loading state
* Counter value
* Network status

State always has one latest value.

---

# Characteristics of StateFlow

* Hot flow
* Always has one value
* Requires initial value
* Emits only latest state
* New collectors immediately receive current value
* Multiple collectors share same data

---

# Creating StateFlow

```kotlin
val counter = MutableStateFlow(0)
```

Initial value is mandatory.

```
Current Value = 0
```

---

# Updating State

Use

```kotlin
counter.value = 1

counter.value = 2

counter.value = 3
```

Current state becomes

```
3
```

Only one value exists at any time.

---

# Collecting StateFlow

```kotlin
val counter = MutableStateFlow(0)

launch {
    counter.collect {
        println(it)
    }
}

counter.value = 1
counter.value = 2
counter.value = 3
```

Output

```
0
1
2
3
```

Notice

The collector immediately received

```
0
```

because StateFlow always has a current value.

---

# New Collector Example

```kotlin
val counter = MutableStateFlow(0)

counter.value = 10

launch {
    counter.collect {
        println(it)
    }
}
```

Output

```
10
```

Why?

Because StateFlow always sends the latest value to every new collector.

---

## Timeline

```
Initial

Current State = 0

↓

1

↓

2

↓

3
```

A new collector joins here

```
Current State = 3
```

It immediately receives

```
3
```

Not

```
0
1
2
```

Those values are gone.

---

# Duplicate Values

StateFlow does **not** emit a value if it's equal to the current one.

```kotlin
val state = MutableStateFlow(10)

state.value = 10
```

Nothing happens because the value didn't change.

Example

```kotlin
state.collect {
    println(it)
}

state.value = 10
state.value = 10
state.value = 20
```

Output

```
10
20
```

The duplicate `10` values are ignored.

---

# StateFlow in Android

A common pattern in a `ViewModel`:

```kotlin
class CounterViewModel : ViewModel() {

    private val _count = MutableStateFlow(0)

    val count: StateFlow<Int> = _count

    fun increment() {
        _count.value++
    }
}
```

UI

```kotlin
lifecycleScope.launch {
    viewModel.count.collect {
        textView.text = it.toString()
    }
}
```

Whenever the state changes, the UI updates automatically.

---

# SharedFlow

## Definition

A `SharedFlow` is a hot flow used to **broadcast events**.

Unlike StateFlow, it **doesn't represent state**.

Examples

* Toast message
* Snackbar
* Navigation
* Button click
* Login success
* Payment completed

These are events.

They happen once.

---

# Creating SharedFlow

```kotlin
val events = MutableSharedFlow<String>()
```

Notice

No initial value.

---

# Emitting

```kotlin
events.emit("Login Successful")

events.emit("Welcome")
```

---

# Collecting

```kotlin
launch {
    events.collect {
        println(it)
    }
}
```

Output

```
Login Successful
Welcome
```

---

# New Collector Example

```kotlin
events.emit("Hello")

launch {
    events.collect {
        println(it)
    }
}
```

Output

```
(nothing)
```

Why?

Because SharedFlow does **not** keep the latest value by default.

The event already happened.

The new collector missed it.

---

# Timeline

```
Event A

↓

Event B

↓

Event C
```

Collector joins here

```
        ↑
```

It only receives future events.

```
Event D

Event E
```

---

# Replay

SharedFlow can remember previous events using `replay`.

```kotlin
val events = MutableSharedFlow<String>(
    replay = 1
)
```

Now

```kotlin
events.emit("Hello")

launch {
    events.collect {
        println(it)
    }
}
```

Output

```
Hello
```

Because one previous event is replayed.

---

## replay = 2

```kotlin
val flow = MutableSharedFlow<Int>(
    replay = 2
)

flow.emit(1)
flow.emit(2)
flow.emit(3)
```

New collector gets

```
2
3
```

because only the last two values are replayed.

---

# Multiple Collectors

```kotlin
val events = MutableSharedFlow<String>()
```

Collector 1

```kotlin
launch {
    events.collect {
        println("A $it")
    }
}
```

Collector 2

```kotlin
launch {
    events.collect {
        println("B $it")
    }
}
```

Emit

```kotlin
events.emit("Hello")
```

Output

```
A Hello

B Hello
```

Both collectors receive the same event.

---

# Buffer

SharedFlow can store extra events temporarily.

```kotlin
MutableSharedFlow(
    replay = 0,
    extraBufferCapacity = 5
)
```

Meaning

```
Collector Busy

↓

Event1
Event2
Event3
Event4
Event5
```

The buffer prevents immediate suspension of the emitter until it fills up.

---

# StateFlow vs SharedFlow

| Feature                                | StateFlow      | SharedFlow                 |
| -------------------------------------- | -------------- | -------------------------- |
| Hot Flow                               | ✅              | ✅                          |
| Holds state                            | ✅              | ❌                          |
| Initial value required                 | ✅              | ❌                          |
| Current value available                | ✅ (`value`)    | ❌                          |
| Replays latest value to new collectors | Always 1       | Configurable with `replay` |
| Duplicate equal values emitted         | ❌ (suppressed) | ✅                          |
| Good for UI state                      | ✅              | ❌                          |
| Good for one-time events               | ❌              | ✅                          |

---

# Android Examples

## StateFlow

Current screen state

```kotlin
data class UiState(
    val loading: Boolean,
    val users: List<String>
)

private val _uiState = MutableStateFlow(
    UiState(true, emptyList())
)

val uiState: StateFlow<UiState> = _uiState
```

UI

```kotlin
viewModel.uiState.collect { state ->
    progressBar.isVisible = state.loading
    adapter.submitList(state.users)
}
```

The UI always reflects the latest state, even after configuration changes.

---

## SharedFlow

Show a toast

```kotlin
private val _events = MutableSharedFlow<String>()

val events = _events
```

Emit

```kotlin
_events.emit("Saved Successfully")
```

UI

```kotlin
viewModel.events.collect {
    Toast.makeText(this, it, Toast.LENGTH_SHORT).show()
}
```

Each event is handled once and is not automatically re-shown to future collectors unless `replay` is configured.

---

# When to Use Which?

Use **StateFlow** when you need to represent the current state of something:

* Login state
* User profile
* Theme
* Loading status
* Form data
* Counter
* Network connectivity

Use **SharedFlow** when you need to send one-time events:

* Navigation commands
* Toast or Snackbar messages
* Dialog requests
* Payment success notifications
* Button click events
* Refresh triggers

---

# Visual Summary

```
StateFlow
──────────────

Current State
      │
      ▼
Loading

      ▼
Success

      ▼
Error

New collector
      │
      ▼
Immediately gets "Error"
```

```
SharedFlow
──────────────

Event A

Event B

Event C

New collector joins
      │
      ▼
Gets only future events (unless replay > 0)

Event D

Event E
```

### Quick Rule to Remember

* **StateFlow answers:** *"What is the current state right now?"*
* **SharedFlow answers:** *"What events should all current subscribers be notified about?"*

---
