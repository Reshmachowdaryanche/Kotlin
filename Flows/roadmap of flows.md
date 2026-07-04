

### 1. Learn the basics of Kotlin Flow

Start by understanding what a Flow is and why it exists.

Topics to cover:

* What is a Flow?
* Cold streams vs Hot streams
* Flow lifecycle
* Creating flows

  * `flow {}`
  * `flowOf()`
  * `asFlow()`
* Collecting values

  * `collect`
  * `collectLatest`
  * `launchIn`

Example:

```kotlin
val numbers = flow {
    emit(1)
    emit(2)
    emit(3)
}

numbers.collect {
    println(it)
}
```

---

### 2. Learn Flow operators

These are used constantly in real projects.

Transformation:

* `map`
* `filter`
* `transform`
* `flatMapConcat`
* `flatMapLatest`
* `flatMapMerge`

Combination:

* `zip`
* `combine`

Terminal operators:

* `collect`
* `first`
* `last`
* `single`
* `toList`

Example:

```kotlin
flowOf(1,2,3,4)
    .filter { it % 2 == 0 }
    .map { it * 10 }
    .collect {
        println(it)
    }
```

---

### 3. Learn coroutine context in Flow

Understand:

* `flowOn`
* Dispatchers
* Cancellation
* Structured concurrency

Example:

```kotlin
flow {
    emit(fetchData())
}.flowOn(Dispatchers.IO)
```

---

### 4. Learn exception handling

Topics:

* `catch`
* `retry`
* `retryWhen`
* `onCompletion`

Example:

```kotlin
flow {
    emit(apiCall())
}
.catch {
    emit("Error")
}
.collect {
    println(it)
}
```

---

### 5. Learn Hot Flows

This is one of the most important concepts in Android.

Understand:

* `StateFlow`
* `SharedFlow`
* Differences from Flow
* State management
* Events

Example:

```kotlin
private val _count = MutableStateFlow(0)
val count = _count

_count.value++
```

---

## Android Flow

Now learn how Flow is used in Android architecture.

### Repository

```kotlin
class UserRepository {

    fun getUsers(): Flow<List<User>> {
        return flow {
            emit(api.getUsers())
        }
    }
}
```

---

### ViewModel

```kotlin
class UserViewModel(
    private val repository: UserRepository
) : ViewModel() {

    val users = repository.getUsers()
}
```

---

### Activity / Fragment

```kotlin
lifecycleScope.launch {
    viewModel.users.collect {
        // update UI
    }
}
```

---

### Using `repeatOnLifecycle`

This is the recommended approach.

```kotlin
lifecycleScope.launch {
    repeatOnLifecycle(Lifecycle.State.STARTED) {
        viewModel.users.collect {
            // update UI
        }
    }
}
```

---

### Room + Flow

Room can automatically emit updates.

```kotlin
@Query("SELECT * FROM user")
fun getUsers(): Flow<List<User>>
```

Whenever the database changes, the Flow emits new data automatically.

---

### StateFlow in ViewModel

```kotlin
private val _uiState = MutableStateFlow(UiState())
val uiState: StateFlow<UiState> = _uiState
```

Collect in UI:

```kotlin
lifecycleScope.launch {
    repeatOnLifecycle(Lifecycle.State.STARTED) {
        viewModel.uiState.collect {
            // Render UI
        }
    }
}
```

---

### SharedFlow for events

Use it for one-time events like:

* Toast
* Navigation
* Snackbar

```kotlin
private val _events = MutableSharedFlow<String>()
val events = _events
```

---

## Important interview topics

Make sure you understand:

* Flow vs LiveData
* Flow vs StateFlow
* Flow vs SharedFlow
* Cold Flow vs Hot Flow
* `collect` vs `collectLatest`
* `zip` vs `combine`
* `map` vs `flatMapLatest`
* `flowOn`
* Buffering (`buffer`, `conflate`)
* `debounce`
* `distinctUntilChanged`
* `stateIn`
* `shareIn`
* Exception handling
* Cancellation

---

## Recommended learning order

1. Coroutines basics (`launch`, `async`, `suspend`)
2. Flow basics
3. Flow operators
4. Exception handling
5. Hot Flows (`StateFlow`, `SharedFlow`)
6. Flow with Room
7. Flow with ViewModel
8. Flow with `repeatOnLifecycle`
9. Flow with Jetpack Compose (`collectAsState`)
10. Advanced operators (`combine`, `flatMapLatest`, `stateIn`, `shareIn`)

