Excellent. **Module 6 is probably the single most important networking topic for Android interviews.**

I've interviewed Android developers, and one pattern stands out:

* Almost everyone has **used** an interceptor.
* Very few can **explain** how it works internally.

If you can explain interceptors well, interviewers immediately see that you understand the networking stack.

---

# Module 6 – OkHttp Interceptors (Deep Dive)

## Interview Goal

By the end of this module, you should be able to answer:

* What is an Interceptor?
* Why do we need Interceptors?
* How do they work internally?
* What is `Interceptor.Chain`?
* What is `chain.proceed()`?
* Why should `proceed()` usually be called exactly once?
* Application vs Network Interceptors
* Order of execution
* Logging Interceptor
* Authentication Interceptor
* Retry Interceptor
* Caching Interceptor
* Interceptor vs Authenticator

---

# Part 1 — Why Do Interceptors Exist?

Imagine OkHttp without interceptors.

Request flow:

```text
App
  ↓
OkHttp
  ↓
Server
```

Suppose every request needs:

* Bearer Token
* Logging
* Retry
* Custom Headers
* Analytics
* Timing

Without interceptors, you'd repeat code everywhere.

```kotlin
api.getUsers(token)

api.getProducts(token)

api.getMovies(token)

api.getOrders(token)
```

Every request repeats the same logic.

That's a violation of the **DRY (Don't Repeat Yourself)** principle.

---

## Solution

Instead of changing every API call,

change the request **once**.

```text
App

↓

Interceptor

↓

Server
```

Now every request automatically gets modified.

---

# Interview Definition

> An Interceptor is a component that can observe, modify, or even short-circuit an HTTP request or response as it passes through the OkHttp request pipeline.

Notice three important words:

* Observe
* Modify
* Short-circuit

We'll discuss each.

---

# Part 2 — Think of Airport Security

Imagine you're flying.

```text
Passenger

↓

Passport Check

↓

Security Check

↓

Immigration

↓

Boarding

↓

Flight
```

Each checkpoint:

* Reads information
* Modifies state
* Can stop you

HTTP requests are similar.

```text
Request

↓

Authentication

↓

Logging

↓

Compression

↓

Retry

↓

Internet
```

Each stage gets a chance to inspect the request.

---

# Part 3 — The Interceptor Interface

Every interceptor implements:

```kotlin
interface Interceptor {

    fun intercept(chain: Chain): Response

}
```

Interview question:

Why does it return `Response`?

Because every interceptor is responsible for forwarding the request and eventually returning the response back up the chain.

---

# Part 4 — What is Chain?

This is where many candidates get confused.

Suppose you have:

```text
Logging

↓

Authentication

↓

Retry

↓

Server
```

How does Logging know who comes next?

Through `Chain`.

Think of Chain as:

> "The remaining pipeline."

It knows:

* Current Request
* Next Interceptor
* Eventually the network

---

# Part 5 — What is `chain.request()`?

Inside an interceptor:

```kotlin
override fun intercept(chain: Interceptor.Chain): Response {

    val request = chain.request()

}
```

This returns the **current request**.

At this point, you can:

* Read URL
* Read Headers
* Read Method
* Read Body

Example:

```kotlin
val method = request.method
```

or

```kotlin
val url = request.url
```

Nothing has been sent yet.

---

# Part 6 — Modifying Requests

Suppose every request needs:

```http
Authorization: Bearer token
```

Instead of modifying every API,

modify the request.

```kotlin
val newRequest = request.newBuilder()
    .addHeader("Authorization", "Bearer token")
    .build()
```

Notice:

Request objects are **immutable**.

You don't change the original.

You build a new one.

Interview point:

OkHttp `Request` is immutable.

`Request.Builder` creates a modified copy.

---

# Part 7 — `chain.proceed()`

This is **the heart of interceptors**.

Suppose:

```text
Logging

↓

Authentication

↓

Server
```

Logging executes.

Now what?

It calls:

```kotlin
chain.proceed(request)
```

Meaning:

> "Continue the chain."

