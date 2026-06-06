# `snapshotFlow`

## Purpose

Used to:

```text id="jlwmu4"
Convert Compose State into Kotlin Flow
```

This allows Compose state to use:

* Flow operators
* collect
* map
* filter
* debounce
* distinctUntilChanged

---

# Why do we need snapshotFlow?

Compose state works well inside Compose UI.

But sometimes we want to:

✅ observe Compose state as Flow
✅ use Flow operators
✅ collect state changes in coroutines

`snapshotFlow` creates this bridge.

---

# Mental Model

```text id="jlwmg6"
Compose State
      ↓
snapshotFlow
      ↓
Kotlin Flow
      ↓
Use Flow operators
```

---

# Key Points

```kotlin id="jlwmt1"
snapshotFlow {

}
```

* converts Compose state into Flow
* emits values when observed state changes
* emits only when new value differs from previous value
* behaves similar to `distinctUntilChanged()`
* Flow is cold

---

# What is a cold Flow?

```text id="jlwmy3"
Flow does nothing
until someone collects it
```

`snapshotFlow` block runs only during collection.

---

# Example

```kotlin id="jlwmb7"
val listState = rememberLazyListState()

LazyColumn(state = listState) {
    // items
}

LaunchedEffect(listState) {

    snapshotFlow {

        listState.firstVisibleItemIndex
    }

    .map { index ->

        index > 0
    }

    .distinctUntilChanged()

    .filter { it == true }

    .collect {

        MyAnalyticsService
            .sendScrolledPastFirstItemEvent()
    }
}
```

---

# Step-by-step Explanation

---

## 1. Scroll position changes

```kotlin id="jlwmm9"
listState.firstVisibleItemIndex
```

Values become:

```text id="jlwmu8"
0,1,2,3,4,5...
```

This is Compose State.

---

## 2. Convert Compose State → Flow

```kotlin id="jlwmg0"
snapshotFlow {
    listState.firstVisibleItemIndex
}
```

Now state changes become Flow emissions.

Equivalent emitted values:

```text id="jlwms7"
0 → 1 → 2 → 3 → 4...
```

---

## 3. Transform values

```kotlin id="jlwmd3"
.map { index -> index > 0 }
```

Now values become:

| Index | Result |
| ----- | ------ |
| 0     | false  |
| 1     | true   |
| 2     | true   |
| 3     | true   |

---

## 4. Remove duplicates

```kotlin id="jlwmo5"
.distinctUntilChanged()
```

Now Flow emits only when boolean changes.

Result:

```text id="jlwmt9"
false → true
```

NOT:

```text id="jlwmy7"
true,true,true,true...
```

---

## 5. Filter only true

```kotlin id="jlwmb4"
.filter { it == true }
```

Now only:

```text id="jlwmm2"
true
```

passes through.

---

## 6. Collect result

```kotlin id="jlwmu2"
.collect {

    MyAnalyticsService
        .sendScrolledPastFirstItemEvent()
}
```

Analytics event fires only once when user scrolls past first item.

---

# Full Flow

```text id="jlwmg3"
Scroll changes
      ↓
snapshotFlow converts state to Flow
      ↓
map converts index → boolean
      ↓
distinctUntilChanged removes duplicates
      ↓
filter keeps only true
      ↓
collect triggers analytics event
```

---

# Why use snapshotFlow here?

Without `snapshotFlow`:

```text id="jlwms9"
Cannot directly use Flow operators
on Compose state
```

With `snapshotFlow`:

```text id="jlwmd8"
Compose state becomes reactive Flow stream
```

---

# Important Behavior

`snapshotFlow` emits only when:

```text id="jlwmo0"
New value != previous value
```

Similar to:

```kotlin id="jlwmt2"
distinctUntilChanged()
```

---

# Another Example — Search Query

```kotlin id="jlwmy5"
var searchQuery by remember {
    mutableStateOf("")
}

LaunchedEffect(Unit) {

    snapshotFlow {

        searchQuery
    }

    .debounce(500)

    .distinctUntilChanged()

    .collect { query ->

        viewModel.search(query)
    }
}
```

---

# What happens?

---

## User types

```text id="jlwmb1"
"H"
"He"
"Hel"
"Hell"
"Hello"
```

---

## debounce(500)

Waits until typing stops for 500ms.

---

## collect

Only final value:

```text id="jlwmm6"
"Hello"
```

triggers API call.

---

# Common Use Cases

* scroll tracking
* analytics
* search debounce
* observing Compose state in coroutines
* Flow operator usage
* event streams from Compose state

---

# Important Characteristics

✅ Compose State → Flow conversion
✅ works with Flow operators
✅ lifecycle-aware when collected in effects
✅ distinctUntilChanged-like behavior

---

# Difference from `derivedStateOf`

| derivedStateOf         | snapshotFlow                    |
| ---------------------- | ------------------------------- |
| Produces Compose State | Produces Kotlin Flow            |
| UI optimization        | Flow/reactive stream processing |
| Used inside Compose UI | Used with Flow operators        |

---

# Difference from `produceState`

| produceState             | snapshotFlow         |
| ------------------------ | -------------------- |
| External → Compose State | Compose State → Flow |
| Produces State<T>        | Produces Flow<T>     |

---

# Interview Point

Use `snapshotFlow` when:

```text id="jlwmu7"
Compose State needs to be observed
as a Kotlin Flow.
```

---

# Simple Analogy

```text id="jlwmg5"
"Convert UI state changes
into a Flow stream
so Flow operators can process them."
```
