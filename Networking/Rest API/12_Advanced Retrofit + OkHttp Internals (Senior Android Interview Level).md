# Module 12 — Advanced Retrofit + OkHttp Internals (Senior Android Interview Level)

This is the final deep networking module.

Most Android developers can explain:

```
Retrofit
↓
API Interface
↓
Response
```

Senior Android developers should understand what happens underneath:

```
Your Code

↓

Retrofit Proxy

↓

Converter

↓

OkHttp Call

↓

Dispatcher

↓

Connection Pool

↓

DNS

↓

TCP

↓

TLS

↓

HTTP/2

↓

Server
```

This module focuses on **internals, performance, and security**.

---

# Interview Goals

After this module, you should be able to answer:

* How Retrofit works internally?
* How does Retrofit create API implementations?
* What happens when calling a suspend API?
* How OkHttp manages connections?
* What is Dispatcher?
* What is Connection Pool?
* What is HTTP/2?
* How DNS resolution works?
* What happens during HTTPS handshake?
* What is Certificate Pinning?
* How to optimize networking performance?

---

# Part 1 — Retrofit Internal Architecture

Developers write:

```kotlin
interface UserApi {

    @GET("users")
    suspend fun getUsers():
        List<User>

}
```

Question:

Where does this implementation come from?

You never write:

```kotlin
class UserApiImpl
```

So who creates it?

---

## Retrofit Uses Dynamic Proxy

When you call:

```kotlin
val api =
    retrofit.create(UserApi::class.java)
```

Retrofit creates a proxy object.

Conceptually:

```
UserApi Interface

        |

        ↓

Dynamic Proxy

        |

        ↓

Retrofit Handler

        |

        ↓

OkHttp Request
```

---

The proxy:

1. Reads annotations

Example:

```kotlin
@GET("users")
```

2. Creates HTTP request

3. Executes through OkHttp

---

# Part 2 — Retrofit Call Flow

When you call:

```kotlin
api.getUsers()
```

Internally:

```
api.getUsers()

↓

Retrofit Proxy

↓

Parse Method Annotation

↓

Create Request

↓

Create OkHttp Call

↓

Execute Network

↓

Receive Response

↓

Converter

↓

Return Object
```

---

# Part 3 — Converter Factory

Server returns:

```json
{
"id":1,
"name":"John"
}
```

But your app needs:

```kotlin
data class User(
    val id:Int,
    val name:String
)
```

Who converts JSON → Object?

Answer:

Converter.

Examples:

* Gson Converter
* Moshi Converter
* Kotlin Serialization Converter

---

Flow:

```
JSON

↓

Converter

↓

Kotlin Object
```

---

Example:

```kotlin
Retrofit.Builder()
.addConverterFactory(
    MoshiConverterFactory.create()
)
```

---

# Interview Question

## Does Retrofit parse JSON?

Correct answer:

> Retrofit itself does not parse JSON. Retrofit delegates serialization and deserialization to converter factories such as Moshi, Gson, or Kotlin Serialization.

---

# Part 4 — OkHttp Internal Architecture

OkHttp responsibilities:

* HTTP requests
* Connection management
* TLS
* Interceptors
* Caching
* Retries

Architecture:

```
Retrofit

↓

OkHttpClient

↓

Interceptor Chain

↓

RealCall

↓

Dispatcher

↓

Connection Pool

↓

Network
```

---

# Part 5 — OkHttp Dispatcher

Very important.

Dispatcher manages:

* Async calls
* Thread execution
* Maximum concurrent requests

Example:

App starts:

```
Request 1
Request 2
Request 3
...
Request 100
```

Dispatcher controls execution.

---

Default limits:

Maximum requests:

```
64
```

Maximum per host:

```
5
```

(Defaults can vary by version.)

---

Why?

Imagine:

```
App

↓

1000 API Calls

↓

Server
```

Without control:

* Battery drain
* Memory pressure
* Server overload

---

# Part 6 — Async vs Sync Calls in OkHttp

## Synchronous

Example:

```kotlin
call.execute()
```

Runs on current thread.

Usually not used on Android main thread.

---

## Asynchronous

Example:

```kotlin
call.enqueue()
```

OkHttp:

```
Main Thread

↓

Dispatcher

↓

Background Thread

↓

Response Callback
```

---

With coroutines:

```kotlin
suspend
```

uses this asynchronous model internally.

