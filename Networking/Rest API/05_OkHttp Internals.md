Excellent. This is my favorite module because **OkHttp is the real networking engine**.

Once you understand OkHttp, you'll understand:

* Retrofit
* Interceptors
* Authentication
* Logging
* Timeouts
* Connection Pooling
* Caching
* Retry
* Dispatcher

Almost every Android networking interview eventually reaches OkHttp.

---

# Module 5 – OkHttp Internals (Deep Dive)

## Interview Goal

After this module you should confidently answer:

* What is OkHttp?
* Why does Retrofit need OkHttp?
* How does OkHttp execute requests?
* What is RealCall?
* What is Dispatcher?
* What is Connection Pool?
* What is HTTP/2 Multiplexing?
* How are requests executed?
* How are synchronous and asynchronous calls different?

---

# Before We Start

Remember this architecture.

```
Compose

↓

ViewModel

↓

Repository

↓

Retrofit

↓

OkHttp

↓

Internet

↓

Server
```

Many developers think Retrofit is networking.

**It isn't.**

Retrofit is just an abstraction layer.

OkHttp is the networking library.

---

# Part 1 — Why OkHttp Exists

Suppose Retrofit didn't exist.

How would Android communicate with the server?

It still needs something that can:

* Open sockets
* Resolve DNS
* Perform TLS handshake
* Send HTTP requests
* Receive HTTP responses
* Reuse connections
* Retry requests
* Cache responses

That "something" is OkHttp.

Interview definition:

> OkHttp is an HTTP client responsible for executing network requests and managing low-level networking concerns such as connections, TLS, caching, retries, and HTTP/2 support.

---

# Part 2 — Retrofit vs OkHttp

One of the most common interview questions.

```
Retrofit

↓

Reads annotations

↓

Creates HTTP request

↓

Calls OkHttp
```

OkHttp

```
Receives Request

↓

Opens Connection

↓

Sends Bytes

↓

Receives Bytes

↓

Returns Response
```

Think of ordering food.

Retrofit:

> Writes your order nicely.

OkHttp:

> Drives to the restaurant, picks up the food, and brings it back.

---

# Part 3 — OkHttpClient

This is the heart of OkHttp.

Everyone writes:

```kotlin
val client = OkHttpClient.Builder()
    .build()
```

Question:

What exactly is `OkHttpClient`?

Interview answer:

> OkHttpClient is a thread-safe, reusable object that manages the configuration and resources required to execute HTTP requests.

Notice:

**Reusable**

This is important.

---

## Why Reuse It?

Imagine creating a new client for every API.

```
API 1

New Client

↓

Connection Pool

↓

Thread Pool
```

API 2

```
New Client

↓

Another Pool
```

API 3

```
Another Pool
```

Very expensive.

Instead:

```
One OkHttpClient

↓

Many Requests
```

The same client shares:

* Connection Pool
* Thread Pool
* Cache
* DNS
* SSL Configuration

This is why interviewers often ask:

> Should OkHttpClient be singleton?

Answer:

**Yes.**

---

# Part 4 — Builder Pattern

Everyone sees

```kotlin
OkHttpClient.Builder()
```

Why Builder?

Imagine the constructor.

```kotlin
OkHttpClient(
 timeout,
 cache,
 retry,
 auth,
 dns,
 logging,
 proxy,
 cookie,
 ...
)
```

Huge.

Instead

Builder lets you configure step by step.

```kotlin
OkHttpClient.Builder()

.addInterceptor(...)

.readTimeout(...)

.cache(...)

.build()
```

Builder simply collects configuration before creating the client.

---

# Part 5 — RealCall

This is rarely discussed in tutorials but often appreciated in interviews.

Suppose you write

```kotlin
api.getUsers()
```

Retrofit eventually creates an OkHttp `Request`.

Then:

```kotlin
client.newCall(request)
```

returns a `Call`.

Internally this is actually a class named:

```
RealCall
```

Flow:

```
Retrofit

↓

Request

↓

OkHttpClient

↓

RealCall

↓

Execute
```

RealCall represents **one HTTP request**.

One request.

One response.

---

# Part 6 — Synchronous vs Asynchronous

OkHttp supports both.

---

## Synchronous

```
execute()
```

Current thread waits.

```
Thread

↓

Waiting...

↓

Response

↓

Continue
```

---

## Asynchronous

```
enqueue()
```

Current thread continues immediately.

Background thread performs networking.

Later:

Callback.

```
Main Thread

↓

enqueue()

↓

Continue UI

↓

Background Thread

↓

Network

↓

Callback
```

Retrofit coroutines internally build on asynchronous execution.

---

# Part 7 — Dispatcher

Interview favorite.

Question:

If 100 requests happen simultaneously...

Who manages them?

Answer:

Dispatcher.

```
Request

↓

Dispatcher

↓

Worker Thread

↓

Execute
```

Dispatcher controls:

* Thread usage
* Request scheduling
* Parallel execution

Without Dispatcher:

Imagine 1000 threads.

Phone crashes.

Dispatcher prevents that.

---

# Dispatcher Limits

OkHttp doesn't start unlimited requests simultaneously.

It limits:

* Total concurrent requests
* Requests per host

This prevents overwhelming the device or server.

---

# Part 8 — Connection Pool

One of the highest ROI interview topics.

Suppose your app calls

```
GET /users
```

One second later

```
GET /posts
```

Should OkHttp open another TCP connection?

No.

Opening TCP connections is expensive.

Instead

```
Connection Pool

↓

Reuse Existing Connection
```

Huge performance improvement.

---

## Without Pool

