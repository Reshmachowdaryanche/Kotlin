
As an Android developer, you don't need to memorize APIs. You need to understand **the journey of one network request**.

---

# The Complete Flow of an Android REST API Request

Imagine your app calls:

```kotlin
api.getUsers()
```

What actually happens?

```
UI (Compose/Activity)
       │
       ▼
Repository
       │
       ▼
Retrofit
       │
       ▼
OkHttp
       │
       ├── Interceptors
       │
       ├── Authentication
       │
       ├── Logging
       │
       ├── Retry
       │
       ├── Timeout
       │
       ▼
HTTP Request
       │
Internet
       │
Server
       │
HTTP Response
       │
OkHttp
       │
Interceptors
       │
Retrofit Converter
       │
Repository
       │
ViewModel
       │
UI
```

Everything fits somewhere in this pipeline.

---

# Step 1: Retrofit

Retrofit is **not** the networking library.

Retrofit is just a wrapper that makes APIs easy.

Instead of

```kotlin
Request request = ...
```

you simply write

```kotlin
interface ApiService {

    @GET("users")
    suspend fun getUsers(): List<User>

}
```

Retrofit converts this

into

```
HTTP Request
```

It delegates the actual networking to **OkHttp**.

Think of Retrofit as

> "I convert Kotlin functions into HTTP requests."

---

# Step 2: OkHttp

OkHttp is the real networking engine.

It

* opens sockets
* sends requests
* downloads responses
* handles HTTPS
* handles redirects
* caches
* retries

Retrofit cannot work without OkHttp.

```
Retrofit
      │
      ▼
OkHttp
      │
Internet
```

---

# Step 3: Builder Pattern

People get scared seeing this.

```kotlin
OkHttpClient.Builder()
```

Builder simply means

> Build an object step by step.

Instead of

```kotlin
OkHttpClient(
 timeout,
 cache,
 auth,
 logging,
 retry,
 ...
)
```

we do

```kotlin
OkHttpClient.Builder()
    .connectTimeout(...)
    .readTimeout(...)
    .writeTimeout(...)
    .addInterceptor(...)
    .build()
```

It is just configuration.

Think Lego.

```
Builder

↓

Add timeout

↓

Add logging

↓

Add auth

↓

Add cache

↓

Build
```

---

# Step 4: Chaining

Why can we do this?

```kotlin
builder
    .addInterceptor(...)
    .connectTimeout(...)
    .build()
```

Because every function returns

```kotlin
this
```

Example

```kotlin
class CarBuilder {

    fun color(): CarBuilder {
        return this
    }

    fun wheels(): CarBuilder {
        return this
    }

}
```

Now

```kotlin
CarBuilder()
    .color()
    .wheels()
```

That's chaining.

---

# Step 5: Interceptors

This is where most Android developers get confused.

Imagine every request passes through security checkpoints.

```
Request

↓

Interceptor 1

↓

Interceptor 2

↓

Interceptor 3

↓

Server
```

Every interceptor can

* inspect request
* modify request
* stop request
* retry request
* inspect response
* modify response

Think

```
Airport Security

↓

Passport Check

↓

Bag Scan

↓

Boarding
```

Exactly the same.

---

Example

```
GET /users
```

Logging interceptor prints

```
GET /users
```

Auth interceptor adds

```
Authorization: Bearer xyz
```

Network interceptor compresses

```
gzip
```

Then request goes.

---

# Step 6: Authentication

Suppose API requires

```
Authorization: Bearer token
```

Without interceptor

Every API

```kotlin
@GET("users")
fun getUsers(
    @Header("Authorization") token:String
)
```

Terrible.

Instead

One interceptor

```kotlin
request.newBuilder()
    .header("Authorization", "Bearer token")
```

Now

ALL requests automatically receive the token.

---

# Step 7: Timeout

Sometimes server is slow.

Without timeout

Phone waits forever.

Timeout tells

```
Wait only 30 seconds.
```

Example

```kotlin
.connectTimeout(30, TimeUnit.SECONDS)
```

Different timeouts

### Connect timeout

How long to establish connection.

---

### Read timeout

Connected.

Waiting for server response.

---

### Write timeout

