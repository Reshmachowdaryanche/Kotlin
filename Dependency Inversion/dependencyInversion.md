Here’s your content converted into clean **GitHub README.md notes format** for interview preparation and revision.
Based on your uploaded PDF 

---

# Dependency Inversion Principle (DIP) & Dependency Injection (DI)

# 1. Dependency Inversion Principle (DIP)

## Definition

Dependency Inversion Principle is one of the **SOLID principles**.

It states:

1. High-level modules should not depend on low-level modules.
   Both should depend on abstractions.

2. Abstractions should not depend on details.
   Details should depend on abstractions.

---

## Why DIP?

DIP helps achieve:

* Loose coupling
* Better maintainability
* Easier testing
* Better scalability
* Easier replacement of implementations

---

# ❌ DIP Violation (Bad Example)

```kotlin
class RealmDatabase {
    fun fetchUser() {
        println("Fetching user from Realm Database")
    }
}

class UserRepository {

    private val database = RealmDatabase()

    fun getUser() {
        database.fetchUser()
    }
}

fun main() {
    val repository = UserRepository()
    repository.getUser()
}
```

---

## Problems in Above Code

* `UserRepository` directly depends on `RealmDatabase`
* Tight coupling
* Difficult to replace database
* Violates Open-Closed Principle

---

## Problem When Switching Database

If we switch to `RoomDatabase`:

```kotlin
class RoomDatabase {
    fun fetchUser() {
        println("Fetching user from Room Database")
    }
}

class UserRepository {

    private val database = RoomDatabase()

    fun getUser() {
        database.fetchUser()
    }
}
```

---

## Issue

Every time database changes:

* `UserRepository` must change
* Existing code gets modified
* Hard to maintain
* Difficult to scale

---

# ✅ Applying DIP (Good Example)

```kotlin
interface Database {
    fun fetchUser()
}

class RealmDatabase : Database {
    override fun fetchUser() {
        println("Fetching user from Realm Database")
    }
}

class RoomDatabase : Database {
    override fun fetchUser() {
        println("Fetching user from Room Database")
    }
}

class UserRepository(
    private val database: Database
) {

    fun getUser() {
        database.fetchUser()
    }
}

fun main() {

    val realmRepository =
        UserRepository(RealmDatabase())

    realmRepository.getUser()

    val roomRepository =
        UserRepository(RoomDatabase())

    roomRepository.getUser()
}
```

---

# ✅ Benefits of DIP

* Repository depends on abstraction (`Database`)
* Easy to swap implementations
* Follows Open-Closed Principle
* Better testing and mocking
* Cleaner architecture

---

# Important Interview Point

Without understanding DIP:

* You may use Hilt blindly
* Debugging DI becomes difficult
* Code becomes hard to maintain

---

# 2. Dependency Injection (DI)

# Definition

Dependency Injection is a design pattern used to achieve:

# Inversion of Control (IoC)

Instead of creating dependencies inside class,
dependencies are injected from outside.

---

# Benefits of DI

* Loose coupling
* Better testing
* Easier mocking
* Better maintainability
* Reusable components

---

# Types of Dependency Injection

1. Constructor Injection
2. Field Injection
3. Method Injection
4. Interface Injection

---

# 1. Constructor Injection (Preferred)

## Definition

Dependencies are passed through constructor.

---

## Example

```kotlin
class Database {

    fun queryUser(userId: String): String {
        return "User data for $userId"
    }
}

class UserRepository(
    private val database: Database
) {

    fun getUser(userId: String): String {
        return database.queryUser(userId)
    }
}

fun main() {

    val database = Database()

    val userRepository =
        UserRepository(database)

    val userData =
        userRepository.getUser("12345")

    println(userData)
}
```

---

# Why Constructor Injection is Best?

* Dependency becomes mandatory
* Immutable (`val`)
* Easier unit testing
* Cleaner architecture
* Prevents null issues

---

# Constructor Injection Using Hilt

```kotlin
@HiltAndroidApp
class MyApp : Application()

class Database @Inject constructor() {

    fun queryUser(userId: String): String {
        return "User data for $userId"
    }
}

class UserRepository @Inject constructor(
    private val database: Database
) {

    fun getUser(userId: String): String {
        return database.queryUser(userId)
    }
}

@AndroidEntryPoint
class MainActivity : AppCompatActivity() {

    @Inject
    lateinit var userRepository: UserRepository

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        val userData =
            userRepository.getUser("12345")

        Log.d("HiltExample", userData)
    }
}
```

---

# 2. Field Injection

## Definition

Dependencies are injected directly into fields.

---

## Example

```kotlin
class Database {

    fun queryUser(userId: String): String {
        return "User data for $userId"
    }
}

class UserRepository {

    lateinit var database: Database

    fun getUser(userId: String): String {
        return database.queryUser(userId)
    }
}

fun main() {

    val database = Database()

    val userRepository = UserRepository()

    userRepository.database = database

    val userData =
        userRepository.getUser("12345")

    println(userData)
}
```

---

# Problems with Field Injection

* Mutable dependency
* Can cause null issues
* Harder to test
* Dependency not guaranteed

---

# Field Injection Using Hilt

```kotlin
@HiltAndroidApp
class MyApp : Application()

@Module
@InstallIn(SingletonComponent::class)
object DatabaseModule {

    @Provides
    fun provideDatabase(): Database {
        return Database()
    }
}

@AndroidEntryPoint
class UserRepository @Inject constructor() {

    @Inject
    lateinit var database: Database

    fun getUser(userId: String): String {
        return database.queryUser(userId)
    }
}
```

---

# 3. Method Injection

## Definition

Dependencies are passed via methods/setters.

---

## Example

