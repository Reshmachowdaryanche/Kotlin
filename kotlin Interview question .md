## 1. How does Kotlin work on Android?

Just like Java, Kotlin code is compiled into **Java bytecode**. When you write Kotlin code in a file such as `Main.kt`, the Kotlin compiler compiles it into a class file named `MainKt.class` (for top-level functions) and generates the corresponding bytecode.

During the Android build process, this bytecode is converted into **DEX (Dalvik Executable)** bytecode, which is then executed by the **Android Runtime (ART)**. Since Kotlin is fully interoperable with Java, it can seamlessly use all Java libraries and Android APIs.

## 2. Why should we use Kotlin?**

* **Kotlin is concise:** It reduces boilerplate code, making the code shorter, cleaner, and easier to maintain.
* **Kotlin is null-safe:** It provides built-in null safety, helping prevent `NullPointerException` errors at compile time.
* **Kotlin is interoperable:** It works seamlessly with Java, allowing developers to use existing Java code and libraries without any issues.

## 3. What is the difference between `var` and `val` in Kotlin?**

* **`var` (Mutable Variable):** Used to declare a variable whose value can be changed or reassigned after initialization.
* **`val` (Immutable Variable):** Used to declare a variable whose value cannot be changed once it is assigned.

**Example:**

```kotlin
var name = "John"
name = "David"   // Allowed

val age = 25
age = 30         // Error: Val cannot be reassigned
```

## 4. What is the difference between `val` and `const` in Kotlin?

Both `val` and `const` are used to declare **immutable (read-only)** values, meaning their values cannot be changed after initialization. However, they differ in when and where their values are assigned.

- **`val`** is initialized only once, but its value can be determined **at runtime**.
- **`const val`** must be initialized with a value that is known **at compile time**.

### Key Differences

| `val` | `const val` |
|------|------------|
| Value is assigned once and cannot be changed. | Value is assigned once and cannot be changed. |
| Value can be determined at **runtime**. | Value must be known at **compile time**. |
| Can be used for objects, function results, or computed values. | Can only be initialized with primitive types or `String` constants. |
| Can be declared inside classes, functions, or at the top level. | Can only be declared at the **top level**, inside an **object**, or in a **companion object**. |

### Example

```kotlin
val currentTime = System.currentTimeMillis()   // Runtime value

const val APP_NAME = "MyApp"                  // Compile-time constant
const val MAX_USERS = 100
```

### Interview Answer

> Both `val` and `const val` are immutable, meaning their values cannot be changed after initialization. The main difference is that a `val` can be initialized with a value that is known at runtime, whereas a `const val` must be initialized with a compile-time constant. Additionally, `const val` can only be declared at the top level, inside an object, or in a companion object, while `val` can be declared almost anywhere.

