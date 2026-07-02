# State & State Management in Jetpack Compose

## 7. What is State?

**State in Jetpack Compose is any value that can change over time and affect the UI.**

Compose follows a declarative UI model where the UI is generated based on the current state. Whenever the state changes, Compose automatically updates the affected UI through recomposition.

### Examples of State

* Text entered in a TextField
* Checkbox selection
* Loading state
* Counter value
* API response data
* Selected item in a list

---

### State Drives UI

In Jetpack Compose:

```text
UI = f(State)
```

Meaning:

* Same State → Same UI
* Different State → Different UI

Whenever state changes:

```text
State Change
      ↓
Recomposition
      ↓
UI Updated
```

Compose automatically updates the required UI components.

---

## 8. Creating and Remembering State

### 8.1 What is `remember`?

Composable functions can recompose many times. During recomposition, local variables normally get recreated.

Example:

```kotlin
@Composable
fun Counter() {
    var count = 0
}
```

During every recomposition:

```text
count becomes 0 again
```

To retain values across recompositions, Compose provides the `remember` API.

```kotlin
val state = remember { ... }
```

`remember` stores an object in the Composition memory during the initial composition and returns the stored value during recomposition.

#### Important

> `remember` retains state only as long as the composable remains in the Composition.

If the composable leaves the Composition, the remembered object is forgotten.

---

### 8.2 What is `mutableStateOf`?

`mutableStateOf()` creates an observable `MutableState<T>` object that is integrated with the Compose runtime.

Example:

```kotlin
val count = mutableStateOf(0)
```

Internally:

```kotlin
interface MutableState<T> : State<T> {
    override var value: T
}
```

Whenever the value changes:

```kotlin
count.value++
```

Compose automatically schedules recomposition for composables reading that state.

---

### 8.3 How Compose Tracks State

Example:

```kotlin
Text("$count")
```

Compose internally tracks:

```text
This composable depends on count
```

When `count` changes:

```text
State Updated
      ↓
Composable Invalidated
      ↓
Recomposition Happens
      ↓
UI Updated
```

This makes Compose highly efficient because only affected composables are recomposed.

---

## 9. Different Ways to Declare State

Compose provides multiple syntax styles for state declaration.

---

### 9.1 Direct `MutableState` Object

```kotlin
val mutableState = remember {
    mutableStateOf(default)
}
```

Here:

* `mutableState` is a `MutableState<T>` object.
* To read the value:

```kotlin
mutableState.value
```

* To update the value:

```kotlin
mutableState.value = newValue
```

#### Example

```kotlin
val countState = remember { mutableStateOf(0) }

Text(text = countState.value.toString())

Button(
    onClick = {
        countState.value++
    }
) {
    Text("Increment")
}
```

#### What is Happening Internally?

`mutableStateOf()` creates a state holder object.

Something like:

```kotlin
MutableState<Int>
```

Compose observes `.value`.

Whenever `.value` changes:

* Compose marks the composable for recomposition.
* UI updates automatically.

---

### 9.2 Property Delegation (`by`) — Most Preferred

```kotlin
var value by remember {
    mutableStateOf(default)
}
```

Here:

* `value` is not a `MutableState`.
* `value` directly contains the actual value.
* Kotlin property delegation (`by`) automatically uses `.value` internally.

So this:

```kotlin
var count by remember {
    mutableStateOf(0)
}
```

is equivalent to:

```kotlin
val countState = remember {
    mutableStateOf(0)
}

var count
    get() = countState.value
    set(value) {
        countState.value = value
    }
```

#### Example

```kotlin
var count by remember {
    mutableStateOf(0)
}

Text(text = count.toString())

Button(
    onClick = {
        count++
    }
) {
    Text("Increment")
}
```

#### Why Preferred?

* Cleaner syntax
* No `.value`
* Easier to read
* Less boilerplate

Required imports:

```kotlin
import androidx.compose.runtime.getValue
import androidx.compose.runtime.setValue
```

---

### 9.3 Destructuring Declaration

```kotlin
val (value, setValue) = remember {
    mutableStateOf(default)
}
```

This uses Kotlin destructuring.