```kotlin
class Database {

    fun queryUser(userId: String): String {
        return "User data for $userId"
    }
}

class UserRepository {

    private lateinit var database: Database

    fun setDatabase(database: Database) {
        this.database = database
    }

    fun getUser(userId: String): String {
        return database.queryUser(userId)
    }
}

fun main() {

    val database = Database()

    val repository = UserRepository()

    repository.setDatabase(database)

    println(repository.getUser("12345"))
}
```

---

# When to Use Method Injection?

Use when:

* Dependency changes dynamically
* Optional dependency
* Runtime dependency changes

---

# 4. Interface Injection

## Definition

Dependency is injected through interface contract.

---

## Example

```kotlin
interface DatabaseService {

    fun queryUser(userId: String): String
}

class RealDatabase : DatabaseService {

    override fun queryUser(userId: String): String {
        return "User data for $userId"
    }
}

class UserRepository {

    private lateinit var databaseService: DatabaseService

    fun setDatabaseService(
        databaseService: DatabaseService
    ) {
        this.databaseService = databaseService
    }

    fun getUser(userId: String): String {
        return databaseService.queryUser(userId)
    }
}
```

---

# Constructor vs Field vs Method vs Interface Injection

| Type                  | How Injected      | Best Use Case                         |
| --------------------- | ----------------- | ------------------------------------- |
| Constructor Injection | Via constructor   | Required dependencies                 |
| Field Injection       | Via fields        | When constructor injection impossible |
| Method Injection      | Via setter method | Dynamic dependencies                  |
| Interface Injection   | Via interface     | Swappable implementations             |

---

# 3. Dependency Injection Using Hilt (Android)

---

# Step 1 — Add Dependencies

```gradle
implementation("com.google.dagger:hilt-android:2.48")
kapt("com.google.dagger:hilt-compiler:2.48")
```

---

# Step 2 — Enable Hilt

```kotlin
@HiltAndroidApp
class MyApp : Application()
```

---

# Step 3 — Create Interface

```kotlin
interface UserRepository {

    suspend fun getUserById(userId: String): String
}
```

---

# Step 4 — Database Service Interface

```kotlin
interface DatabaseService {

    suspend fun queryUser(userId: String): String
}
```

---

# Step 5 — Room Entity

```kotlin
@Entity(tableName = "users")
data class User(

    @PrimaryKey
    val id: String,

    val name: String
)
```

---

# Step 6 — DAO

```kotlin
@Dao
interface UserDao {

    @Query("SELECT * FROM users WHERE id = :userId")
    suspend fun getUser(userId: String): User?

    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insertUser(user: User)
}
```

---

# Step 7 — Room Database

```kotlin
@Database(
    entities = [User::class],
    version = 1
)
abstract class AppDatabase : RoomDatabase() {

    abstract fun userDao(): UserDao
}
```

---

# Step 8 — Database Service Implementation

```kotlin
class RoomDatabaseService @Inject constructor(
    private val userDao: UserDao
) : DatabaseService {

    override suspend fun queryUser(
        userId: String
    ): String {

        return userDao.getUser(userId)?.name
            ?: "User not found"
    }
}
```

---

# Step 9 — Repository Implementation

```kotlin
class UserRepositoryImpl @Inject constructor(
    private val databaseService: DatabaseService
) : UserRepository {

    override suspend fun getUserById(
        userId: String
    ): String {

        return databaseService.queryUser(userId)
    }
}
```

---

# Step 10 — Hilt Module

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object DatabaseModule {

    @Provides
    fun provideDatabase(
        application: Application
    ): AppDatabase {

        return Room.databaseBuilder(
            application,
            AppDatabase::class.java,
            "app_db"
        ).build()
    }

    @Provides
    fun provideUserDao(
        database: AppDatabase
    ): UserDao {

        return database.userDao()
    }
}
```

---

# Step 11 — Bind Interfaces

```kotlin
@Module
@InstallIn(SingletonComponent::class)
abstract class RepositoryModule {

    @Binds
    abstract fun bindUserRepository(
        impl: UserRepositoryImpl
    ): UserRepository
}

@Module
@InstallIn(SingletonComponent::class)
abstract class DatabaseServiceModule {

    @Binds
    abstract fun bindDatabaseService(
        impl: RoomDatabaseService
    ): DatabaseService
}
```

---

# Step 12 — ViewModel Injection

```kotlin
@HiltViewModel
class UserViewModel @Inject constructor(
    private val userRepository: UserRepository
) : ViewModel() {

    fun fetchUser(userId: String) {

        viewModelScope.launch {

            val user =
                userRepository.getUserById(userId)

            Log.d("HiltExample", user)
        }
    }
}
```

---

# Step 13 — Inject ViewModel in Activity

```kotlin
@AndroidEntryPoint
class MainActivity : ComponentActivity() {

    private val userViewModel: UserViewModel by viewModels()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        userViewModel.fetchUser("12345")
    }
}
```

---

# Interview Summary (1-Minute Answer)

# What is DIP?

> High-level modules should not depend on low-level modules.
> Both should depend on abstractions.

Example:
Repository should depend on `Database` interface instead of `RoomDatabase`.

---

# What is DI?

> DI means providing dependencies from outside instead of creating them inside class.

Benefits:

* Loose coupling
* Better testing
* Easier maintenance
* Cleaner architecture

---

# Best Injection Type?

✅ Constructor Injection

Because:

* Mandatory dependency
* Immutable
* Easy testing
* Cleaner code

---

# Hilt Internally

Hilt:

* Generates dependency graph
* Creates objects automatically
* Resolves dependencies at compile time
* Uses Dagger internally

---

# Important Interview Line

> DIP is the principle.
> DI is the implementation of that principle.
