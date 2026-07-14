
# Module 1: Internet & Networking Fundamentals

## Learning Objectives

By the end of this module, you should be able to answer:

> **"Explain what happens when an Android app makes an API call."**

This is one of the most common networking interview questions.

---

# Module Structure

We'll cover:

1. Why Networking Exists
2. Client & Server
3. Internet
4. IP Address
5. Domain Name
6. DNS
7. URL
8. URI
9. Port
10. Protocol
11. HTTP
12. HTTPS
13. SSL/TLS
14. Statelessness
15. Complete Flow of an Android API Request
16. Interview Questions

---

# Part 1 — Why Do We Need Networking?

### Interview Question

> Why does an Android app need networking?

Most candidates answer:

> To fetch data from the server.

This isn't wrong, but it's incomplete.

### Better Answer

Networking enables **communication between distributed systems**.

An Android app runs on the user's device, while business data (users, products, payments, orders) is stored on remote servers. Networking allows the app to exchange data with those servers over standardized protocols such as HTTP.

This answer shows you understand the broader purpose.

---

## Example

Imagine you're building an e-commerce app.

Where is the product data stored?

Not inside the APK.

Because:

* Products change.
* Prices change.
* Stock changes.
* Images change.

If all of this were inside the app:

```text
APK
 ├── Products
 ├── Prices
 ├── Orders
```

every price update would require publishing a new app version.

Instead:

```text
Android App
      │
      ▼
Internet
      │
      ▼
Backend Server
      │
      ▼
Database
```

The app becomes a **client** that requests fresh data whenever needed.

---

# Part 2 — Client and Server

This is one of the most fundamental concepts.

## What is a Client?

A client is a program that **initiates communication** by requesting a service or data from another system.

Examples:

* Android app
* iOS app
* Chrome browser
* Postman
* curl

Interview definition:

> A client is a software application that initiates requests to a server and consumes the responses.

Notice the phrase **initiates requests**.

The client always starts the conversation.

---

## What is a Server?

A server is a program that listens for incoming requests, processes them, and returns responses.

It may:

* Validate authentication.
* Apply business logic.
* Read/write a database.
* Call external services.
* Return an HTTP response.

---

## Restaurant Analogy

```text
Customer
      │
      ▼
Waiter
      │
      ▼
Kitchen
```

Customer → Client

Kitchen → Server

Menu → API

Order → HTTP Request

Food → HTTP Response

The customer cannot enter the kitchen. They interact through the menu. Similarly, your app cannot directly access the database; it communicates through an API.

---

## Android Example

You open Instagram.

The app shows:

* Followers
* Likes
* Posts

Where is that data?

Not inside your phone.

Instead:

```text
Instagram App

↓

Instagram Server

↓

Database
```

The app asks:

```http
GET /feed
```

The server responds with the latest feed.

---

## Common Interview Follow-up

> Can a server also act as a client?

**Answer: Yes.**

Example:

```text
Android App
      │
      ▼
API Gateway
      │
      ▼
Payment Service
      │
      ▼
Bank API
```

The Payment Service is a **server** for the Android app, but it becomes a **client** when it calls the Bank API.

This demonstrates that "client" and "server" describe **roles in a communication**, not fixed types of software.

---

# Part 3 — What is the Internet?

Many people answer:

> The internet is a network.

That's too vague.

### Better Answer

The internet is a **global network of interconnected computer networks** that communicate using the **TCP/IP protocol suite**.

Let's unpack that.

Your phone isn't connected directly to a server in another country. Data travels through many intermediate devices.

```text
Phone
   │
Wi-Fi
   │
Router
   │
ISP
   │
Regional Network
   │
International Backbone
   │
Data Center
   │
Server
```

---

## How Does Data Travel?

Suppose you're downloading:

```text
10 MB
```

Does the network send one huge block?

No.

It breaks the data into many **packets**.

```text
Packet 1

Packet 2

Packet 3

Packet 4
```

Each packet can take a different route across the internet and is reassembled at the destination.

This packet-based design makes the internet scalable and resilient.

---

# Part 4 — IP Address

### Interview Question

> What is an IP address?

### Good Answer

An IP address is a **unique logical address** assigned to a device on a network, allowing other devices to identify and communicate with it.

Example:

```text
192.168.1.10
```

(Local/private IPv4)

```text
142.250.190.78
```

(Public IPv4)

---

## Why Do We Need IP Addresses?

Imagine sending a letter without an address.

Impossible.

Similarly, a server needs to know where to send the response.

```text
Client IP

↓

Server

↓

Response returns to Client IP
```

---

## IPv4 vs IPv6

Interviewers sometimes ask this.

### IPv4

32 bits

Example:

```text
192.168.1.10
```

About 4.3 billion unique addresses.

---

### IPv6

128 bits

Example:

```text
2001:db8::8a2e:370:7334
```

Designed to provide a vastly larger address space.

You don't need to memorize the exact numbers, but know **why IPv6 exists**.

---

# Part 5 — Public vs Private IP

Private IP:

```text
192.168.x.x

10.x.x.x

172.16.x.x
```

Used within local networks (home, office).

Public IP:

Assigned by an ISP and reachable over the internet.

Example:

```text
Android Phone

↓

Private IP

↓

Wi-Fi Router

↓

Public IP

↓

Internet
```

Your router performs **NAT (Network Address Translation)** so multiple devices can share a single public IP.

---

# Interview Questions

### Q1. What is a client?

**Expected Answer:**

A client is a software application that initiates requests to another system to consume data or services.

---

### Q2. What is a server?

A server is a software system that listens for requests, processes them, and returns responses.

---

### Q3. Can a server become a client?

**Yes.** When it calls another service, it acts as a client in that interaction.

---

### Q4. What is an IP address?

A unique logical address used to identify devices and route network traffic.

---

### Q5. Why do we need IP addresses?

To uniquely identify communication endpoints so data can be delivered correctly.

---

### Q6. Why doesn't an Android app connect directly to a database?

Because exposing the database would create serious security risks, bypass business rules, complicate scaling, and make authentication and authorization difficult. Instead, the app communicates with a backend API that enforces these concerns.

---

# Interview Tip

One thing interviewers notice is whether you stop at definitions or explain **why** something exists.

For example:

❌ Weak:

> "An IP address identifies a device."

✅ Strong:

> "An IP address uniquely identifies a device on a network and enables routers to deliver packets to the correct destination. Without IP addressing, devices wouldn't know where to send requests or responses."

The second answer shows understanding, not memorization.

---