---

# Part 7 — Connection Pool

Opening a network connection is expensive.

Without pooling:

Every request:

```
Create TCP Connection

↓

TLS Handshake

↓

Request

↓

Close Connection
```

Very expensive.

---

Connection pooling:

```
Connection 1

↓

Reuse

↓

Request 1

Request 2

Request 3
```

---

Benefits:

* Faster requests
* Less battery usage
* Less latency

---

OkHttp maintains:

```
Connection Pool

|
|-- Connection A
|-- Connection B
|-- Connection C
```

---

# Part 8 — Keep Alive

HTTP connections can stay alive.

Example:

Server:

```
Connection:
keep-alive
```

Meaning:

"Reuse this connection for future requests."

---

Without keep-alive:

```
Request

↓

Open Connection

↓

Close Connection
```

---

With keep-alive:

```
Request 1

↓

Existing Connection

↓

Request 2

↓

Same Connection
```

---

# Part 9 — DNS Resolution

Before connecting:

App knows:

```
api.example.com
```

but network needs:

```
192.168.1.10
```

DNS converts:

```
Domain

↓

IP Address
```

---

Flow:

```
URL

↓

DNS Lookup

↓

IP Address

↓

TCP Connection
```

---

Possible failures:

```
UnknownHostException
```

Meaning:

DNS failed.

---

# Part 10 — TCP Connection

After DNS:

Android creates TCP connection.

Three-way handshake:

```
Client              Server

 SYN  ------------>

      <------------ SYN ACK

 ACK  ------------>
```

Connection established.

---

Then data transfer starts.

---

# Part 11 — HTTPS and TLS

Most APIs use:

```
https://
```

not:

```
http://
```

Why?

Security.

---

HTTPS adds TLS.

Flow:

```
Client

↓

TLS Handshake

↓

Certificate Verification

↓

Encrypted Connection

↓

HTTP Request
```

---

TLS provides:

## Encryption

Others cannot read data.

---

## Authentication

Verify server identity.

---

## Integrity

Detect tampering.

---

# Part 12 — Certificate Verification

Server provides:

```
Certificate
```

containing:

* Domain name
* Public key
* Issuer

Android checks:

```
Is this certificate trusted?
```

---

If invalid:

Exception:

```
SSLHandshakeException
```

---

# Part 13 — Certificate Pinning

Senior interview topic.

Normal verification:

```
Trust any certificate
from trusted authority
```

Problem:

A compromised certificate authority could issue fake certificates.

---

Certificate pinning:

App says:

"I trust only this certificate/public key."

Flow:

```
Server Certificate

↓

Compare with pinned value

↓

Match?

YES → Continue

NO → Reject
```

---

OkHttp example:

```kotlin
CertificatePinner.Builder()
    .add(
      "api.example.com",
      "sha256/xxxx"
    )
    .build()
```

---

Benefits:

* Protection against man-in-the-middle attacks

Risks:

* Certificate rotation problems

---

# Part 14 — HTTP/1.1 vs HTTP/2

Important performance topic.

---

## HTTP/1.1

Multiple requests:

```
Request 1
Request 2
Request 3
```

Often needs multiple connections.

---

## HTTP/2

Features:

### Multiplexing

Multiple requests over one connection.

```
Single Connection

 ├── Request 1
 ├── Request 2
 └── Request 3
```

---

### Header Compression

Repeated headers compressed.

Example:

Instead of sending:

```
Authorization header
every time
```

compresses repeated data.

---

### Binary Protocol

More efficient than text.

---

# Part 15 — Interceptor Internals

We learned:

```
Interceptor

↓

chain.proceed()
```

Internally:

```
Application Interceptor

↓

Retry Interceptor

↓

Bridge Interceptor

↓

Cache Interceptor

↓

Connect Interceptor

↓

Network Interceptors

↓

Call Server
```

---

OkHttp itself has internal interceptors.

Examples:

## RetryAndFollowUpInterceptor

Handles:

* Redirects
* Authentication challenges
* Retries

---

## BridgeInterceptor

Adds:

* Headers
* Cookies
* Compression

---

## CacheInterceptor

Handles:

* Cache lookup
* Cache updates

---

## ConnectInterceptor

Creates:

* Connection

---

# Part 16 — Request Lifecycle Complete

Full flow:

