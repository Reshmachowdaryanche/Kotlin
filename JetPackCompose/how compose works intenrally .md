Jetpack Compose works by replacing the traditional XML + imperative UI approach with a **declarative UI runtime**. Instead of telling Android *how* to update views, you describe *what* the UI should look like for a given state. The Compose runtime determines what actually needs to change.

Here's a breakdown of what happens internally.

## 1. `@Composable` functions are transformed by the compiler

When you write:

```kotlin
@Composable
fun Greeting(name: String) {
    Text("Hello $name")
}
```

The Compose compiler rewrites it into something conceptually like:

```kotlin
fun Greeting(
    name: String,
    composer: Composer,
    changed: Int
) {
    // generated runtime code
}
```

The compiler inserts:

* A hidden `Composer` parameter
* Flags to detect parameter changes
* Calls that record the UI structure
* Logic for skipping recomposition when possible

You never see this generated code.

---

## 2. Compose builds a Slot Table

Unlike the View system, Compose does **not** create a `View` for every composable.

Instead, it stores UI structure in a data structure called the **Slot Table**.

For example:

```kotlin
Column {
    Text("A")
    Text("B")
}
```

Internally becomes something like:

```
Root
 └── Column
      ├── Text("A")
      └── Text("B")
```

The Slot Table stores:

* composable hierarchy
* remembered values
* state locations
* keys
* parameter information

Think of it as a lightweight tree describing the UI.

---

## 3. Composition phase

During the first render:

```kotlin
setContent {
    Greeting("John")
}
```

Compose executes every composable.

Execution looks like:

```
Greeting()

    ↓

Text()

    ↓

Layout()

    ↓

Draw()
```

Every composable registers itself with the Composer.

The runtime records

* where it is
* what parameters it received
* what state it depends on

This process is called **Composition**.

---

## 4. State observation

Suppose:

```kotlin
var count by remember {
    mutableStateOf(0)
}
```

Internally:

```
MutableState<Int>
      |
      |
 Snapshot System
      |
      |
 Composer observes reads
```

Whenever you read

```kotlin
Text("$count")
```

Compose records

```
This Text depends on count
```

This dependency tracking is automatic.

---

## 5. Snapshot system

Compose has its own state management engine called the **Snapshot System**.

Instead of directly storing values:

```
count = 0
```

It stores

```
MutableState

value = 0
version = 1
```

When you do

```kotlin
count++
```

the snapshot:

```
version++

notify observers
```

Every composable that previously read that state becomes invalid.

---

## 6. Recomposition

Suppose:

```kotlin
Column {

    Text("$count")

    Button(...)
}
```

When

```kotlin
count++
```

Compose does **not** execute everything again.

Only the affected parts are recomposed.

```
Before

Column
   Text
   Button

After

Column
   Text   ← recomposed
   Button ← skipped
```

This is called **selective recomposition**.

---

## 7. Skipping unchanged composables

Consider:

```kotlin
Greeting(name)
```

If `name` hasn't changed, the compiler-generated code checks change flags and can skip executing the function.

Conceptually:

```kotlin
if (parameterChanged) {
    Greeting(...)
} else {
    skip()
}
```

Skipping is one of the main reasons Compose performs well.

---

## 8. `remember`

Example:

```kotlin
val random = remember {
    Random.nextInt()
}
```

Without `remember`:

```
Every recomposition

↓

Random.nextInt()

↓

different value
```

With `remember`:

```
First composition

↓

store value in Slot Table

↓

Later recomposition

↓

reuse stored value
```

The value lives in the Slot Table until the composable leaves the composition.

---

## 9. Layout phase

After composition, Compose measures layouts.

Every layout implements a measure policy.

Example:

```kotlin
Column {
    Text(...)
    Button(...)
}
```

Measurement proceeds roughly as:

```
Column.measure()

↓

measure Text

↓

measure Button

↓

calculate own size
```

The parent supplies constraints:

```
minWidth
maxWidth

minHeight
maxHeight
```

Children return their measured sizes.

---

## 10. Placement

After measurement:

```
Column

Text at y = 0

Button at y = 40
```

Compose determines where each child should be placed.

---

## 11. Drawing

Finally:

```
Canvas

↓

draw Text

↓

draw Button

↓

draw background
```

Compose renders using Android's graphics system rather than creating a separate `View` for each composable.

---

## 12. What happens when state changes?

Example:

```kotlin
var count by remember {
    mutableStateOf(0)
}

Text("$count")

Button {
    count++
}
```

Flow:

```
Button Click
      │
      ▼
count++
      │
      ▼
Snapshot version changes
      │
      ▼
Composer notified
      │
      ▼
Invalidate Text composable
      │
      ▼
Recompose Text only
      │
      ▼
Measure if needed
      │
      ▼
Draw updated UI
```

The rest of the UI remains unchanged unless it also depends on `count`.

---

## 13. Side effects

Composable functions should be free of side effects, so Compose provides dedicated APIs:

* `remember` → retain values across recompositions.
* `LaunchedEffect` → run suspend functions tied to the composition lifecycle.
* `DisposableEffect` → register and clean up resources when entering/leaving the composition.
* `SideEffect` → publish state to objects outside Compose after a successful recomposition.
* `produceState` → bridge asynchronous data into Compose state.

This separation keeps recomposition predictable.

---

## 14. End-to-end flow

```text
Kotlin Code
      │
      ▼
Compose Compiler Plugin
      │
      ▼
Generated Composer Calls
      │
      ▼
Composition
      │
      ▼
Slot Table Created
      │
      ▼
Snapshot State Observed
      │
      ▼
Measure
      │
      ▼
Place
      │
      ▼
Draw
      │
      ▼
State Changes
      │
      ▼
Selective Recomposition
      │
      ▼
Only Changed UI Redrawn
```

## How this differs from the View system

| Android Views                                      | Jetpack Compose                             |
| -------------------------------------------------- | ------------------------------------------- |
| XML defines UI                                     | Kotlin code defines UI                      |
| One `View` object per widget                       | Lightweight composition tree and slot table |
| Manual updates (`textView.text = ...`)             | UI automatically updates when state changes |
| `findViewById()` and view references               | Direct function calls and state             |
| Entire view hierarchy can be traversed for updates | Only affected composables are recomposed    |

The key performance ideas behind Compose are:

* **Compiler-generated code** that tracks changes.
* A **Slot Table** that stores composition information and remembered values.
* A **Snapshot** state system that automatically observes reads and writes.
* **Selective recomposition**, so only composables affected by state changes are re-executed.
* Separate **composition**, **layout**, and **drawing** phases, allowing work to be minimized when only certain aspects of the UI change.
