# Module 8 — Network Error Handling & Production-Grade API Result Handling (Deep Dive)

This module is extremely important for Android interviews.

Many developers know:

```kotlin
val response = api.getUsers()
```

but real production apps need to answer:

* What if there is no internet?
* What if the server returns 500?
* What if JSON parsing fails?
* What if the token expires?
* What if the request times out?
* How does UI know whether to show loading, error, or data?

This module connects **Retrofit + OkHttp + Coroutines + Clean Architecture**.

---

# Interview Goals

After this module, you should be able to explain:

* Different types of network failures
* HTTP errors vs network errors
* Retrofit exception handling
* `Response<T>` vs throwing exceptions
* Timeout handling
* Parsing errors
* Creating a `NetworkResult`
* Repository-level error handling
* UI state management
* Retry mechanisms

---

# Part 1 — Types of Failures in Networking

A network call can fail at different layers.

Complete flow:

```text
Android App

↓

OkHttp

↓

Internet

↓

Server

↓

Database
```

Failure can happen anywhere.

---

## Category 1: Network Failure

The request never reaches the server.

Examples:

* No internet
* Airplane mode
* DNS failure
* Connection refused
* Socket error

Example:

```text
Android

X

Server
```

Common exception:

```kotlin
IOException
```

---

## Category 2: HTTP Failure

The request reaches the server.

Server responds with an error.

Example:

```http
HTTP/1.1 404 Not Found
```

or:

```http
HTTP/1.1 500 Internal Server Error
```

The network worked.

The server rejected the request.

---

## Category 3: Serialization / Parsing Failure

Server returns:

```json
{
 "username":"John"
}
```

App expects:

```kotlin
data class User(
    val id:Int,
    val name:String
)
```

Conversion fails.

Example:

```text
Expected Int
Received String
```

---

## Category 4: Business Logic Error

HTTP is successful:

```http
200 OK
```

But application says:

```json
{
 "success":false,
 "message":"Account blocked"
}
```

Technically successful.

Logically failed.

---

# Part 2 — HTTP Status Codes

Interviewers expect you to know these.

---

# 2xx Success

## 200 OK

Request successful.

Example:

```http
GET /users
```

---

## 201 Created

Resource created.

Example:

```http
POST /users
```

---

## 204 No Content

Successful but no response body.

Example:

```http
DELETE /user/10
```

---

# 4xx Client Errors

Client made a bad request.

---

## 400 Bad Request

Invalid input.

Example:

```json
{
 "email":"wrong-format"
}
```

---

## 401 Unauthorized

Authentication failed.

Usually:

* Missing token
* Expired token
* Invalid token

Example:

```http
Authorization: Bearer expired_token
```

---

## 403 Forbidden

User is authenticated but doesn't have permission.

Example:

Normal user:

```text
DELETE ALL USERS
```

Server:

```http
403 Forbidden
```

---

## 404 Not Found

Resource does not exist.

Example:

```http
/users/999999
```

---

# 5xx Server Errors

Server problem.

---

## 500 Internal Server Error

Backend crashed.

---

## 502 Bad Gateway

Gateway received invalid response.

---

## 503 Service Unavailable

Server temporarily unavailable.

---

# Part 3 — Retrofit Error Handling Approaches

There are two common styles.

---

# Approach 1 — Using Response<T>

Example:

```kotlin
@GET("users")
suspend fun getUsers():
    Response<List<User>>
```

Now Retrofit does not throw HTTP errors.

You manually check:

```kotlin
val response = api.getUsers()

if(response.isSuccessful){

}
else{

}
```

---

Example:

```kotlin
if(response.code()==401){

}
```

---

Advantages:

* Full control
* Easy access to status codes
* Good for complex APIs

---

Disadvantages:

More boilerplate.

---

# Approach 2 — Direct Return Type

Example:

```kotlin
@GET("users")
suspend fun getUsers():
    List<User>
```

Now:

Success:

```kotlin
List<User>
```

Failure:

Exception.

Example:

```kotlin
try {

 val users = api.getUsers()

}
catch(e:Exception){

}
```

---

Advantages:

Cleaner code.

---

Disadvantages:

Need good exception mapping.

---

# Interview Question

Which approach do you prefer?

Good answer:

> It depends on the architecture. For simple APIs, throwing exceptions with centralized handling is cleaner. For APIs requiring detailed status-code handling, using Response<T> provides more control. Many production apps wrap both approaches inside a repository-level Result or sealed class.

---

# Part 4 — Important Exceptions

Now let's understand common exceptions.

---

# IOException

Most common network failure.

Examples:

* No internet
* Connection timeout
* DNS failure

Example:

```kotlin
catch(e: IOException){

}
```

Meaning:

The request could not complete at network level.

---

# HttpException

Occurs when using Retrofit coroutine support and a non-2xx response happens.

Example:

Server:

```http
404
```

Exception:

```kotlin
HttpException
```

You can get:

```kotlin
e.code()
```

---

# JsonDataException / SerializationException

Parsing problem.

Example:

Expected:

```json
{
"id":1
}
```

Received:

```json
{
"id":"abc"
}
```

---

# TimeoutException

Request took too long.

Caused by:

* Slow server
* Poor network
* Wrong timeout configuration

---

# Part 5 — Timeouts

OkHttp provides three important timeouts.

---

# Connect Timeout

How long to establish connection.

Example:

```text
Phone

↓

Server
```

If connection cannot be created:

Timeout.

---

# Read Timeout

How long to wait for server response data.

Example:

Server accepts request but takes too long.

---

# Write Timeout

How long to send request body.

Example:

Uploading image.

---

Configuration:

```kotlin
val client =
    OkHttpClient.Builder()
        .connectTimeout(30, TimeUnit.SECONDS)
        .readTimeout(30, TimeUnit.SECONDS)
        .writeTimeout(30, TimeUnit.SECONDS)
        .build()
```

---

# Interview Question

Difference between connect timeout and read timeout?

Answer:

Connect timeout is the maximum time allowed to establish a connection.

Read timeout is the maximum time waiting for response data after connection is established.

---

# Part 6 — Why We Need NetworkResult

Imagine ViewModel:

```kotlin
try{

}
catch{

}
```

Every ViewModel handles errors.

Bad design.

Instead:

Repository converts everything into a common result.

Example:

```kotlin
sealed class NetworkResult<T>{

    data class Success<T>(
        val data:T
    ):NetworkResult<T>()

    data class Error<T>(
        val message:String
    ):NetworkResult<T>()

    class Loading<T>:
        NetworkResult<T>()

}
```

---

Now:

Repository:

```text
API Error

↓

NetworkResult.Error
```

ViewModel:

```text
Success
↓

Show Data


Error
↓

Show Message
```

---

# Part 7 — Production API Flow

Real architecture:

```text
UI

↓

ViewModel

↓

Repository

↓

SafeApiCall

↓

Retrofit

↓

OkHttp

↓

Server
```

---

The repository owns:

* Exception conversion
* Error mapping
* Retry decisions

Not the UI.

---

# Part 8 — Safe API Call Pattern

Common production pattern:

```kotlin
suspend fun <T> safeApiCall(
    apiCall: suspend () -> T
): NetworkResult<T>{

    return try {

        NetworkResult.Success(
            apiCall()
        )

    } catch(e: IOException){

        NetworkResult.Error(
            "No internet connection"
        )

    } catch(e: HttpException){

        NetworkResult.Error(
            "Server error ${e.code()}"
        )

    }

}
```

---

Usage:

```kotlin
repository.getUsers()
```

returns:

```kotlin
Success(users)

or

Error(message)
```

---

# Part 9 — Handling Loading State

UI usually has three states:

```text
Loading

Success

Error
```

Example:

```kotlin
sealed class UiState{

    object Loading:UiState()

    data class Success(
        val users:List<User>
    ):UiState()

    data class Error(
        val message:String
    ):UiState()
}
```

---

Flow:

User opens screen:

```text
Loading
```

