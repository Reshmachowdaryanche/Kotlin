# Delegation in Kotlin

Delegation is a **design pattern** where a class **delegates the responsibility of performing a task to another object** instead of implementing it itself.

Kotlin provides **built-in support for delegation** using the `by` keyword.

---

# 1. Why Delegation?

Delegation helps to:

* Reduce **boilerplate code**
* Improve **code reuse**
* Follow **composition over inheritance**
* Keep classes **small and maintainable**

Instead of implementing functionality inside a class, we **forward the work to another object**.

### Concept

```
Client → Class A → Delegate Class → Work Done
```

---

# 2. Manual Delegation

```kotlin
interface Engine {
    fun start()
}

class PetrolEngine : Engine {
    override fun start() {
        println("Engine started")
    }
}

class Car : Engine {

    private val engine = PetrolEngine()

    override fun start() {
        engine.start()
    }
}
```

### Flow

```
Car.start()
   ↓
engine.start()
```

Here **Car forwards the call manually** to the `engine`.

---

# 3. Kotlin Interface Delegation (`by`)

Kotlin simplifies manual delegation using **`by`**.

```kotlin
interface Engine {
    fun start()
}

class PetrolEngine : Engine {
    override fun start() {
        println("Engine started")
    }
}

class Car(engine: Engine) : Engine by engine
```

### Usage

```kotlin
fun main() {
    val car = Car(PetrolEngine())
    car.start()
}
```

Output

```
Engine started
```

---

# 4. What Kotlin Generates Internally

Kotlin automatically creates something like this:

```kotlin
class Car(private val engine: Engine) : Engine {

    override fun start() {
        engine.start()
    }
}
```

So you **don’t write forwarding code manually**.

---

# 5. Execution Flow

```
Car.start()
     ↓
Delegated to
     ↓
PetrolEngine.start()
```

---

# 6. Realistic Example

```kotlin
interface Engine {
    fun start()
}

class PetrolEngine : Engine {
    override fun start() {
        println("Petrol engine started")
    }
}

class ElectricEngine : Engine {
    override fun start() {
        println("Electric engine started")
    }
}

class Car(engine: Engine) : Engine by engine
```

Usage

```kotlin
val petrolCar = Car(PetrolEngine())
petrolCar.start()

val electricCar = Car(ElectricEngine())
electricCar.start()
```

Output

```
Petrol engine started
Electric engine started
```

---

# 7. Types of Delegation in Kotlin

Kotlin supports **two main delegation types**.

## 1️⃣ Interface Delegation

Delegating an **interface implementation** to another object.

Example:

```kotlin
class Car(engine: Engine) : Engine by engine
```

Used when:

* One class wants to reuse behavior of another class
* Avoid inheritance
* Reduce boilerplate code

---

## 2️⃣ Property Delegation

Property value handling is delegated to another object.

Syntax:

```kotlin
val/var propertyName by delegate
```

Example:

```kotlin
val name by lazy {
    "Reshma"
}
```

Here **lazy** is the delegate that controls property initialization.

---

# 8. Built-in Kotlin Delegates

Kotlin provides several **built-in delegates**.

## 1️⃣ lazy

Initializes a value **only when first used**.

```kotlin
val data by lazy {
    loadDataFromNetwork()
}
```

Common use cases:

* Expensive objects
* Database initialization
* Repository initialization

---

## 2️⃣ observable

Notifies when a value changes.

```kotlin
import kotlin.properties.Delegates

var name: String by Delegates.observable("Initial") { property, old, new ->
    println("$old -> $new")
}
```

Use case:

* State tracking
* UI updates
* Logging value changes

---

## 3️⃣ vetoable

Allows **rejecting value changes**.

```kotlin
var age: Int by Delegates.vetoable(18) { _, old, new ->
    new >= 18
}
```

Use case:

* Validation
* Business rules

---

## 4️⃣ map delegation

Properties stored in a **map**.

```kotlin
class User(val map: Map<String, Any?>) {
    val name: String by map
    val age: Int by map
}
```

Use case:

* JSON parsing
* Dynamic data models

---

# 9. Custom Delegates

You can create your own delegate.

Example:

```kotlin
import kotlin.reflect.KProperty

class LoggerDelegate {

    operator fun getValue(thisRef: Any?, property: KProperty<*>): String {
        return "Hello"
    }

    operator fun setValue(thisRef: Any?, property: KProperty<*>, value: String) {
        println("Value changed to $value")
    }
}

class User {
    var name: String by LoggerDelegate()
}
```

Usage

```kotlin
val user = User()
user.name = "Reshma"
```

---

# 10. Most Used Delegates in Android

Delegation is **heavily used in Android development**.

## 1️⃣ viewModels()

Used in **Activities and Fragments**.

```kotlin
class MainActivity : AppCompatActivity() {

    private val viewModel: MainViewModel by viewModels()
}
```

What happens internally:

* A **ViewModelProvider delegate** creates the ViewModel.
* Lifecycle aware.
* Avoids manual ViewModel creation.

---

## 2️⃣ activityViewModels()

Used when **multiple fragments share the same ViewModel**.

```kotlin
private val viewModel: MainViewModel by activityViewModels()
```

Used for:

* Fragment communication
* Shared state

---

## 3️⃣ lazy in Android

Very common in Android.

Example:

```kotlin
private val repository by lazy {
    UserRepository()
}
```

Used for:

* Repository initialization
* Database setup
* Retrofit creation

---

## 4️⃣ viewBinding delegation

Common in modern Android.

```kotlin
private val binding by lazy {
    ActivityMainBinding.inflate(layoutInflater)
}
```

---

## 5️⃣ Compose state delegation

Jetpack Compose uses delegation heavily.

Example:

```kotlin
var count by remember { mutableStateOf(0) }
```

`by` automatically unwraps `MutableState`.

Without delegation:

```kotlin
val countState = remember { mutableStateOf(0) }
countState.value
```

With delegation:

```kotlin
var count by remember { mutableStateOf(0) }
```

Cleaner and easier.

---

# 11. Rule to Remember

Whenever you see:

```
: Interface by object
```

It means:

> This class **implements the interface but delegates the work to another object.**

Example

```kotlin
class Car(engine: Engine) : Engine by engine
```

Meaning:

```
Car implements Engine
       ↓
Engine implementation delegated to engine object
```

---

# 12. Summary

Delegation in Kotlin:

* Reduces **boilerplate code**
* Encourages **composition over inheritance**
* Improves **maintainability**
* Built into the language using **`by`**

Main delegation types:

| Type                 | Example                                        |
| -------------------- | ---------------------------------------------- |
| Interface Delegation | `class Car(engine: Engine) : Engine by engine` |
| Property Delegation  | `val name by lazy {}`                          |

Common built-in delegates:

* `lazy`
* `observable`
* `vetoable`
* `map`

Most used Android delegates:

* `by viewModels()`
* `by activityViewModels()`
* `by lazy`
* `by remember { mutableStateOf() }`

---

# Quick Cheat Sheet

```
Interface delegation
class Car(engine: Engine) : Engine by engine

Property delegation
val data by lazy { load() }

Android
private val viewModel by viewModels()

Compose
var count by remember { mutableStateOf(0) }
```

Delegation is one of the **most powerful and widely used features in Kotlin**, especially in **Android and Jetpack Compose**.
