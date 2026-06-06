In Jetpack Compose, you don't usually override lifecycle methods like `onCreate()`, `onStart()`, or `onDestroy()` inside a Composable. Instead, Compose provides **side-effect APIs** that are lifecycle-aware.

Here are the most common ways to handle lifecycle events.

---

# 1. `LaunchedEffect` (Run when composable enters composition)

Equivalent to starting work when a screen appears.

```kotlin
@Composable
fun UserScreen() {

    LaunchedEffect(Unit) {
        println("Screen entered composition")
        loadUserData()
    }

    Text("User Screen")
}
```

### When it runs

```text
Composable enters composition
       ↓
LaunchedEffect starts
       ↓
Coroutine runs
```

If the Composable leaves composition, the coroutine is automatically cancelled.

---

# 2. `DisposableEffect` (Setup + Cleanup)

Equivalent to:

```kotlin
onStart()
onStop()
```

or

```kotlin
registerListener()
unregisterListener()
```

Example:

```kotlin
@Composable
fun LocationScreen() {

    DisposableEffect(Unit) {

        println("Register location listener")

        onDispose {
            println("Remove location listener")
        }
    }

    Text("Location Screen")
}
```

### Lifecycle

```text
Composable enters
    ↓
DisposableEffect executes

Composable leaves
    ↓
onDispose executes
```

---

# 3. Observe Activity Lifecycle Events

Sometimes you need actual Android lifecycle callbacks.

Use:

```kotlin
LocalLifecycleOwner.current
```

Example:

```kotlin
@Composable
fun LifecycleObserverScreen() {

    val lifecycleOwner = LocalLifecycleOwner.current

    DisposableEffect(lifecycleOwner) {

        val observer = LifecycleEventObserver { _, event ->

            when (event) {

                Lifecycle.Event.ON_START ->
                    println("ON_START")

                Lifecycle.Event.ON_RESUME ->
                    println("ON_RESUME")

                Lifecycle.Event.ON_PAUSE ->
                    println("ON_PAUSE")

                Lifecycle.Event.ON_STOP ->
                    println("ON_STOP")

                else -> {}
            }
        }

        lifecycleOwner.lifecycle.addObserver(observer)

        onDispose {
            lifecycleOwner.lifecycle.removeObserver(observer)
        }
    }

    Text("Lifecycle Observer")
}
```

---

# 4. `rememberUpdatedState`

Useful when lifecycle callbacks need the latest lambda.

Bad:

```kotlin
DisposableEffect(Unit) {
    listener.onEvent {
        callback()
    }

    onDispose { }
}
```

The callback may become stale after recomposition.

Better:

```kotlin
@Composable
fun Example(
    onEvent: () -> Unit
) {

    val currentOnEvent by rememberUpdatedState(onEvent)

    DisposableEffect(Unit) {

        listener.onEvent {
            currentOnEvent()
        }

        onDispose { }
    }
}
```

This ensures the latest callback is always used.

---

# 5. Collect Flow with Lifecycle Awareness

For ViewModels, this is very common.

ViewModel:

```kotlin
class UserViewModel : ViewModel() {

    val users = repository.usersFlow
}
```

Composable:

```kotlin
@Composable
fun UserScreen(
    viewModel: UserViewModel
) {

    val users by viewModel.users.collectAsStateWithLifecycle()

    Text("Users: ${users.size}")
}
```

Benefits:

* Automatically starts collecting when UI is active
* Stops collecting when UI is stopped
* Prevents memory leaks

Dependency:

```kotlin
implementation(
    "androidx.lifecycle:lifecycle-runtime-compose:2.8.0"
)
```

---

# 6. Lifecycle Event Example (Analytics)

```kotlin
@Composable
fun ProductScreen() {

    val lifecycleOwner = LocalLifecycleOwner.current

    DisposableEffect(lifecycleOwner) {

        val observer = LifecycleEventObserver { _, event ->

            when (event) {

                Lifecycle.Event.ON_RESUME -> {
                    analytics.trackScreen("Product")
                }

                Lifecycle.Event.ON_PAUSE -> {
                    analytics.flush()
                }

                else -> {}
            }
        }

        lifecycleOwner.lifecycle.addObserver(observer)

        onDispose {
            lifecycleOwner.lifecycle.removeObserver(observer)
        }
    }
}
```

---

# Compose Lifecycle APIs Cheat Sheet

| Requirement                                  | API                             |
| -------------------------------------------- | ------------------------------- |
| Run coroutine when composable appears        | `LaunchedEffect`                |
| Cleanup resources when composable disappears | `DisposableEffect`              |
| Access Activity/Fragment lifecycle events    | `LifecycleEventObserver`        |
| Keep latest callback reference               | `rememberUpdatedState`          |
| Collect Flow safely                          | `collectAsStateWithLifecycle()` |
| Store state across recompositions            | `remember`                      |
| Store state across configuration changes     | `rememberSaveable`              |

### Interview Answer

> In Jetpack Compose, lifecycle handling is typically done using side-effect APIs. `LaunchedEffect` is used for lifecycle-aware coroutines, `DisposableEffect` for setup and cleanup work, and `LifecycleEventObserver` with `LocalLifecycleOwner` for observing Android lifecycle events such as `ON_START` and `ON_RESUME`. For Flow collection, `collectAsStateWithLifecycle()` is the recommended lifecycle-aware approach.
