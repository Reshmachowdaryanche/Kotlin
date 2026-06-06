# `produceState`

## Purpose

Used to:

```text id="jlwmu5"
Convert external/non-Compose data into Compose State
```

Examples of external data:

* API/network response
* Flow
* LiveData
* RxJava
* callback/listener data

---

# Why do we need produceState?

Compose UI reacts automatically only to:

```kotlin id="jlwmg9"
State<T>
```

But external data sources are NOT Compose state.

`produceState` acts as a bridge:

```text id="jlwmt8"
External Data
      ↓
produceState
      ↓
Compose State
      ↓
UI recomposes automatically
```

---

# Key Points

```kotlin id="jlwmy4"
produceState(initialValue, keys...) {

    value = ...
}
```

* creates Compose `State`
* launches coroutine automatically
* updates state using `value`
* recomposes UI when value changes
* coroutine cancelled when composable leaves composition
* restarts if keys change

---

# Mental Model

```text id="jlwmb3"
External async work
      ↓
produceState converts result
      ↓
Compose State updates
      ↓
UI recomposes
```

---

# Example

```kotlin id="jlwmm6"
@Composable
fun loadNetworkImage(
    url: String,
    imageRepository: ImageRepository = ImageRepository()
): State<Result<Image>> {

    return produceState<Result<Image>>(

        initialValue = Result.Loading,

        url,
        imageRepository

    ) {

        // Suspend function call

        val image = imageRepository.load(url)

        // Update Compose State

        value = if (image == null) {
            Result.Error
        } else {
            Result.Success(image)
        }
    }
}
```

---

# Step-by-step Explanation

---

## 1. produceState creates Compose State

```kotlin id="jlwmu0"
initialValue = Result.Loading
```

Initial state becomes:

```text id="jlwmg1"
Loading
```

UI can immediately show:

* progress bar
* shimmer
* loading text

---

## 2. Coroutine starts automatically

Inside `produceState`:

```kotlin id="jlwms3"
{
    val image = imageRepository.load(url)
}
```

runs inside a coroutine automatically.

So suspend functions can be called directly.

---

## 3. Load image from network

```kotlin id="jlwmd7"
val image = imageRepository.load(url)
```

Usually a network/API call.

---

## 4. Update Compose State

```kotlin id="jlwmo2"
value = Result.Success(image)
```

OR:

```kotlin id="jlwmt4"
value = Result.Error
```

Updating `value` updates Compose State.

---

## 5. UI recomposes automatically

When `value` changes:

```text id="jlwmy8"
Loading → Success/Error
```

UI updates automatically.

---

# Full Flow

```text id="jlwmb8"
Composable enters
      ↓
produceState creates State
      ↓
Coroutine starts
      ↓
Network/API call happens
      ↓
value updated
      ↓
UI recomposes
      ↓
Composable removed
      ↓
Coroutine cancelled
```

---

# Keys Behavior

```kotlin id="jlwmm1"
produceState(initialValue, url, imageRepository)
```

Keys are:

```kotlin id="jlwmu7"
url
imageRepository
```

If any key changes:

```text id="jlwmg5"
Old coroutine cancelled
New coroutine started
```

Exactly like `LaunchedEffect(keys)`.

---

# Important Point

Inside `produceState`:

```kotlin id="jlwms9"
value = ...
```

means:

```text id="jlwmd2"
Update Compose State
```

This automatically triggers recomposition.

---

# What happens internally?

`produceState` internally combines:

```text id="jlwmo8"
remember + mutableStateOf + LaunchedEffect
```

Equivalent idea:

```kotlin id="jlwmt0"
val state = remember {
    mutableStateOf(initialValue)
}

LaunchedEffect(keys) {
    state.value = ...
}
```

---

# Why use produceState instead of LaunchedEffect?

Without `produceState`:

```kotlin id="jlwmy2"
val state = remember {
    mutableStateOf(...)
}

LaunchedEffect {
    state.value = ...
}
```

You manually manage state.

`produceState` simplifies this pattern.

---

# awaitDispose

Used when data source is NOT suspend-based.

Examples:

* listeners
* callbacks
* subscriptions

---

# Example

```kotlin id="jlwmb5"
produceState(initialValue = 0) {

    val listener = object : Listener {
        override fun onValue(v: Int) {
            value = v
        }
    }

    source.addListener(listener)

    awaitDispose {
        source.removeListener(listener)
    }
}
```

---

# Why awaitDispose?

Ensures cleanup when composable leaves composition.

Equivalent idea to:

```text id="jlwmm9"
DisposableEffect cleanup
```

---

# Common Use Cases

* image loading
* Flow collection
* LiveData observation
* network/API state
* callback APIs
* repository data

---

# Important Characteristics

✅ creates Compose State
✅ coroutine support
✅ lifecycle-aware
✅ automatic recomposition
✅ restartable using keys

---

# Difference from `LaunchedEffect`

| LaunchedEffect        | produceState            |
| --------------------- | ----------------------- |
| Runs side effects     | Produces Compose State  |
| No state returned     | Returns State<T>        |
| Manual state handling | Built-in state handling |

---

# Difference from `collectAsState`

| collectAsState        | produceState          |
| --------------------- | --------------------- |
| Specifically for Flow | Generic external data |

---

# Interview Point

Use `produceState` when:

```text id="jlwmu3"
External async/non-Compose data
must be exposed as Compose State.
```

---

# Simple Analogy

```text id="jlwmg8"
"Take external async data
and convert it into UI-observable Compose state."
```
