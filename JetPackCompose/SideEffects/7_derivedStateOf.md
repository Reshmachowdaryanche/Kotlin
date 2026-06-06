# `derivedStateOf`

## Purpose

Used to:

```text id="jlwmu6"
Create derived/computed state
that updates less frequently
than its input state
```

Main goal:

✅ reduce unnecessary recomposition

---

# Why do we need it?

Sometimes a state changes VERY frequently, but UI only cares about certain conditions.

Example:

```text id="jlwmg4"
Scroll position:
0,1,2,3,4,5,6,7...
```

UI may only care about:

```text id="jlwmt8"
Is value > 0 ?
```

Without optimization:

```text id="jlwmy1"
UI recomposes on every scroll change
```

Unnecessary recompositions.

`derivedStateOf` helps avoid this.

---

# Mental Model

```text id="jlwmb2"
Frequently changing state
      ↓
derivedStateOf computes result
      ↓
Recompose ONLY if derived result changes
```

---

# Key Points

```kotlin id="jlwmm5"
val derivedValue by remember {
    derivedStateOf {

    }
}
```

* creates derived/computed Compose state
* recomposes only when derived result changes
* useful for expensive/frequent state updates
* similar to Flow's `distinctUntilChanged()`

---

# Correct Usage Example

```kotlin id="jlwmu0"
@Composable
fun MessageList(messages: List<Message>) {

    Box {

        val listState = rememberLazyListState()

        LazyColumn(state = listState) {
            // items
        }

        val showButton by remember {

            derivedStateOf {

                listState.firstVisibleItemIndex > 0
            }
        }

        AnimatedVisibility(
            visible = showButton
        ) {
            ScrollToTopButton()
        }
    }
}
```

---

# Step-by-step Explanation

---

## 1. Scroll position changes frequently

```kotlin id="jlwmg7"
listState.firstVisibleItemIndex
```

While scrolling:

```text id="jlwms3"
0,1,2,3,4,5,6...
```

This changes continuously.

---

## 2. UI only needs boolean

UI only cares about:

```kotlin id="jlwmd9"
firstVisibleItemIndex > 0
```

Meaning:

| Scroll Position | showButton |
| --------------- | ---------- |
| 0               | false      |
| 1               | true       |
| 2               | true       |
| 3               | true       |

---

# Important Observation

Without `derivedStateOf`:

```text id="jlwmo1"
Recomposition happens for:
0 → 1 → 2 → 3 → 4 → 5 ...
```

Even though:

```text id="jlwmt5"
showButton remains true
```

after index becomes > 0.

Unnecessary recompositions.

---

# What derivedStateOf does

```kotlin id="jlwmy4"
derivedStateOf {
    listState.firstVisibleItemIndex > 0
}
```

Now recomposition happens ONLY when result changes:

```text id="jlwmb6"
false → true
OR
true → false
```

NOT on every scroll value.

---

# Full Flow

```text id="jlwmm8"
Scroll index changes continuously
      ↓
derivedStateOf computes boolean
      ↓
Boolean changes only occasionally
      ↓
UI recomposes less
```

---

# Why use remember?

```kotlin id="jlwmu3"
remember {
    derivedStateOf { ... }
}
```

Prevents recreating derived state object on every recomposition.

---

# Important Benefit

```text id="jlwmg0"
Avoid unnecessary recompositions
```

Especially useful for:

* scrolling
* animations
* rapidly changing state

---

# Similarity to Flow

Docs compare it to:

```kotlin id="jlwms7"
distinctUntilChanged()
```

Reason:

Even if input changes many times:

```text id="jlwmd1"
Only emit when final result changes
```

---

# Incorrect Usage

```kotlin id="jlwmo7"
var firstName by remember {
    mutableStateOf("")
}

var lastName by remember {
    mutableStateOf("")
}

val fullNameBad by remember {

    derivedStateOf {
        "$firstName $lastName"
    }
}
```

---

# Why is this bad?

Because:

```text id="jlwmt0"
fullName SHOULD update every time
firstName or lastName changes
```

No unnecessary recompositions exist.

So `derivedStateOf` adds unnecessary overhead.

---

# Correct Version

```kotlin id="jlwmy9"
val fullNameCorrect =
    "$firstName $lastName"
```

Simple and efficient.

---

# Important Rule

Use `derivedStateOf` ONLY when:

```text id="jlwmb4"
Input changes more often
than UI actually needs to update
```

---

# When to Use

✅ scroll position
✅ animation thresholds
✅ expensive derived calculations
✅ rapidly changing state

---

# When NOT to Use

❌ simple string concatenation
❌ normal calculations
❌ values that must update every time anyway

---

# Important Characteristics

✅ reduces unnecessary recomposition
✅ computed/derived state
✅ optimized state observation
✅ useful for frequently changing inputs

---

# Caution

Docs mention:

```text id="jlwmm2"
derivedStateOf is expensive
```

So don't overuse it.

Only use when recomposition optimization is actually needed.

---

# Difference from Normal Calculation

| Normal Calculation             | derivedStateOf                           |
| ------------------------------ | ---------------------------------------- |
| Recomputes every recomposition | Updates only when derived result changes |
| No optimization                | Recomposition optimization               |
| Good for simple values         | Good for frequently changing inputs      |

---

# Interview Point

Use `derivedStateOf` when:

```text id="jlwmu8"
A state changes frequently,
but UI only needs updates
when the derived result changes.
```

---

# Simple Analogy

```text id="jlwmg5"
Temperature changes every second,
but AC only cares whether temperature crossed threshold.
```
