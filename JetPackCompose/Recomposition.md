# Recomposition

## 6. What is Recomposition?

**Recomposition is the process where Jetpack Compose re-executes composable functions when their input state changes in order to update the UI.**

In traditional imperative UI systems, when data changes, developers manually update the affected widgets using setters like:

* `setText()`
* `setVisibility()`
* `notifyDataSetChanged()`

However, in Jetpack Compose, UI is declarative and state-driven.

Instead of manually modifying widgets, we call composable functions again with the updated state. Compose then intelligently updates only the necessary parts of the UI.

This process of re-executing composable functions with new state is called **recomposition**.

---

## 6.1 Example of Recomposition

```kotlin
@Composable
fun ClickCounter(
    clicks: Int,
    onClick: () -> Unit
) {
    Button(onClick = onClick) {
        Text("I've been clicked $clicks times")
    }
}
```

In this example:

* `clicks` is the state.
* Whenever the button is clicked, the value of `clicks` changes.
* Compose calls the composable again with the updated value.
* The `Text()` composable gets recomposed and displays the latest count.

### Flow

```text
State Change
      ↓
 Recomposition
      ↓
   UI Updated
```

Compose automatically updates the UI without developers manually changing views.

---

## 6.2 Intelligent Recomposition

One of the biggest advantages of Compose is that recomposition is intelligent and optimized.

Rebuilding the entire UI tree every time state changes would be computationally expensive and would consume more CPU and battery.

Compose solves this problem by recomposing only the composables whose inputs have changed and skipping the rest.

### Example

```kotlin
Text(header)
```

If only the list changes and `header` remains the same:

* Compose skips recomposing `Text(header)`
* Only the list-related composables are recomposed

This selective recomposition is one of the core reasons why Compose performs efficiently.

---

## 6.3 Recomposition Skips Unchanged UI

Compose tries to skip as much work as possible during recomposition.

Every composable function or lambda can independently recompose.

### Example

```kotlin
@Composable
fun NamePicker(
    header: String,
    names: List<String>
)
```

If:

* `header` changes → only header-related composables recompose
* `names` changes → only list-related composables recompose

Compose avoids recomposing unaffected UI sections.

This is very important for performance because large UI trees can still update efficiently.

---

## 6.4 Recomposition is Optimistic

Recomposition in Compose is optimistic.

This means Compose assumes recomposition will complete successfully.

However, while recomposition is happening, state may change again.

In such cases, Compose may:

* Cancel the current recomposition
* Discard partially built UI
* Restart recomposition with the latest state

### Flow

```text
Recomposition Started
         ↓
 State Changed Again
         ↓
 Current Recomposition Cancelled
         ↓
 New Recomposition Started
```

Because recompositions can be canceled midway, composables should never depend on side-effects.

---

## 6.5 Why Composables Should Be Side-Effect Free

A side-effect is any operation that changes state outside the composable.

### Dangerous Side Effects

* Updating global variables
* Writing to SharedPreferences
* Updating ViewModel state directly
* Database writes
* Network calls

Compose may:

* Recompose frequently
* Skip recompositions
* Cancel recompositions
* Execute composables in different orders

Because of this, side-effects inside composables can create inconsistent or unpredictable behavior.

---

### Bad Example of Side Effects

```kotlin
var count = 0

@Composable
fun Counter() {
    count++
}
```

Since recomposition may happen multiple times:

```text
Counter()
Counter()
Counter()
```

`count` may increase unexpectedly.

This creates unstable UI behavior.

---

### Good Compose Practice

Instead of performing side-effects inside composables:

* Move business logic to ViewModel
* Perform expensive work in coroutines
* Pass state into composables
* Use callbacks like `onClick`

Example:

```kotlin
@Composable
fun SharedPrefsToggle(
    text: String,
    value: Boolean,
    onValueChanged: (Boolean) -> Unit
)
```

Here:

* Composable only displays UI
* ViewModel handles SharedPreferences update
* UI remains side-effect free

This is the recommended Compose architecture.

---

## 6.6 Composable Functions Can Run Frequently

Composable functions may execute very frequently.

During animations, recomposition can happen every frame.

### Example

```text
60 FPS Animation
        ↓
Composable may execute
60 times per second
```

Because of this:

* Composables should be lightweight
* Expensive operations should be avoided
* Heavy work should happen outside composition

### Bad Example

```kotlin
@Composable
fun Screen() {
    val prefs = sharedPreferences.getString(...)
}
```

This may repeatedly read storage during recompositions and cause UI lag or jank.

---

## 6.7 Recomposition and State Observation

Compose automatically tracks state reads.

### Example

```kotlin
Text("$count")
```

Compose internally remembers:

```text
This composable depends on count
```

When `count` changes:

```text
count updated
      ↓
Only affected composables recomposed
```

This is how Compose achieves efficient UI updates.

---

## 6.8 Composable Functions Can Execute in Parallel

Compose is currently mostly single-threaded, but it is designed with future multithreading support in mind.

In the future, Compose may execute composables in parallel on multiple threads for better performance.

This means:

* Composables should not modify shared mutable variables
* They should remain thread-safe
* They should only transform state into UI

---

### Bad Parallel Execution Example

```kotlin
@Composable
fun ListWithBug(myList: List<String>) {

    var items = 0

    Column {
        for (item in myList) {
            items++
        }
    }

    Text("Count: $items")
}
```

This is dangerous because recomposition may happen multiple times or potentially on different threads in the future.

The UI may display incorrect counts.

---

## 6.9 Composable Functions Can Execute in Any Order

Compose does not guarantee execution order between composable functions.

### Example

```kotlin
StartScreen()
MiddleScreen()
EndScreen()
```

Compose may execute them in any order depending on optimization priorities.

Therefore:

* One composable should never depend on another composable's side-effects
* Every composable should be self-contained

---

## 6.10 Key Characteristics of Recomposition

Recomposition is:

* Automatic
* State-driven
* Optimized
* Selective
* Cancelable
* Frequent
* Potentially parallel in the future

---

## Recomposition Summary

### What Triggers Recomposition?

```text
Observable State Change
          ↓
Composable Invalidated
          ↓
Recomposition Scheduled
          ↓
Affected UI Updated
```

### What Happens During Recomposition?

* Compose re-executes composable functions.
* Compose compares the new UI description with the previous one.
* Unchanged parts are skipped.
* Changed parts are updated.
* Only affected UI is refreshed.

### Why is Recomposition Efficient?

Because Compose:

* Tracks state reads automatically
* Rebuilds only affected UI
* Skips unchanged composables
* Cancels unnecessary work when newer state arrives

---

## Interview One-Liner

> Recomposition is the process where Jetpack Compose re-executes composable functions whenever observed state changes and intelligently updates only the affected parts of the UI instead of redrawing the entire screen.
