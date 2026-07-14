
This module is asked frequently in interviews for **2–6 years** Android experience.

---

# Module 3 – REST API (Deep Dive)

## Interview Goal

By the end of this module, you should be able to answer:

* What is REST?
* What is a REST API?
* What are REST constraints?
* What is a Resource?
* URI vs URL
* Idempotency
* Safe Methods
* REST Best Practices
* REST API Design
* Versioning
* Pagination
* Filtering
* Sorting

---

# Part 1 — Why REST Exists

Let's start with a problem.

Imagine you have a server that stores users.

How should the Android app ask for users?

One developer creates:

```text
/getUsers
```

Another creates:

```text
/fetchAllUsers
```

Another:

```text
/users/list
```

Another:

```text
/get_user_data
```

All work.

But imagine 100 developers creating APIs however they like.

Chaos.

REST provides **conventions** so that APIs are consistent and predictable.

---

## Interview Definition

**REST** stands for **Representational State Transfer**.

It is an **architectural style**, **not a protocol**.

> This distinction is important.

HTTP is a protocol.

REST is a set of architectural principles that are commonly implemented over HTTP.

---

## Interview Answer

If an interviewer asks:

> What is REST?

A good answer:

> REST (Representational State Transfer) is an architectural style for designing networked applications. It models data as resources, identifies those resources with URIs, and uses standard HTTP methods such as GET, POST, PUT, PATCH, and DELETE to manipulate those resources. RESTful systems are typically stateless and use standard HTTP semantics.

---

# Part 2 — What is a Resource?

This is the heart of REST.

A **resource** is **anything the server exposes**.

Examples:

```text
User

Product

Order

Movie

Payment

Profile

Comment

Image
```

Everything is treated as a resource.

---

Suppose you have:

```text
User
```

Instead of

```text
/getUsers
```

REST says

```text
/users
```

Notice something.

The URL is a **noun**.

Not a verb.

---

Good

```text
/users

/products

/orders
```

Bad

```text
/getUsers

/createUser

/deleteUser
```

Why?

Because the **HTTP method** already tells us the action.

---

# Resource + HTTP Method

This is one of the biggest interview topics.

Suppose we have

```text
/users
```

Now combine it with methods.

```text
GET /users
```

Meaning

> Give me users.

---

```text
POST /users
```

Meaning

> Create a new user.

---

```text
PUT /users/10
```

Meaning

> Replace user 10.

---

```text
PATCH /users/10
```

Meaning

> Partially update user 10.

---

```text
DELETE /users/10
```

Meaning

> Delete user 10.

Notice:

The URL never changes.

The HTTP method changes.

This is elegant and predictable.

---

# Why Nouns Instead of Verbs?

Imagine a library.

The resource is

```text
Book
```

Operations:

Borrow

Return

Read

Delete

Instead of creating:

```text
/borrowBook

/deleteBook

/readBook
```

REST says:

```text
/books
```

and lets the HTTP method express the action.

---

# Part 3 — URI vs URL

Interviewers love this.

Most candidates say:

"They're the same."

They're not.

---

## URI

Uniform Resource Identifier

A URI identifies a resource.

Examples:

```text
/users/10
```

```text
urn:isbn:9780134685991
```

Both identify something.

---

## URL

Uniform Resource Locator

A URL is a URI that also tells you **where** the resource is located and **how** to access it.

Example:

```text
https://api.example.com/users/10
```

Contains:

* Protocol
* Host
* Path

Every URL is a URI.

Not every URI is a URL.

---

### Interview Trick

> Is every URI a URL?

No.

> Is every URL a URI?

Yes.

---

# Part 4 — REST Constraints

This is the theoretical part that interviewers occasionally ask.

REST defines several constraints.

You don't need to memorize the original paper, but you should understand the ideas.

---

## 1. Client–Server

The client and server have separate responsibilities.

Android app

↓

Requests data

Server

↓

Processes requests

Database

↓

Stores data

This separation allows each side to evolve independently.

---

## 2. Stateless

Very important.

Every request contains all the information needed to process it.

Server:

Doesn't remember previous requests.

Example:

```text
GET /profile
```

The request includes:

```text
Authorization: Bearer token
```

The server validates the token each time.

---

## Interview Question

Why is statelessness useful?

Because it:

* simplifies scaling
* simplifies load balancing
* improves reliability
* avoids server-side session memory

---

## 3. Cacheable

Responses may be cached.

Example:

```text
GET /countries
```

Countries rarely change.

The server can tell the client:

```text
Cache-Control: max-age=86400
```

Meaning:

Cache for one day.

This improves performance and reduces server load.

---

## 4. Uniform Interface

All resources follow consistent conventions.

Example:

```text
/users

/products

/orders
```

instead of random naming.

This consistency makes APIs easier to understand.