`MutableState` provides:

```kotlin
component1() -> current value
component2() -> setter lambda
```

Internally:

```kotlin
val state = remember {
    mutableStateOf(default)
}

val value = state.value

val setValue = { newValue ->
    state.value = newValue
}
```

#### Example

```kotlin
val (count, setCount) = remember {
    mutableStateOf(0)
}

Text(text = count.toString())

Button(
    onClick = {
        setCount(count + 1)
    }
) {
    Text("Increment")
}
```

---

### Comparison Table

| Syntax                                       | Variable Type     | Read          | Update          | Common Usage             |
| -------------------------------------------- | ----------------- | ------------- | --------------- | ------------------------ |
| `val state = remember { mutableStateOf() }`  | `MutableState<T>` | `state.value` | `state.value =` | When state object needed |
| `var value by remember { mutableStateOf() }` | Actual value      | `value`       | `value =`       | Most preferred           |
| `val (value, setValue)`                      | Value + Setter    | `value`       | `setValue()`    | React-style pattern      |

---

### Which One Should You Use?

#### Most Common in Compose

```kotlin
var value by remember {
    mutableStateOf(default)
}
```

because:

* Cleaner
* Readable
* Less boilerplate

---

#### When to Use Direct `MutableState`

Useful when passing the state object itself:

```kotlin
fun MyComposable(
    state: MutableState<Int>
)
```

or when APIs expect `State<T>`.

---

#### When Destructuring is Useful

Less common in Android Compose.

Feels similar to React:

```javascript
const [value, setValue] = useState()
```

Some developers prefer it for a functional style.

---

### Important Compose Concept

All three approaches ultimately create:

```kotlin
MutableState<T>
```

and all trigger recomposition when state changes.

The difference is only:

* Syntax
* Readability
* Access style

---

### Visual Understanding

#### Style 1

```kotlin
state.value
```

You manually access the box.

---

#### Style 2

```kotlin
value
```

Kotlin automatically opens the box for you.

---

#### Style 3

```kotlin
value
setValue()
```

Compose gives you:

* Current value
* Function to update it

similar to React Hooks.

---

## 10. Using State in UI

State can control:

* UI values
* Visibility
* Conditional rendering
* Business logic

### Example

```kotlin
@Composable
fun HelloContent() {

    var name by remember {
        mutableStateOf("")
    }

    if (name.isNotEmpty()) {
        Text("Hello, $name!")
    }

    OutlinedTextField(
        value = name,
        onValueChange = {
            name = it
        }
    )
}
```

Here:

* `name` is state
* Typing updates state
* State change triggers recomposition
* UI updates automatically

---

## 11. What is `rememberSaveable`?

`remember` survives recomposition, but it does not survive configuration changes like:

* Screen rotation
* Process recreation

For this, Compose provides:

```kotlin
rememberSaveable
```

### Example

```kotlin
var text by rememberSaveable {
    mutableStateOf("")
}
```

`rememberSaveable` automatically saves values that can be stored inside a `Bundle`.

For custom objects, custom savers can be provided.

---

# State Management

## 12. How State Management Works in Compose

**State management in Jetpack Compose is based on the principle that UI is a function of state.**

Whenever the state changes, Compose automatically triggers recomposition and updates only the affected UI components.

Instead of manually updating views like in the traditional View system, Compose observes state changes and refreshes the UI automatically.

---

### Core Flow

```text
State → UI
UI Events → State Updates
State Change → Recomposition → UI Update
```

### Example

```kotlin
@Composable
fun Counter() {

    var count by remember {
        mutableStateOf(0)
    }

    Button(
        onClick = {
            count++
        }
    ) {
        Text("Count: $count")
    }
}
```

Here:

* `count` is the state
* UI displays the state
* Button click updates the state
* Compose detects the change
* Recomposition happens
* UI updates automatically

---

## 13. Stateful vs Stateless Composables

### Stateful Composable

A composable that internally owns and manages state.

It usually uses:

- remember
- mutableStateOf
- rememberSaveable

to store and update state inside the composable itself.

Example:

```kotlin
@Composable
fun Counter() {

    var count by remember {
        mutableStateOf(0)
    }

    Button(onClick = {
      count++
 }) {
      Text("Count: $count") }
}
```

The composable owns the state.

** What happens here? **
- count state is created inside the composable.
- The composable owns the state.
- Button click updates the internal state.
- State change triggers recomposition.
- UI updates automatically.

---

### Stateless Composable

A composable that receives state from outside.

Example:

```kotlin
@Composable
fun Counter(
    count: Int,
    onIncrement: () -> Unit
) {
    Button(
        onClick = onIncrement
    ) {
        Text("$count")
    }
}
```

Here:

* State comes from parent/ViewModel
* Composable only displays UI
* Better for testing and reusability

---

### Why Stateless Composables Are Preferred

* Reusable
* Testable
* Easier to maintain

Modern Compose architecture generally prefers stateless composables.

---

## 14. State Hoisting

State hoisting means moving state outside composables to a higher-level owner such as:

* Parent composable
* ViewModel

### Flow

```text
ViewModel → State → UI
UI Events → Callback → ViewModel
```

### Benefits

* Single source of truth
* Better architecture
* Easier testing
* Better reusability

---

## 15. Compose State and ViewModel

In real applications, state is usually managed inside a ViewModel.

### Example

```kotlin
class CounterViewModel : ViewModel() {

    var count by mutableStateOf(0)
        private set

    fun increment() {
        count++
    }
}
```

### UI

```kotlin
@Composable
fun CounterScreen(
    viewModel: CounterViewModel
) {

    Button(
        onClick = {
            viewModel.increment()
        }
    ) {
        Text("${viewModel.count}")
    }
}
```

Here:

* ViewModel manages state
* Compose observes state
* State changes trigger recomposition automatically

---

## 16. StateFlow and LiveData in Compose

Compose can also observe:

* StateFlow
* Flow
* LiveData

---

### StateFlow Example

```kotlin
val uiState by viewModel
    .uiState
    .collectAsState()
```

---

### LiveData Example

```kotlin
val data by liveData
    .observeAsState()
```

When new values are emitted:

```text
New State
     ↓
Recomposition
     ↓
UI Updated
```

---

## 17. Observable State vs Non-Observable State

State used in Compose should be observable.

### Bad Example

```kotlin
val list = mutableListOf<String>()
```

Normal mutable collections are not observable by Compose.

Updating them may not trigger recomposition.

---

### Recommended

```kotlin
val listState =
    mutableStateOf(
        listOf<String>()
    )
```

Prefer:

* Immutable data
* Observable state holders

---

## 18. Unidirectional Data Flow (UDF)

Compose follows Unidirectional Data Flow.

### Flow

```text
State flows downward
Events flow upward
```

### Example

```text
ViewModel
     ↓
 UI State
     ↓
Composable

User Action
     ↓
 Callback
     ↓
ViewModel
```

This architecture makes apps:

* Predictable
* Easier to debug
* Easier to test
* Easier to maintain

---

## 19. Best Practices for Compose State

Good state management in Compose should:

* Use observable state
* Keep UI state-driven
* Hoist state when needed
* Prefer immutable data
* Avoid unnecessary mutable shared state
* Keep composables stateless when possible

---

## Interview One-Liners

### What is State?

> State in Jetpack Compose is any observable value that can change over time and affect the UI. When state changes, Compose automatically triggers recomposition and updates only the affected UI components.

---

### What is `remember`?

> `remember` stores an object in the Composition and retains it across recompositions as long as the composable remains in the Composition.

---

### What is `mutableStateOf`?

> `mutableStateOf()` creates an observable state holder that automatically triggers recomposition when its value changes.

---

### What is State Management?

> State management in Jetpack Compose works by storing observable state values and automatically triggering recomposition whenever the state changes, allowing the UI to stay synchronized with the latest state in a declarative and efficient way.

---

### What is State Hoisting?

> State hoisting is the practice of moving state to a higher-level owner and passing state down while sending events upward through callbacks.

---

### What is Unidirectional Data Flow?

> Unidirectional Data Flow means state flows downward to the UI and events flow upward to the state owner, creating a predictable and maintainable architecture.
