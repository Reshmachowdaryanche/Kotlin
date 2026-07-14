# Module 11 — Complete Android Networking Architecture (Senior Interview Level)

This module is the **integration module**.

Until now we learned individual pieces:

* Retrofit
* OkHttp
* Interceptors
* Authentication
* Error handling
* Caching
* Coroutines
* Flow

Now we combine everything into a production Android architecture.

This is the type of question asked in senior Android interviews:

> "Design a scalable networking layer for a large Android application."

---

# Interview Goal

After this module, you should be able to design and explain:

* Complete API architecture
* Package structure
* Retrofit setup
* OkHttp configuration
* Interceptor placement
* Authentication flow
* Repository responsibility
* Offline-first approach
* Error handling
* Testing strategy
* Dependency injection

---

# Part 1 — The Big Picture

A production Android networking architecture:

```text
                         UI Layer

                            |
                            |

                       ViewModel

                            |
                            |

                      Repository

                    /             \

                   /               \

              Local Source      Remote Source

                  |                  |

                Room             Retrofit

                                     |

                                  OkHttp

                                     |

                              Interceptors

                                     |

                                  Server
```

---

The main idea:

**UI does not know about networking.**

UI should not know:

* Retrofit
* HTTP codes
* Tokens
* Database
* Exceptions

---

# Part 2 — Layer Responsibilities

A common interview question:

> "Where should each networking responsibility live?"

Let's define.

---

# UI Layer

Responsibilities:

* Display data
* Show loading
* Show errors
* Handle user actions

Should NOT:

❌ Call Retrofit
❌ Parse JSON
❌ Handle tokens

Example:

```kotlin
viewModel.loadUsers()
```

---

# ViewModel

Responsibilities:

* Manage UI state
* Call repository
* Survive configuration changes

Example:

```kotlin
viewModelScope.launch {

    repository.getUsers()

}
```

Should NOT:

❌ Create Retrofit instance
❌ Add headers
❌ Handle HTTP logic

---

# Repository

This is the brain.

Responsibilities:

* Decide data source
* Combine local + remote
* Convert errors
* Apply business rules

Example:

```text
Repository

        |
        |
  Should I use:

  Room?

  API?

  Both?
```

---

# Data Sources

Two types:

## Remote Data Source

Responsible for:

* Retrofit calls

Example:

```kotlin
class UserRemoteDataSource(
    private val api: UserApi
)
```

---

## Local Data Source

Responsible for:

* Room database

Example:

```kotlin
class UserLocalDataSource(
    private val dao: UserDao
)
```

---

# Part 3 — Recommended Package Structure

A scalable project:

```
com.example.app

├── data
│
│── remote
│   ├── api
│   ├── dto
│   ├── interceptor
│   └── RemoteDataSource
│
│── local
│   ├── database
│   ├── dao
│   └── LocalDataSource
│
│── repository
│
├── domain
│
│── model
│── usecase
│
├── presentation
│
│── viewmodel
│── ui
```

---

Why separate?

Because:

* Easier testing
* Easier maintenance
* Less coupling

---

# Part 4 — Retrofit Setup

Usually create one Retrofit instance.

Example:

```kotlin
object RetrofitClient {

    fun create(): Retrofit {

        return Retrofit.Builder()
            .baseUrl(BASE_URL)
            .client(okHttpClient)
            .addConverterFactory(
                GsonConverterFactory.create()
            )
            .build()
    }
}
```

---

Important interview point:

Do not create Retrofit for every API call.

Bad:

```kotlin
buttonClick(){

 Retrofit.Builder()

}
```

Why?

* Expensive
* Duplicate objects
* Bad resource usage

---

# Part 5 — OkHttp Setup

OkHttp handles:

* Connections
* Timeouts
* Interceptors
* Cache

Example:

```kotlin
val client =
    OkHttpClient.Builder()
        .addInterceptor(authInterceptor)
        .addInterceptor(loggingInterceptor)
        .connectTimeout(
            30,
            TimeUnit.SECONDS
        )
        .build()
```

---

# Part 6 — Interceptor Architecture

A typical chain:

```text
Request

↓

Logging Interceptor

↓

Header Interceptor

↓

Auth Interceptor

↓

Retry

↓

Network

↓

Server
```

---

Example responsibilities:

## Logging

Development only:

```
Request URL
Headers
Response time
```

---

## Auth

Adds:

```http
Authorization:
Bearer token
```

---

## Header

Adds:

```http
App-Version: 5.0
Platform: Android
```

---

# Part 7 — Authentication Architecture

Complete flow:

```
Login

↓

Server

↓

Access Token
Refresh Token

↓

Secure Storage

↓

API Request

↓

Auth Interceptor

↓

Bearer Token
```

---

When token expires:

```
API Request

↓

401

↓

Authenticator

↓

Refresh Token API

↓

New Access Token

↓

Retry Original Request
```

---

Important:

Token storage:

Use:

* Android Keystore
* Encrypted storage

Avoid:

* Plain SharedPreferences

---

# Part 8 — API Interface Design

Example:

```kotlin
interface UserApi {

    @GET("users")
    suspend fun getUsers():
        List<UserDto>


    @POST("login")
    suspend fun login(
        @Body request:LoginRequest
    ): LoginResponse

}
```

---

Notice:

API returns DTO.

Not UI models.

---

Why?

Because API changes should not break UI.

---

# Part 9 — DTO vs Domain Model

Very common interview topic.

Example API response:

```json
{
"id":1,
"user_name":"John"
}
```

DTO:

```kotlin
data class UserDto(
    val id:Int,
    val user_name:String
)
```

