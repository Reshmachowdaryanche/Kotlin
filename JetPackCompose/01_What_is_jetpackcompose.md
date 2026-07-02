# Jetpack Compose

## 1. What is Jetpack Compose?

**Jetpack Compose is Android's modern declarative UI toolkit developed by Google for building native Android applications using Kotlin.**

Unlike the traditional View system where we create XML layouts and manually update views using methods like `findViewById()` or `setText()`, Compose allows us to describe the UI as a function of state using composable functions.

When the application's state changes, Compose automatically updates only the affected parts of the UI through a process called recomposition. This reduces boilerplate code, improves readability, and makes UI development faster and easier to maintain.

It also integrates well with modern Android architecture components such as ViewModel, StateFlow, and Navigation, making it the recommended approach for Android UI development today.

---

## 2. Jetpack Compose vs Android View System

**Jetpack Compose and the Android View System are two different approaches to building UI in Android.**

The traditional **View System** is based on an **imperative approach**, where we define UI using XML layouts and then manually update views at runtime using methods like `findViewById()`, `setText()`, or `addView()`.

In this approach, developers are responsible for keeping the UI in sync with the data, which often leads to more boilerplate code and higher chances of inconsistencies.

In contrast, **Jetpack Compose** follows a **declarative approach**, where we describe the UI using Kotlin functions called composables.

Instead of manually updating UI elements, we provide state to composables, and whenever the state changes, Compose automatically updates the UI through recomposition.

So, in short:

* The View System is **UI-driven and manually updated**
* Compose is **state-driven and automatically updates the UI based on state changes**

Compose is more modern, reduces boilerplate, and integrates better with Kotlin and modern architecture patterns like ViewModel and StateFlow.

### Comparison Table

| Android View System                   | Jetpack Compose                 |
| ------------------------------------- | ------------------------------- |
| Imperative UI                         | Declarative UI                  |
| XML + Kotlin/Java                     | Kotlin only                     |
| Manual UI updates                     | Automatic UI updates            |
| Uses `findViewById()`                 | No `findViewById()`             |
| More boilerplate                      | Less boilerplate                |
| UI-driven                             | State-driven                    |
| Harder to maintain                    | Easier to maintain              |
| Manual synchronization of UI and data | UI automatically reflects state |

---

## 3. Imperative UI vs Declarative UI

**Declarative UI and Imperative UI are two different approaches to building user interfaces.**

### Imperative UI

In an **imperative UI approach**, developers focus on **how to update the UI**.

We manually create UI components and explicitly modify them whenever data changes.

For example, in the traditional Android View system, we define layouts using XML and update views using methods like:

* `findViewById()`
* `setText()`
* `setVisibility()`
* `addView()`

Here, the developer is responsible for keeping the UI synchronized with the data.

### Declarative UI

In contrast, **declarative UI focuses on what the UI should look like for a given state**.

Instead of manually updating views, we describe the UI as a function of state.

Jetpack Compose follows this declarative approach. We pass state to composable functions, and whenever the state changes, Compose automatically updates the required parts of the UI through recomposition.

So, rather than writing code to manipulate views step by step, we simply describe the desired UI for the current state, and Compose handles the UI updates internally.

### Summary

* **Imperative UI → Focuses on how to update UI manually**
* **Declarative UI → Focuses on describing UI based on state**

Declarative UI reduces boilerplate code, improves readability, and makes UI management simpler and more predictable.

---

## 4. Declarative UI in Jetpack Compose

Declarative UI in Jetpack Compose means that instead of building and updating UI widgets manually, we describe the UI as a function of state.

### Traditional UI Approach

In traditional imperative UI toolkits:

* We create a tree of widgets (often using XML)
* Each widget maintains its own internal state with getter and setter methods
* We directly manipulate widgets to update the UI

### Compose Approach

In Compose:

* UI components (composables) are stateless
* They do not expose getters or setters
* They are not treated as objects to be modified

Instead, we simply call composable functions with the current state as input, and they return a UI representation.

The app logic (often through a ViewModel) provides state to top-level composables.

These composables then pass the state down the hierarchy and describe the UI at each level.

### Data Flow

```text
ViewModel/App Logic
         ↓
       State
         ↓
   Composable Functions
         ↓
          UI
```

### Event Flow

When the user interacts with the UI:

```text
User Action
      ↓
     Event
      ↓
 App Logic / ViewModel
      ↓
 State Updated
      ↓
 Recomposition
      ↓
 UI Updated
```

Events like `onClick` are triggered and sent back to the app logic.

The app logic updates the state, and when that state changes, Compose automatically re-executes the composable functions.

This process is called **recomposition**.

### Key Idea

```text
UI = f(State)
```

Meaning:

* Same State → Same UI
* Different State → Different UI

### Summary

Declarative UI means the UI is rebuilt from state whenever it changes, and we focus on describing what the UI should look like rather than how to update it manually.

---

# Core Building Blocks

## 5. Composable Functions

**Composable functions are the basic building blocks of UI in Jetpack Compose.**

They are Kotlin functions annotated with the `@Composable` annotation and are used to describe how the UI should look for a given state.

Instead of creating XML layouts and manually manipulating View objects, Compose allows us to build UI by calling composable functions directly from Kotlin code.

### Example

```kotlin
@Composable
fun Greeting(name: String) {
    Text(text = "Hello $name")
}
```

In this example:

* `Greeting()` is a composable function.
* It accepts data as input through the `name` parameter.
* It emits UI by calling another composable function `Text()`.

---

### 5.1 Annotated with `@Composable`

Every composable function must use the `@Composable` annotation.

This tells the Compose compiler that the function is intended to convert state/data into UI.

---

### 5.2 Accept Data as Input

Composable functions can take parameters.

This allows the app logic or ViewModel to pass state into the UI.

Example:

```kotlin
Greeting(name = "Reshma")
```

---

### 5.3 Emit UI Instead of Returning UI Objects

Composable functions do not return Views or UI objects.

Instead, they describe the UI by calling other composable functions.

Example:

```kotlin
Text("Hello")
```

Compose internally manages the UI hierarchy.

---

### 5.4 Can Be Re-Executed Multiple Times

Composable functions may run many times during recomposition whenever observed state changes.

Because of this, composables should ideally be:

* Fast
* Idempotent
* Side-effect free

Meaning:

* Calling the composable multiple times with the same input should produce the same UI.
* They should not modify global variables or perform operations like network calls directly.

---

### 5.5 Build UI Hierarchies

Composable functions can call other composable functions, which helps create reusable and modular UI components.

Example:

```kotlin
@Composable
fun ProfileScreen() {
    Column {
        ProfileImage()
        UserName()
        UserBio()
    }
}
```

---

### Summary

Composable functions are declarative Kotlin functions that take state as input and emit UI.

They are:

* Lightweight
* Reusable
* Automatically recomposed when state changes

They form the foundation of Jetpack Compose UI development.

---

### Interview One-Liner

> A composable function is a Kotlin function annotated with `@Composable` that describes UI for a given state and serves as the basic building block of Jetpack Compose.
