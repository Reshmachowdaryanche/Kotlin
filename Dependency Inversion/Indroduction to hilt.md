# Introduction to Hilt


# What is Hilt?

Hilt is a **Dependency Injection (DI)** library for Android built on top of **Dagger**.

It reduces boilerplate code and provides predefined components tied to Android lifecycles.

---

# Why Hilt?

Without Hilt:

* Manual object creation
* Difficult dependency management
* Hard testing
* Boilerplate Dagger setup

With Hilt:

* Automatic dependency injection
* Lifecycle-aware components
* Easier testing
* Less boilerplate
* Better scalability

---

# Hilt Components

Hilt creates components automatically.

Each component is tied to a specific Android lifecycle.

---

# Hilt Component Hierarchy

```text id="c9q1sv"
SingletonComponent
├── ActivityRetainedComponent
│   ├── ViewModelComponent
├── ActivityComponent
│   ├── FragmentComponent
│   ├── ViewComponent
│   ├── ViewWithFragmentComponent
├── ServiceComponent
```

---

# Important Interview Point

Child components can access dependencies from parent components.

Example:

* FragmentComponent can access ActivityComponent dependencies
* ViewModelComponent can access ActivityRetainedComponent dependencies

---

# Lifecycle of Hilt Components

| Component                 | Scope                     | Lifecycle                      |
| ------------------------- | ------------------------- | ------------------------------ |
| SingletonComponent        | `@Singleton`              | Entire app lifecycle           |
| ActivityRetainedComponent | `@ActivityRetainedScoped` | Survives configuration changes |
| ViewModelComponent        | `@ViewModelScoped`        | Until ViewModel cleared        |
| ActivityComponent         | `@ActivityScoped`         | Activity lifecycle             |
| FragmentComponent         | `@FragmentScoped`         | Fragment lifecycle             |
| ViewComponent             | `@ViewScoped`             | View lifecycle                 |
| ViewWithFragmentComponent | No direct scope           | Views inside Fragment          |
| ServiceComponent          | `@ServiceScoped`          | Service lifecycle              |

---

# 1. SingletonComponent

# Definition

Root component of Hilt.

Lives as long as application lives.

---

# Used For

* Retrofit
* Room Database
* SharedPreferences
* Repository
* Network clients

---

# Example

```kotlin id="aq9d2f"
@Module
@InstallIn(SingletonComponent::class)
object AppModule {

    @Provides
    @Singleton
    fun provideRetrofit(): Retrofit {

        return Retrofit.Builder()
            .baseUrl("https://example.com/")
            .addConverterFactory(
                GsonConverterFactory.create()
            )
            .build()
    }
}
```

---

# Important Point

Only one instance exists throughout app.

---

# 2. ActivityRetainedComponent

# Definition

Survives configuration changes.

Used mainly for ViewModel dependencies.

---

# Lifecycle

* Screen rotation → survives
* Activity destroyed permanently → destroyed

---

# Example

```kotlin id="y6vbgw"
@HiltViewModel
class MyViewModel @Inject constructor(
    private val repository: UserRepository
) : ViewModel()
```

---

# Module Example

```kotlin id="nt56v9"
@Module
@InstallIn(ActivityRetainedComponent::class)
object ViewModelModule {

    @Provides
    @ActivityRetainedScoped
    fun provideUserRepository(
        apiService: ApiService
    ): UserRepository {

        return UserRepository(apiService)
    }
}
```

---

# Important Interview Point

`UserRepository` will NOT recreate during rotation.

---

# 3. ViewModelComponent

# Definition

Lives as long as ViewModel exists.

---

# Used For

* UI state
* ViewModel-specific repository
* UseCases

---

# Example

```kotlin id="8rlcjp"
@HiltViewModel
class UserViewModel @Inject constructor(
    private val repository: UserRepository
) : ViewModel()
```

---

# Scope

```kotlin id="u9g2j6"
@ViewModelScoped
```

---

# 4. ActivityComponent

# Definition

Exists only during Activity lifecycle.

Destroyed when Activity is destroyed.

---

# Used For

* Adapter
* NavigationController
* UI objects

---

# Example

```kotlin id="f0w4km"
@AndroidEntryPoint
class MainActivity : AppCompatActivity() {

    @Inject
    lateinit var adapter: MyAdapter
}
```

---

# Module Example

```kotlin id="yus9e8"
@Module
@InstallIn(ActivityComponent::class)
object ActivityModule {

    @Provides
    @ActivityScoped
    fun provideAdapter(): MyAdapter {

        return MyAdapter()
    }
}
```

---

# Important Point

New Activity = new instance.

---

# 5. FragmentComponent

# Definition

Lives during Fragment lifecycle.

Destroyed when Fragment is destroyed.

---

# Used For

* Fragment-specific adapters
* UI objects
* Fragment utilities

---

# Example

```kotlin id="olp3n8"
@AndroidEntryPoint
class UserFragment : Fragment() {

    @Inject
    lateinit var adapter: UserAdapter
}
```

---

# Module Example

```kotlin id="bknq9r"
@Module
@InstallIn(FragmentComponent::class)
object FragmentModule {

    @Provides
    @FragmentScoped
    fun provideUserAdapter(): UserAdapter {

        return UserAdapter()
    }
}
```

---

# 6. ViewComponent

