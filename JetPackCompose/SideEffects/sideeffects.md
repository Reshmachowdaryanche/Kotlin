
# 🌱 Side Effects in Jetpack Compose

## 📖 Definition

In Jetpack Compose, a **side effect** is **any action that affects the outside world beyond just drawing UI**, such as:

* Fetching data from network
* Logging events
* Updating system UI (status bar, clipboard, toast)
* Starting coroutines
* Registering/unregistering listeners

⚠️ Since composables **recompose frequently**, doing side effects *directly in composables* can cause them to repeat unexpectedly.
👉 To solve this, Compose provides **special side-effect APIs** that are **lifecycle-aware**.

---

# 📦 Side Effect APIs — with Examples and Usage

---

### 1. **`SideEffect { }`**

* Runs **after every successful recomposition**.
* Use it to **synchronize Compose state with external objects**.
* No suspend functions allowed.

✅ Example:

```kotlin
@Composable
fun StatusBar(color: Color) {
    SideEffect {
        // Sync status bar with current color
        systemUiController.setStatusBarColor(color)
    }
}
```

📌 **Usage:** Update external UI (status bar, analytics, logs) after recomposition.

---

### 2. **`LaunchedEffect(key)`**

* Launches a coroutine tied to the composable’s lifecycle.
* Runs **once when composed**, re-runs if `key` changes.
* Cancels automatically when composable leaves.

✅ Example:

```kotlin
@Composable
fun UserProfile(userId: String) {
    var user by remember { mutableStateOf<User?>(null) }

    LaunchedEffect(userId) {
        user = repository.getUser(userId) // suspend fun
    }

    Text("Hello ${user?.name ?: "Loading..."}")
}
```

📌 **Usage:** Fetch data, run animations, timers when screen or state changes.

---

### 3. **`DisposableEffect(key)`**

* Provides **setup and cleanup** around composition.
* Runs setup when entering composition, cleanup (`onDispose`) when leaving or key changes.

✅ Example:

```kotlin
@Composable
fun LocationObserver(locationManager: LocationManager) {
    DisposableEffect(Unit) {
        val listener = LocationListener { location -> println(location) }
        locationManager.requestLocationUpdates(listener)

        onDispose {
            locationManager.removeUpdates(listener)
        }
    }
}
```

📌 **Usage:** Register/unregister listeners, broadcast receivers, sensors.

---

### 4. **`rememberCoroutineScope()`**

* Returns a coroutine scope tied to the composable’s lifecycle.
* Unlike `LaunchedEffect`, doesn’t run automatically — you use it in **response to events**.

✅ Example:

```kotlin
@Composable
fun SaveButton() {
    val scope = rememberCoroutineScope()
    Button(onClick = {
        scope.launch {
            repository.saveData()
        }
    }) {
        Text("Save")
    }
}
```

📌 **Usage:** Start coroutines on demand (button clicks, gestures).

---

### 5. **`rememberUpdatedState(newValue)`**

* Keeps the **latest value** inside long-running effects without restarting them.

✅ Example:

```kotlin
@Composable
fun Timer(onTick: () -> Unit) {
    val updatedOnTick by rememberUpdatedState(onTick)

    LaunchedEffect(Unit) {
        while (true) {
            delay(1000)
            updatedOnTick() // always latest onTick
        }
    }
}
```

📌 **Usage:** Avoid restarting effects when only a callback changes.

---

### 6. **`produceState`**

* Converts suspend functions or Flows into a `State<T>` that Compose can observe.

✅ Example:

```kotlin
@Composable
fun UserData(userId: String): State<User?> {
    return produceState<User?>(null, userId) {
        value = repository.loadUser(userId)
    }
}
```

📌 **Usage:** Fetch data and expose it as State in UI.

---

### 7. **`derivedStateOf`**

* Creates a **derived/computed state** from other states efficiently.
* Only recomputes when the input changes.

✅ Example:

```kotlin
val activeCount by remember {
    derivedStateOf { items.count { it.active } }
}
```

📌 **Usage:** Derived values (like counts, filters, calculations).

---

# 📝 Quick Usage Cheat Sheet

| API                        | When to Use                                                                                            |
| -------------------------- | ------------------------------------------------------------------------------------------------------ |
| **SideEffect**             | After every recomposition → sync state with external systems (logs, status bar, analytics)             |
| **LaunchedEffect**         | Run suspend function / coroutine when entering composition or when key changes (data fetch, animation) |
| **DisposableEffect**       | Setup + cleanup when entering/leaving composition (listeners, sensors)                                 |
| **rememberCoroutineScope** | Launch coroutine manually in response to events (button clicks, gestures)                              |
| **rememberUpdatedState**   | Keep latest value inside long-running effect without restarting it                                     |
| **produceState**           | Convert suspend/Flow → Compose State                                                                   |
| **derivedStateOf**         | Efficiently compute derived state from other state                                                     |

---

👉 Would you like me to now build a **full mini example (like a Chat screen)** showing *multiple side effects working together* so you see how they interact in a real scenario?
