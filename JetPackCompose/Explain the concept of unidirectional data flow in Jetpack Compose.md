# Unidirectional Data Flow (UDF) in Jetpack Compose

## Definition

Unidirectional Data Flow (UDF) is a design pattern in Jetpack Compose where data moves in **one direction only**:

* **State flows down** from parent to child composables.
* **Events flow up** from child composables to parent composables.

This makes UI behavior predictable, maintainable, and easier to debug.

---

## Flow Diagram

```text
State → UI → User Action → Event → State Update → Recomposition
```

### Working Process

1. State is stored in a parent composable or ViewModel.
2. State is passed to child composables as parameters.
3. User interacts with the UI.
4. Child composable sends an event through a callback.
5. Parent/ViewModel updates the state.
6. Compose recomposes the UI with the updated state.

---

## Example

```kotlin
@Composable
fun CounterScreen() {
    var count by remember { mutableStateOf(0) }

    Counter(
        count = count,
        onIncrement = { count++ }
    )
}

@Composable
fun Counter(
    count: Int,
    onIncrement: () -> Unit
) {
    Column {
        Text("Count: $count")

        Button(onClick = onIncrement) {
            Text("Increment")
        }
    }
}
```

---

## State Hoisting

State Hoisting is the practice of moving state ownership to a higher-level composable and passing it down as parameters.

```kotlin
@Composable
fun Counter(
    count: Int,
    onIncrement: () -> Unit
)
```

### Benefits

* Reusable composables
* Better testability
* Clear separation of UI and logic
* Easier state management

---

## UDF with ViewModel

```kotlin
class CounterViewModel : ViewModel() {
    private val _count = MutableStateFlow(0)
    val count = _count.asStateFlow()

    fun increment() {
        _count.value++
    }
}
```

```kotlin
@Composable
fun CounterScreen(viewModel: CounterViewModel) {
    val count by viewModel.count.collectAsState()

    Counter(
        count = count,
        onIncrement = viewModel::increment
    )
}
```

### Flow

```text
ViewModel State
      ↓
Composable UI
      ↓
User Action
      ↓
ViewModel Event Handler
      ↓
State Update
      ↓
Recomposition
```

---

## Advantages

* Predictable state management
* Single source of truth
* Easier debugging
* Better testability
* Improved maintainability
* Automatic UI updates through recomposition

---

## Key Principle

> **State flows down, Events flow up.**

This principle forms the foundation of Unidirectional Data Flow in Jetpack Compose and helps build scalable, maintainable Android applications.
