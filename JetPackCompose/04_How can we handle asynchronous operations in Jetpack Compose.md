# How can we handle asynchronous operations in Jetpack Compose?

In Jetpack Compose, asynchronous operations are mainly handled using:

* Kotlin Coroutines
* Flow / StateFlow
* Compose Side Effect APIs

Compose itself is declarative, so async work should not run directly inside composable bodies because recomposition can happen multiple times.

---

# 1. Using `LaunchedEffect`

Used to run suspend functions tied to composable lifecycle.

```kotlin id="jlwm0q"
LaunchedEffect(Unit) {

    val user = repository.getUser()
}
```

### Key Points

* launches coroutine automatically
* lifecycle-aware
* coroutine cancelled when composable leaves composition
* restarts if keys change

### Common Use Cases

* API calls
* timers
* animations
* Flow collection

---

# 2. Using `rememberCoroutineScope`

Used to manually launch coroutines from user events.

```kotlin id="jlwmu5"
val scope = rememberCoroutineScope()

Button(onClick = {

    scope.launch {

        snackbarHostState.showSnackbar("Saved")
    }
})
```

### Key Points

* used in callbacks like `onClick`
* lifecycle-aware CoroutineScope
* manual coroutine launching

### Common Use Cases

* snackbar
* scrolling
* button click async work

---

# 3. Using `ViewModel + StateFlow` (Recommended)

Modern recommended architecture.

### ViewModel

```kotlin id="jlwmg7"
private val _uiState =
    MutableStateFlow<UserUiState>(
        UserUiState.Loading
    )

val uiState = _uiState.asStateFlow()
```

### Compose

```kotlin id="jlwmt9"
val uiState by viewModel.uiState
    .collectAsStateWithLifecycle()
```

### Flow

```text id="jlwmy2"
Coroutine updates StateFlow
        ↓
Compose observes StateFlow
        ↓
UI recomposes automatically
```

### Why Recommended?

* lifecycle-aware
* survives configuration changes
* clean architecture
* separation of concerns

---

# 4. Using `collectAsState()`

Used to observe Flow in Compose.

```kotlin id="jlwmb4"
val users by usersFlow
    .collectAsState(initial = emptyList())
```

### Key Points

* converts Flow → Compose State
* recomposes UI automatically

---

# 5. Using `collectAsStateWithLifecycle()` (Preferred)

```kotlin id="jlwmm6"
val users by usersFlow
    .collectAsStateWithLifecycle()
```

### Why preferred?

Collects Flow only when lifecycle is active.

Prevents:

* unnecessary work
* background collection
* memory waste

---

# 6. Using `produceState`

Used to convert async/external data into Compose State.

```kotlin id="jlwmu8"
val user by produceState<User?>(
    initialValue = null
) {

    value = repository.getUser()
}
```

### Key Points

* internally uses coroutine
* updates Compose State using `value`
* triggers recomposition automatically

---

# 7. Using `snapshotFlow`

Used to convert Compose State into Flow.

```kotlin id="jlwmg1"
snapshotFlow {

    listState.firstVisibleItemIndex
}
```

### Common Use Cases

* analytics
* scroll tracking
* debounce
* Flow operators on Compose state

---

# 8. Using `DisposableEffect`

Used for async listeners/subscriptions with cleanup.

```kotlin id="jlwms3"
DisposableEffect(Unit) {

    registerListener()

    onDispose {

        unregisterListener()
    }
}
```

### Common Use Cases

* BroadcastReceiver
* LifecycleObserver
* sensor listeners

---

# Important Interview Point

Never run async work directly inside composable body.

❌ Bad

```kotlin id="jlwmd0"
@Composable
fun Screen() {

    repository.getUser()
}
```

Because composables can recompose multiple times.

✅ Correct

```kotlin id="jlwmo5"
LaunchedEffect(Unit) {

    repository.getUser()
}
```

---

# Recommended Modern Architecture

```text id="jlwmt7"
Repository
     ↓
ViewModel (Coroutines + StateFlow)
     ↓
Compose observes using collectAsStateWithLifecycle()
     ↓
UI recomposes automatically
```

---

# Quick Revision Table

| API                    | Purpose                             |
| ---------------------- | ----------------------------------- |
| LaunchedEffect         | Run suspend functions automatically |
| rememberCoroutineScope | Launch coroutine manually           |
| collectAsState         | Observe Flow                        |
| observeAsState         | Observe LiveData                    |
| produceState           | Async data → Compose State          |
| snapshotFlow           | Compose State → Flow                |
| DisposableEffect       | Setup + cleanup listeners           |

---

# One-line Summary

Jetpack Compose handles asynchronous operations using Coroutines, Flow, StateFlow, and side-effect APIs to safely perform async work and automatically update UI through recomposition.
