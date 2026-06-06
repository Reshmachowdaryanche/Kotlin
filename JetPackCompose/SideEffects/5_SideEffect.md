# `SideEffect`

## Purpose

Used to:

```text id="jlwmu1"
Publish Compose state to external/non-Compose objects
```

Examples:

* analytics SDK
* logging system
* external Java/Kotlin objects
* legacy APIs

---

# Why do we need SideEffect?

Compose state is managed inside Compose.

But sometimes external systems also need updated values.

`SideEffect` safely synchronizes:

```text id="jlwmt4"
Compose State
      ↓
External Object / SDK
```

---

# Key Points

```kotlin id="jlwmg2"
SideEffect {

}
```

* runs after every successful recomposition
* used for external side effects
* no coroutine
* no suspend functions
* no cleanup
* no keys

---

# Important Rule

`SideEffect` runs:

✅ AFTER successful recomposition

NOT during composition.

---

# Why is this important?

Composable functions should ideally be:

```text id="jlwmo9"
Side-effect free
```

If you directly update external objects inside composable body:

```kotlin id="jlwmy6"
analytics.setUserProperty(...)
```

it may run:

* too early
* during failed composition
* during cancelled recomposition

This can cause inconsistent external state.

---

# SideEffect guarantees

```text id="jlwmb2"
Composition completed successfully
      ↓
Now execute external side effect
```

---

# Example

```kotlin id="jlwmm4"
@Composable
fun rememberFirebaseAnalytics(user: User): FirebaseAnalytics {

    val analytics: FirebaseAnalytics = remember {
        FirebaseAnalytics()
    }

    SideEffect {

        analytics.setUserProperty(
            "userType",
            user.userType
        )
    }

    return analytics
}
```

---

# Step-by-step Explanation

---

## 1. Create analytics object

```kotlin id="jlwmu8"
val analytics = remember {
    FirebaseAnalytics()
}
```

`remember` ensures object is created only once.

Without `remember`:

```text id="jlwmg7"
New FirebaseAnalytics object
on every recomposition
```

Bad.

---

## 2. SideEffect runs

```kotlin id="jlwms2"
SideEffect {

}
```

Runs after every successful recomposition.

---

## 3. Update external object

```kotlin id="jlwmd8"
analytics.setUserProperty(
    "userType",
    user.userType
)
```

This updates analytics SDK with latest user type.

Example:

```text id="jlwmo3"
premium
guest
admin
```

Now all future analytics events include this metadata.

---

# Full Flow

```text id="jlwmt6"
Compose state changes
      ↓
Recomposition happens
      ↓
Composition succeeds
      ↓
SideEffect runs
      ↓
External SDK updated
```

---

# Why not directly call analytics?

BAD:

```kotlin id="jlwmy3"
@Composable
fun Screen(user: User) {

    analytics.setUserProperty(
        "type",
        user.type
    )
}
```

Problem:

Composable execution can be:

* cancelled
* skipped
* restarted

External side effect becomes unreliable.

---

# SideEffect guarantees safe timing

```text id="jlwmb7"
Only run after successful recomposition
```

---

# Common Use Cases

* analytics updates
* logging
* updating SDK configuration
* updating Activity title
* syncing Compose state externally

---

# Example — Activity Title

```kotlin id="jlwmm9"
SideEffect {
    activity.title = screenTitle
}
```

---

# Important Characteristics

✅ runs after successful recomposition
✅ safe external synchronization
✅ lifecycle-aware with composition
✅ useful for non-Compose objects

---

# What SideEffect does NOT support

❌ suspend functions
❌ coroutines
❌ cleanup
❌ restart keys

---

# Difference from `LaunchedEffect`

| LaunchedEffect             | SideEffect                      |
| -------------------------- | ------------------------------- |
| Coroutine-based            | No coroutine                    |
| Supports suspend functions | No suspend functions            |
| Async work                 | External object synchronization |

---

# Difference from `DisposableEffect`

| DisposableEffect   | SideEffect               |
| ------------------ | ------------------------ |
| Setup + cleanup    | Publish state externally |
| Requires onDispose | No cleanup               |

---

# Interview Point

Use `SideEffect` when:

```text id="jlwmg0"
Compose state must be shared
with external/non-Compose code safely.
```

---

# Simple Analogy

```text id="jlwmt1"
"After UI updates successfully,
inform external systems about latest state."
```