# Definition

Exists only during View lifecycle.

Used for custom views.

---

# Example

```kotlin id="j3q1as"
@AndroidEntryPoint
class CustomView @JvmOverloads constructor(
    context: Context,
    attrs: AttributeSet? = null
) : View(context, attrs) {

    @Inject
    lateinit var customPainter: CustomPainter
}
```

---

# Module Example

```kotlin id="m7t4ku"
@Module
@InstallIn(ViewComponent::class)
object ViewModule {

    @Provides
    @ViewScoped
    fun provideCustomPainter(): CustomPainter {

        return CustomPainter()
    }
}
```

---

# 7. ViewWithFragmentComponent

# Definition

Special component used for Views inside Fragments.

---

# Why Needed?

Normally:

* View can access only ViewComponent dependencies
* Cannot access Fragment dependencies

`ViewWithFragmentComponent` solves this problem.

---

# Problem Without It

```kotlin id="1f6rqe"
class MyFragment : Fragment() {

    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View {

        return CustomView(requireContext())
    }
}
```

CustomView cannot access Fragment dependencies.

---

# Solution

Hilt internally uses:

# ViewWithFragmentComponent

to inherit Fragment dependencies.

---

# Example

## Dependency

```kotlin id="f8m5ya"
class MyDependency @Inject constructor() {

    fun doSomething() =
        "Dependency Working!"
}
```

---

## Custom View

```kotlin id="n1r5tx"
@AndroidEntryPoint
class CustomView @JvmOverloads constructor(
    context: Context,
    attrs: AttributeSet? = null
) : View(context, attrs) {

    @Inject
    lateinit var myDependency: MyDependency
}
```

---

## Fragment

```kotlin id="e6h8kv"
@AndroidEntryPoint
class MyFragment : Fragment() {

    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View {

        return CustomView(requireContext())
    }
}
```

---

# Important Interview Point

Without `ViewWithFragmentComponent`,
Views inside Fragments cannot access Fragment dependencies.

---

# 8. ServiceComponent

# Definition

Used for Android Services.

---

# Used For

* Foreground services
* Background tasks
* Firebase Messaging
* Location tracking

---

# Example

```kotlin id="t6r9ox"
@AndroidEntryPoint
class MyService : Service() {

    @Inject
    lateinit var locationTracker: LocationTracker
}
```

---

# Module Example

```kotlin id="gho2u4"
@Module
@InstallIn(ServiceComponent::class)
object ServiceModule {

    @Provides
    @ServiceScoped
    fun provideLocationTracker(): LocationTracker {

        return LocationTracker()
    }
}
```

---

# Scopes in Hilt

# What is Scope?

Scope decides:

* How long dependency lives
* When object recreated
* Which component owns object

---

# Common Scopes

| Scope                     | Lifetime        |
| ------------------------- | --------------- |
| `@Singleton`              | Entire app      |
| `@ActivityRetainedScoped` | Across rotation |
| `@ViewModelScoped`        | ViewModel       |
| `@ActivityScoped`         | Activity        |
| `@FragmentScoped`         | Fragment        |
| `@ViewScoped`             | View            |
| `@ServiceScoped`          | Service         |

---

# Important Interview Questions

# Why Hilt over Dagger?

Hilt:

* Reduces boilerplate
* Predefined Android components
* Easier lifecycle management
* Easier setup

---

# Difference Between SingletonComponent & ActivityRetainedComponent

| Singleton               | ActivityRetained                 |
| ----------------------- | -------------------------------- |
| Entire app              | Activity + rotation              |
| App-wide objects        | ViewModel related                |
| Destroyed when app dies | Destroyed when activity finishes |

---

# Difference Between ActivityScoped & FragmentScoped

| ActivityScoped          | FragmentScoped      |
| ----------------------- | ------------------- |
| Shared across fragments | Unique per fragment |
| Lives with activity     | Lives with fragment |

---

# Important Hilt Annotations

| Annotation           | Purpose                       |
| -------------------- | ----------------------------- |
| `@Inject`            | Inject dependency             |
| `@Module`            | Dependency provider class     |
| `@Provides`          | Provide object manually       |
| `@Binds`             | Bind interface implementation |
| `@InstallIn`         | Define component              |
| `@HiltAndroidApp`    | Initialize Hilt               |
| `@AndroidEntryPoint` | Enable injection              |
| `@HiltViewModel`     | Inject ViewModel              |

---

# Hilt Internal Working

# Internally Hilt:

1. Generates Dagger code
2. Creates dependency graph
3. Resolves dependencies at compile time
4. Creates scoped containers
5. Injects dependencies automatically

---

# 1-Minute Interview Answer

# What is Hilt?

> Hilt is a dependency injection library built on top of Dagger for Android.

It reduces boilerplate and provides lifecycle-aware dependency injection.

---

# What are Hilt Components?

> Components are generated containers that manage dependencies according to Android lifecycle.

Example:

* SingletonComponent → app lifecycle
* ActivityComponent → activity lifecycle
* FragmentComponent → fragment lifecycle

---

# What is Scope?

> Scope defines how long dependency should live.

Example:

* `@Singleton`
* `@ActivityScoped`
* `@FragmentScoped`

---

# Most Important Interview Line

> Hilt manages object creation and lifecycle automatically using generated Dagger components.
