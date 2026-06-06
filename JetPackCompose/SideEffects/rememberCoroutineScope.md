# `rememberCoroutineScope`

## Purpose

Used to:

* get a coroutine scope inside a composable
* manually launch coroutines from callbacks/events
* run suspend functions outside composable body

---

# Why do we need it?

`LaunchedEffect` can only run directly inside composable scope.

But sometimes coroutine work should happen because of:

* button click
* user interaction
* gesture
* event callback

Example:

```kotlin id="jlwm3f"
onClick = {

}
```

This is NOT a composable function.

So you cannot use:

```kotlin id="jlwmy0"
LaunchedEffect { }
```

inside `onClick`.

That’s why `rememberCoroutineScope` exists.

---

# Key Points

```kotlin id="jlwmt8"
val scope = rememberCoroutineScope()
```

* returns a `CoroutineScope`
* scope is tied to composable lifecycle
* automatically cancelled when composable leaves composition
* coroutines are launched manually using `scope.launch`

---

# Mental Model

```text id="jlwmv3"
Composable appears
      ↓
Get CoroutineScope
      ↓
Wait for user event
      ↓
Launch coroutine manually
      ↓
Composable removed
      ↓
Scope cancelled automatically
```

---

# Example

```kotlin id="jlwmf7"
@Composable
fun MoviesScreen(snackbarHostState: SnackbarHostState) {

    // Creates a CoroutineScope bound to the MoviesScreen's lifecycle
    val scope = rememberCoroutineScope()

    Scaffold(
        snackbarHost = {
            SnackbarHost(hostState = snackbarHostState)
        }
    ) { contentPadding ->

        Column(Modifier.padding(contentPadding)) {

            Button(
                onClick = {

                    // Launch coroutine manually

                    scope.launch {
                        snackbarHostState.showSnackbar(
                            "Something happened!"
                        )
                    }
                }
            ) {
                Text("Press me")
            }
        }
    }
}
```

---

# Step-by-step Explanation

---

## 1. Create CoroutineScope

```kotlin id="jlwmu9"
val scope = rememberCoroutineScope()
```

Creates a lifecycle-aware coroutine scope.

Bound to:

```text id="jlwmh1"
MoviesScreen lifecycle
```

---

## 2. Button click happens

```kotlin id="jlwmy5"
onClick = {

}
```

`onClick` is NOT suspend.

Cannot directly call suspend functions here.

---

## 3. Launch coroutine manually

```kotlin id="jlwmn6"
scope.launch {

}
```

Starts a coroutine inside click handler.

---

## 4. Call suspend function

```kotlin id="jlwmb4"
snackbarHostState.showSnackbar(...)
```

`showSnackbar()` is suspend.

That’s why coroutine is required.

---

# Full Flow

```text id="jlwmq2"
User clicks button
      ↓
scope.launch starts coroutine
      ↓
showSnackbar() executes
      ↓
Snackbar appears
```

---

# Why not use LaunchedEffect?

Because snackbar should appear:

❌ NOT automatically when screen opens
✅ ONLY when user clicks button

This is event-driven behavior.

---

# Common Use Cases

* show snackbar
* scroll LazyColumn
* open/close drawer
* retry button
* animation on click
* manual coroutine control

---

# Important Lifecycle Behavior

```text id="jlwmz7"
Composable removed from screen
        ↓
CoroutineScope cancelled automatically
        ↓
All launched coroutines cancelled
```

Prevents memory leaks.

---

# Important Characteristics

✅ lifecycle-aware
✅ manual coroutine launching
✅ works in event handlers
✅ supports suspend functions

---

# Difference from `LaunchedEffect`

| LaunchedEffect                 | rememberCoroutineScope       |
| ------------------------------ | ---------------------------- |
| Coroutine starts automatically | You start coroutine manually |
| Runs when composable enters    | Runs on events/callbacks     |
| Restartable using keys         | No automatic restart         |
| Used for side effects          | Used for user interactions   |

---

# Interview Point

`rememberCoroutineScope()`:

❌ does NOT launch coroutine itself
✅ only provides CoroutineScope

You must call:

```kotlin id="jlwmm0"
scope.launch {

}
```

manually.

---

# Simple Analogy

```text id="jlwmg8"
"Wait until user performs an action,
then start async work."
```
