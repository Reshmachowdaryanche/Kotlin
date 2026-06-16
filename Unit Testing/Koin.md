
## What is MockK?

MockK is a mocking library designed specifically for Kotlin. It handles Kotlin features such as:

* Final classes (default in Kotlin)
* Data classes
* Coroutines
* Extension functions
* Object singletons
* Top-level functions

Dependency (Gradle):

```kotlin
testImplementation("io.mockk:mockk:<version>")
```

---

# Basic Example

Class under test:

```kotlin
class UserService(
    private val repository: UserRepository
) {
    fun getUser(id: Long): String {
        return repository.findById(id).name
    }
}
```

Test:

```kotlin
class UserServiceTest {

    private val repository = mockk<UserRepository>()

    private val service = UserService(repository)

    @Test
    fun shouldReturnUserName() {

        every {
            repository.findById(1L)
        } returns User("John")

        val result = service.getUser(1L)

        assertEquals("John", result)

        verify {
            repository.findById(1L)
        }
    }
}
```

---

# Important Interview Questions

## 1. How do you create a mock?

```kotlin
val repository = mockk<UserRepository>()
```

Relaxed mock:

```kotlin
val repository = mockk<UserRepository>(relaxed = true)
```

Relaxed mocks return default values automatically.

Example:

```kotlin
repository.count()
```

returns:

```kotlin
0
```

without stubbing.

---

## 2. What is `every {}`?

Equivalent to Mockito's `when().thenReturn()`.

```kotlin
every {
    repository.findById(1L)
} returns User("John")
```

---

## 3. How do you verify method calls?

```kotlin
verify {
    repository.findById(1L)
}
```

Verify count:

```kotlin
verify(exactly = 2) {
    repository.save(any())
}
```

Never called:

```kotlin
verify(exactly = 0) {
    repository.delete(any())
}
```

---

## 4. What is a Relaxed Mock?

```kotlin
val service = mockk<MyService>(
    relaxed = true
)
```

Returns default values:

* String → ""
* Int → 0
* Boolean → false

Useful when return values are not important.

---

## 5. Mocking Coroutines

Very common interview topic.

Function:

```kotlin
suspend fun getUser(id: Long): User
```

Test:

```kotlin
coEvery {
    repository.getUser(1)
} returns User("John")
```

Verification:

```kotlin
coVerify {
    repository.getUser(1)
}
```

### Interview Rule

| Normal Function | Coroutine Function |
| --------------- | ------------------ |
| `every`         | `coEvery`          |
| `verify`        | `coVerify`         |

---

## 6. Mocking Exceptions

```kotlin
every {
    repository.findById(1)
} throws RuntimeException("Not found")
```

Coroutine:

```kotlin
coEvery {
    repository.getUser(1)
} throws RuntimeException()
```

---

## 7. Capturing Arguments

```kotlin
val slot = slot<User>()

every {
    repository.save(capture(slot))
} returns Unit
```

Use:

```kotlin
verify {
    repository.save(any())
}

assertEquals(
    "John",
    slot.captured.name
)
```

---

## 8. Mocking Object Singletons

Kotlin object:

```kotlin
object Config {
    fun version() = "1.0"
}
```

Test:

```kotlin
mockkObject(Config)

every {
    Config.version()
} returns "2.0"
```

Cleanup:

```kotlin
unmockkObject(Config)
```

Frequently asked because Mockito traditionally struggles with static-like behavior.

---

## 9. Mocking Companion Objects

```kotlin
class Utils {
    companion object {
        fun getValue() = "real"
    }
}
```

```kotlin
mockkObject(Utils.Companion)

every {
    Utils.getValue()
} returns "mocked"
```

---

## 10. Mocking Top-Level Functions

File:

```kotlin
fun generateToken() = "real"
```

Test:

```kotlin
mockkStatic(::generateToken)

every {
    generateToken()
} returns "mocked"
```

Very common MockK interview question.

---

## 11. Spies in MockK

Create spy:

```kotlin
val service = spyk(MyService())
```

Stub selected methods:

```kotlin
every {
    service.calculate()
} returns 100
```

Real methods execute unless overridden.

---

## 12. Ordered Verification

Verify call order:

```kotlin
verifyOrder {
    repository.save(user)
    repository.publish(user)
}
```

Strict sequence:

