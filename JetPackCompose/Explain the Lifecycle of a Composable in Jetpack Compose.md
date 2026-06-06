
# Explain the Lifecycle of a Composable in Jetpack Compose

The Jetpack Compose lifecycle is similar to the traditional Android lifecycle, but because Compose is declarative and state-driven, composables follow the lifecycle of the Composition instead of Activity or Fragment callbacks.

A composable mainly goes through 3 phases:

1. Composition
2. Recomposition
3. Disposal

---

# 1. Composition

Composition is the initial phase where UI is created for the first time.

When a composable function is called for the first time, Jetpack Compose:

* evaluates the composable functions
* builds the UI tree
* creates the UI elements
* places them on the screen

This phase happens only once for a particular UI unless that composable is removed and recreated.

---

# Example

```kotlin id="jlwmu5"
@Composable
fun Greeting(name: String) {

    Text("Hello $name")
}
```

When `Greeting()` is first displayed:

```text id="jlwmg8"
Initial Composition happens
```

---

# 2. Recomposition

Recomposition is the process of updating the UI when state changes.

Compose automatically tracks state used inside composables.

Whenever observed state changes:

* only affected composables are re-executed
* unaffected composables may be skipped

This makes Compose efficient.

---

# Example

```kotlin id="jlwmt1"
var count by remember {

    mutableStateOf(0)
}

Text("Count: $count")
```

When:

```kotlin id="jlwmy2"
count++
```

Compose detects state change and recomposes only the required UI.

---

# Important Points About Recomposition

* recomposition happens only when observed state changes
* Compose performs smart recomposition
* entire UI is not recreated
* only affected composables are updated

---

# Example

```kotlin id="jlwmb7"
Column {

    Header()

    Counter(count)

    Footer()
}
```

If `count` changes:

```text id="jlwmm5"
Only Counter() recomposes
```

---

# 3. Disposal

Disposal happens when a composable is removed from the UI tree.

This can happen when:

* navigating away from a screen
* conditional UI disappears
* composable is replaced

During disposal, Compose:

* clears remembered state
* cancels coroutines
* removes listeners
* runs cleanup logic

---

# Example

```kotlin id="jlwmu9"
if (showScreen) {

    HomeScreen()
}
```

When:

```kotlin id="jlwmg0"
showScreen = false
```

`HomeScreen()` leaves composition and gets disposed.

---

# DisposableEffect Cleanup

If `DisposableEffect` exists:

```kotlin id="jlwms4"
DisposableEffect(Unit) {

    onDispose {

        // cleanup
    }
}
```

Compose executes cleanup automatically during disposal.

---

# Lifecycle-aware APIs

| API                    | Lifecycle Behavior                         |
| ---------------------- | ------------------------------------------ |
| remember               | survives recomposition                     |
| LaunchedEffect         | starts on composition, cancels on disposal |
| DisposableEffect       | setup on composition, cleanup on disposal  |
| rememberCoroutineScope | cancelled on disposal                      |

---

# Lifecycle Flow

```text id="jlwmd2"
Composable enters composition
        ↓
Initial composition
        ↓
State changes
        ↓
Recomposition
        ↓
Composable removed
        ↓
Disposal
```

---

# Difference from Traditional Android Lifecycle

| Traditional Views   | Compose       |
| ------------------- | ------------- |
| onCreate()          | Composition   |
| UI updates manually | Recomposition |
| onDestroy()         | Disposal      |

---

# Important Interview Point

Composable functions should ideally be:

```text id="jlwmo5"
Side-effect free
```

because recomposition can happen multiple times.

Async work and side effects should be handled using:

* LaunchedEffect
* DisposableEffect
* rememberCoroutineScope

---

# One-line Summary

The lifecycle of a composable consists of Composition, Recomposition, and Disposal, where Compose automatically creates, updates, and cleans up UI based on state changes in a declarative and lifecycle-aware manner.
