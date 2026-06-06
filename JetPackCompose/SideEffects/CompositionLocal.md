# CompositionLocal in Jetpack Compose

`CompositionLocal` is used to share data across the composable tree **without passing it manually through every composable parameter**. 

---

# Why do we need CompositionLocal?

Normally in Compose:

```text id="jlwmu5"
Data flows from parent → child using parameters
```

Example:

```kotlin id="jlwmg8"
App(user)
```

↓

```kotlin id="jlwmt1"
HomeScreen(user)
```

↓

```kotlin id="jlwmy2"
Toolbar(user)
```

↓

```kotlin id="jlwmb6"
Profile(user)
```

Even if `Toolbar` doesn't use `user`, it still has to pass it down.

This problem is called:

```text id="jlwmm4"
Prop Drilling
```

---

# CompositionLocal Solves This

Instead of passing value manually:

```text id="jlwmu9"
Provide value once at parent
        ↓
Any child composable can access it directly
```

---

# Simple Analogy

Think of WiFi in a house.

```text id="jlwmg0"
Router provides WiFi
        ↓
All rooms can access it
```

You don't manually carry internet to every room.

Similarly:

```text id="jlwms4"
CompositionLocalProvider provides value
        ↓
All child composables can access it
```

---

# Steps to Use CompositionLocal

---

# 1. Create CompositionLocal

```kotlin id="jlwmd2"
val LocalUser = compositionLocalOf<String> {

    "Guest"
}
```

Creates a shared value holder.

---

# 2. Provide Value

```kotlin id="jlwmo5"
CompositionLocalProvider(

    LocalUser provides "Reshma"
) {

    HomeScreen()
}
```

Now `"Reshma"` becomes available to all child composables.

---

# 3. Access Value

```kotlin id="jlwmt8"
@Composable
fun Profile() {

    val user = LocalUser.current

    Text(user)
}
```

Output:

```text id="jlwmy4"
Reshma
```

---

# Complete Example

```kotlin id="jlwmb9"
val LocalUser = compositionLocalOf<String> {

    "Guest"
}

@Composable
fun App() {

    CompositionLocalProvider(

        LocalUser provides "Reshma"
    ) {

        HomeScreen()
    }
}

@Composable
fun HomeScreen() {

    Profile()
}

@Composable
fun Profile() {

    val user = LocalUser.current

    Text("Hello $user")
}
```

---

# Output

```text id="jlwmm0"
Hello Reshma
```

---

# Important Point

`Profile()` accesses value directly using:

```kotlin id="jlwmu2"
LocalUser.current
```

without parameter passing.

---

# Built-in CompositionLocals

Compose already uses this internally. 

Examples:

| CompositionLocal    | Purpose              |
| ------------------- | -------------------- |
| LocalContext        | Android Context      |
| LocalLifecycleOwner | LifecycleOwner       |
| LocalDensity        | Screen density       |
| LocalConfiguration  | Device configuration |

---

# Example

```kotlin id="jlwmg9"
val context = LocalContext.current
```

You never pass Context manually everywhere because Compose provides it using CompositionLocal.

---

# compositionLocalOf vs staticCompositionLocalOf

| compositionLocalOf         | staticCompositionLocalOf                  |
| -------------------------- | ----------------------------------------- |
| Tracks reads               | No read tracking                          |
| Better for changing values | Better for static values                  |
| More recomposition-aware   | Slightly better performance for constants |

---

# When to Use

✅ theme/colors
✅ typography
✅ dimensions
✅ localization
✅ shared config
✅ environment values

---

# When NOT to Use

❌ screen UI state
❌ ViewModel sharing
❌ frequently changing business state

Docs specifically mention passing ViewModel through CompositionLocal is a bad practice. 

---

# Important Interview Point

`CompositionLocal` should mainly be used for:

```text id="jlwms7"
Cross-cutting values
needed by many composables
```

and not as a replacement for proper state management.

---

# Interview-style Definition

`CompositionLocal` is a Jetpack Compose API used to implicitly share data across the composable hierarchy without manually passing it through every composable parameter.

---

# One-line Summary

```text id="jlwmd1"
Provide value once,
access it anywhere in child composables.
```
