# Explain Jetpack Compose Phases

Jetpack Compose rendering happens in different phases to efficiently create, measure, layout, and draw the UI.

The main phases are:

1. Composition
2. Layout

   * Measurement
   * Placement
3. Drawing

---

# Mental Model

```text id="jlwmu5"
State changes
      ↓
Composition
      ↓
Layout
   ↓       ↓
Measure  Placement
      ↓
Drawing
      ↓
UI displayed
```

---

# 1. Composition Phase

This is the phase where Compose executes composable functions and builds the UI tree.

---

# What happens in Composition?

Compose:

* executes composable functions
* creates UI nodes
* tracks state reads
* determines UI structure

---

# Example

```kotlin id="jlwmg7"
@Composable
fun Greeting(name: String) {

    Text("Hello $name")
}
```

Compose executes:

```kotlin id="jlwmt1"
Greeting("Reshma")
```

and creates UI representation.

---

# Important Point

During composition:

```text id="jlwmy2"
Compose decides WHAT UI should look like
```

---

# Recomposition

If observed state changes:

```kotlin id="jlwmb6"
count++
```

Compose reruns affected composables.

This is:

```text id="jlwmm4"
Recomposition
```

---

# 2. Layout Phase

After composition, Compose calculates size and position of UI elements.

Layout phase has 2 steps:

1. Measurement
2. Placement

---

# A. Measurement Phase

Compose measures each UI element.

Questions answered:

```text id="jlwmu9"
How big should this composable be?
```

---

# Example

```kotlin id="jlwmg0"
Text("Hello")
```

Compose measures:

* width
* height

based on constraints.

---

# Constraints Flow

```text id="jlwms4"
Parent gives constraints
        ↓
Child measures itself
        ↓
Child returns size
```

---

# B. Placement Phase

After measurement, Compose places elements on screen.

Questions answered:

```text id="jlwmd2"
Where should this composable appear?
```

---

# Example

```kotlin id="jlwmo5"
Column {

    Text("A")

    Text("B")
}
```

Compose calculates:

```text id="jlwmt8"
A → top
B → below A
```

---

# Important Point

During layout:

```text id="jlwmy4"
Compose decides WHERE UI should appear
```

---

# 3. Drawing Phase

After layout, Compose draws pixels on screen.

---

# What happens?

Compose renders:

* text
* shapes
* colors
* images
* borders

onto Canvas.

---

# Example

```kotlin id="jlwmb9"
Text(
    "Hello",
    color = Color.Red
)
```

During drawing:

```text id="jlwmm0"
Red text is drawn on screen
```

---

# Important Point

During drawing:

```text id="jlwmu2"
Compose decides HOW UI appears visually
```

---

# Full Flow Example

```kotlin id="jlwmg9"
Text(
    text = "Hello",
    modifier = Modifier.padding(8.dp)
)
```

---

# Composition

```text id="jlwms7"
Create Text composable
```

---

# Measurement

```text id="jlwmd1"
Calculate text size + padding
```

---

# Placement

```text id="jlwmo4"
Determine screen position
```

---

# Drawing

```text id="jlwmt7"
Render text on screen
```

---

# Optimization in Compose

Compose can skip phases intelligently.

---

# Example

If only color changes:

```kotlin id="jlwmy5"
color = Color.Red
```

Maybe only:

```text id="jlwmb0"
Drawing phase reruns
```

No need for remeasurement.

---

# Example

If text changes:

```kotlin id="jlwmm8"
"Hi" → "Hello World"
```

Compose may need:

* recomposition
* remeasurement
* redraw

because size changed.

---

# Important Interview Point

Compose optimizes performance by rerunning only necessary phases.

---

# Phase Dependency

| Change             | Affected Phase |
| ------------------ | -------------- |
| State/UI structure | Composition    |
| Size/layout change | Layout         |
| Visual-only change | Drawing        |

---

# Example — Only Drawing Re-executes

```kotlin id="jlwmu1"
Box(
    Modifier.background(color)
)
```

If only background color changes:

```text id="jlwmg3"
Mostly redraw happens
```

No layout recalculation needed.

---

# Difference from Traditional Views

| Android Views           | Compose                        |
| ----------------------- | ------------------------------ |
| Measure → Layout → Draw | Composition → Layout → Draw    |
| Imperative UI updates   | Declarative UI updates         |
| Manual invalidation     | Automatic state-driven updates |

---

# Common Interview Questions

---

# Q1. What are the phases in Compose?

* Composition
* Layout

  * Measurement
  * Placement
* Drawing

---

# Q2. What happens during Composition?

Composable functions execute and UI tree is built.

---

# Q3. What happens during Layout?

Compose measures and positions UI elements.

---

# Q4. What happens during Drawing?

Compose renders UI pixels on screen.

---

# Q5. Does Compose rerun all phases every time?

No. Compose intelligently skips unnecessary phases for better performance.

---

# One-line Summary

Jetpack Compose UI rendering happens in three phases: Composition (build UI tree), Layout (measure and place UI), and Drawing (render pixels on screen), with Compose optimizing by rerunning only the required phases when state changes.