```
Retrofit API Call

↓

Create OkHttp Request

↓

Application Interceptors

↓

Retry Handling

↓

Headers/Cookies

↓

Cache Check

↓

Connection Creation

↓

Network Interceptors

↓

Server

↓

Response travels backward

↓

Converter

↓

Coroutine resumes

↓

UI updates
```

---

# Part 17 — Performance Optimization

Senior interview question:

> How would you improve API performance?

Answers:

---

## 1. Enable HTTP caching

Reduce unnecessary requests.

---

## 2. Use Pagination

Don't download huge lists.

---

## 3. Compress responses

Use:

```
gzip
```

---

## 4. Reuse connections

OkHttp does this automatically.

---

## 5. Avoid duplicate requests

Example:

Bad:

```
Screen opens

↓

3 identical API calls
```

---

Better:

Share data using:

* Repository
* Flow
* Cache

---

## 6. Optimize images

Use:

* Image compression
* CDN
* Proper sizes

---

# Part 18 — Security Best Practices

Interviewers like this.

---

## Always use HTTPS

Never send:

```
password
tokens
```

over HTTP.

---

## Secure token storage

Use:

* Android Keystore
* Encrypted storage

---

## Avoid logging sensitive data

Bad:

```
Authorization:
Bearer abc123
```

in production logs.

---

## Certificate pinning for sensitive apps

Examples:

* Banking
* Finance
* Enterprise

---

# Part 19 — Senior Interview Questions

---

## Q1. How does Retrofit work internally?

Strong answer:

> Retrofit creates a dynamic proxy implementation of the API interface. It reads annotations, creates an OkHttp request, executes it through OkHttp, converts the response using a converter factory, and returns the result to the caller.

---

## Q2. What does OkHttp Dispatcher do?

Answer:

> Dispatcher manages asynchronous request execution, thread usage, and limits concurrent network calls.

---

## Q3. Why does OkHttp use connection pooling?

Answer:

> Creating TCP and TLS connections is expensive. Connection pooling allows reuse of existing connections, reducing latency and resource consumption.

---

## Q4. Explain HTTPS flow.

Answer:

> The client resolves DNS, establishes a TCP connection, performs TLS handshake, validates the server certificate, creates an encrypted channel, and then sends HTTP requests.

---

## Q5. What is certificate pinning?

Answer:

> Certificate pinning restricts an application to trust only specific certificates or public keys, reducing the risk of man-in-the-middle attacks.

---

## Q6. HTTP/2 advantages?

Answer:

* Multiplexing
* Header compression
* Better connection utilization

---

# Complete Networking Architecture Diagram

```
                 UI

                  |

              ViewModel

                  |

             Repository

             /        \

            /          \

        Room          Retrofit

          |              |

   Local Database     OkHttp

                         |

              -------------------

              |        |        |

        Auth      Logging    Cache

       Interceptor Interceptor

              |

          Dispatcher

              |

       Connection Pool

              |

              DNS

              |

              TLS

              |

            Server
```

---

# Final Senior Interview Answer

If asked:

> "Explain the complete lifecycle of an API request in Android."

Answer:

> "The ViewModel calls the repository, which invokes a Retrofit service. Retrofit creates an OkHttp request using a dynamic proxy and converter configuration. The request passes through OkHttp's interceptor chain where authentication, logging, caching, and retry logic are handled. OkHttp's Dispatcher manages execution and uses connection pooling for efficient network communication. The client resolves DNS, establishes TCP and TLS connections, communicates using HTTP protocols, receives the response, converts it into Kotlin objects, and returns it through the repository to update UI state."

---

# Module 12 Summary

You now understand:

✅ Retrofit internals
✅ Dynamic proxy
✅ Converter factories
✅ OkHttp architecture
✅ Dispatcher
✅ Connection pooling
✅ DNS
✅ TCP
✅ TLS
✅ Certificate pinning
✅ HTTP/2
✅ Internal interceptor chain
✅ Performance optimization
✅ Security practices

---

## You have completed the Android REST API Mastery Path.

Your learning progression:

1. REST API fundamentals
2. HTTP methods & status codes
3. Retrofit basics
4. OkHttp architecture
5. Request/Response handling
6. Interceptors
7. Authentication & JWT
8. Error handling
9. Offline-first & caching
10. Coroutines + networking
11. Complete architecture
12. Retrofit/OkHttp internals

At this level, you should be able to handle most **Android networking interview discussions from junior to senior level**.
