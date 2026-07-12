# Chapter 2 – Problems with REST APIs

## GraphQL for Android Developers

> **Learning Goal**
>
> By the end of this chapter, you will understand:
>
> * Why REST APIs can become inefficient
> * Over-fetching and under-fetching
> * Multiple network requests
> * The N+1 request problem (client perspective)
> * Mobile bandwidth and battery concerns
> * API versioning challenges
> * Why Facebook created GraphQL

---

# Table of Contents

1. Introduction
2. Why REST Was Successful
3. Limitations of REST
4. Over-Fetching
5. Under-Fetching
6. Multiple Network Requests
7. N+1 Problem (Client Side)
8. API Versioning
9. Mobile Performance Issues
10. Real Android Example
11. How GraphQL Solves These Problems
12. Summary
13. Interview Questions

---

# 1. Introduction

REST has been the dominant API architecture for many years.

Companies like:

* Twitter (early APIs)
* GitHub
* Spotify
* Amazon
* Netflix

have all used REST extensively.

REST works well for many applications. However, as mobile apps became more complex, developers encountered several challenges:

* Downloading unnecessary data
* Making many API calls for one screen
* Increased latency
* Wasted bandwidth
* Difficult API evolution

These problems motivated Facebook to create **GraphQL**.

---

# 2. Why REST Was Successful

REST became popular because it is:

* Easy to understand
* Built on HTTP
* Resource-oriented
* Supported by almost every framework
* Simple to cache
* Easy to test

Example REST endpoints:

```text
GET    /users
GET    /users/1
POST   /users
PUT    /users/1
DELETE /users/1
```

Each endpoint represents a specific resource.

---

# 3. Limitations of REST

As applications grow, REST APIs often face these challenges:

```text
REST Problems
      │
      ├── Over-fetching
      ├── Under-fetching
      ├── Multiple API calls
      ├── API Versioning
      ├── Large payloads
      ├── Slow mobile networks
      └── High battery usage
```

Let's examine each one.

---

# 4. Over-Fetching

## What is Over-Fetching?

Over-fetching happens when the server returns **more data than the client actually needs**.

Imagine the database contains:

```text
User
-------------------------
id
name
email
phone
address
birthday
profilePicture
bio
followers
following
posts
createdDate
lastLogin
```

Your Android profile screen only needs:

* name
* profilePicture

REST request:

```http
GET /users/101
```

Server response:

```json
{
  "id":101,
  "name":"John",
  "email":"john@gmail.com",
  "phone":"9876543210",
  "address":"New York",
  "birthday":"1995-06-10",
  "profilePicture":"profile.jpg",
  "bio":"Android Developer",
  "followers":450,
  "following":210,
  "posts":120,
  "createdDate":"2021-01-01",
  "lastLogin":"2025-07-15"
}
```

The app only uses:

```text
✓ name
✓ profilePicture
```

Everything else is downloaded but ignored.

### Visual Representation

```text
Server sends:

✔ name
✔ profilePicture
✖ email
✖ phone
✖ address
✖ birthday
✖ followers
✖ following
✖ posts
✖ lastLogin
```

This wastes:

* Network bandwidth
* Mobile data
* Processing time
* Memory

---

## Android Example

Suppose your `RecyclerView` only displays:

```text
John

(Profile Image)
```

Downloading:

```text
email
phone
bio
followers
address
birthday
```

adds no value to that screen.

---

# 5. Under-Fetching

## What is Under-Fetching?

Under-fetching occurs when **one API doesn't provide enough information**, forcing the client to make additional requests.

Example:

User Profile Screen

Needs:

```text
User Information

Recent Posts

Followers Count

Following Count

Profile Picture
```

REST endpoints:

```http
GET /users/101

GET /users/101/posts

GET /users/101/followers

GET /users/101/following
```

The Android app must call four different APIs.

---

### Flow

```text
Android

   │

GET /users

   │

GET /posts

   │

GET /followers

   │

GET /following
```

More requests mean:

* More waiting
* More battery usage
* More chances for failure

---

# 6. Multiple Network Requests

Suppose you're building an e-commerce home screen.

It displays:

* User
* Categories
* Products
* Offers
* Cart Count

REST may require:

```http
GET /users/1

GET /categories

GET /products

GET /offers

GET /cart
```

Five separate network requests.

### Timeline

```text
Request 1
------------>

Request 2
------------>

Request 3
------------>

Request 4
------------>

Request 5
------------>
```

Each request has:

* DNS lookup
* TCP connection (or reuse)
* TLS negotiation (when applicable)
* Request processing
* Response

Even with connection reuse, each additional request adds overhead and latency.

---

# 7. N+1 Problem (Client Perspective)

Imagine a social media feed.

First API:

```http
GET /posts
```

Response:

```json
[
 { "id":1, "userId":10 },
 { "id":2, "userId":20 },
 { "id":3, "userId":30 }
]
```

The UI also needs each author's name.

Now Android must call:

```http
GET /users/10

GET /users/20

GET /users/30
```

Total:

