
The goal of Lesson 1 is:

> **Understand exactly what happens from the moment a user taps a button until data comes back to your app.**

---

# Lesson 1: Internet, HTTP & REST (Deep Dive)

Imagine you're building a weather app.

The user taps:

```text
Get Weather
```

In your code, this eventually becomes:

```kotlin
viewModel.getWeather()
```

But what happens after that?

Most developers imagine this:

```text
App
 ↓
Server
```

Reality is much more interesting.

```text
┌───────────────────────┐
│ Android App           │
└──────────┬────────────┘
           │
           ▼
     Internet Provider
           │
           ▼
     DNS Server
           │
           ▼
   Finds Server IP Address
           │
           ▼
      Internet Routers
           │
           ▼
      Server Computer
           │
           ▼
     Application Server
           │
           ▼
        Database
           │
           ▼
     Application Server
           │
           ▼
     Internet Routers
           │
           ▼
      Android App
```

Your request travels across many systems before it reaches the server.

---

# What is a Client?

A client is simply something that **requests** information.

Examples:

* Android App
* Chrome
* Firefox
* Postman
* Curl

All of these are clients.

Example:

```text
Android App
      │
      ▼
Please send me all users.
```

The client starts the conversation.

---

# What is a Server?

A server waits for requests.

Example:

```text
Android
   │
   ▼
GET /users

──────────────►

Server
```

The server processes the request.

It might:

* check permissions
* access a database
* perform calculations
* call another API

Then it responds.

```text
Server
    │
    ▼
Here are your users.
```

A server never randomly sends data to your app. It responds when asked.

---

# A Real-Life Analogy

Think of a restaurant.

Customer = Client

Kitchen = Server

Menu = API Documentation

Order = HTTP Request

Food = HTTP Response

```
Customer
    │
"I want Pizza"
    │
    ▼
Waiter
    │
    ▼
Kitchen
    │
Makes Pizza
    │
    ▼
Customer
```

Notice something important:

The customer never enters the kitchen.

They use the menu.

Your Android app never touches the database directly.

It talks through an API.

---

# What is an API?

API stands for **Application Programming Interface**.

A simpler definition:

> An API is a set of rules that tells clients how to communicate with a server.

Think of it as a menu.

Restaurant Menu:

```
Pizza
Burger
Pasta
```

API Menu:

```
GET /users

POST /login

GET /products

DELETE /cart/5
```

The API says:

"You may ask for these things, but only in these ways."

---

# Why Can't an App Access the Database Directly?

Many beginners ask:

> "Why doesn't my app connect directly to MySQL?"

Imagine every user's phone connected directly to your production database.

```
Millions of Phones

        │

        ▼

Database
```

Problems:

* Huge security risk
* Anyone could modify data
* No business rules
* No authentication
* Difficult to scale

Instead:

```
Android

↓

API Server

↓

Database
```

The server decides:

* Can this user access data?
* Is the token valid?
* Does the user have permission?
* Which data should be returned?

The app never sees the database.

---

# What is the Internet?

The internet is simply a massive network of connected computers.

```
Phone

↓

Wi-Fi

↓

ISP

↓

Routers

↓

Other Routers

↓

Server
```

Data is broken into small packets and routed across many devices until it reaches its destination.

---

# What is an IP Address?

Every device connected to the internet has an address.

Example:

```
142.250.190.78
```

A server also has an IP address.

Without an IP address, nothing knows where to send data.

---

# Why Don't We Use IP Addresses?

Would you rather type:

```
142.250.190.78
```

or

```
google.com
```

Humans prefer names.

Computers use numbers.

That's why DNS exists.

---

# DNS (Domain Name System)

DNS is like the internet's phone book.

When you type:

```
google.com
```

your phone asks:

```
DNS

Where is google.com?
```

DNS replies:

```
142.250.190.78
```

Now your phone knows where to send the request.

Flow:

```
google.com

↓

DNS

↓

142.250.190.78

↓

Server
```

Without DNS, you'd have to remember IP addresses.

---

# What is a URL?

Example:

```
https://api.github.com/users/octocat
```

Break it into parts:

```
https://
```

Protocol

```
api.github.com
```

Host (Domain)

```
/users/octocat
```

Path

Another example:

```
https://example.com/products/15/reviews
```

Parts:

```
Protocol

↓

Domain

↓

Path

↓

Resource
```

---

# What is a Protocol?

A protocol is simply an agreed set of rules.

Humans use languages.

Computers use protocols.

Examples:

```
HTTP

HTTPS

FTP

SMTP

WebSocket
```

For Android REST APIs, the important ones are **HTTP** and **HTTPS**.

---

# HTTP

HTTP stands for **HyperText Transfer Protocol**.

It defines:

* how requests are sent
* how responses are structured
* status codes
* headers
* methods

HTTP doesn't define *what* data is sent, only *how* communication happens.

Think of it as the grammar of a conversation.

---

# HTTPS

HTTPS is HTTP plus encryption.

```
HTTP

↓

Encrypt Everything

↓

HTTPS
```

Without HTTPS, someone on the same network could potentially read your traffic.

With HTTPS, the contents are encrypted in transit.

---

# What Actually Gets Sent?

Suppose you call:

```kotlin
api.getUsers()
```

The network doesn't send Kotlin code.

It sends plain text that follows the HTTP protocol.

A simplified request looks like this:

```http
GET /users HTTP/1.1
Host: api.example.com
Accept: application/json
```

The server responds with something like:

```http
HTTP/1.1 200 OK
Content-Type: application/json

[
  {
    "id": 1,
    "name": "Alice"
  }
]
```

This is why understanding HTTP matters even if you use Retrofit.

---

# Request and Response

Every HTTP interaction has two parts.

```
Client
   │
   │ Request
   ▼
Server
   │
   │ Response
   ▼
Client
```

A request asks for something.

A response answers.

No response can exist without a request.

---

# What is Stateless?

HTTP is **stateless**.

This means the server treats each request independently.

Example:

Request 1:

```
GET /users
```

Server responds.

Five minutes later:

```
GET /users
```

The server does **not** automatically remember the earlier request.

If the server needs to identify you, you usually send something like an authentication token with each request. We'll cover that in a later lesson.

---

# Why This Matters for Android Developers

When you eventually write:

```kotlin
@GET("users")
suspend fun getUsers(): List<User>
```

Retrofit is ultimately producing an HTTP request like:

```http
GET /users HTTP/1.1
Host: api.example.com
Accept: application/json
```

The server doesn't know or care that your app is written in Kotlin or that you're using Retrofit. It only understands the HTTP request it receives.