```
API

↓

Open TCP

↓

Close

↓

API

↓

Open Again

↓

Close
```

Very slow.

---

## With Pool

```
TCP Connection

↓

Pool

↓

Reuse

↓

Reuse

↓

Reuse
```

Much faster.

---

Interview answer:

> Connection Pool stores reusable HTTP connections so future requests can avoid the cost of creating new TCP connections.

---

# Part 9 — HTTP/2 Multiplexing

Very popular interview topic.

Old HTTP:

```
Connection 1

↓

Request 1

↓

Close
```

Second request:

```
Connection 2

↓

Request 2
```

Many connections.

---

HTTP/2

```
One Connection

↓

Request 1

↓

Request 2

↓

Request 3

↓

Request 4
```

Multiple requests share one connection.

This is called

**Multiplexing**.

OkHttp supports this automatically when the server does.

---

# Part 10 — DNS

Before sending a request

OkHttp asks

```
Where is api.example.com?
```

DNS replies

```
142.250.xxx.xxx
```

Then connection begins.

---

# Part 11 — TLS Handshake

If HTTPS

Before data transfer

Client and server perform

```
TLS Handshake

↓

Certificates

↓

Encryption Keys

↓

Secure Connection
```

Only after this

HTTP data starts flowing.

Interviewers may ask:

> Does Retrofit perform TLS?

No.

OkHttp handles it.

---

# Part 12 — Complete Execution Flow

Suppose

```kotlin
api.getUsers()
```

Internally

```
Retrofit

↓

Generated Proxy

↓

Create Request

↓

OkHttpClient

↓

RealCall

↓

Dispatcher

↓

Connection Pool

↓

DNS

↓

TCP Connection

↓

TLS Handshake

↓

Interceptors

↓

Server

↓

Response

↓

Converter

↓

User Object

↓

Repository

↓

ViewModel

↓

UI
```

This is the entire networking pipeline.

---

# Part 13 — Why Singleton?

Interview question.

Why don't we create

```kotlin
OkHttpClient()
```

inside every Repository?

Because every client has

* Connection Pool
* Thread Pool
* Dispatcher
* Cache

Creating many clients wastes memory and prevents connection reuse.

Best practice:

```
Application

↓

Singleton OkHttpClient

↓

Singleton Retrofit

↓

Repositories
```

---

# Frequently Asked Interview Questions

## Q1

What is OkHttp?

Answer:

> OkHttp is a networking library that executes HTTP requests and manages low-level networking features such as connections, TLS, HTTP/2, caching, retries, and connection pooling.

---

## Q2

Does Retrofit use sockets?

Answer:

No.

OkHttp handles socket communication.

---

## Q3

What is Connection Pool?

Answer:

A collection of reusable TCP connections that reduces latency by avoiding repeated connection creation.

---

## Q4

What is Dispatcher?

Answer:

Dispatcher schedules and manages asynchronous request execution using a shared thread pool while enforcing concurrency limits.

---

## Q5

Why Singleton OkHttpClient?

Answer:

To reuse expensive resources like connection pools, thread pools, caches, and DNS configuration.

---

## Q6

What is RealCall?

Answer:

RealCall is OkHttp's internal implementation of a Call. It represents a single HTTP request/response execution.

---

## Q7

What is HTTP/2 Multiplexing?

Answer:

Multiple HTTP requests share the same TCP connection simultaneously, reducing latency and improving throughput.

---

# Common Interview Mistakes

❌

"Retrofit sends requests."

Better:

> Retrofit constructs requests and delegates execution to OkHttp.

---

❌

"OkHttp opens a new connection for every API."

Better:

> OkHttp reuses connections through its Connection Pool whenever possible.

---

❌

"Every Repository should have its own OkHttpClient."

Better:

> A single shared OkHttpClient should be reused across the application.

---

# Interview-Level Summary

If I asked you in an interview:

> **"Explain how OkHttp executes a request."**

A strong answer would be:

> "When Retrofit creates an HTTP request, it delegates execution to OkHttp. OkHttp creates a `RealCall` for that request. The `Dispatcher` schedules the call and executes it on an appropriate thread. OkHttp checks its `ConnectionPool` for an existing reusable connection; if none is available, it performs DNS resolution, establishes a TCP connection, and, for HTTPS, completes a TLS handshake. The request then passes through the interceptor chain before being sent to the server. After receiving the response, the data flows back through the interceptors, Retrofit's converter deserializes the response body into Kotlin objects, and the result is returned to the repository and ultimately the UI."

---

## One Important Correction

Many tutorials simplify the flow as:

```
Retrofit
    ↓
OkHttp
    ↓
Interceptors
```

The more accurate internal flow is:

```
Retrofit
    ↓
OkHttpClient
    ↓
RealCall
    ↓
Dispatcher
    ↓
Interceptor Chain
    ↓
Connection Management
    ↓
Network
```

Interceptors are **one stage** in OkHttp's execution pipeline, not the first thing that happens.

---

## Next Module Preview: Interceptors

Module 6 is one of the most important modules for Android interviews because it covers the **Chain of Responsibility** design pattern in a real production context.

We'll answer questions like:

* What exactly is an Interceptor?
* What is `chain.proceed()`?
* Why do we call `proceed()` only once?
* What is the difference between **Application Interceptors** and **Network Interceptors**?
* How do Logging, Authentication, Retry, and Caching interceptors work together?
* What is the execution order of multiple interceptors?
* What is the difference between an **Interceptor** and an **Authenticator**?

Understanding Module 6 will also deepen your understanding of design patterns, since OkHttp's interceptor mechanism is a practical implementation of the **Chain of Responsibility** pattern.