```text
1 request

+

3 requests

=

4 requests
```

If there are 100 posts:

```text
1

+

100

=

101 requests
```

This is commonly referred to as an **N+1 request problem** from the client's perspective.

> **Note:** There is also a server-side N+1 problem inside GraphQL resolvers, which we'll cover in a later chapter.

---

# 8. API Versioning

REST APIs often introduce new versions when breaking changes are needed.

Example:

```text
/api/v1/users

/api/v2/users

/api/v3/users
```

Problems:

* Multiple versions to maintain
* Larger codebase
* Old clients continue using old versions
* Documentation becomes harder

Android developers sometimes have to support multiple API versions simultaneously.

---

# 9. Mobile Performance Issues

Every unnecessary API call affects mobile devices.

## More Bandwidth

Example:

Needed:

```text
20 KB
```

Downloaded:

```text
250 KB
```

230 KB is wasted.

---

## More Battery Usage

Each network request activates network hardware.

More requests generally mean:

* Higher battery consumption
* Longer radio usage
* More background work

---

## Slower UI

Imagine a profile screen waiting for four API responses.

```text
Loading...

User

Loading...

Posts

Loading...

Followers
```

The user may see partial or delayed content.

---

## Higher Latency

Suppose each request takes:

```text
300 ms
```

Four sequential requests:

```text
300

+

300

+

300

+

300

=

1200 ms
```

The user waits over a second.

---

# 10. Real Android Example

## Instagram Profile Screen

Needs:

```text
Profile Photo

Username

Bio

Followers

Following

Posts

Stories
```

REST may require:

```http
GET /profile

GET /posts

GET /stories

GET /followers

GET /following
```

Five requests.

GraphQL can fetch all this in a single operation (we'll see this in the next chapter).

---

# 11. How GraphQL Solves These Problems

Instead of multiple endpoints:

```text
/users

/posts

/followers

/comments
```

GraphQL typically uses one endpoint:

```http
POST /graphql
```

The client asks only for the fields it needs.

Example:

```graphql
query {
  user(id:101){
    name
    profilePicture
    followers
    posts(limit:5){
      title
    }
  }
}
```

The server responds with only the requested data.

```json
{
  "data": {
    "user": {
      "name": "John",
      "profilePicture": "profile.jpg",
      "followers": 450,
      "posts": [
        { "title": "Learning GraphQL" },
        { "title": "Android Tips" }
      ]
    }
  }
}
```

Advantages:

* One request
* No over-fetching
* Reduced under-fetching
* Smaller payloads
* Better suited for mobile apps

---

# REST vs GraphQL Comparison

| Feature           | REST           | GraphQL                |
| ----------------- | -------------- | ---------------------- |
| Endpoints         | Many           | Usually one            |
| Data selection    | Server decides | Client decides         |
| Over-fetching     | Common         | Minimized              |
| Under-fetching    | Common         | Reduced                |
| Multiple requests | Often          | Usually fewer          |
| API versioning    | Often required | Schema evolves instead |
| Mobile efficiency | Can vary       | Often better           |

---

# 12. Key Takeaways

* REST is simple and widely used, but it has limitations for data-heavy applications.
* Over-fetching wastes bandwidth by returning unnecessary fields.
* Under-fetching forces clients to make additional API calls.
* Multiple network requests increase latency and battery usage.
* Versioning can complicate API maintenance.
* These challenges led Facebook to develop GraphQL.

---

# 13. Interview Questions

### 1. What is over-fetching?

**Answer:** Over-fetching occurs when the server returns more data than the client needs. This wastes bandwidth, memory, and processing time.

---

### 2. What is under-fetching?

**Answer:** Under-fetching occurs when one API response doesn't include all the required data, forcing the client to make additional API requests.

---

### 3. Why are multiple API calls a problem in Android?

**Answer:**

* Increased latency
* Higher battery usage
* More network overhead
* More complex error handling
* Slower user experience

---

### 4. Why is API versioning challenging?

**Answer:** Maintaining multiple API versions increases development effort, documentation complexity, and server maintenance costs while older clients may continue relying on outdated versions.

---

### 5. Why did Facebook create GraphQL?

**Answer:** Facebook created GraphQL to address REST limitations such as over-fetching, under-fetching, and the need for multiple network requests, especially in mobile applications with varying network conditions.

---

### 6. Does GraphQL completely replace REST?

**Answer:** No. REST remains an excellent choice for many APIs. GraphQL is particularly useful when clients need flexible data fetching or when multiple related resources are required in a single request. Many organizations use both REST and GraphQL depending on the use case.

---

# What's Next?

**Chapter 3 – Introduction to GraphQL**

In the next chapter, we'll answer:

* What exactly is GraphQL?
* How does it work internally?
* Why is it called a "query language"?
* What is a GraphQL server?
* Why does GraphQL usually expose a single endpoint?
* How does an Android app communicate with a GraphQL server using Apollo Kotlin?

From this point onward, we'll begin moving from REST concepts into the GraphQL ecosystem itself.
