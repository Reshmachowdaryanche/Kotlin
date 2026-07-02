
---

## How do you handle orientation changes in Jetpack Compose?

In Jetpack Compose, orientation changes (portrait ↔ landscape) cause the Activity to be recreated by default. To preserve UI state across configuration changes, I use `ViewModel` for screen-level state and `rememberSaveable` for simple UI state.

---

## 1. Using rememberSaveable

For small UI state like text fields, selected tabs, or counters:

```kotlin
@Composable
fun CounterScreen() {

    var count by rememberSaveable {
        mutableStateOf(0)
    }

    Button(
        onClick = { count++ }
    ) {
        Text("Count: $count")
    }
}
```

If the device rotates:

```
Count: 5
```

remains:

```
Count: 5
```

because `rememberSaveable` saves and restores the state.

---

## 2. Using ViewModel

For business logic and screen data:

```kotlin
class UserViewModel : ViewModel() {

    private val _name = mutableStateOf("Prasad")
    val name: State<String> = _name
}

@Composable
fun ProfileScreen(
    viewModel: UserViewModel = viewModel()
) {
    Text(viewModel.name.value)
}
```

`ViewModel` survives configuration changes, so data is not lost during rotation.

---

## 3. Detecting Current Orientation

Sometimes the UI layout itself must change.

```kotlin
val configuration = LocalConfiguration.current

if (configuration.orientation ==
    Configuration.ORIENTATION_LANDSCAPE
) {
    LandscapeLayout()
} else {
    PortraitLayout()
}
```

Example:

```
Portrait  -> Column layout
Landscape -> Row layout
```

---

## 4. Responsive Layouts (Preferred)

Instead of explicitly checking orientation, Compose encourages adaptive layouts.

```kotlin
BoxWithConstraints {
    if (maxWidth < 600.dp) {
        PhoneLayout()
    } else {
        TabletLayout()
    }
}
```

This works for:

* Orientation changes
* Tablets
* Foldables
* Multi-window mode

---

## Short Interview Answer

In Jetpack Compose, orientation changes trigger Activity recreation. To preserve state, I use `rememberSaveable` for simple UI state and `ViewModel` for screen-level and business data. If the UI needs to adapt to orientation, I use `LocalConfiguration.current` or responsive layouts with `BoxWithConstraints` and window size classes. This ensures state is retained and the UI adapts correctly across configuration changes.

---

If you want, I can also convert this into a **very short cheat-sheet version (1/4 size)** for interviews.
