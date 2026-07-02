In Jetpack Compose, user input and events are handled using:

* **State**
* **Event callbacks**
* **Modifiers**
* **Gesture detectors**

Compose follows a **unidirectional data flow** approach:

1. State flows down to UI
2. Events flow up from UI

---

# 1. Handling Text Input

Use `TextField` with state.

```kotlin id="ej39go"
@Composable
fun NameScreen() {

    var name by remember {
        mutableStateOf("")
    }

    TextField(
        value = name,
        onValueChange = {
            name = it
        },
        label = {
            Text("Enter Name")
        }
    )
}
```

---

## Explanation

* `value` → current state
* `onValueChange` → called whenever user types
* Updating state triggers recomposition

---

# 2. Handling Button Clicks

```kotlin id="a9ofdf"
Button(
    onClick = {
        println("Button Clicked")
    }
) {
    Text("Login")
}
```

`onClick` handles button events.

---

# 3. Handling Checkbox Events

```kotlin id="xqk8zk"
var checked by remember {
    mutableStateOf(false)
}

Checkbox(
    checked = checked,
    onCheckedChange = {
        checked = it
    }
)
```

---

# 4. Handling Switch Events

```kotlin id="vdf5up"
Switch(
    checked = isEnabled,
    onCheckedChange = {
        isEnabled = it
    }
)
```

---

# 5. Handling Gestures

Compose provides gesture modifiers.

## Click Gesture

```kotlin id="h3i93r"
Modifier.clickable {
    println("Clicked")
}
```

---

## Long Click

```kotlin id="y4xjpj"
Modifier.combinedClickable(
    onClick = {},
    onLongClick = {
        println("Long Click")
    }
)
```

---

# 6. Handling Pointer Input

For custom gestures:

```kotlin id="njlwm0"
Modifier.pointerInput(Unit) {
    detectTapGestures(
        onDoubleTap = {
            println("Double Tap")
        }
    )
}
```

---

# 7. Event Handling with ViewModel

Best practice is:

* UI sends events
* ViewModel handles business logic

---

## Example

```kotlin id="n0gj8q"
Button(
    onClick = {
        viewModel.login()
    }
) {
    Text("Login")
}
```

---

# 8. State Hoisting (Recommended)

Move state outside composable.

```kotlin id="v1l4l8"
@Composable
fun NameField(
    name: String,
    onNameChange: (String) -> Unit
) {
    TextField(
        value = name,
        onValueChange = onNameChange
    )
}
```

Parent controls state:

```kotlin id="rw5gwl"
var name by remember {
    mutableStateOf("")
}

NameField(
    name = name,
    onNameChange = {
        name = it
    }
)
```

---

# Why State Hoisting?

Benefits:

* Reusable composables
* Better testing
* Single source of truth
* Easier ViewModel integration

---

# Common Event Handlers in Compose

| Component  | Event                |
| ---------- | -------------------- |
| Button     | `onClick`            |
| TextField  | `onValueChange`      |
| Checkbox   | `onCheckedChange`    |
| Switch     | `onCheckedChange`    |
| Slider     | `onValueChange`      |
| LazyColumn | Scroll events        |
| Modifier   | Click/Gesture events |

---

# Interview Style Answer

> In Jetpack Compose, user input and events are handled using state and callback functions. Components like TextField, Button, Checkbox, and Switch expose event callbacks such as onValueChange and onClick. When the user interacts with the UI, these callbacks update the state, which triggers recomposition. Compose follows unidirectional data flow, where state flows down and events flow up. For complex applications, event handling is usually delegated to the ViewModel using state hoisting.
