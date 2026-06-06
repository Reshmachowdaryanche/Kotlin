# How do you handle lifecycle events in Compose functions?

In Jetpack Compose, composables do not directly have lifecycle callbacks like:

```text id="jlwmu4"
onCreate()
onResume()
onDestroy()
```

Instead, lifecycle events are handled using:

* side-effect APIs
* LifecycleOwner
* lifecycle observers
* lifecycle-aware state collection

---

# Common Ways to Handle Lifecycle Events

| API                         | Purpose                                      |
| --------------------------- | -------------------------------------------- |
| LaunchedEffect              | Run code when composable enters composition  |
| DisposableEffect            | Register and clean up lifecycle observers    |
| LifecycleEventObserver      | Listen to Activity/Fragment lifecycle events |
| collectAsStateWithLifecycle | Lifecycle-aware Flow collection              |
| rememberCoroutineScope      | Launch lifecycle-aware coroutines manually   |

---

# 1. Using `LaunchedEffect`

Used to run suspend functions tied to composable lifecycle.

```kotlin id="jlwmg7"
LaunchedEffect(Unit) {

    viewModel.loadData()
}
```

### Behavior

```text id="jlwmt1"
Composable enters composition
        ↓
Coroutine starts
        ↓
Composable leaves composition
        ↓
Coroutine cancelled automatically
```

### Common Use Cases

* API calls
* initial data loading
* timers
* analytics

---

# 2. Using `DisposableEffect`

Used when lifecycle observers or listeners require cleanup.

```kotlin id="jlwmy2"
@Composable
fun HomeScreen() {

    val lifecycleOwner =
        LocalLifecycleOwner.current

    DisposableEffect(lifecycleOwner) {

        val observer =
            LifecycleEventObserver { _, event ->

                when(event) {

                    Lifecycle.Event.ON_START -> {
                        println("ON_START")
                    }

                    Lifecycle.Event.ON_STOP -> {
                        println("ON_STOP")
                    }

                    else -> Unit
                }
            }

        lifecycleOwner.lifecycle
            .addObserver(observer)

        onDispose {

            lifecycleOwner.lifecycle
                .removeObserver(observer)
        }
    }
}
```

### Why use DisposableEffect?

Because observers need:

```text id="jlwmb5"
register + unregister
```

It ensures proper cleanup and prevents memory leaks.

---

# 3. Using `LifecycleEventObserver`

Used to observe Android lifecycle events.

### Common Events

| Event      | Meaning                 |
| ---------- | ----------------------- |
| ON_CREATE  | Lifecycle created       |
| ON_START   | Screen visible          |
| ON_RESUME  | Screen interactive      |
| ON_PAUSE   | Screen partially hidden |
| ON_STOP    | Screen not visible      |
| ON_DESTROY | Lifecycle destroyed     |

---

# 4. Using `collectAsStateWithLifecycle`

Used for lifecycle-aware Flow collection.

```kotlin id="jlwmm4"
val uiState by viewModel.uiState
    .collectAsStateWithLifecycle()
```

### Why preferred?

Flow collection automatically pauses when lifecycle is inactive.

Prevents:

* unnecessary work
* background collection
* memory waste

---

# 5. Using `rememberCoroutineScope`

Used to manually launch lifecycle-aware coroutines.

```kotlin id="jlwmu8"
val scope = rememberCoroutineScope()

scope.launch {

    snackbarHostState.showSnackbar(
        "Saved"
    )
}
```

---

# Important Interview Point

Compose lifecycle is based on:

```text id="jlwmg0"
Composition lifecycle
```

not traditional View lifecycle.

Composable lifecycle mainly consists of:

* entering composition
* recomposition
* leaving composition

---

# Best Practices

✅ Use ViewModel for business logic
✅ Use StateFlow + collectAsStateWithLifecycle
✅ Use DisposableEffect for observers/listeners
✅ Use LaunchedEffect for startup side effects

---

# One-line Summary

Lifecycle events in Compose are handled using lifecycle-aware side-effect APIs like `LaunchedEffect`, `DisposableEffect`, `LifecycleEventObserver`, and `collectAsStateWithLifecycle`, which safely integrate composables with the Android lifecycle.