Flow:

```text
Logging

↓

chain.proceed()

↓

Authentication

↓

chain.proceed()

↓

Server
```

---

## Interview Definition

`chain.proceed()` forwards the request to the next interceptor in the chain or, if there are no more interceptors, executes the network request.

---

# Why Usually Call `proceed()` Only Once?

This is a common interview question.

If you call:

```kotlin
chain.proceed(request)
```

twice,

you're asking OkHttp to execute the request twice.

```text
Request 1

↓

Server

Request 2

↓

Server
```

For a `POST`, that might create two orders or process two payments.

So in a normal interceptor:

```kotlin
val response = chain.proceed(request)

return response
```

one call is expected.

> **Exception:** Retry interceptors intentionally call `proceed()` again after deciding a retry is appropriate. They do this carefully and under controlled conditions.

---

# Part 8 — Response Flow

Many developers think interceptors only see requests.

Wrong.

They also see responses.

Flow:

```text
Request

↓

Logging

↓

Authentication

↓

Server

↑

Authentication

↑

Logging

↑

App
```

Notice:

Responses travel **backwards**.

This is why Logging can print both:

```text
Request

Response
```

---

# Part 9 — Logging Interceptor

One of the simplest examples.

Before:

```text
GET /users
```

Logging prints:

```text
--> GET /users
```

Calls:

```kotlin
chain.proceed()
```

Receives response.

Prints:

```text
<-- 200 OK
```

Returns response.

Logging doesn't change anything.

It only observes.

---

# Part 10 — Authentication Interceptor

Suppose API requires

```http
Authorization: Bearer xyz
```

Without interceptor:

Every API needs

```kotlin
@Header("Authorization")
```

Very repetitive.

Authentication interceptor:

```text
Request

↓

Add Token

↓

Continue
```

Every request now includes the token automatically.

---

# Part 11 — Retry Interceptor

Suppose internet drops.

```text
Request

↓

IOException
```

Retry interceptor:

```text
Request

↓

Failure

↓

Wait

↓

Retry

↓

Success
```

This is one of the few cases where calling `proceed()` more than once is intentional.

---

# Part 12 — Short-Circuiting

Earlier I said:

Interceptors can **short-circuit**.

What does that mean?

Instead of:

```text
Request

↓

Internet

↓

Server
```

an interceptor can immediately return a response.

Example:

Cache hit.

```text
Request

↓

Cache

↓

Response
```

No network call.

That's a short-circuit.

---

# Part 13 — Multiple Interceptors

Suppose

```kotlin
.addInterceptor(logging)
.addInterceptor(auth)
.addInterceptor(retry)
```

Request flow:

```text
Logging

↓

Auth

↓

Retry

↓

Server
```

Response flow:

```text
Server

↓

Retry

↓

Auth

↓

Logging
```

Remember this.

Requests go **down**.

Responses come **back up**.

---

# Part 14 — Application vs Network Interceptors

This is asked very often.

## Application Interceptor

Runs once for the logical request.

It:

* Can short-circuit
* Sees cached responses
* Runs before the network

Configured with:

```kotlin
.addInterceptor()
```

---

## Network Interceptor

Runs only when a real network request occurs.

Configured with:

```kotlin
.addNetworkInterceptor()
```

Useful for:

* Observing network headers
* Compression
* Redirects
* Network-level behavior

If the response comes entirely from cache, a network interceptor won't run because no network request was made.

---

### Interview Comparison

| Application                   | Network                                   |
| ----------------------------- | ----------------------------------------- |
| `.addInterceptor()`           | `.addNetworkInterceptor()`                |
| Can short-circuit             | Runs only for network calls               |
| Sees cached responses         | Doesn't run for cache-only responses      |
| Best for auth, logging, retry | Best for observing actual network traffic |

---

# Part 15 — Execution Flow

Suppose:

```text
Logging

↓

Authentication

↓

Retry

↓

Network

↓

Server
```

Execution:

```text
Logging Request

↓

Auth Request

↓

Retry Request

↓

Server

↓

Retry Response

↓

Auth Response

↓

Logging Response
```

