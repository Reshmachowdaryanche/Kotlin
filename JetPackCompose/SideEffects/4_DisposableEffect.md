# `DisposableEffect`

## Purpose

Used for side effects that need:

```text id="jlwmg4"
setup + cleanup
```

Examples:

* register/unregister observer
* subscribe/unsubscribe listener
* add/remove callback
* start/stop resource

---

# Key Points

```kotlin id="jlwmf0"
DisposableEffect(key) {

    onDispose {

    }
}
```

* effect starts when composable enters composition
* cleanup happens when:

  * composable leaves composition
  * keys change
* `onDispose` is mandatory

---

# Mental Model

```text id="jlwmq6"
Composable enters
      ↓
Setup effect
      ↓
Composable removed OR key changes
      ↓
Cleanup effect
```

---

# Why use DisposableEffect?

Some side effects create external objects/resources that must be cleaned up.

Without cleanup:

❌ memory leaks
❌ duplicate listeners
❌ multiple callbacks
❌ crashes/unexpected behavior

---

# Example

```kotlin id="jlwmt0"
@Composable
fun HomeScreen(
    lifecycleOwner: LifecycleOwner = LocalLifecycleOwner.current,
    onStart: () -> Unit,
    onStop: () -> Unit
) {

    // Always use latest lambdas

    val currentOnStart by rememberUpdatedState(onStart)
    val currentOnStop by rememberUpdatedState(onStop)

    // Restart effect if lifecycleOwner changes

    DisposableEffect(lifecycleOwner) {

        // Create lifecycle observer

        val observer = LifecycleEventObserver { _, event ->

            if (event == Lifecycle.Event.ON_START) {
                currentOnStart()
            } else if (event == Lifecycle.Event.ON_STOP) {
                currentOnStop()
            }
        }

        // Register observer

        lifecycleOwner.lifecycle.addObserver(observer)

        // Cleanup

        onDispose {
            lifecycleOwner.lifecycle.removeObserver(observer)
        }
    }

    /* UI */
}
```

---

# Step-by-step Explanation

---

## 1. lifecycleOwner

```kotlin id="jlwmh8"
lifecycleOwner: LifecycleOwner
```

Usually:

* Activity lifecycle
* Fragment lifecycle

---

## 2. rememberUpdatedState

```kotlin id="jlwmy1"
val currentOnStart by rememberUpdatedState(onStart)
val currentOnStop by rememberUpdatedState(onStop)
```

Ensures observer always calls latest callbacks.

Without restarting DisposableEffect.

---

## 3. DisposableEffect starts

```kotlin id="jlwmu4"
DisposableEffect(lifecycleOwner)
```

When composable enters composition:

```text id="jlwmm2"
Effect starts
```

Key is:

```kotlin id="jlwmz0"
lifecycleOwner
```

If lifecycleOwner changes:

```text id="jlwmg6"
Old effect disposed
New effect created
```

---

## 4. Create observer

```kotlin id="jlwmp8"
val observer = LifecycleEventObserver { _, event -> }
```

Observer listens to lifecycle events like:

* ON_START
* ON_STOP

---

## 5. Register observer

```kotlin id="jlwmk9"
lifecycleOwner.lifecycle.addObserver(observer)
```

Now observer starts receiving lifecycle callbacks.

---

## 6. Handle lifecycle events

```kotlin id="jlwmr3"
if (event == Lifecycle.Event.ON_START)
```

Calls:

```kotlin id="jlwms0"
currentOnStart()
```

Similarly:

```kotlin id="jlwmd5"
ON_STOP
```

calls:

```kotlin id="jlwmf2"
currentOnStop()
```

---

## 7. Cleanup using onDispose

```kotlin id="jlwmb9"
onDispose {
    lifecycleOwner.lifecycle.removeObserver(observer)
}
```

When composable leaves composition:

```text id="jlwmt9"
Observer removed
```

Prevents leaks and duplicate observers.

---

# Full Flow

```text id="jlwmo6"
Composable enters
      ↓
Observer registered
      ↓
Lifecycle events received
      ↓
Composable removed
      ↓
Observer unregistered
```

---

# Why is `onDispose` mandatory?

Compose must know:

```text id="jlwmn3"
How to clean up the effect
```

Without cleanup:

* resources may leak
* listeners remain active

---

# Important Note

This is BAD:

```kotlin id="jlwmu7"
onDispose { }
```

Empty cleanup usually means:

```text id="jlwmg1"
Maybe DisposableEffect is not the correct API
```

---

# Common Use Cases

* LifecycleObserver
* BroadcastReceiver
* sensor listeners
* callback registration
* subscriptions
* media player listeners

---

# Example — BroadcastReceiver

```kotlin id="jlwmy4"
DisposableEffect(Unit) {

    val receiver = MyReceiver()

    context.registerReceiver(receiver, filter)

    onDispose {
        context.unregisterReceiver(receiver)
    }
}
```

---

# Important Characteristics

✅ setup + cleanup
✅ lifecycle-aware
✅ restartable using keys
✅ prevents memory leaks

---

# Difference from `LaunchedEffect`

| LaunchedEffect              | DisposableEffect               |
| --------------------------- | ------------------------------ |
| Coroutine side effects      | Resource setup/cleanup         |
| Suspend functions           | Observer/listener management   |
| Auto coroutine cancellation | Manual cleanup using onDispose |

---

# Difference from `SideEffect`

| SideEffect               | DisposableEffect             |
| ------------------------ | ---------------------------- |
| No cleanup               | Requires cleanup             |
| Runs after recomposition | Used for resource management |

---

# Interview Point

Use `DisposableEffect` when:

```text id="jlwmq1"
Something must be registered when composable starts
and unregistered when composable ends.
```

---

# Simple Analogy

```text id="jlwmv8"
"Install CCTV camera when entering room,
remove it when leaving room."
```
