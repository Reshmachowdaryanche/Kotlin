# `LaunchedEffect`

## Purpose

Used to:

* run coroutines inside a composable
* call suspend functions
* perform async work tied to composable lifecycle

---

# Key Points

```kotlin id="7jftjlwm"
LaunchedEffect(key) {

}
```

* launches a coroutine when composable enters composition
* coroutine is automatically cancelled when composable leaves composition
* if key changes:

  * old coroutine is cancelled
  * new coroutine starts

---

# Mental Model

```text id="8vjlwm"
Composable appears
      ↓
Coroutine starts
      ↓
Suspend functions run
      ↓
Composable removed
      ↓
Coroutine cancelled automatically
```

---

# Example

```kotlin id="jlwmz1"
// Allow the pulse rate to be configured, so it can be sped up if the user is running
// out of time

var pulseRateMs by remember { mutableLongStateOf(3000L) }

val alpha = remember { Animatable(1f) }

LaunchedEffect(pulseRateMs) {

    // Restart the effect when the pulse rate changes

    while (isActive) {

        // Wait for pulseRateMs milliseconds

        delay(pulseRateMs)

        // Fade out

        alpha.animateTo(0f)

        // Fade in

        alpha.animateTo(1f)
    }
}
```

---

# Step-by-step Explanation

---

## 1. State for pulse speed

```kotlin id="jlwmk3"
var pulseRateMs by remember {
    mutableLongStateOf(3000L)
}
```

Initial value:

```text id="vjlwm7"
3000 milliseconds = 3 seconds
```

Controls how frequently animation happens.

---

## 2. Create Animatable

```kotlin id="jlwm52"
val alpha = remember {
    Animatable(1f)
}
```

`alpha` controls opacity:

| Value | Meaning           |
| ----- | ----------------- |
| `1f`  | fully visible     |
| `0f`  | fully transparent |

---

## 3. LaunchedEffect starts coroutine

```kotlin id="jlwmo2"
LaunchedEffect(pulseRateMs)
```

When composable enters composition:

```text id="jlwm44"
Coroutine starts automatically
```

Key is:

```kotlin id="jlwm0q"
pulseRateMs
```

If pulse rate changes:

```text id="jlwmc1"
Old coroutine cancelled
New coroutine started
```

---

## 4. while(isActive)

```kotlin id="jlwm5f"
while (isActive)
```

Loop keeps running while coroutine is active.

When composable leaves composition:

```text id="jlwmk8"
Coroutine cancelled
isActive becomes false
Loop stops
```

---

## 5. delay()

```kotlin id="jlwmj6"
delay(pulseRateMs)
```

Suspend function.

Waits for:

```text id="9jlwmq"
pulseRateMs milliseconds
```

---

## 6. Animate alpha

```kotlin id="jlwmr0"
alpha.animateTo(0f)
```

Fade out:

```text id="jlwm4g"
1f → 0f
```

Then:

```kotlin id="jlwmf4"
alpha.animateTo(1f)
```

Fade in:

```text id="jlwm9t"
0f → 1f
```

---

# Full Flow

```text id="jlwmu2"
Wait 3 seconds
    ↓
Fade out
    ↓
Fade in
    ↓
Repeat forever
```

---

# Why use LaunchedEffect here?

Because:

```kotlin id="jlwm2m"
delay()
animateTo()
```

are suspend functions.

Suspend functions require a coroutine.

`LaunchedEffect` provides a coroutine scope tied to composable lifecycle.

---

# Important Lifecycle Behavior

```text id="jlwmm6"
Composable removed from screen
        ↓
Coroutine automatically cancelled
        ↓
Animation stops
```

Prevents memory leaks.

---

# What happens if pulseRateMs changes?

Example:

```kotlin id="jlwmz8"
pulseRateMs = 1000L
```

Then:

```text id="jlwmn7"
Old coroutine cancelled
New coroutine started
Animation now pulses every 1 second
```

---

# Common Use Cases

* API calls
* timers
* animations
* Flow collection
* startup work
* delayed navigation

---

# Important Characteristics

✅ supports suspend functions
✅ lifecycle-aware
✅ automatically cancelled
✅ restartable using keys

---

# Interview Point

`LaunchedEffect(key)` means:

```text id="jlwmy9"
Run coroutine for this composable
and restart when key changes
```

---

# Simple Analogy

```text id="jlwmv0"
"Automatically start work when screen appears,
and stop when screen disappears."
```
