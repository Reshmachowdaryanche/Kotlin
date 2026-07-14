

# Module 2 – HTTP Deep Dive

## Big Picture

Let's start with a question.

Suppose your Android app executes:

```kotlin
api.getUsers()
```

Does Retrofit send this Kotlin function over the internet?

**No.**

Retrofit converts it into an **HTTP request**.

The server doesn't understand Kotlin, Java, or Android. It understands **HTTP**.

Think of HTTP as the **language** that both the client and server agree to speak.

```
Android App
      │
      ▼
HTTP Request
      │
      ▼
Server
      │
      ▼
HTTP Response
      │
      ▼
Android App
```

---

# What is HTTP?

**Interview Definition:**

> HTTP (HyperText Transfer Protocol) is an application-layer protocol used for communication between clients and servers.

Let's break that down.

* **Protocol** = A set of rules.
* **Application Layer** = It's designed for applications like browsers, mobile apps, and APIs to exchange data.

HTTP defines:

* How to make a request.
* How to structure a response.
* What methods are allowed.
* What status codes mean.
* How headers are sent.

It does **not** define your business logic.

---

# Structure of an HTTP Request

An HTTP request has four main parts:

```
Request Line
Headers
Blank Line
Body (optional)
```

Example:

```http
POST /users HTTP/1.1
Host: api.example.com
Authorization: Bearer abc123
Content-Type: application/json

{
  "name": "John",
  "age": 25
}
```

Let's examine each part.

---

# 1. Request Line

```
POST /users HTTP/1.1
```

It contains:

* **HTTP Method** → `POST`
* **Path** → `/users`
* **HTTP Version** → `HTTP/1.1`

Think of it as:

> "What do you want to do, and to which resource?"

---

# 2. Headers

Headers contain **metadata** about the request.

Example:

```
Authorization: Bearer token
Content-Type: application/json
Accept: application/json
User-Agent: Android
```

Notice something important:

Headers are **not** the actual data.

They're instructions or additional information.

Think of sending a courier package.

The package:

```
Laptop
```

The label:

```
Fragile
Handle with care
Destination
```

Headers are the label.

Body is the package.

---

# Common Request Headers

## Authorization

Used for authentication.

```
Authorization: Bearer eyJhbGci...
```

Without it:

Server:

```
401 Unauthorized
```

---

## Content-Type

Tells the server what you're sending.

```
application/json
```

means:

"I'm sending JSON."

Other examples:

```
multipart/form-data
```

(File uploads)

```
text/plain
```

```
application/xml
```

---

## Accept

Tells the server what format you want back.

```
Accept: application/json
```

Meaning:

"Please return JSON if possible."

---

## User-Agent

Identifies the client.

Example:

```
User-Agent: Android
```

Browsers also send one.

Servers may use it for analytics or compatibility.

---

# 3. Body

The body contains the actual data.

Example:

```json
{
    "name":"John",
    "age":25
}
```

Only some HTTP methods typically include a body (such as POST, PUT, and PATCH). GET requests generally do **not** include one.

---

# HTTP Response

The server sends back:

```
Status Line
Headers
Blank Line
Body
```

Example:

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
    "id":1,
    "name":"John"
}
```

---

# Status Line

```
HTTP/1.1 200 OK
```

Contains:

* Version
* Status Code
* Reason Phrase

Example:

```
HTTP/1.1 404 Not Found
```

---

# Response Headers

Example:

```
Content-Type: application/json
Content-Length: 128
Cache-Control: max-age=60
```

These provide metadata about the response.

---

# Response Body

Usually JSON.

```json
{
   "id":1,
   "name":"John"
}
```

Retrofit converts this into:

```kotlin
User(
   id = 1,
   name = "John"
)
```

using a converter like Gson, Moshi, or Kotlin Serialization.

---

# HTTP Methods (Very Important)

## GET

Purpose:

Retrieve data.

Example:

```
GET /users
```

Returns:

```
[
 { ... },
 { ... }
]
```

Characteristics:

* Retrieves data
* Should not modify server state
* Safe
* Idempotent

### Interview Question

**Why should GET not modify data?**

Because GET is intended for retrieval. Caches, browsers, and proxies assume it has no side effects. If a GET endpoint deleted data, it would violate HTTP semantics and could cause unexpected behavior.

---

## POST

Purpose:

Create a new resource.

Example:

```
POST /users
```

Body:

```json
{
   "name":"John"
}
```

Server:

Creates a new user.

Characteristics:

* Usually changes server state
* Not idempotent

### Why isn't POST idempotent?

If you send:

```
POST /orders
```

twice,

you might create:

```
Order #101

