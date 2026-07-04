If you're referring to **Jetpack Compose**, its internals are very different from the traditional Android View system. Instead of creating and mutating a tree of UI widgets directly, Compose describes the UI as a function of state and efficiently updates only what changes.

Here's how it works internally.

### 1. Composable functions describe the UI

A composable is just a Kotlin function annotated with `@Composable`.

```kotlin
@Composable
fun Greeting(name: String) {
    Text("Hello $name")
}
```

This function doesn't directly draw anything. Instead, it emits instructions that Compose records.

---

### 2. The Compose Compiler transforms your code

The Compose compiler plugin rewrites your composable functions.

Your code conceptually becomes something like:

```kotlin
fun Greeting(
    name: String,
    composer: Composer,
    changed: Int
)
```

The compiler automatically inserts:

* a `Composer` parameter
* change-tracking flags
* calls that start/end groups
* logic for skipping recomposition

You never write this manually.

---

### 3. Composer records the UI

The **Composer** builds a data structure called the **Slot Table**.

Imagine your composables:

```
App
 ├── Column
 │    ├── Text
 │    └── Button
```

Instead of creating a permanent object tree immediately, Compose records:

```
Group 1: App
Group 2: Column
Group 3: Text
Group 4: Button
```

Each group stores:

* parameters
* remembered values
* state
* keys
* child relationships

This Slot Table is the heart of Compose.

---

### 4. `remember` stores values in the Slot Table

Example:

```kotlin
val counter = remember {
    mutableStateOf(0)
}
```

Compose does something like:

```
Slot 10:
    mutableStateOf(0)
```

On recomposition:

* it reaches Slot 10
* returns the existing object
* doesn't recreate it

Without `remember`, the object would be recreated every recomposition.

---

### 5. State objects are observable

```kotlin
var count by remember {
    mutableStateOf(0)
}
```

`mutableStateOf` is a special observable object.

```
count = 5
```

doesn't simply assign a value.

Internally it roughly does:

```
state.value = 5

↓

notifySnapshotSystem()

↓

scheduleRecomposition()
```

Compose now knows some UI depends on this value.

---

### 6. Snapshot system tracks reads

Compose has a **Snapshot** system.

When a composable reads:

```kotlin
Text("$count")
```

Compose records:

```
Greeting
    reads
        count
```

Later:

```
count++
```

Compose knows exactly:

> Greeting depends on count

Only that part of the UI needs recomposition.

---

### 7. Recomposition

Suppose:

```kotlin
Column {
    Text("$count")
    Button(...)
}
```

When `count` changes:

Compose does **not** rebuild everything.

It recomposes only:

```
Column
    Text   ← recomposed
    Button ← skipped
```

This is much more efficient than redrawing the entire UI.

---

### 8. Skipping unchanged composables

The compiler generates checks like:

```
if (parametersChanged) {
    executeComposable()
} else {
    skip()
}
```

For example:

```kotlin
Greeting("John")
```

If `"John"` hasn't changed and the function is stable, Compose skips executing it entirely.

This optimization is called **skipping**.

---

### 9. Layout phase

After recomposition, Compose performs layout.

Every layout follows a measure-and-place process.

```
Column
    measure()
        Text
        Button

    place()
        Text at y=0
        Button at y=40
```

Each layout receives constraints:

```
minWidth
maxWidth
minHeight
maxHeight
```

Children choose their own size within those constraints.

---

### 10. Drawing

Finally, Compose issues drawing commands.

For example:

```
Draw Text
Draw Rectangle
Draw Image
Draw Circle
```

These are translated into Android's graphics APIs (or platform-specific rendering on other Compose targets).

---

## Complete flow

```
State changes
      │
      ▼
Snapshot detects change
      │
      ▼
Schedule recomposition
      │
      ▼
Composer re-executes affected composables
      │
      ▼
Update Slot Table
      │
      ▼
Measure layouts
      │
      ▼
Place children
      │
      ▼
Draw to Canvas
```

---

## Why Compose is fast

Compose achieves good performance through several mechanisms:

