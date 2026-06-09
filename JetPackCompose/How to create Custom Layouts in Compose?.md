# Creating Custom Layouts in Jetpack Compose

## Introduction

Jetpack Compose provides built-in layouts such as `Row`, `Column`, and `Box`. When these layouts do not meet specific UI requirements, you can create **custom layouts** using the `Layout` composable.

Custom layouts allow developers to control:

* Measurement of child composables
* Positioning of child composables
* Layout size and arrangement

---

## Creating a Custom Layout

The `Layout` composable requires two parameters:

1. Content (child composables)
2. Measure policy (measurement and placement logic)

### Syntax

```kotlin
@Composable
fun CustomLayout(
    content: @Composable () -> Unit
) {
    Layout(
        content = content
    ) { measurables, constraints ->

        // Measure children
        val placeables = measurables.map {
            it.measure(constraints)
        }

        // Define layout size
        layout(
            width = constraints.maxWidth,
            height = constraints.maxHeight
        ) {

            // Place children
            placeables.forEach {
                it.place(x = 0, y = 0)
            }
        }
    }
}
```

---

## Example: Vertical Layout

This custom layout arranges items vertically similar to a `Column`.

```kotlin
@Composable
fun VerticalLayout(
    content: @Composable () -> Unit
) {
    Layout(content = content) { measurables, constraints ->

        val placeables = measurables.map {
            it.measure(constraints)
        }

        val totalHeight = placeables.sumOf { it.height }
        val maxWidth = placeables.maxOfOrNull { it.width } ?: 0

        layout(maxWidth, totalHeight) {

            var yPosition = 0

            placeables.forEach { placeable ->
                placeable.place(0, yPosition)
                yPosition += placeable.height
            }
        }
    }
}
```

### Usage

```kotlin
VerticalLayout {
    Text("Item 1")
    Text("Item 2")
    Text("Item 3")
}
```

---

## Layout Process in Compose

Custom layouts follow three steps:

### 1. Measure

Determine the size of child composables.

```kotlin
val placeable = measurable.measure(constraints)
```

### 2. Define Layout Size

Specify the width and height of the custom layout.

```kotlin
layout(width, height) {
}
```

### 3. Place Children

Position children within the layout.

```kotlin
placeable.place(x, y)
```

---

## Important Components

### Constraints

Constraints define the minimum and maximum size available to a composable.

```kotlin
constraints.maxWidth
constraints.maxHeight
```

### Measurable

Represents a child composable before measurement.

```kotlin
measurable.measure(constraints)
```

### Placeable

Represents a measured child composable.

```kotlin
placeable.width
placeable.height
placeable.place(x, y)
```

---

## Advantages of Custom Layouts

* Full control over UI arrangement
* Optimized layout performance
* Ability to create unique UI designs
* Reusable layout components
* Supports complex positioning logic

---

## Use Cases

* Flow layouts (wrapping items)
* Custom grids
* Overlapping views
* Circular layouts
* Timeline views
* Dashboard components
* Advanced animations

---

## Key Points

* Use `Layout` to create custom layouts in Compose.
* Measure children using `measure()`.
* Define parent size using `layout()`.
* Position children using `place()`.
* Useful when `Row`, `Column`, or `Box` cannot satisfy design requirements.

> Custom layouts in Jetpack Compose provide complete control over how composables are measured and positioned on the screen.