Order #102
```

Two different resources.

---

## PUT

Purpose:

Replace the entire resource.

Example:

```
PUT /users/5
```

If the existing user is:

```json
{
   "name":"John",
   "age":25
}
```

and you send:

```json
{
   "name":"Alice",
   "age":30
}
```

The server replaces the existing representation with the new one (subject to how that API is designed).

PUT is **idempotent**.

Sending the same PUT request multiple times should result in the same final state.

---

## PATCH

Purpose:

Partially update a resource.

Existing user:

```json
{
   "name":"John",
   "age":25
}
```

Request:

```json
{
   "age":26
}
```

Only the age changes.

---

### Interview Question

**Difference between PUT and PATCH?**

| PUT                                       | PATCH                                                                  |
| ----------------------------------------- | ---------------------------------------------------------------------- |
| Replaces the full resource representation | Updates part of a resource                                             |
| Usually sends the complete object         | Sends only changed fields                                              |
| Idempotent                                | Often idempotent, but not guaranteed—it depends on the API's semantics |

Many interviewers appreciate if you mention that PATCH **can** be idempotent, but it is **not guaranteed by definition**.

---

## DELETE

Deletes a resource.

```
DELETE /users/5
```

Usually idempotent.

Deleting the same user twice leaves the server in the same final state (although the second request may return a different status such as `404 Not Found`).

---

# Safe vs Idempotent

This is a favorite interview topic.

## Safe

A safe method should not modify server state.

Safe methods:

* GET
* HEAD
* OPTIONS

---

## Idempotent

Making the same request multiple times results in the same **final state**.

Examples:

```
PUT
```

Update age to 30.

First request:

Age = 30

Second request:

Age = 30

Third request:

Age = 30

Final state remains unchanged.

---

POST is not idempotent:

```
POST /orders
```

Each request may create a new order.

---

# HTTP Status Codes

Interviewers expect you to know the categories.

## 1xx

Informational.

Rare in Android apps.

---

## 2xx

Success.

```
200 OK
```

Request succeeded.

```
201 Created
```

New resource created.

```
204 No Content
```

Success with no response body.

---

## 3xx

Redirection.

```
301 Moved Permanently
302 Found
304 Not Modified
```

### 304 Not Modified

Important for caching.

It means:

> "Use your cached copy."

The server indicates the resource hasn't changed, so it doesn't resend the body.

---

## 4xx

Client Errors.

```
400 Bad Request
```

Malformed request.

---

```
401 Unauthorized
```

Authentication is missing or invalid.

---

```
403 Forbidden
```

Authenticated, but not allowed to access the resource.

Example:

Normal user trying to access an admin endpoint.

---

```
404 Not Found
```

Resource doesn't exist.

---

```
409 Conflict
```

Conflict with the current state of the resource (for example, trying to create a duplicate unique record).

---

## 5xx

Server Errors.

```
500 Internal Server Error
```

Unexpected server failure.

---

```
502 Bad Gateway
```

A gateway/proxy received an invalid response from an upstream server.

---

```
503 Service Unavailable
```

Server temporarily unavailable (maintenance or overload).

---

# Android Interview Mapping

When you write:

```kotlin
@GET("users")
suspend fun getUsers(): Response<List<User>>
```

and receive:

```
200
```

You process the body.

When you receive:

```
404
```

The request **reached the server**, but the resource wasn't found.

When there's:

```
UnknownHostException
```

The request **never reached the server**—it's a network problem, not an HTTP error.

This distinction is very important in interviews.

---

# Frequently Asked Interview Questions

### 1. Why doesn't GET usually have a request body?

Because GET is defined for retrieving resources, and many clients, servers, and intermediaries do not support or expect request bodies with GET. While the HTTP specification doesn't outright forbid a body on GET, relying on one leads to poor interoperability.

---

### 2. What's the difference between 401 and 403?

**401 Unauthorized**

* Authentication is missing or invalid.
* Example: Missing or expired Bearer token.

**403 Forbidden**

* Authentication succeeded.
* The user doesn't have permission.

---

### 3. Why is PUT idempotent?

Because sending the same PUT request multiple times results in the same final resource state.

---

### 4. Why is POST not idempotent?

Because each POST request may create a new resource or trigger another action.

---

### 5. Is DELETE idempotent?

Yes, in terms of the final state. Repeating the same DELETE request should leave the resource deleted, even if later requests return a different status code.

---

## Homework (Interview Practice)

Try answering these aloud without looking at the notes:

1. What are the four parts of an HTTP request?
2. What's the difference between request headers and the request body?
3. Explain GET, POST, PUT, PATCH, and DELETE with real-world examples.
4. What's the difference between **safe** and **idempotent** methods?
5. Explain the difference between **401** and **403**.
6. Why is **304 Not Modified** useful?
7. Why doesn't GET typically include a request body?
8. What's the difference between an HTTP error (like `404`) and a network error (like `UnknownHostException`)?
9. How does Retrofit turn an annotated interface method into an HTTP request?
10. If an interviewer asks, "What is HTTP?" can you explain not only the definition but also **why it exists**?

---

### One small correction to something many tutorials teach

You'll often hear:

> **401 = Unauthorized**

But in practice, it's more accurate to think of it as:

* **401** → *"I can't verify who you are."* (Authentication problem)
* **403** → *"I know who you are, but you're not allowed to do this."* (Authorization problem)

Interviewers appreciate candidates who understand the difference between **authentication** (identity) and **authorization** (permissions), because those concepts come up again when discussing JWTs, interceptors, and authenticators.
