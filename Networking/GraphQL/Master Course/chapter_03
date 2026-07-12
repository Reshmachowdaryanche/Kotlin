# Chapter 3 – Introduction to GraphQL

## GraphQL for Android Developers

> **Learning Goal**
>
> By the end of this chapter, you will understand:
>
> * What GraphQL is
> * Why Facebook created GraphQL
> * How GraphQL works internally
> * Why GraphQL is called a *query language*
> * GraphQL Architecture
> * GraphQL Request Lifecycle
> * GraphQL vs REST
> * Why GraphQL is popular for Android applications

---

# Table of Contents

1. What is GraphQL?
2. Why Facebook Created GraphQL
3. Why is it Called a Query Language?
4. GraphQL Architecture
5. How GraphQL Works
6. GraphQL Components
7. GraphQL Request Flow
8. GraphQL vs REST
9. Why Android Developers Like GraphQL
10. Advantages & Disadvantages
11. Summary
12. Interview Questions

---

# 1. What is GraphQL?

GraphQL is a **query language for APIs** and a **runtime for executing those queries**.

Let's break that sentence into two parts.

### Part 1 – Query Language

A **query language** is simply a language that lets you ask for data.

Examples:

* SQL → Queries databases
* GraphQL → Queries APIs

SQL example:

```sql
SELECT name, email
FROM Users
WHERE id = 10;
```

GraphQL example:

```graphql
query {
  user(id: 10) {
    name
    email
  }
}
```

Notice something interesting.

In SQL, we ask a **database**.

In GraphQL, we ask an **API**.

---

### Part 2 – Runtime

GraphQL is also a runtime.

The runtime is responsible for:

* Reading your query
* Validating it
* Finding the requested fields
* Executing business logic
* Returning the result

Think of the runtime as the **engine** that understands GraphQL.

---

# Definition

GraphQL is:

> A language that allows clients to request exactly the data they need from an API.

This is the biggest idea in GraphQL.

---

# 2. Why Facebook Created GraphQL

Around 2012, Facebook faced several challenges.

Imagine opening the Facebook mobile app.

The Home Screen needs:

* User information
* Stories
* Feed posts
* Comments
* Likes
* Advertisements
* Notifications
* Friend suggestions

Using REST, the app might call:

```text
GET /user

GET /stories

GET /posts

GET /comments

GET /notifications

GET /friends
```

Many API requests.

---

## Mobile Networks Were Slow

At that time,

* 2G
* 3G

were common.

Bandwidth was limited.

Every extra request increased loading time.

---

## Different Devices Needed Different Data

Desktop Facebook

Needs

```text
Profile

Posts

Videos

Ads

Friend Suggestions

Notifications

Marketplace
```

Mobile Facebook

Needs

```text
Profile

Posts

Notifications
```

Why should both download exactly the same data?

They shouldn't.

Facebook wanted clients to request **only what they need**.

This became one of the main motivations behind GraphQL.

---

# 3. Why is it Called a Query Language?

Many beginners think:

> "GraphQL is an API."

Not exactly.

GraphQL is a **language** for describing the data you want.

Imagine ordering pizza.

Instead of saying:

> "Give me Pizza #5"

You say:

```text
Large Pizza

Extra Cheese

No Onion

Mushrooms

Thin Crust
```

You describe exactly what you want.

GraphQL works the same way.

Instead of requesting an endpoint, you describe the fields you need.

Example:

```graphql
query {
  user(id: 10) {
    name
    profilePicture
  }
}
```

You are literally describing your desired response.

---

# 4. GraphQL Architecture

Let's see the complete architecture.

```text
+----------------------+
|    Android App       |
+----------+-----------+
           |
           | GraphQL Query
           |
           ▼
+----------------------+
|   Apollo Client      |
+----------+-----------+
           |
           |
           ▼
+----------------------+
|  GraphQL Server      |
+----------+-----------+
           |
           ▼
+----------------------+
|   GraphQL Runtime    |
+----------+-----------+
           |
           ▼
+----------------------+
|      Resolvers       |
+----------+-----------+
           |
           ▼
+----------------------+
| Database / Services  |
+----------------------+
```

Let's understand every component.

---

## Android App

Responsible for:

* UI
* User interactions
* Displaying data

The Android app never directly accesses the database.

---

## Apollo Client

Apollo is the GraphQL client for Android.

Responsibilities:

* Send GraphQL queries
* Download responses
* Convert JSON into Kotlin objects
* Handle caching
* Handle errors

Apollo is similar to what Retrofit is for REST APIs.

---

## GraphQL Server

The GraphQL server receives every GraphQL request.

Usually there is one endpoint:

```text
POST /graphql
```

Unlike REST:

```text
/users

/posts

/comments

/orders
```

GraphQL commonly uses a single endpoint.

---

## GraphQL Runtime

The runtime is the heart of GraphQL.

Responsibilities:

* Parse the query
* Validate fields against the schema
* Execute resolvers
* Build the response

Without the runtime, GraphQL cannot execute requests.

---

## Resolvers

Resolvers fetch actual data.

Example:

```graphql
query {
    user(id: 10){
        name
    }
}
```

The resolver for `user` may execute:

```sql
SELECT name
FROM Users
WHERE id = 10;
```

The client doesn't know or care where the data comes from.

---

## Database

Could be:

* MySQL
* PostgreSQL
* MongoDB
* Firebase
* Another REST API
* Another GraphQL service
* Microservices

GraphQL is independent of the underlying data source.

---

# 5. How GraphQL Works

Let's follow a request from beginning to end.

Suppose Android needs:

* Name
* Email

The app sends:

```graphql
query {
  user(id: 10) {
    name
    email
  }
}
```