API succeeds:

```text
Success(data)
```

API fails:

```text
Error(message)
```

---

# Part 10 — Retry Strategy

Should every request retry?

No.

Important interview topic.

---

Safe retry candidates:

GET:

```http
GET /users
```

Usually safe.

---

Dangerous:

POST:

```http
POST /payment
```

Retrying may create:

```text
Double payment
```

---

Retry should consider:

* HTTP method
* Error type
* Network state
* Number of attempts

---

# Part 11 — Exponential Backoff

Common retry strategy.

Instead of:

```text
Retry immediately
```

Do:

```
Attempt 1
wait 1 sec

Attempt 2
wait 2 sec

Attempt 3
wait 4 sec
```

Formula:

```
delay = 2^attempt
```

Benefits:

* Reduces server load
* Handles temporary failures

---

# Part 12 — Network Availability Check

A common mistake:

```kotlin
if(internetAvailable)
    callApi()
```

Why?

Because network can disappear after the check.

Example:

```
Check internet

YES

↓

Request

↓

Network gone
```

The API call still needs exception handling.

---

Better:

Use connectivity check only for UX:

Example:

"No internet connection"

But always handle actual request failure.

---

# Part 13 — Error Mapping

Bad:

```text
Exception message directly to user
```

Example:

```
java.net.SocketTimeoutException
```

User doesn't understand.

---

Better:

Convert:

```
SocketTimeoutException

↓

"Server is taking too long. Try again."
```

---

Example mapping:

| Exception        | User Message              |
| ---------------- | ------------------------- |
| IOException      | Check internet connection |
| TimeoutException | Server timeout            |
| 401              | Session expired           |
| 403              | Access denied             |
| 500              | Server unavailable        |

---

# Part 14 — Interview Architecture Question

Interviewer:

> Where should API error handling happen?

Strong answer:

> The repository layer should handle network exceptions and convert them into a common result type. ViewModels should only consume the result and decide UI state. This keeps networking concerns separate from presentation logic.

---

# Common Interview Mistakes

❌

"404 means no internet."

Wrong.

404 means server responded but resource doesn't exist.

---

❌

"401 and 403 are the same."

Wrong.

401 = authentication failure.

403 = authorization failure.

---

❌

"Retry every failed request."

Wrong.

POST requests may create duplicate operations.

---

❌

"Checking internet before every API call solves network errors."

Wrong.

Network can change after the check.

---

# Senior-Level Interview Question

## Design a robust API handling system.

Answer:

```
Retrofit
    |
OkHttp
    |
Interceptors
    |
Repository
    |
SafeApiCall
    |
NetworkResult
    |
ViewModel
    |
UI State
```

Responsibilities:

### OkHttp

* Connection
* Timeout
* Authentication
* Logging

### Retrofit

* API definition
* Serialization

### Repository

* Error conversion
* Business decisions

### ViewModel

* UI state

---

# Final Interview Answer

If asked:

> "How do you handle API errors in Android?"

A strong answer:

> "I handle errors at the repository layer rather than directly in the UI. Retrofit and OkHttp can throw different types of failures such as IOException for network problems, HttpException for HTTP errors, and serialization exceptions for parsing failures. I convert these into a common Result or sealed class like Loading, Success, and Error. The ViewModel observes this state and updates the UI accordingly. For retries, I consider the HTTP method and error type because retrying unsafe operations like payments can create duplicate actions."

---

# Module 8 Summary

You now understand:

✅ HTTP errors
✅ Network errors
✅ Retrofit exceptions
✅ Timeouts
✅ Response handling
✅ Safe API calls
✅ NetworkResult pattern
✅ UI state management
✅ Retry strategies
✅ Error mapping

---

## Next Module: Module 9 — Caching & Offline-First Networking

This is where we connect:

* OkHttp Cache
* Room Database
* Repository pattern
* Offline-first architecture
* Single Source of Truth
* Network Bound Resource
* Sync strategies

This topic is heavily asked in senior Android interviews because modern apps are expected to work even with poor connectivity.
