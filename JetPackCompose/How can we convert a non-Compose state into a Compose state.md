# How can we convert a non-Compose state into a Compose state?

In Jetpack Compose, UI automatically updates only when it observes:

```kotlin id="jlwmu4"
State<T>
```

But many external data sources are not Compose State.

Examples:

* Flow
* StateFlow
* LiveData
* RxJava
* callbacks
* API responses

So we convert them into Compose State using Compose APIs.

---

# Mental Model

```text id="jlwmg8"
Non-Compose State
        ↓
Convert to Compose State
        ↓
Compose observes it
        ↓
UI recomposes automatically
```

---

# 1. Using `collectAsState()` for Flow / StateFlow

---

# Example — StateFlow

## ViewModel

```kotlin id="jlwmt1"
private val _name =
    MutableStateFlow("Reshma")

val name = _name.asStateFlow()
```

---

## Compose UI

```kotlin id="jlwmy3"
val name by viewModel.name
    .collectAsState()
```

---

# What happens?

```text id="jlwmb7"
StateFlow emits value
        ↓
collectAsState converts it
        ↓
Compose State updates
        ↓
UI recomposes
```

---

# For Normal Flow

```kotlin id="jlwmm5"
val users by usersFlow
    .collectAsState(initial = emptyList())
```

`initial` value is needed because normal Flow may not emit immediately.

---

# 2. Using `observeAsState()` for LiveData

---

# ViewModel

```kotlin id="jlwmu9"
private val _name =
    MutableLiveData("Reshma")

val name: LiveData<String> = _name
```

---

# Compose UI

```kotlin id="jlwmg0"
val name by viewModel.name
    .observeAsState("")
```

---

# What happens?

```text id="jlwms6"
LiveData emits value
        ↓
observeAsState converts it
        ↓
Compose State updates
        ↓
UI recomposes
```

---

# 3. Using `produceState()`

Used for generic async or external data conversion.

---

# Example

```kotlin id="jlwmd2"
val user by produceState<User?>(
    initialValue = null
) {

    value = repository.getUser()
}
```

---

# What happens?

```text id="jlwmo5"
Coroutine runs
      ↓
External data fetched
      ↓
value updated
      ↓
Compose State updates
      ↓
UI recomposes
```

---

# Why use produceState?

Useful for:

* API calls
* callback APIs
* repository data
* custom subscriptions

---

# 4. Using `mutableStateOf()`

Sometimes external values are manually copied into Compose State.

---

# Example

```kotlin id="jlwmt8"
var userName by remember {
    mutableStateOf("")
}

LaunchedEffect(Unit) {

    userName = repository.getName()
}
```

---

# Common APIs

| Non-Compose Source | Compose Conversion API  |
| ------------------ | ----------------------- |
| StateFlow          | collectAsState()        |
| Flow               | collectAsState(initial) |
| LiveData           | observeAsState()        |
| Generic async data | produceState()          |

---

# Recommended Modern Approach

```text id="jlwmy6"
ViewModel exposes StateFlow
        ↓
Compose observes using collectAsStateWithLifecycle()
```

---

# Important Interview Point

Compose UI can automatically recompose only for Compose State.

So non-Compose reactive sources must first be converted into Compose State.

---

# One-line Summary

Non-Compose state can be converted into Compose State using APIs like `collectAsState()`, `observeAsState()`, and `produceState()`, allowing Compose UI to automatically recompose when data changes.
