# What are the best practices for performance optimization in Jetpack Compose?

Performance optimization in Jetpack Compose mainly focuses on:

```text id="jlwmu5"
Reducing unnecessary recompositions
and avoiding expensive work during recomposition.
```

---

# 1. Minimize Unnecessary Recompositions

Compose recomposes when observed state changes.

Best practice is to keep recomposition scope small by splitting UI into smaller composables.

```kotlin id="jlwmg7"
@Composable
fun Counter(count: Int) {

    Text("$count")
}
```

Only `Counter()` recomposes when `count` changes.

---

# 2. Use `remember`

Use `remember` to avoid recreating expensive objects during recomposition.

```kotlin id="jlwmt1"
val formatter = remember {

    SimpleDateFormat(...)
}
```

Without `remember`, object recreates on every recomposition.

---

# 3. Use `derivedStateOf`

Used when state changes frequently but UI only cares about derived result.

```kotlin id="jlwmy2"
val showButton by remember {

    derivedStateOf {

        listState.firstVisibleItemIndex > 0
    }
}
```

Helps reduce unnecessary recompositions.

---

# 4. Use Stable and Immutable Data

Prefer immutable UI state models.

```kotlin id="jlwmb6"
data class UserUiState(
    val name: String
)
```

Stable objects help Compose skip recompositions.

---

# 5. Use `key` in Lazy Lists

Always provide stable keys in `LazyColumn`.

```kotlin id="jlwmm4"
items(
    users,
    key = { it.id }
) {

}
```

Improves item reuse and preserves item state.

---

# 6. Avoid Heavy Work Inside Composables

Do not perform:

* API calls
* database operations
* complex calculations

directly inside composables.

❌ Bad

```kotlin id="jlwmu8"
@Composable
fun Screen() {

    repository.getUsers()
}
```

✅ Use ViewModel or side effects.

---

# 7. Use `LaunchedEffect` for Async Work

```kotlin id="jlwmg0"
LaunchedEffect(Unit) {

    viewModel.loadData()
}
```

Prevents repeated API calls during recomposition.

---

# 8. Prefer `collectAsStateWithLifecycle()`

Instead of:

```kotlin id="jlwms4"
collectAsState()
```

use:

```kotlin id="jlwmd2"
collectAsStateWithLifecycle()
```

because it is lifecycle-aware and avoids unnecessary Flow collection.

---

# 9. Use Lazy Layouts for Large Lists

Use:

* LazyColumn
* LazyRow

instead of:

* Column
* Row

for large datasets.

Lazy layouts compose only visible items.

---

# 10. Hoist State Properly

Keep composables stateless when possible.

```kotlin id="jlwmo5"
@Composable
fun Counter(
    count: Int,
    onIncrement: () -> Unit
)
```

Benefits:

* reusable UI
* easier testing
* controlled recomposition

---

# 11. Avoid Unstable Lambdas and Objects

Use `remember` for lambdas if needed.

```kotlin id="jlwmt8"
val onClick = remember {

    { viewModel.onAction() }
}
```

Helps avoid unnecessary recompositions.

---

# 12. Use `rememberSaveable`

```kotlin id="jlwmy4"
var text by rememberSaveable {

    mutableStateOf("")
}
```

Preserves UI state across configuration changes.

---

# 13. Profile and Measure Performance

Use tools like:

* Layout Inspector
* Recomposition Counts
* Macrobenchmark
* Compose Tracing

---

# Important Interview Point

Compose performance optimization is mainly about:

```text id="jlwmb9"
Avoiding unnecessary recompositions
and keeping composables lightweight.
```

---

# Common Interview Questions

### Q1. What causes recomposition?

* mutableStateOf changes
* StateFlow updates
* parameter changes
* collectAsState updates

---

### Q2. Why use remember?

To avoid recreating objects during recomposition.

---

### Q3. Why use derivedStateOf?

To reduce recomposition when state changes frequently.

---

### Q4. Why use LazyColumn instead of Column?

LazyColumn composes only visible items, improving list performance.

---

# One-line Summary

Performance optimization in Jetpack Compose focuses on minimizing recompositions, using stable state management, avoiding expensive work inside composables, and using lifecycle-aware and lazy APIs for efficient UI rendering.
