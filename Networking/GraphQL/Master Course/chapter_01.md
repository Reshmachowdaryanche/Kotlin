# Chapter 1 – APIs and Client-Server Architecture

## GraphQL for Android Developers

> **Learning Goal**
>
> By the end of this chapter, you will understand:
>
> * What an API is
> * Why Android applications need APIs
> * Client-Server Architecture
> * HTTP basics
> * Request and Response lifecycle
> * JSON
> * REST API fundamentals
> * Why GraphQL was introduced (high-level overview)
> * How Android communicates with servers

---

# Table of Contents

1. Introduction
2. What is an API?
3. Why Android Apps Need APIs
4. Client-Server Architecture
5. Types of APIs
6. HTTP Fundamentals
7. HTTP Request
8. HTTP Response
9. HTTP Methods
10. HTTP Status Codes
11. JSON
12. REST APIs
13. Android Networking
14. Complete Request Flow
15. Real World Example
16. Summary
17. Interview Questions

---

# 1. Introduction

Every Android application needs data.

Examples:

* Instagram needs posts
* WhatsApp needs messages
* Amazon needs products
* YouTube needs videos
* Swiggy needs restaurants
* Banking apps need account details

Where does this data come from?

**Answer:** A server.

The Android app communicates with the server using an **API**.

---

# 2. What is an API?

API stands for

> **Application Programming Interface**

Think of an API as a **messenger** between two applications.

```
Android App
      │
      │ Request
      ▼
     API
      │
      ▼
 Server
      │
      ▼
 Database
```

The API receives requests from the app, interacts with the server or database, and sends back a response.

---

## Real-Life Analogy: Restaurant

Imagine you're at a restaurant.

```
Customer
    │
    ▼
 Waiter
    │
    ▼
 Kitchen
```

* Customer → Android App
* Waiter → API
* Kitchen → Server
* Food → Data

You don't go into the kitchen yourself. You tell the waiter what you want. The waiter brings back your food.

Similarly:

```
Android App
      │
      ▼
      API
      │
      ▼
   Database
```

The app never talks directly to the database.

---

# 3. Why Android Apps Need APIs

Android apps cannot directly access a company's database because of security, scalability, and business logic.

Suppose an app directly connected to a database:

```
Android App
      │
      ▼
 Database
```

Problems:

* Database credentials would be exposed.
* Anyone could modify data.
* No validation.
* No authentication.
* High security risk.

Instead:

```
Android App
      │
      ▼
      API
      │
      ▼
 Server
      │
      ▼
 Database
```

The server checks permissions, validates input, applies business rules, and only then accesses the database.

---

# 4. Client-Server Architecture

Most Android apps use a client-server architecture.

```
+----------------------+
|    Android App       |
|      (Client)        |
+----------+-----------+
           |
      Internet
           |
           v
+----------+-----------+
|       Server         |
|  Business Logic      |
+----------+-----------+
           |
           v
+----------+-----------+
|      Database        |
+----------------------+
```

### Responsibilities

**Client (Android App)**

* Display UI
* Accept user input
* Send API requests
* Show server responses

**Server**

* Authenticate users
* Validate requests
* Execute business logic
* Query the database
* Return responses

**Database**

* Store data
* Retrieve data
* Update data
* Delete data

---

# Example Flow

User logs in.

```
User
 │
 ▼
Android App
 │
 ▼
Login API
 │
 ▼
Server
 │
 ▼
Database
 │
 ▼
Server
 │
 ▼
Android
 │
 ▼
User Logged In
```

---

# 5. Types of APIs

### REST API

Example:

```
GET /users

GET /products

POST /login
```

Most traditional Android apps use REST.

---

### GraphQL API

Example:

```
POST /graphql
```

A single endpoint handles all operations. The client specifies exactly what data it needs.

---

### SOAP API

An older XML-based protocol. It is still used in some enterprise systems but is uncommon in modern Android development.

---

### gRPC

Uses Protocol Buffers instead of JSON.

Benefits:

* Fast
* Compact
* Common in microservices
* Useful for high-performance systems

---

# 6. HTTP Fundamentals

HTTP stands for

> **HyperText Transfer Protocol**

It is the protocol most apps use to communicate with web servers.

```
Android
    │
HTTP Request
    │
    ▼
Server
    │
HTTP Response
    │
    ▼
Android
```

Without HTTP, your app couldn't communicate over the web.

---

# 7. HTTP Request

A request consists of:

```
Method

URL

Headers

Body
```

Example:

```
POST https://api.company.com/login
```

Headers:

```
Authorization

Content-Type

Accept
```

Body:

```json
{
  "email":"john@gmail.com",
  "password":"123456"
}
```

---

# 8. HTTP Response

Example:

```json
{
    "token":"abc123",
    "name":"John"
}
```

The server returns:

* Status code
* Headers
* Body

---

