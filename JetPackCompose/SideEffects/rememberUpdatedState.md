# `rememberUpdatedState`

## Purpose

Used to:

* get the latest value inside an effect
* without restarting the effect

---

# Problem It Solves

Normally:

```kotlin id="jlwm94"
LaunchedEffect(key)
```

restarts whenever the key changes.

Sometimes this is bad.

You may want:

✅ latest value
❌ effect restart

---

# Mental Model

```text id="jlwm70"
Keep effect running
      ↓
Always use latest value
      ↓
Without restarting effect
```

---

# Why is this needed?

Effects like:

* timers
* delays
* animations
* long-running coroutines

can be expensive or wrong to restart.

---

# Example

```kotlin id="jlwmp5"
@Composable
fun LandingScreen(onTimeout: () -> Unit) {

    // Always holds latest lambda

    val currentOnTimeout by rememberUpdatedState(onTimeout)

    // Runs once for this composable lifecycle

    LaunchedEffect(true) {

        delay(SplashWaitTimeMillis)

        currentOnTimeout()
    }

    /* Landing screen content */
}
```

---

# Step-by-step Explanation

---

## 1. onTimeout callback

```kotlin id="jlwmd1"
onTimeout: () -> Unit
```

Usually something like:

```kotlin id="jlwmx9"
navigateToHome()
```

---

## 2. Problem without rememberUpdatedState

Suppose we write:

```kotlin id="jlwmu3"
LaunchedEffect(true) {
    delay(3000)
    onTimeout()
}
```

Looks fine initially.

But coroutine captures OLD value of `onTimeout`.

---

# Problem Scenario

---

## Initial composition

```text id="jlwmz2"
onTimeout = navigateV1
```

Coroutine starts.

---

## Recomposition happens

```text id="jlwmb0"
onTimeout = navigateV2
```

BUT coroutine still remembers:

```text id="jlwmk4"
navigateV1
```

After delay:

```text id="jlwmg9"
OLD callback executes
```

This is called:

```text id="jlwmt2"
stale capture
```

---

# Why not add onTimeout as key?

You may think:

```kotlin id="jlwmj8"
LaunchedEffect(onTimeout)
```

But then:

```text id="jlwmu6"
onTimeout changes
      ↓
Effect restarts
      ↓
delay starts again
```

Splash timer resets.

Bad UX.

---

# Solution: rememberUpdatedState

```kotlin id="jlwmo7"
val currentOnTimeout by rememberUpdatedState(onTimeout)
```

This creates a reference that:

✅ always updates to latest value
✅ without restarting effect

---

# What happens now?

---

## Initial composition

```text id="jlwmc5"
currentOnTimeout → V1
```

Effect starts.

---

## Recomposition happens

```text id="jlwmm7"
currentOnTimeout → V2
```

BUT:

```text id="jlwmn1"
LaunchedEffect does NOT restart
```

---

## Delay finishes

```kotlin id="jlwmr4"
currentOnTimeout()
```

Now latest callback executes:

```text id="jlwmh6"
V2
```

Perfect.

---

# Why use `LaunchedEffect(true)`?

```kotlin id="jlwmf3"
LaunchedEffect(true)
```

means:

```text id="jlwmq0"
Run effect once
for composable lifecycle
```

Because `true` never changes.

Equivalent to:

```text id="jlwms8"
Start when screen appears
Stop when screen disappears
Never restart during recomposition
```

---

# Important Point

`rememberUpdatedState`:

❌ does NOT stop recomposition
❌ does NOT stop state updates

It ONLY helps effects use latest value without restart.

---

# Common Use Cases

* splash screen timers
* delayed navigation
* long-running animations
* timeout handlers
* expensive effects

---

# Important Characteristics

✅ latest value access
✅ avoids stale captures
✅ prevents unnecessary effect restart
✅ useful with long-running effects

---

# Difference from LaunchedEffect keys

| Put value in key         | rememberUpdatedState       |
| ------------------------ | -------------------------- |
| Effect restarts          | Effect keeps running       |
| New coroutine created    | Same coroutine continues   |
| Good when restart needed | Good when restart is wrong |

---

# Interview Point

`rememberUpdatedState` is used when:

```text id="jlwmt7"
"I need latest value inside effect,
but restarting effect would be expensive or incorrect."
```

---

# Simple Analogy

```text id="jlwmv5"
Timer keeps running,
but the phone number to call
gets updated continuously.
```