```kotlin
verifySequence {
    repository.save(user)
    repository.publish(user)
}
```

Interviewers often ask the difference:

* `verifyOrder` → calls happen in order, extra calls allowed.
* `verifySequence` → exact sequence, no extra calls.

---

## MockK Annotations

```kotlin
@MockK
lateinit var repository: UserRepository

@InjectMockKs
lateinit var service: UserService
```

Initialization:

```kotlin
@BeforeEach
fun setup() {
    MockKAnnotations.init(this)
}
```

Or with JUnit 5:

```kotlin
@ExtendWith(MockKExtension::class)
class UserServiceTest
```

---

# Mockito vs MockK

| Feature             | Mockito                | MockK     |
| ------------------- | ---------------------- | --------- |
| Kotlin Support      | Good                   | Excellent |
| Coroutines          | Extra setup            | Native    |
| Final Classes       | Historically difficult | Native    |
| Object Mocking      | More complex           | Simple    |
| Top-level Functions | More complex           | Simple    |
| Kotlin-first API    | No                     | Yes       |

Interview answer:

> MockK was built specifically for Kotlin and provides native support for coroutines, object mocking, top-level functions, and final classes, making tests more idiomatic and concise than Mockito in Kotlin projects.

---

# Most Asked MockK Interview Questions

Here are concise interview-ready answers for the most common MockK questions.

---

## 1. What is MockK?

**Answer:**
MockK is a Kotlin-first mocking framework used for unit testing. It provides native support for Kotlin features such as coroutines, final classes, object singletons, companion objects, and top-level functions.

**Interview version:**

> MockK is a mocking library designed specifically for Kotlin. It makes testing easier by providing built-in support for coroutines and Kotlin language features that are difficult to mock with traditional Java-based frameworks.

---

## 2. Why use MockK instead of Mockito in Kotlin?

**Answer:**

* Better Kotlin support
* Native coroutine support
* Easier object and static mocking
* No need for special configuration for final classes

**Interview version:**

> MockK is more Kotlin-friendly than Mockito. It supports suspend functions, object declarations, companion objects, and top-level functions out of the box, making tests more concise and idiomatic.

---

## 3. What is the difference between `mockk()` and `spyk()`?

**Answer:**

`mockk()`

```kotlin
val repo = mockk<UserRepository>()
```

Creates a completely fake object.

`spyk()`

```kotlin
val service = spyk(UserService())
```

Wraps a real object and allows partial mocking.

**Interview version:**

> `mockk()` creates a fully mocked object where all behavior must be defined. `spyk()` creates a spy around a real object, so real methods execute unless explicitly mocked.

---

## 4. What is a Relaxed Mock?

**Answer:**

```kotlin
val repo = mockk<UserRepository>(relaxed = true)
```

Returns default values automatically.

Examples:

* Int → 0
* String → ""
* Boolean → false

**Interview version:**

> A relaxed mock automatically returns default values for unstubbed methods, reducing boilerplate in tests.

---

## 5. What is `every {}`?

**Answer:**

Used for stubbing behavior.

```kotlin
every {
    repo.findById(1)
} returns User("John")
```

**Interview version:**

> `every {}` defines how a mock should behave when a specific method is called. It is similar to Mockito's `when().thenReturn()`.

---

## 6. What is `verify {}`?

**Answer:**

Used to verify interactions.

```kotlin
verify {
    repo.findById(1)
}
```

**Interview version:**

> `verify {}` checks whether a method was called on a mock and can also validate the number of invocations.

---

## 7. Difference between `every` and `verify`?

**Answer:**

| every            | verify            |
| ---------------- | ----------------- |
| Defines behavior | Verifies behavior |
| Before execution | After execution   |

Example:

```kotlin
every { repo.findById(1) } returns user
```

```kotlin
verify { repo.findById(1) }
```

**Interview version:**

> `every` is used for stubbing method responses, while `verify` is used to confirm that methods were actually invoked.

---

## 8. Difference between `coEvery` and `every`?

**Answer:**

For suspend functions:

```kotlin
coEvery {
    repo.getUser(1)
} returns user
```

For normal functions:

```kotlin
every {
    repo.findById(1)
} returns user
```

**Interview version:**

> `coEvery` is used for suspending functions and `every` is used for regular functions.

---

