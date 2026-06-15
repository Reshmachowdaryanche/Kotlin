# Hilt Annotations Notes

Based on your uploaded PDF 

---

# 1. `@HiltAndroidApp`

# Purpose

Marks the `Application` class as the entry point for Hilt.

It initializes the entire Hilt Dependency Injection graph.

---

# Example

```kotlin id="z6y7tp"
@HiltAndroidApp
class MyApplication : Application()
```

---

# What Happens Internally?

Hilt generates:

```text id="vlp0zb"
Hilt_MyApplication
```

which manages:

* Dependency graph
* Component creation
* Application-level injection

---

# Generated Flow (Simplified)

```text id="4mby1s"
MyApplication
    ↓
Hilt_MyApplication
    ↓
DaggerApplicationComponent
```

---

# Internal Generated Code (Simplified)

```kotlin id="2hfr0x"
class Hilt_MyApplication : Application() {

    private lateinit var applicationComponent:
        ApplicationComponent

    override fun onCreate() {
        super.onCreate()

        applicationComponent =
            DaggerApplicationComponent.create()
    }
}
```

---

# Important Interview Point

`@HiltAndroidApp`:

* Bootstraps Hilt
* Creates root dependency graph
* Creates components automatically

---

# Dependency Graph Meaning

A graph showing object relationships.

Example:

```text id="r3x4a1"
Activity
   ↓
Repository
   ↓
ApiService
```

---

# 2. `@AndroidEntryPoint`

# Purpose

Enables dependency injection into Android components.

Used on:

* Activity
* Fragment
* Service
* BroadcastReceiver
* View

---

# Example

```kotlin id="4a6jhk"
@AndroidEntryPoint
class MyActivity : AppCompatActivity() {

    @Inject
    lateinit var repository: MyRepository
}
```

---

# Internal Working

Hilt generates:

```text id="7kq4wr"
Hilt_MyActivity
```

which injects dependencies before `onCreate()`.

---

# Generated Code (Simplified)

```kotlin id="ie4lmn"
class Hilt_MyActivity : AppCompatActivity() {

    @Inject
    lateinit var repository: MyRepository

    override fun onCreate(
        savedInstanceState: Bundle?
    ) {

        inject()

        super.onCreate(savedInstanceState)
    }

    private fun inject() {

        MyApplication
            .getApplicationComponent()
            .inject(this)
    }
}
```

---

# Important Interview Point

Injection happens BEFORE:

```kotlin id="frz1c3"
super.onCreate()
```

---

# 3. `@Inject`

# Purpose

Marks constructor or field for dependency injection.

---

# Constructor Injection Example

```kotlin id="x8d4mj"
class MyRepository @Inject constructor(
    private val apiService: ApiService
) {

    fun fetchUser() {
        apiService.fetch()
    }
}
```

---

# Field Injection Example

```kotlin id="ob9tx4"
@Inject
lateinit var repository: MyRepository
```

---

# Internal Working

Hilt generates Factory classes automatically.

---

# Generated Factory (Simplified)

```kotlin id="2ktjlwm"
class MyRepository_Factory(
    private val apiServiceProvider:
        Provider<ApiService>
) : Factory<MyRepository> {

    override fun get(): MyRepository {

        return MyRepository(
            apiServiceProvider.get()
        )
    }
}
```

---

# Important Concepts

# `Provider<T>`

Lazy provider of dependency.

Object created only when needed.

---

# Important Interview Point

`@Inject constructor()` allows Hilt to know:

* How object should be created
* What dependencies required

---

# 4. `@Module`

# Purpose

Marks class/object as dependency provider container.

---

# Example

```kotlin id="t6g4nh"
@Module
@InstallIn(SingletonComponent::class)
object AppModule {

    @Provides
    fun provideApiService(): ApiService {

        return Retrofit.Builder()
            .baseUrl("https://api.example.com")
            .build()
            .create(ApiService::class.java)
    }
}
```

---

# When Needed?

Use `@Module` when:

* Cannot use constructor injection
* Third-party libraries
* Retrofit
* Room
* OkHttp
* SharedPreferences

---

# 5. `@InstallIn`

# Purpose

Specifies which Hilt component owns dependency.

---

# Example

```kotlin id="n5okx7"
@InstallIn(SingletonComponent::class)
```

---

# Common Components

| Component          | Lifetime   |
| ------------------ | ---------- |
| SingletonComponent | Entire app |
| ActivityComponent  | Activity   |
| FragmentComponent  | Fragment   |
| ViewModelComponent | ViewModel  |

---

# Important Interview Point

`@InstallIn` decides dependency lifecycle.

---

# 6. `@Provides`

# Purpose

Provides dependencies manually.

Used inside modules.

---

# Example

```kotlin id="pt8qrm"
@Module
@InstallIn(SingletonComponent::class)
object NetworkModule {

    @Provides
    fun provideHttpClient(): OkHttpClient {

        return OkHttpClient.Builder()
            .build()
    }
}
```

---

# When to Use `@Provides`?

When dependency:

* Has no `@Inject constructor`
* Needs configuration
* Third-party class

---

# Example

* Retrofit
* OkHttp
* Room Database

---

# 7. `@Binds`

# Purpose

Binds interface to implementation.

---

# Example