Think of it like nested function calls.

---

# Part 16 — Interceptor vs Authenticator

Interview favorite.

Many candidates confuse these.

## Interceptor

Runs **before** request.

Purpose:

Modify outgoing request.

Example:

```http
Authorization: Bearer token
```

---

## Authenticator

Runs **after** server responds:

```http
401 Unauthorized
```

Purpose:

Refresh expired token.

Retry original request.

Flow:

```text
Request

↓

401

↓

Authenticator

↓

Refresh Token

↓

Retry Original Request
```

We'll study this deeply in the Authentication module.

---

# Common Production Interceptors

Logging

```text
Print requests
```

Authentication

```text
Add Bearer token
```

Retry

```text
Retry failed requests
```

Analytics

```text
Measure API timing
```

Headers

```text
Add common headers
```

Caching

```text
Return cached data
```

---

# Real Interview Questions

### Q1

What is an Interceptor?

**Answer:**

An Interceptor is a component that can inspect, modify, or short-circuit HTTP requests and responses as they pass through the OkHttp pipeline.

---

### Q2

What does `chain.proceed()` do?

**Answer:**

It forwards the request to the next interceptor or, if none remain, executes the network request and returns the resulting response.

---

### Q3

Why usually call `proceed()` only once?

**Answer:**

Because each call can trigger another execution of the request. Calling it multiple times unintentionally may send duplicate requests. Controlled retries are an intentional exception.

---

### Q4

Why is `Request` immutable?

**Answer:**

Immutability makes request objects thread-safe and predictable. To change a request, you create a new one using `Request.Builder`.

---

### Q5

Difference between Application and Network Interceptors?

You should mention:

* Cache behavior
* Execution timing
* Network visibility
* Typical use cases

---

### Q6

Can an Interceptor stop a request?

Yes.

It can return a response directly without contacting the server, such as serving data from a local cache.

---

# Common Interview Mistakes

❌

"Interceptor is only for logging."

Better:

Logging is **one use case**. Interceptors are a general mechanism for inspecting and modifying requests and responses.

---

❌

"`chain.proceed()` sends the request."

Better:

`chain.proceed()` passes control to the next interceptor, or ultimately to the network if no interceptors remain.

---

❌

"Authentication and Authenticator are the same."

Better:

An authentication interceptor typically **adds** credentials before a request is sent. An `Authenticator` typically **reacts** to authentication failures (like `401`) by obtaining fresh credentials and retrying the request.

---

# Interview-Level Summary

If an interviewer asks:

> **"Explain OkHttp Interceptors."**

A strong answer would be:

> "OkHttp interceptors implement the Chain of Responsibility pattern. Each interceptor receives the current request through `Interceptor.Chain`, can inspect or modify it, and typically calls `chain.proceed()` to pass it to the next interceptor. After the network call completes, the response travels back through the same chain in reverse order, allowing interceptors to inspect or modify responses as well. Common use cases include adding authentication headers, logging requests and responses, implementing retries, measuring request duration, and serving cached responses. Application interceptors operate at the logical request level, while network interceptors observe actual network traffic."

---

# What Makes This Module Special

This module isn't just about networking.

It teaches a classic software design pattern:

## Chain of Responsibility

```text
Request

↓

Handler 1

↓

Handler 2

↓

Handler 3

↓

Server
```

Each handler decides:

* Handle it
* Modify it
* Pass it on
* Stop it

Once you recognize this pattern, you'll start seeing it in many frameworks beyond OkHttp.

---

## Next Module (Authentication & JWT)

This is one of the most commonly asked Android interview topics.

We'll cover:

* Authentication vs Authorization
* Sessions vs JWT
* Access Token
* Refresh Token
* Token expiration
* How an `Authenticator` refreshes a token
* Why token refresh should not be implemented in a normal interceptor
* Avoiding infinite refresh loops
* Production-ready token refresh flow

By the end of that module, you'll be able to confidently explain the complete authentication lifecycle used in modern Android applications.
