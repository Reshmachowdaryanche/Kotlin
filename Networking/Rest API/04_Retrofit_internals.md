Excellent. This is where Android interviews start separating **developers who use Retrofit** from **developers who understand Retrofit**.

Many candidates know this:

```kotlin
interface ApiService {

    @GET("users")
    suspend fun getUsers(): List<User>

}
```

But when the interviewer asks:

> **"Where is the implementation of `ApiService`?"**

Most people get stuck.

---

# Module 4 – Retrofit Internals (Deep Dive)

## Interview Goal

After this module, you should be able to answer:

* What is Retrofit?
* Why Retrofit exists?
* How Retrofit works internally?
* How does Retrofit implement an interface?
* What is Dynamic Proxy?
* What is Reflection?
* Why annotations?
* How Retrofit creates HTTP requests?
* Retrofit request flow
* Common interview questions

---

# Part 1 — Why Retrofit Exists

Let's go back in time.

Before Retrofit, Android developers had to manually create HTTP requests.

Imagine calling:

```
GET https://api.example.com/users
```

Without Retrofit, you would have to:

1. Create URL
2. Open connection
3. Set method
4. Add headers
5. Read InputStream
6. Convert bytes to String
7. Parse JSON
8. Handle errors
9. Close connection

Hundreds of lines of code.

Retrofit removes this boilerplate.

Instead, you write

```kotlin
@GET("users")
suspend fun getUsers(): List<User>
```

That's all.

---

# Part 2 — What is Retrofit?

Interview Definition:

> Retrofit is a type-safe HTTP client for Android and Java that converts Kotlin/Java interface methods into HTTP requests.

Notice the wording.

Retrofit is **not** the networking engine.

Retrofit does **not** directly talk to the internet.

---

## Biggest Misconception

Many developers think

```
Retrofit
      │
Internet
```

Wrong.

Actual architecture

```
Your App

↓

Retrofit

↓

OkHttp

↓

TCP/IP

↓

Internet

↓

Server
```

Retrofit sits **above** OkHttp.

OkHttp performs networking.

Retrofit performs abstraction.

---

# Part 3 — Why Interface?

Consider this.

```kotlin
interface ApiService {

    @GET("users")
    suspend fun getUsers(): List<User>

}
```

Question:

Where is the implementation?

There isn't one.

So why doesn't Kotlin complain?

Because Retrofit creates it **at runtime**.

This surprises many interview candidates.

---

# Part 4 — retrofit.create()

Everyone writes

```kotlin
val api = retrofit.create(ApiService::class.java)
```

Question:

What does this actually do?

Most people answer

> It creates an object.

Not enough.

Actual answer:

Retrofit uses Java's **Dynamic Proxy** mechanism to generate an implementation of your interface at runtime.

No class is generated in your source code.

No class is written by you.

It exists only while the application is running.

---

# Part 5 — Dynamic Proxy

This is one of the most frequently misunderstood interview topics.

Let's simplify.

Suppose you have

```kotlin
interface Animal {

    fun speak()

}
```

Normally you'd write

```kotlin
class Dog : Animal {

    override fun speak() {
        println("Bark")
    }

}
```

Retrofit doesn't do this.

Instead it creates something like

```
Anonymous Generated Object

↓

Implements Animal

↓

Created During Runtime
```

Think of it as

```
interface

↓

Retrofit reads it

↓

Builds hidden implementation

↓

Returns object
```

You never see this implementation.

---

# Internal Idea

When you call

```kotlin
api.getUsers()
```

You're actually calling

```
Generated Proxy

↓

Invocation Handler

↓

Retrofit

↓

OkHttp
```

The proxy intercepts the method call.

---

# Part 6 — Reflection

Question:

How does Retrofit know

```
@GET("users")
```

exists?

Answer:

Reflection.

Reflection allows a program to inspect classes, methods, annotations, and fields while the application is running.

Normally

```kotlin
class User
```

is compiled.

Reflection lets us ask

```
What methods does this class have?

What annotations exist?

What are their values?
```

Retrofit reads

```kotlin
@GET("users")
```

using reflection.

---

# Part 7 — Why Annotations?

Instead of

```kotlin
getUsers("GET","users")
```

Retrofit uses

```kotlin
@GET("users")
```

Annotations are metadata.

They're instructions for Retrofit.

Retrofit scans them.

Example

```
Method

↓

Read Annotation

↓

Discover HTTP Method

↓

Discover Path

↓

Create Request
```

---

# Example

```kotlin
@POST("login")
suspend fun login(
    @Body request: LoginRequest
)
```

Retrofit extracts

```
Method

POST

↓

Path

login

↓

Body

LoginRequest

↓

Headers

↓

Request
```

---

# Part 8 — Building the Request

Suppose

```kotlin
@GET("users/{id}")
suspend fun getUser(
    @Path("id") id:Int
)
```

Call

```kotlin
api.getUser(10)
```

Retrofit builds

```
GET /users/10
```

If

```kotlin
@Query("page") page:Int
```

becomes

```
?page=2
```

Result

```
GET /users?page=2
```

---

# Part 9 — Converter Factory

Server returns

```json
{
 "id":1,
 "name":"John"
}
```

Android wants

```kotlin
User(
 id=1,
 name="John"
)
```

