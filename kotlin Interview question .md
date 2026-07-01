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

Got it — here is your **clean, interview-ready Markdown format only for Q5 and Q6**, matching your style:


## 5. How to ensure null safety in Kotlin?

One of the major advantages of using Kotlin is **null safety**. In Java, if you access some null variable then you will get a `NullPointerException`. So, the following code in Kotlin will produce a compile-time error:

```kotlin
var name: String = "MindOrks"
name = null // error
```

So, to assign null values to a variable, you need to declare the `name` variable as a nullable string and then during the access of this variable, you need to use a safe call operator i.e. `?`.

```kotlin
var name: String? = "MindOrks"
print(name?.length) // ok
name = null // ok
```


## 6. What is the difference between safe calls (?.) and null check (!!)?

Safe call operator i.e. `?.` is used to check if the value of the variable is null or not. If it is null then null will be returned otherwise it will return the desired value.

```kotlin
var name: String? = "MindOrks"
println(name?.length) // 8

name = null
println(name?.length) // null
```

If you want to throw `NullPointerException` when the value of the variable is null, then you can use the null check or `!!` operator.

```kotlin
var name: String? = "MindOrks"
println(name?.length) // 8

name = null
println(name!!.length) // KotlinNullPointerException

```
## 7. Do we have a ternary operator in Kotlin just like Java?**

No, Kotlin does not have a ternary operator like Java.
Instead, we use **`if-else` expression** or the **Elvis operator (`?:`)** to achieve the same functionality.


## 8. What is the Elvis operator in Kotlin?**

The **Elvis operator (`?:`)** is used to handle null values by providing a default value when an expression is `null`.

It checks whether a value is null; if it is not null, it returns the value, otherwise it returns the default value on the right side.

**Example:**

```kotlin
var name: String? = "Mindorks"
val nameLength = name?.length ?: -1
println(nameLength)
```

**Explanation:**

* If `name` is not null → returns `name.length`
* If `name` is null → returns `-1`

---

**Interview Answer:**

> "Kotlin does not have a ternary operator like Java. Instead, we use `if-else` expressions or the Elvis operator `?:`. The Elvis operator is used to handle null values by providing a default value when the expression is null."

## 9. How to convert a Kotlin source file to a Java source file?

Kotlin code can be converted into Java equivalent code using built-in tools in **IntelliJ IDEA / Android Studio**.

### Steps to convert Kotlin to Java:

1. Open your Kotlin project in **IntelliJ IDEA / Android Studio**  
2. Navigate to:  
   **Tools > Kotlin > Show Kotlin Bytecode**
3. Click on the **Decompile** button  
4. You will get the **equivalent Java code generated from Kotlin bytecode**

---

### Key Point:
- Kotlin is first compiled into **bytecode**, and then decompiled into Java code for viewing
- The generated Java code is mainly for understanding, not for production use

---

## 10. What is the use of `@JvmStatic`, `@JvmOverloads`, and `@JvmField` in Kotlin?

Kotlin is designed for **Java interoperability**, meaning Kotlin code can be called from Java and vice versa. However, some Kotlin features (like default parameters, objects, and properties) do not directly map to Java. To solve this, Kotlin provides JVM annotations like `@JvmStatic`, `@JvmOverloads`, and `@JvmField`.

---

### 1. `@JvmStatic`

### Purpose:
Used to make a Kotlin function behave like a **static method in Java**.

---

#### Problem without `@JvmStatic`

```kotlin
object AppUtils {
    fun install() {
        println("Installing...")
    }
}
```

#### Kotlin call:
```kotlin
AppUtils.install()
```

#### Java call:
```java
AppUtils.install(); // ❌ Error
AppUtils.INSTANCE.install(); // ✅ Works
```

---

##### Solution using `@JvmStatic`

```kotlin
object AppUtils {

    @JvmStatic
    fun install() {
        println("Installing...")
    }
}
```

#### Java call:
```java
AppUtils.install(); // ✅ Works like static method
```

---

#### Key Point:
- Removes `.INSTANCE`
- Exposes function as a **static method in Java**

---

### 2. `@JvmOverloads`

#### Purpose:
Generates overloaded methods/constructors for functions that use **default parameters**, so Java can call them easily.

---

#### Problem without `@JvmOverloads`

```kotlin
data class Session(
    val name: String,
    val date: Date = Date()
)
```

#### Kotlin usage:
```kotlin
Session("Session One")
```

#### Java usage:
```java
new Session("Session One"); // ❌ Error
```

---

#### Solution using `@JvmOverloads`

```kotlin
data class Session @JvmOverloads constructor(
    val name: String,
    val date: Date = Date()
)
```

---

#### Java now works:
```java
new Session("Session One"); // ✅ Works
new Session("Session One", new Date()); // ✅ Works
```

---

#### Key Point:
- Converts default parameters into **Java-compatible overloads**

---

#### 3. `@JvmField`

#### Purpose:
Exposes Kotlin properties as **direct Java fields (no getters/setters)**.

---

#### Problem without `@JvmField`

```kotlin
class User {
    val name: String = "MindOrks"
}
```

#### Java usage:
```java
user.getName(); // getter required
```

---

#### Solution using `@JvmField`

```kotlin
class User {
    @JvmField
    val name: String = "MindOrks"
}
```

#### Java usage:
```java
user.name; // ✅ Direct field access
```

---

#### Key Point:
- Removes getters/setters
- Provides direct field access in Java

---

### Summary Table

| Annotation | Purpose | Java Benefit |
|------------|--------|--------------|
| `@JvmStatic` | Makes function static | No `.INSTANCE` required |
| `@JvmOverloads` | Supports default parameters | Generates overloaded methods |
| `@JvmField` | Exposes field directly | No getter/setter needed |

---

#### Final Interview Answer (Combined)

`@JvmStatic`, `@JvmOverloads`, and `@JvmField` are Kotlin JVM interoperability annotations used to make Kotlin code more Java-friendly. `@JvmStatic` exposes Kotlin functions as static methods in Java. `@JvmOverloads` generates overloaded methods or constructors for functions with default parameters so Java can call them easily. `@JvmField` exposes Kotlin properties as direct fields, allowing Java to access them without getters and setters. These annotations help bridge differences between Kotlin and Java and improve interoperability.