Uploading data.

Example

Uploading image.

---

### Call timeout

Entire request.

If total exceeds

```
30 sec
```

Stop.

---

# Step 8: Logging Interceptor

During development

You want to see

```
GET /users

Headers

Response

Status

Body
```

Just add

```kotlin
HttpLoggingInterceptor()
```

Never use BODY logging in production.

---

# Step 9: Retrofit Converter

Server returns

```json
{
  "id":1,
  "name":"John"
}
```

Retrofit converts

↓

```kotlin
User(
    id=1,
    name="John"
)
```

using

```
Gson

or

Moshi

or

Kotlinx Serialization
```

---

# Step 10: Error Handling

Server returns

```
200
```

Success.

```
404
```

Not found.

```
401
```

Unauthorized.

```
500
```

Server error.

Network failure

```
IOException
```

Timeout

```
SocketTimeoutException
```

Unknown

```
Exception
```

Typical handling in a repository:

```kotlin
try {
    val response = api.getUsers()

    if (response.isSuccessful) {
        // Success
    } else {
        // HTTP error (4xx/5xx)
    }

} catch (e: IOException) {
    // No internet, timeout, connection issue
} catch (e: Exception) {
    // Unexpected error
}
```

---

# Step 11: Repository

Repository decides

```
API

or

Database

or

Cache
```

UI never knows.

```
UI

↓

ViewModel

↓

Repository

↓

Retrofit
```

---

# Step 12: ViewModel

ViewModel calls repository

```kotlin
viewModelScope.launch {

    repository.getUsers()

}
```

No networking code should be in Activity or Compose UI.

---

# Putting Everything Together

A common setup looks like this:

```kotlin
val logging = HttpLoggingInterceptor().apply {
    level = HttpLoggingInterceptor.Level.BODY
}

val authInterceptor = AuthInterceptor(tokenManager)

val client = OkHttpClient.Builder()
    .addInterceptor(authInterceptor)
    .addInterceptor(logging)
    .connectTimeout(30, TimeUnit.SECONDS)
    .readTimeout(30, TimeUnit.SECONDS)
    .writeTimeout(30, TimeUnit.SECONDS)
    .build()

val retrofit = Retrofit.Builder()
    .baseUrl(BASE_URL)
    .client(client)
    .addConverterFactory(GsonConverterFactory.create())
    .build()

val api = retrofit.create(ApiService::class.java)
```

The request flow is:

```
UI
 ↓
ViewModel
 ↓
Repository
 ↓
Retrofit
 ↓
OkHttpClient
 ↓
AuthInterceptor
 ↓
LoggingInterceptor
 ↓
Timeout checks
 ↓
Internet
 ↓
Server
 ↓
Response
 ↓
LoggingInterceptor
 ↓
Retrofit Converter (JSON → Kotlin object)
 ↓
Repository
 ↓
ViewModel
 ↓
UI
```

## A roadmap to master this in one week

Instead of reading everything at once, build one feature each day:

1. **Day 1:** Learn HTTP basics (GET, POST, PUT, DELETE, headers, status codes, request/response).
2. **Day 2:** Build a simple Retrofit app with one GET API.
3. **Day 3:** Learn `OkHttpClient`, the Builder pattern, and method chaining by configuring timeouts.
4. **Day 4:** Create your own logging and authentication interceptors to see how requests are modified.
5. **Day 5:** Handle errors properly using `Response<T>`, exceptions, and Kotlin's `Result` or a sealed `NetworkResult`.
6. **Day 6:** Implement token refresh with an `Authenticator` and understand when to use an interceptor versus an authenticator.
7. **Day 7:** Build a small production-style networking module using dependency injection (for example, with Hilt), Retrofit, OkHttp, interceptors, and a repository.

If you work through these in order, you'll stop seeing Retrofit, OkHttp, interceptors, authentication, and error handling as separate topics—they'll become parts of a single, understandable request pipeline.

Since you're aiming to **master Android networking**, I can also teach this as a **visual, production-level course** with diagrams, real interview questions, and progressively built projects, from HTTP basics all the way to JWT refresh tokens, multipart uploads, pagination, WebSockets, and clean architecture integration.