Retrofit itself cannot parse JSON.

It delegates to a converter.

Examples

```
Gson

Moshi

Kotlin Serialization
```

Flow

```
JSON

↓

Converter

↓

Kotlin Object
```

---

# Part 10 — Call Adapter

Interview topic.

Why can Retrofit return

```kotlin
Call<User>
```

or

```kotlin
suspend fun
```

or

```kotlin
Flow<User>
```

Because of **Call Adapters**.

Retrofit internally converts network calls into different return types.

Think of it as an adapter layer between the networking engine and the programming model you want to use.

---

# Part 11 — Complete Internal Flow

Suppose UI calls

```kotlin
api.getUsers()
```

Here's the real sequence.

```
ViewModel

↓

Repository

↓

Generated Retrofit Proxy

↓

Reflection

↓

Read @GET

↓

Read @Headers

↓

Read @Path

↓

Read @Query

↓

Build HTTP Request

↓

OkHttp

↓

Interceptors

↓

Internet

↓

Server

↓

HTTP Response

↓

OkHttp

↓

Converter Factory

↓

User Objects

↓

Repository

↓

ViewModel

↓

UI
```

This is the complete lifecycle.

---

# Part 12 — Why OkHttp?

Interview question.

If Retrofit creates requests,

why do we need OkHttp?

Because Retrofit is only responsible for

* Reading annotations
* Creating requests
* Parsing responses

OkHttp handles

* TCP connection
* HTTPS
* SSL
* HTTP/2
* Connection pooling
* Timeouts
* Retries
* Cookies
* Caching
* Interceptors

Think of Retrofit as a translator.

Think of OkHttp as the delivery truck.

---

# Part 13 — What Happens When You Call api.getUsers()?

Interview answer.

```
1. ViewModel calls Repository.

2. Repository calls api.getUsers().

3. Generated Proxy intercepts the call.

4. Reflection reads annotations.

5. Retrofit creates an HTTP request.

6. Retrofit passes the request to OkHttp.

7. OkHttp executes interceptors.

8. OkHttp sends the request.

9. Server processes the request.

10. Server sends HTTP response.

11. OkHttp receives response.

12. Retrofit converter converts JSON to Kotlin objects.

13. Repository returns data.

14. ViewModel updates state.

15. UI refreshes.
```

This is a strong interview answer because it shows the complete flow.

---

# Frequently Asked Interview Questions

### Q1. Is Retrofit a networking library?

A better answer is:

> Retrofit is a type-safe HTTP client that provides a higher-level API for defining REST endpoints. It relies on OkHttp to perform the actual network communication.

---

### Q2. Does Retrofit make socket connections?

No.

OkHttp handles socket connections.

---

### Q3. Why do we define APIs as interfaces?

Because Retrofit creates a runtime implementation using Java Dynamic Proxies. This lets you declare the API contract without writing the implementation yourself.

---

### Q4. What is Reflection?

Reflection is the ability of a program to inspect classes, methods, fields, and annotations at runtime. Retrofit uses it to read annotations like `@GET`, `@POST`, `@Path`, and `@Query`.

---

### Q5. What is Dynamic Proxy?

A Dynamic Proxy is a runtime-generated object that implements one or more interfaces. It intercepts method calls and can provide behavior without requiring a concrete implementation class.

---

### Q6. Why use annotations instead of strings?

Annotations make the API declaration declarative, easier to read, and less error-prone. Retrofit can inspect them using reflection to build HTTP requests.

---

# Common Interview Mistakes

❌ **"Retrofit sends the request to the internet."**

Better:

> Retrofit constructs the request and delegates execution to OkHttp.

---

❌ **"Retrofit implements the interface at compile time."**

Better:

> Retrofit creates a runtime implementation using Java's Dynamic Proxy mechanism.

---

❌ **"Retrofit parses JSON."**

Better:

> Retrofit delegates JSON serialization/deserialization to a configured Converter Factory such as Gson, Moshi, or Kotlin Serialization.

---

# Interview-Level Summary

If an interviewer asks:

> **"Explain Retrofit internals."**

A strong answer is:

> "Retrofit is a type-safe HTTP client that allows us to define REST APIs as interfaces. When `retrofit.create(ApiService::class.java)` is called, Retrofit uses Java Dynamic Proxies to generate a runtime implementation of that interface. When a method like `getUsers()` is invoked, the proxy intercepts the call, uses reflection to inspect annotations such as `@GET`, `@Path`, and `@Query`, constructs an HTTP request, and delegates execution to OkHttp. Once the response is received, Retrofit uses the configured Converter Factory to deserialize the response body into Kotlin objects and returns the result to the caller."

---

## Interview Insight

For **2–4 years** of Android experience, knowing that Retrofit uses **Dynamic Proxies**, **Reflection**, **OkHttp**, and **Converter Factories** is often enough.

For **5+ years** or senior roles, interviewers may go deeper into **how Dynamic Proxies work**, **how suspend functions are adapted**, **how call adapters integrate with coroutines**, and **how OkHttp executes the request internally**.

Those topics naturally lead into **Module 5: OkHttp Internals**, where we'll follow the request from Retrofit into OkHttp and explore interceptors, connection pooling, HTTP/2, retries, timeouts, and the networking pipeline in detail.