```kotlin id="9zc7xa"
interface MyRepository {

    fun getData(): String
}

class MyRepositoryImpl @Inject constructor()
    : MyRepository {

    override fun getData() = "Hello"
}
```

---

# Module

```kotlin id="h4pk9f"
@Module
@InstallIn(SingletonComponent::class)
abstract class RepositoryModule {

    @Binds
    abstract fun bindMyRepository(
        impl: MyRepositoryImpl
    ): MyRepository
}
```

---

# Internal Working

Hilt generates binding factory.

---

# Generated Code (Simplified)

```kotlin id="vmk4z1"
class RepositoryModule_BindMyRepositoryFactory
    : Factory<MyRepository> {

    override fun get(): MyRepository {

        return MyRepositoryImpl()
    }
}
```

---

# Difference Between `@Provides` and `@Binds`

| `@Provides`              | `@Binds`                         |
| ------------------------ | -------------------------------- |
| Creates object manually  | Maps interface to implementation |
| Works with object/module | Works with abstract module       |
| More flexible            | More efficient                   |

---

# Important Interview Point

Prefer `@Binds` for interfaces because:

* Less generated code
* Better performance

---

# 8. `@Singleton`

# Purpose

Creates single shared instance in app lifecycle.

---

# Example

```kotlin id="a8h5wc"
@Singleton
class UserManager @Inject constructor()
```

---

# Used For

* Retrofit
* Room Database
* Repository
* Managers

---

# Important Interview Point

Single instance exists only inside:

```kotlin id="jlwmn6"
SingletonComponent
```

---

# 9. `@EntryPoint`

# Purpose

Access Hilt dependencies where:

```text id="53jlwm"
@AndroidEntryPoint
```

cannot be used.

---

# Used In

* Utility classes
* Third-party classes
* Plain Kotlin classes

---

# Define EntryPoint

```kotlin id="n6y5kp"
@EntryPoint
@InstallIn(SingletonComponent::class)
interface MyEntryPoint {

    fun getMyRepository(): MyRepository
}
```

---

# Access Dependency

```kotlin id="7vcfwr"
class MyUtilityClass {

    private val repository: MyRepository

    init {

        val entryPoint =
            EntryPointAccessors.fromApplication(
                applicationContext,
                MyEntryPoint::class.java
            )

        repository =
            entryPoint.getMyRepository()
    }
}
```

---

# Important Interview Point

`@EntryPoint` is manual dependency retrieval.

---

# 10. `@HiltViewModel`

# Purpose

Inject dependencies into ViewModel.

---

# Example

```kotlin id="6at0wn"
@HiltViewModel
class MyViewModel @Inject constructor(
    private val repository: MyRepository
) : ViewModel()
```

---

# Internal Working

Hilt creates:

```text id="n4j3os"
ViewModelFactory
```

automatically.

---

# Without Hilt

You would manually create:

```kotlin id="4b0lcz"
ViewModelProvider.Factory
```

---

# 11. Custom Scope

# Purpose

Create custom lifecycle scope.

---

# Create Scope

```kotlin id="jv8p2t"
@Scope
@Retention(AnnotationRetention.RUNTIME)
annotation class ActivityScoped
```

---

# Use Scope

```kotlin id="jkw4fa"
@Module
@InstallIn(ActivityComponent::class)
object MyActivityModule {

    @ActivityScoped
    @Provides
    fun provideUserManager(): UserManager {

        return UserManager()
    }
}
```

---

# Inject in Activity

```kotlin id="whq1mz"
@AndroidEntryPoint
class MainActivity : AppCompatActivity() {

    @Inject
    lateinit var userManager: UserManager
}
```

---

# Important Point

Same instance shared during Activity lifecycle.

---

# Full Hilt Flow

```text id="3q9dkl"
@HiltAndroidApp
        ↓
Creates Root Component
        ↓
@Module Provides Dependencies
        ↓
@Inject Marks Injection Points
        ↓
@AndroidEntryPoint Enables Injection
        ↓
Hilt Generates Factories & Components
        ↓
Dependencies Injected Automatically
```

---

# Most Important Hilt Interview Questions

# Difference Between `@Provides` and `@Inject constructor`

| `@Inject constructor` | `@Provides`         |
| --------------------- | ------------------- |
| Automatic creation    | Manual creation     |
| Our own classes       | Third-party classes |

---

# Difference Between `@Provides` and `@Binds`

| `@Provides`      | `@Binds`       |
| ---------------- | -------------- |
| Creates instance | Maps interface |
| More code        | Less code      |
| Supports logic   | No logic       |

---

# Why Hilt Uses Compile-Time DI?

Because:

* Faster runtime
* Detect errors at compile time
* Better performance
* Safer dependency graph

---

# Most Important Interview Line

> Hilt generates Dagger code at compile time to create and manage dependency graphs automatically.

---

# 1-Minute Interview Explanation

# What is Hilt?

> Hilt is a dependency injection library built on top of Dagger that reduces boilerplate and manages object creation automatically.

---

# What does `@Inject` do?

> It tells Hilt how to create dependencies.

---

# What does `@Provides` do?

> It manually provides objects that cannot use constructor injection.

---

# What does `@AndroidEntryPoint` do?

> Enables injection into Android classes like Activity and Fragment.

---

# What does `@HiltAndroidApp` do?

> Initializes the root dependency graph for the entire app.