---

## 5. Layered System

The client doesn't need to know if it's talking directly to the application server, a load balancer, or an API gateway.

```text
Android App
      │
      ▼
API Gateway
      │
      ▼
Load Balancer
      │
      ▼
Application Server
```

Each layer has a specific responsibility.

---

# Part 5 — Path Parameters

Example:

```text
/users/15
```

15 identifies a specific resource.

Retrofit:

```kotlin
@GET("users/{id}")
suspend fun getUser(
    @Path("id") id: Int
): User
```

Result:

```text
/users/15
```

---

# Part 6 — Query Parameters

Query parameters modify or filter the result.

Example:

```text
/users?page=2
```

```text
/products?category=mobile
```

```text
/products?sort=price
```

Retrofit:

```kotlin
@GET("users")
suspend fun getUsers(
    @Query("page") page: Int
): List<User>
```

---

## Interview Question

Difference between Path and Query?

Path:

```text
/users/10
```

Identifies a specific resource.

Query:

```text
/users?page=2
```

Filters or customizes the returned data.

---

# Part 7 — Pagination

Suppose there are

```text
20 million users
```

Should the server return all of them?

No.

Instead:

```text
/users?page=1
```

Returns:

20 users.

Next:

```text
/users?page=2
```

Returns:

Next 20 users.

---

Modern APIs often use:

```text
limit

offset
```

or

```text
cursor
```

Cursor-based pagination is generally preferred for very large datasets because it avoids some issues with changing data during pagination.

---

# Part 8 — Filtering

Example:

```text
/products?category=laptop
```

Returns only laptops.

---

Multiple filters:

```text
/products?brand=Apple&price=1000
```

---

# Part 9 — Sorting

Example:

```text
/products?sort=price
```

Descending:

```text
/products?sort=-price
```

(Exact syntax depends on the API.)

---

# Part 10 — Versioning

Interview question.

Why version APIs?

Suppose today's response is:

```json
{
   "name":"John"
}
```

Tomorrow:

```json
{
   "firstName":"John"
}
```

Older apps would break.

Versioning allows old and new clients to coexist.

Examples:

```text
/v1/users
```

```text
/v2/users
```

---

# REST Best Practices

Use nouns:

```text
/users
```

Not:

```text
/getUsers
```

Use proper HTTP methods.

Return appropriate status codes.

Support pagination.

Support filtering.

Support sorting.

Version APIs.

Keep endpoints predictable.

---

# Android Interview Questions

### What is REST?

REST is an architectural style for designing networked applications using resources, URIs, HTTP methods, and stateless communication.

---

### Why are resources nouns?

Because the HTTP method expresses the action, while the URI identifies the resource.

---

### Difference between URI and URL?

Every URL is a URI.

Not every URI is a URL.

---

### Difference between Path and Query?

Path identifies a specific resource.

Query customizes or filters the response.

---

### Why is REST stateless?

Each request contains all required information, allowing servers to remain independent of previous requests. This makes systems easier to scale and load balance.

---

### Why do APIs use versioning?

To introduce changes without breaking existing clients.

---

### Explain Pagination

Pagination divides large datasets into smaller pages, reducing bandwidth usage, improving response times, and lowering memory consumption on both the server and the client.

---

# Common Interview Mistakes

❌ "REST is a protocol."

**Correct:** REST is an architectural style.

---

❌ "GET `/getUsers` is REST."

**Better:** `GET /users`.

---

❌ "URL and URI are the same."

**Better:** Every URL is a URI, but not every URI is a URL.

---

# Interview-Level Summary

If an interviewer asks:

> "What is REST and how is it used in Android?"

A strong answer would be:

> "REST is an architectural style for designing web APIs. It models data as resources identified by URIs and uses standard HTTP methods like GET, POST, PUT, PATCH, and DELETE to perform operations on those resources. REST APIs are typically stateless, meaning each request contains all the information needed to process it, such as authentication tokens. In Android, libraries like Retrofit allow us to map these REST endpoints to Kotlin interface methods, making communication with backend services simple while still relying on HTTP under the hood."

---

## Before Module 4

At this point, you understand the **networking concepts**:

* ✅ HTTP
* ✅ REST
* ✅ Resources
* ✅ URIs
* ✅ Methods
* ✅ Status codes
* ✅ Pagination
* ✅ Query vs Path parameters

Now we're ready to move into the **Android implementation**.

**Module 4: Retrofit Internals** is where we answer questions like:

* How does `retrofit.create(ApiService::class.java)` work?
* How can Retrofit implement an interface without you writing a class?
* What are dynamic proxies?
* What happens internally when you call `api.getUsers()`?
* Where does OkHttp fit into the flow?

These are common interview questions for mid-level and senior Android roles because they test whether you understand the libraries beyond just using their annotations.
