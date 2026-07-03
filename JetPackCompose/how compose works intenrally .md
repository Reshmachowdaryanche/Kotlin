# Jetpack Compose Internals (Interview Notes)

## What is Jetpack Compose?

Jetpack Compose is Android's **declarative UI toolkit**.

Instead of telling the UI **how to update**, we describe **what the UI should look like** for the current state.

> **UI = f(State)**

Whenever the state changes, Compose automatically updates only the affected parts of the UI.

---

# Internal Working Flow

```
Composable Function
        │
        ▼
Compiler Plugin
        │
        ▼
Composer
        │
        ▼
Slot Table
        │
        ▼
Snapshot State System
        │
State Changes
        │
        ▼
Recomposer
        │
        ▼
Recomposition
        │
        ▼
Layout
        │
        ▼
Draw
```

---

# Step 1: Compiler Plugin

Every function annotated with `@Composable`

```kotlin
@Composable
fun Greeting(name: String) {
    Text("Hello $name")
}
```

is transformed by the Compose Compiler.

It converts the function into something similar to:

```kotlin
fun Greeting(
    name: String,
    composer: Composer,
    changed: Int
)
```

The compiler also inserts code to

- track parameter changes
- record state reads
- support recomposition
- support skipping

This generated code is invisible to developers.

---

# Step 2: Composition

Composition is the process of executing composable functions for the first time.

Example

```kotlin
Column {
    Text("Hello")
    Button { }
}
```

Compose executes these composables and creates a tree internally.

```
Column
 ├── Text
 └── Button
```

During composition, the runtime records this information in the **Slot Table**.

---

# Step 3: Slot Table

The Slot Table is Compose's internal memory.

It stores

- composable hierarchy
- remembered objects
- parameter values
- state references
- keys
- group information

Example

```kotlin
@Composable
fun Counter() {
    var count by remember {
        mutableStateOf(0)
    }

    Text("$count")
}
```

Internally

```
Counter
    remember -> MutableState(0)
    Text
```

When recomposition happens, Compose reuses this information.

---

# Step 4: remember()

Without remember

```kotlin
val random = Random()
```

Every recomposition creates a new object.

With remember

```kotlin
val random = remember {
    Random()
}
```

Compose stores the object in the Slot Table.

Later recompositions simply retrieve it.

---

# Step 5: Snapshot State System

Compose uses observable state.

```kotlin
var count by remember {
    mutableStateOf(0)
}
```

Whenever

```kotlin
count++
```

is executed,

the Snapshot system detects the change.

It then informs the Recomposer.

---

# Step 6: State Read Tracking

Suppose

```kotlin
Text("$count")
```

During composition Compose records

```
Text
   reads count
```

Now Compose knows

```
count changes
        ↓
only Text depends on it
```

instead of recomposing the whole screen.

---

# Step 7: Recomposer

The Recomposer watches for state changes.

Whenever state changes

```
count = count + 1
```

it schedules recomposition.

The entire screen is **not** rebuilt.

Only affected composables are executed again.

---

# Step 8: Recomposition

Example

```kotlin
Column {
    Header()
    Counter()
    Footer()
}
```

Suppose Counter's state changes.

Compose recomposes only

```
Counter()
```

Header and Footer are skipped.

---

# Step 9: Skipping

Compose checks whether parameters changed.

Example

```kotlin
Greeting(name)
```

Old value

```
John
```

New value

```
John
```

Since nothing changed,

Compose skips executing `Greeting()`.

This makes recomposition very fast.

Stable and immutable objects improve skipping.

---

# Step 10: Layout Phase

After recomposition,

Compose performs layout.

Every layout has two steps.

## Measure

Parent sends constraints.

```
Child.measure(constraints)
```

Child returns

```
width
height
```

---

## Placement

Parent positions children.

```
child.place(x, y)
```

---

# Step 11: Draw Phase

After layout,

Compose draws UI.

```
Canvas

Draw Background

Draw Text

Draw Images

Draw Shapes
```

Only changed nodes are redrawn.

---

# Applier

Compose itself doesn't know Android Views.

Instead it generates operations like

```
Insert Node

Remove Node

Move Node

Update Node
```

The **Applier** converts these into platform-specific UI updates.

For Android, this means updating the underlying rendering nodes.

---

# Complete Lifecycle

```
Composable Called
        │
        ▼
Compiler Generated Code
        │
        ▼
Composer Executes
        │
        ▼
Slot Table Created
        │
        ▼
State Read Tracking
        │
        ▼
Snapshot State
        │
User Updates State
        │
        ▼
Snapshot Detects Change
        │
        ▼
Recomposer Schedules Work
        │
        ▼
Affected Composables Recompose
        │
        ▼
Skip Unchanged Composables
        │
        ▼
Layout
        │
        ▼
Draw
```

---

# Key Components

| Component | Responsibility |
|------------|----------------|
| Compose Compiler | Transforms @Composable functions into runtime-aware code |
| Composer | Executes composables and builds the composition |
| Slot Table | Stores composition information and remembered state |
| Snapshot State | Observes state changes |
| Recomposer | Schedules recomposition |
| remember | Persists objects across recompositions |
| Applier | Applies UI changes to the platform |
| Layout | Measures and places UI elements |
| Draw | Renders UI |

---

# Why Compose is Fast

- Declarative UI
- No XML inflation
- Fine-grained state observation
- Recompose only affected composables
- Skips unchanged composables
- Lightweight Slot Table instead of a large View hierarchy
- Efficient layout and drawing

---

# Interview Answer (2–3 Minutes)

> Jetpack Compose is a declarative UI toolkit where the UI is a function of state. When a composable is first called, the Compose Compiler transforms it into runtime-aware code. The Composer executes it and records the UI structure in the Slot Table. During composition, Compose tracks which state each composable reads using the Snapshot state system. When a state value changes, the Snapshot system notifies the Recomposer, which schedules recomposition. Instead of rebuilding the whole screen, Compose re-executes only the composables that depend on the changed state. If a composable's stable inputs haven't changed, it is skipped. After recomposition, Compose performs layout (measure and place) and then draws only the necessary updates. This state-driven approach reduces unnecessary work and improves UI performance.

---

# Common Interview Questions

### What is Composition?
The initial execution of composable functions to build the UI tree.

### What is Recomposition?
Re-executing only the composables affected by state changes.

### What is the Slot Table?
Compose's internal data structure that stores the composition, remembered values, and metadata needed for recomposition.

### What is the Snapshot System?
The observable state system that tracks reads and writes to state objects and triggers recomposition when values change.

### What is `remember`?
A mechanism that stores an object in the Slot Table so it survives recompositions.

### What is the Recomposer?
A runtime component that listens for state changes and schedules recomposition.

### Why doesn't Compose recompose the whole screen?
Because it tracks which composables read which state values and recomposes only those that depend on changed state.

### Why is Compose faster than the View system?
- No XML inflation
- Fine-grained state observation
- Selective recomposition
- Skipping of unchanged composables
- Lightweight runtime data structures
- Minimal UI updates