# 9. HTTP Methods

### GET

Retrieve data.

```
GET /users/10
```

---

### POST

Create new data.

```
POST /users
```

---

### PUT

Replace an existing resource.

```
PUT /users/10
```

---

### PATCH

Update part of a resource.

```
PATCH /users/10
```

---

### DELETE

Remove data.

```
DELETE /users/10
```

---

# 10. HTTP Status Codes

### 200 OK

Everything worked.

---

### 201 Created

A new resource was created.

---

### 204 No Content

The request succeeded but there is no response body.

---

### 400 Bad Request

The client sent invalid data.

---

### 401 Unauthorized

Authentication is required or invalid.

---

### 403 Forbidden

Authenticated, but not allowed to perform the action.

---

### 404 Not Found

The requested resource doesn't exist.

---

### 500 Internal Server Error

Something went wrong on the server.

---

# 11. JSON

Most APIs exchange data using JSON.

Example:

```json
{
  "id":101,
  "name":"John",
  "email":"john@gmail.com",
  "skills":[
    "Android",
    "Kotlin",
    "GraphQL"
  ]
}
```

### Why JSON?

* Lightweight
* Easy to read
* Easy to parse
* Supported by almost every programming language

---

# 12. REST APIs

REST stands for

> **Representational State Transfer**

REST organizes resources into different endpoints.

Example:

```
GET /users

GET /users/10

GET /products

POST /login

DELETE /users/10
```

Each endpoint represents a resource or action.

---

# 13. Android Networking

Android doesn't access the internet directly through UI code. Typically, the app uses networking libraries.

Popular libraries:

* Retrofit (REST)
* OkHttp (HTTP client)
* Apollo Kotlin (GraphQL)
* Ktor Client

Example architecture:

```
Activity
     │
     ▼
ViewModel
     │
     ▼
Repository
     │
     ▼
Retrofit / Apollo
     │
     ▼
Internet
```

This separation keeps the UI clean and makes the app easier to test.

---

# 14. Complete Request Flow

Imagine a user opens a profile screen.

```
User taps Profile
        │
        ▼
Activity / Compose Screen
        │
        ▼
ViewModel
        │
        ▼
Repository
        │
        ▼
Network Client
        │
        ▼
HTTP Request
        │
        ▼
API Server
        │
        ▼
Database
        │
        ▼
Response
        │
        ▼
Repository
        │
        ▼
ViewModel
        │
        ▼
UI Updates
```

Each layer has a clear responsibility, making the app easier to maintain.

---

# 15. Real-World Example: Shopping App

When a user opens a product page:

```
User opens Product Screen
        │
        ▼
Android App
        │
GET /products/101
        │
        ▼
Server
        │
        ▼
Database
        │
        ▼
Product Information
        │
        ▼
Android Displays:
- Product Name
- Price
- Images
- Reviews
```

The app never reads the database directly; it always communicates through the API.

---

# 16. Key Takeaways

* An API is the communication layer between the client and the server.
* Android apps rely on APIs to fetch and update data.
* Most apps follow a client-server architecture.
* HTTP is the standard protocol used for communication.
* JSON is the most common data format for API responses.
* REST organizes APIs into multiple endpoints.
* GraphQL (covered in later chapters) often uses a single endpoint and lets the client request only the data it needs.

---

# 17. Interview Questions

### 1. What is an API?

**Answer:** An API (Application Programming Interface) allows different software applications to communicate with each other. In Android, APIs are used to exchange data with backend servers.

---

### 2. Why can't an Android app connect directly to a database?

**Answer:** Direct database access would expose credentials, bypass security, and allow unauthorized data access. A server sits between the app and the database to authenticate users, validate requests, and enforce business rules.

---

### 3. What is client-server architecture?

**Answer:** The client (Android app) sends requests to the server. The server processes those requests, interacts with the database if needed, and returns a response.

---

### 4. What is HTTP?

**Answer:** HTTP (HyperText Transfer Protocol) is the protocol used for communication between clients and web servers.

---

### 5. What is JSON?

**Answer:** JSON (JavaScript Object Notation) is a lightweight, text-based data format commonly used to exchange structured data between clients and servers.

---

### 6. What is REST?

**Answer:** REST is an architectural style for designing web APIs. It exposes resources through multiple endpoints and typically uses HTTP methods such as GET, POST, PUT, PATCH, and DELETE.

---

### 7. Which networking libraries are commonly used in Android?

**Answer:**

* **Retrofit** for REST APIs
* **OkHttp** as the HTTP client
* **Apollo Kotlin** for GraphQL
* **Ktor Client** for Kotlin-based networking

---

## What's Next?

**Chapter 2 – Problems with REST APIs**

You'll learn why GraphQL was created by exploring the limitations of REST, including over-fetching, under-fetching, multiple network requests, the N+1 problem, API versioning, and the impact these issues have on Android applications.