## 9. Difference between `coVerify` and `verify`?

**Answer:**

```kotlin
coVerify {
    repo.getUser(1)
}
```

```kotlin
verify {
    repo.findById(1)
}
```

**Interview version:**

> `coVerify` verifies calls to suspend functions, while `verify` is used for normal methods.

---

## 10. How do you mock a suspend function?

**Answer:**

```kotlin
coEvery {
    repo.getUser(1)
} returns User("John")
```

Verification:

```kotlin
coVerify {
    repo.getUser(1)
}
```

**Interview version:**

> Suspend functions are mocked using `coEvery` and verified using `coVerify`.

---

## 11. How do you mock exceptions?

**Answer:**

```kotlin
every {
    repo.findById(1)
} throws RuntimeException("Not Found")
```

Coroutine:

```kotlin
coEvery {
    repo.getUser(1)
} throws RuntimeException()
```

**Interview version:**

> The `throws` keyword is used inside stubbing to simulate error scenarios and test exception handling logic.

---

## 12. How do you capture method arguments?

**Answer:**

```kotlin
val slot = slot<User>()

every {
    repo.save(capture(slot))
} returns Unit
```

Access value:

```kotlin
slot.captured.name
```

**Interview version:**

> A slot captures arguments passed to mocked methods, allowing validation of the exact values used during execution.

---

## 13. How do you mock Kotlin Objects?

Given:

```kotlin
object Config {
    fun version() = "1.0"
}
```

Test:

```kotlin
mockkObject(Config)

every {
    Config.version()
} returns "2.0"
```

Cleanup:

```kotlin
unmockkObject(Config)
```

**Interview version:**

> `mockkObject()` allows mocking Kotlin singleton objects, which is a common requirement in Kotlin applications.

---

## 14. How do you mock top-level functions?

Function:

```kotlin
fun generateToken() = "real"
```

Test:

```kotlin
mockkStatic(::generateToken)

every {
    generateToken()
} returns "mocked"
```

**Interview version:**

> MockK can mock top-level functions using `mockkStatic`, which is one of its key advantages in Kotlin.

---

## 15. Difference between `verifyOrder` and `verifySequence`?

**Answer:**

```kotlin
verifyOrder {
    repo.save()
    repo.publish()
}
```

Checks relative order.

```kotlin
verifySequence {
    repo.save()
    repo.publish()
}
```

Checks exact order and no extra calls.

**Interview version:**

> `verifyOrder` ensures calls occur in the specified order, while `verifySequence` ensures the exact sequence with no additional interactions.

---

## 16. What is `confirmVerified()`?

**Answer:**

```kotlin
verify {
    repo.save()
}

confirmVerified(repo)
```

**Interview version:**

> `confirmVerified()` ensures there are no unverified interactions with a mock, helping detect unexpected method calls.

---

## 17. What is `clearMocks()`?

**Answer:**

```kotlin
clearMocks(repo)
```

Removes:

* Stubs
* Recorded calls
* Verifications

**Interview version:**

> `clearMocks()` resets a mock's state so it can be reused in another test scenario.

---

## 18. What is `@MockK`?

**Answer:**

```kotlin
@MockK
lateinit var repository: UserRepository
```

Creates mock automatically.

**Interview version:**

> `@MockK` is an annotation-based way to create mocks instead of manually calling `mockk()`.

---

## 19. What is `@InjectMockKs`?

**Answer:**

```kotlin
@MockK
lateinit var repo: UserRepository

@InjectMockKs
lateinit var service: UserService
```

Injects mocks into the tested class.

**Interview version:**

> `@InjectMockKs` automatically injects mocked dependencies into the class under test, similar to dependency injection.

---

## 20. What are the advantages of MockK?

**Answer:**

* Kotlin-native API
* Coroutine support
* Final class mocking
* Object mocking
* Companion object mocking
* Top-level function mocking
* Cleaner syntax

**Interview version:**

> MockK provides comprehensive Kotlin support and reduces test complexity by handling Kotlin-specific constructs directly.

---

### One-Liner Interview Summary

> MockK is a Kotlin-first mocking framework. I primarily use `mockk`, `every`, `verify`, `coEvery`, and `coVerify` for unit testing, and MockK's biggest strengths are coroutine support, object mocking, top-level function mocking, and seamless handling of Kotlin final classes.