* **Selective recomposition:** Only composables that depend on changed state are recomposed.
* **Skipping:** Unchanged composables are not re-executed.
* **Slot Table:** Stores composition state efficiently without rebuilding everything.
* **Snapshot system:** Tracks exactly which state each composable reads.
* **Compiler optimizations:** The Compose compiler inserts bookkeeping code automatically, reducing runtime overhead.

Unlike the traditional Android View system, which manages a mutable tree of `View` objects, Compose primarily rebuilds descriptions of the affected UI and applies only the necessary updates, leading to more efficient rendering and simpler state management.








---


# Jetpack Compose Internals: Slot Table, Groups & Slot Array

> This note explains how Compose internally connects **Groups**, the **Slot Table**, and the **Slot Array**.

---

# 1. Big Picture

Jetpack Compose does NOT build a traditional UI tree.

Instead, it maintains a single internal structure called:

```

Slot Table

```

The Slot Table contains two tightly linked parts:

- **Groups → structure (UI layout hierarchy)**
- **Slot Array → stored runtime values (remember, lambdas, etc.)**

---

# 2. Slot Table = Groups + Slots

```

Slot Table
│
├── Groups (structure of composables)
│
└── Slot Array (values stored per group)

````

---

# 3. Group (Structure Layer)

A **Group** represents one composable call.

Example:

```kotlin
@Composable
fun Example() {
    Text("Hello")
}
````

Creates a group:

```
Group: Example
```

---

## Each Group stores:

* composable identity
* parent group index
* child grouping info
* slotStart (where its values begin in Slot Array)
* slotCount (how many slots it owns)
* metadata (keys, flags, etc.)

---

## Example UI:

```
App
 └── Column
      ├── Text
      └── Button
```

---

## Groups representation:

```
Group 0 → App
Group 1 → Column
Group 2 → Text
Group 3 → Button
Group 4 → Text (inside Button)
```

Groups are stored **linearly**, not as a tree.

---

# 4. Slot Array (Value Storage Layer)

The **Slot Array** stores persistent values.

These include:

* remember values
* state objects
* lambdas (onClick)
* cached objects

---

## Example Slot Array:

```
Slot 0 → mutableStateOf(0)
Slot 1 → onClick lambda
Slot 2 → cached object
```

---

# 5. Relationship Between Group and Slot Array

Each group does NOT store values directly.

Instead, it points to a section of the Slot Array.

---

## Key concept:

```
Group → defines structure
Slots → store data for that group
```

---

## Each group has:

```
slotStart  → starting index in Slot Array
slotCount  → number of slots owned
```

---

## Example mapping:

```
Group: Button
slotStart = 1
slotCount = 2
```

Means:

```
Slot Array:
Slot 1 → value A
Slot 2 → value B
```

---

# 6. Full Example

## UI:

```kotlin
@Composable
fun Example() {
    val a = remember { 10 }
    val b = remember { 20 }

    Text("Hello")

    Button(onClick = { }) {
        Text("Click")
    }
}
```

---

## Groups:

```
0 Example
1 remember a
2 remember b
3 Text
4 Button
5 Text (inside Button)
```

---

## Slot Array:

```
0 → 10   (a)
1 → 20   (b)
2 → onClick lambda
```

---

# 7. How Access Works

When Compose executes:

### Step 1: Enter Group

```
Group → Example
slotStart = 0
```

---

### Step 2: Read remember

```
Slot Array[0] → 10
Slot Array[1] → 20
```

---

### Step 3: Recomposition

On recomposition:

* Compose re-enters SAME group
* uses same slotStart
* reuses existing slots
* does NOT recreate values

---

# 8. Why this design works

This structure allows Compose to:

### 1. Fast skipping

Skip entire groups using metadata

### 2. State retention

remember survives recomposition via slots

### 3. Efficient updates

Only changed groups are recomposed

### 4. Linear traversal

No need for tree traversal during execution

---

# 9. Simple Analogy

## Group = Folder (structure)

## Slot Array = Files (data)

```
App Folder
│
├── Column Folder
│     ├── Text Folder
│     └── Button Folder
│            ├── file (onClick)
│            └── file (state)
```

---

# 10. One-line Summary

> The Slot Table is a linear structure where **Groups define UI structure**, and the **Slot Array stores persistent values**, connected through `slotStart` and `slotCount`.