---

## Step 1

Android sends the request.

```text
Android

↓

GraphQL Query
```

---

## Step 2

Server receives it.

```text
GraphQL Server

↓

Reads Query
```

---

## Step 3

Runtime validates.

Questions it asks:

* Does `user` exist?
* Does `name` exist?
* Does `email` exist?

If not,

Error.

---

## Step 4

Resolver executes.

```text
Resolver

↓

Database Query
```

---

## Step 5

Database returns data.

```text
John

john@gmail.com
```

---

## Step 6

GraphQL builds JSON.

```json
{
  "data": {
    "user": {
      "name": "John",
      "email": "john@gmail.com"
    }
  }
}
```

---

## Complete Flow

```text
Android App
      │
      ▼
Apollo Client
      │
      ▼
GraphQL Server
      │
      ▼
Runtime
      │
      ▼
Resolver
      │
      ▼
Database
      │
      ▼
Resolver
      │
      ▼
Runtime
      │
      ▼
Apollo
      │
      ▼
Android UI
```

This lifecycle happens for every GraphQL operation.

---

# 6. GraphQL Components

GraphQL has three main operation types.

```text
GraphQL

│

├── Query

├── Mutation

└── Subscription
```

### Query

Reads data.

Example:

```graphql
query {
    users {
        name
    }
}
```

---

### Mutation

Changes data.

```graphql
mutation {
    createUser(name:"John"){
        id
    }
}
```

---

### Subscription

Receives real-time updates.

```graphql
subscription{
   newMessage{
      text
   }
}
```

Used in:

* Chat apps
* Stock prices
* Notifications
* Live sports

---

# 7. GraphQL Request Flow

Suppose you're building a shopping app.

The Product screen needs:

* Product name
* Price
* Images
* Reviews
* Seller

GraphQL query:

```graphql
query {
  product(id: 1) {
    name
    price
    images
    reviews {
      rating
      comment
    }
    seller {
      name
    }
  }
}
```

Everything comes back in one structured response.

```text
Android

↓

GraphQL Query

↓

GraphQL Server

↓

Database

↓

JSON Response

↓

Apollo Models

↓

RecyclerView / Compose UI
```

---

# 8. GraphQL vs REST

Suppose the Profile screen needs:

* Name
* Followers
* Recent Posts

REST:

```text
GET /users/10

GET /users/10/followers

GET /users/10/posts
```

Three requests.

GraphQL:

```graphql
query {
  user(id: 10) {
    name
    followers
    posts(limit: 5) {
      title
    }
  }
}
```

One request.

---

### Response Comparison

REST:

```json
{
  "id": 10,
  "name": "John",
  "email": "john@example.com",
  "phone": "123456789",
  "address": "New York",
  "followers": 100
}
```

If you only need the name, you still receive everything.

GraphQL:

```graphql
query {
  user(id: 10) {
    name
  }
}
```

Response:

```json
{
  "data": {
    "user": {
      "name": "John"
    }
  }
}
```

Only the requested field is returned.

---

# 9. Why Android Developers Like GraphQL

GraphQL is especially useful for mobile applications because it allows the client to fetch exactly what a screen requires.

Benefits include:

* Fewer API calls
* Smaller payloads
* Less bandwidth usage
* Better performance on slower networks
* Strongly typed schema
* Automatic model generation with Apollo Kotlin
* Easier evolution of APIs without creating multiple versions

For example, if one screen needs only a user's name and avatar while another needs the full profile, each screen can request only the fields it needs.

---

# 10. Advantages & Disadvantages

## Advantages

* Flexible queries
* Single endpoint
* Reduces over-fetching
* Reduces under-fetching
* Strong typing through schemas
* Great tooling and introspection
* Well suited for modern mobile apps

## Disadvantages

* More complex server implementation
* Query optimization requires careful design
* Caching strategies can be more involved than REST
* Developers need to learn GraphQL concepts such as schemas and resolvers

---

# 11. Summary

* GraphQL is both a **query language** and a **runtime**.
* Clients describe the exact data they need.
* The GraphQL runtime validates and executes requests.
* A GraphQL server commonly exposes a single endpoint.
* Resolvers fetch data from databases or other services.
* Apollo Kotlin makes GraphQL easy to use in Android applications.

---

# 12. Interview Questions

### 1. What is GraphQL?

**Answer:** GraphQL is a query language for APIs and a runtime that executes those queries. It allows clients to request exactly the data they need.

---

### 2. Why did Facebook create GraphQL?

**Answer:** Facebook created GraphQL to address limitations of REST, such as over-fetching, under-fetching, and the need for multiple network requests, especially for mobile applications.

---

### 3. Why is GraphQL called a query language?

**Answer:** Because clients write queries that describe the exact data they want, similar to how SQL queries describe the data to retrieve from a database.

---

### 4. What is a GraphQL runtime?

**Answer:** The runtime parses incoming queries, validates them against the schema, executes the appropriate resolvers, and builds the response.

---

### 5. What is a resolver?

**Answer:** A resolver is a server-side function responsible for fetching the data for a particular field in a GraphQL query. It may retrieve data from a database, another API, or another service.

---

### 6. Why does GraphQL usually use a single endpoint?

**Answer:** Unlike REST, where different resources have different URLs, GraphQL sends all operations to a single endpoint (commonly `POST /graphql`). The query itself specifies what data should be returned.

---

## Next Chapter

**Chapter 4 – GraphQL Basics**

We'll start writing actual GraphQL operations and learn:

* GraphQL syntax
* Query structure
* Mutations
* Subscriptions
* Request and response format
* GraphQL Playground
* GraphiQL
* Apollo Studio
* Your first GraphQL query from an Android app