Domain:

```kotlin
data class User(
    val id:Int,
    val name:String
)
```

Mapping:

```kotlin
fun UserDto.toUser():

User {

}
```

---

Why?

Because:

Backend model ≠ Application model

---

# Part 10 — Repository Example

Example:

```kotlin
class UserRepository(
    private val api:UserApi,
    private val dao:UserDao
){

    fun users():Flow<List<User>>{

        return dao.observeUsers()

    }


    suspend fun refresh(){

        val users =
            api.getUsers()

        dao.insert(users)

    }

}
```

---

Flow:

```
UI

↓

Room Flow

↓

Repository

↓

API updates Room

↓

Flow emits new data
```

---

# Part 11 — Error Handling Architecture

Never expose Retrofit exceptions directly.

Bad:

```kotlin
try {

api.call()

}
catch(Exception e){

}
```

everywhere.

---

Better:

```
Retrofit Exception

↓

Repository

↓

NetworkResult

↓

ViewModel

↓

UI State
```

---

Example:

```kotlin
sealed class Result<T>{

data class Success<T>(
 val data:T
):Result<T>()

data class Error<T>(
 val message:String
):Result<T>()

}
```

---

# Part 12 — UI State Architecture

Modern approach:

```kotlin
sealed interface UiState {

data object Loading:
UiState


data class Success(
 val users:List<User>
):UiState


data class Error(
 val message:String
):UiState

}
```

---

ViewModel:

```kotlin
private val _state =
MutableStateFlow<UiState>(
 UiState.Loading
)
```

---

UI:

```kotlin
state.collect {

when(it){

Loading -> showLoader()

Success -> showData()

Error -> showError()

}

}
```

---

# Part 13 — Offline First Architecture

Production flow:

```
User Opens Screen

↓

Room emits cached data

↓

UI displays immediately

↓

Repository refreshes API

↓

Save response into Room

↓

Room emits updated data

↓

UI updates
```

---

This gives:

* Fast startup
* Offline capability
* Consistent data

---

# Part 14 — Dependency Injection

A common interview question:

> How do you provide Retrofit and repositories?

Usually:

Hilt

Example:

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object NetworkModule{


@Provides
fun provideRetrofit():

Retrofit {


}

}
```

---

Benefits:

* Single Retrofit instance
* Easier testing
* Cleaner dependencies

---

# Part 15 — Testing Networking Layer

Senior developers should mention testing.

---

## API Testing

Use:

* MockWebServer

Flow:

```
Fake Server

↓

Retrofit

↓

Repository

↓

Test
```

Test:

* Success response
* 404
* 500
* Timeout

---

## Repository Testing

Fake:

```kotlin
FakeApi
FakeDatabase
```

Test business logic.

---

## ViewModel Testing

Test:

```
Input

↓

ViewModel

↓

StateFlow

↓

Expected UI State
```

---

# Part 16 — Common Senior Interview Questions

---

## Q1. Design a scalable networking layer.

Strong answer:

> I would separate responsibilities using Retrofit for API communication, OkHttp for networking concerns like authentication and logging, repositories for coordinating data sources, Room for local persistence, and ViewModels for exposing UI state. The UI observes data through Flow, while errors are converted into application-level results.

---

## Q2. Where do you handle token refresh?

Answer:

> Token attachment can happen in an interceptor, but refresh handling should happen in OkHttp Authenticator because it is designed for authentication challenges like 401 responses.

---

## Q3. Where should caching happen?

Answer:

> Caching should usually be handled in the repository layer. HTTP caching can be handled by OkHttp, while application data caching should typically use Room.

---

## Q4. Why not call Retrofit directly from ViewModel?

Answer:

Because it couples presentation with networking and makes testing difficult. Repository provides abstraction.

---

## Q5. Why use Flow with Room?

Answer:

Because Room can emit database changes reactively. The UI automatically updates whenever local data changes.

---

# Part 17 — Complete Production Flow

Let's combine everything:

```
User Action

↓

ViewModel

↓

Repository

↓

Check Local Database

↓

Emit Cached Data

↓

Retrofit Suspend API

↓

OkHttp

↓

Interceptors

↓

Authentication

↓

Server

↓

Response

↓

Save Database

↓

Room Flow Emits

↓

StateFlow Updates

↓

UI Refresh
```

---

# Final Interview Answer

If an interviewer asks:

> "Explain your Android networking architecture."

Answer:

> "I use a layered architecture where the UI communicates with ViewModels, ViewModels communicate with repositories, and repositories coordinate between local and remote data sources. Retrofit handles API definitions, OkHttp manages networking concerns such as interceptors, authentication headers, logging, and timeouts. For offline support, Room acts as the single source of truth, with network responses updating the database and UI observing database changes through Flow. Errors are converted into domain-level results inside the repository, and dependency injection provides reusable instances of networking components."

---

# Module 11 Summary

You can now explain:

✅ Complete networking architecture
✅ Retrofit setup
✅ OkHttp responsibility
✅ Interceptor placement
✅ Token architecture
✅ Repository pattern
✅ Offline-first design
✅ Room integration
✅ Flow + StateFlow
✅ DTO mapping
✅ Error architecture
✅ Testing strategy
✅ Dependency injection

---

## Next Module: Module 12 — Advanced Retrofit + OkHttp Internals

This is the final deep networking interview module.

Topics:

* OkHttp Dispatcher
* Connection Pool
* HTTP/2
* DNS
* TLS/SSL
* Certificate Pinning
* Interceptor internals
* Retrofit converter internals
* Call lifecycle
* Performance optimization

This module targets **senior Android engineer interviews**.
