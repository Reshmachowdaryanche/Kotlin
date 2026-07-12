# Chapter 4 – GraphQL Basics (Queries, Mutations & Subscriptions)

## GraphQL for Android Developers

> **Learning Goal**
>
> By the end of this chapter, you will understand:
>
> * The basic syntax of GraphQL
> * GraphQL operations
> * Query structure
> * Mutation structure
> * Subscription structure
> * Request and response format
> * GraphQL endpoint
> * GraphQL Playground
> * How Android sends GraphQL requests
> * How Apollo Kotlin executes GraphQL operations

---

# Table of Contents

1. Introduction
2. Anatomy of a GraphQL Operation
3. GraphQL Endpoint
4. GraphQL Request Format
5. GraphQL Response Format
6. Queries
7. Mutations
8. Subscriptions
9. Operation Names
10. Comments
11. GraphQL Playground & GraphiQL
12. Android Request Flow
13. Complete Example
14. Summary
15. Interview Questions

---

# 1. Introduction

In the previous chapter, we learned:

* What GraphQL is
* Why it was created
* How it works internally

Now it's time to write actual GraphQL code.

GraphQL has only **three operation types**:

```text
GraphQL Operations

        │

 ┌──────┼─────────┐

 │      │         │

Query Mutation Subscription

(Read) (Write) (Real-time)
```

Almost every GraphQL application is built using these three operations.

Think of them like CRUD operations.

| CRUD      | GraphQL      |
| --------- | ------------ |
| Read      | Query        |
| Create    | Mutation     |
| Update    | Mutation     |
| Delete    | Mutation     |
| Real-time | Subscription |

---

# 2. Anatomy of a GraphQL Operation

Let's look at a simple query.

```graphql
query {
    user(id: 10) {
        id
        name
        email
    }
}
```

Every GraphQL operation has three parts.

```text
query

↓

Field

↓

Requested Fields
```

Breaking it down:

```graphql
query {
    user(id:10){
        id
        name
        email
    }
}
```

### Part 1

```graphql
query
```

The operation type.

---

### Part 2

```graphql
user
```

The field (or entry point) you're asking the server to resolve.

---

### Part 3

```graphql
id
name
email
```

The fields you want returned.

This is called a **selection set**.

---

# 3. GraphQL Endpoint

REST:

```text
/users

/products

/orders

/comments

/profile
```

Many endpoints.

GraphQL:

```text
POST /graphql
```

Only one endpoint.

A common question is:

> **How does the server know what data to return if every request goes to the same URL?**

The answer is:

The **GraphQL query itself** tells the server what data is needed.

Example:

```graphql
query {
    user(id:10){
        name
    }
}
```

Another request:

```graphql
query {
    products{
        title
        price
    }
}
```

Same endpoint.

Different queries.

Different responses.

---

# 4. GraphQL Request Format

Unlike REST,

where

```text
GET /users
```

identifies the resource,

GraphQL sends the query inside the HTTP request body.

Example HTTP request:

```http
POST /graphql
Content-Type: application/json
```

Body:

```json
{
  "query": "query { user(id:10){ name email } }"
}
```

Notice that the GraphQL query is just a string inside JSON.

---

## Android Example

Apollo sends something similar internally:

```kotlin
apolloClient.query(GetUserQuery(10))
```

Apollo converts it into an HTTP request automatically.

Developers usually never build the JSON manually.

---

# 5. GraphQL Response Format

GraphQL responses always have a predictable structure.

Successful response:

```json
{
  "data": {
    "user": {
      "name": "John"
    }
  }
}
```

Notice the top-level `data` field.

---

## Error Response

```json
{
  "errors": [
    {
      "message": "User not found"
    }
  ]
}
```

---

## Partial Response

One unique feature of GraphQL is that it can return both data and errors.

```json
{
  "data": {
    "user": {
      "name": "John",
      "email": null
    }
  },
  "errors": [
    {
      "message": "Email service unavailable"
    }
  ]
}
```

Some fields succeeded.

Some failed.

REST usually returns either success **or** failure.

GraphQL can return partial success.

---

# 6. Queries

Queries are used to **read data**.

Imagine the database contains:

```text
User

id

name

email

age
```

Query:

```graphql
query {
    user(id:10){
        name
        email
    }
}
```

Response:

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

Notice

Age wasn't requested.

Age wasn't returned.

---

## Nested Queries

One of GraphQL's biggest strengths is nested data.

Suppose

User

has

Posts.

Schema:

```text
User

├── name

└── posts
```

Query:

```graphql
query {
    user(id:10){
        name

        posts{
            title
        }
    }
}
```

Response:

```json
{
  "data": {
    "user": {
      "name": "John",
      "posts": [
        {
          "title": "GraphQL Basics"
        },
        {
          "title": "Apollo Kotlin"
        }
      ]
    }
  }
}
```

One request.

Hierarchical response.

---

# 7. Mutations

Queries only **read** data.

To modify data,

use

```graphql
mutation
```

Example:

```graphql
mutation {
    createUser(
        name:"John"
        email:"john@gmail.com"
    ){
        id
        name
    }
}
```

The server creates a user.

Response:

```json
{
  "data": {
    "createUser": {
      "id": 101,
      "name": "John"
    }
  }
}
```

---

## Update Example

```graphql
mutation {
    updateUser(
        id:10
        name:"Peter"
    ){
        id
        name
    }
}
```

---

## Delete Example

```graphql
mutation {
    deleteUser(id:10)
}
```

Many APIs return a boolean or a status object:

```json
{
  "data": {
    "deleteUser": true
  }
}
```

---

# 8. Subscriptions

Subscriptions provide **real-time data**.

Examples:

* Chat
* Notifications
* Stock prices
* Cricket scores
* Ride tracking

Instead of repeatedly asking the server:

```text
Any new messages?

Any new messages?

Any new messages?
```

The client subscribes once.

```graphql
subscription {
    newMessage{
        text
        sender
    }
}
```

Whenever a new message arrives:

Server pushes:

```json
{
  "data": {
    "newMessage": {
      "text": "Hello",
      "sender": "John"
    }
  }
}
```

No polling required.

---

# 9. Operation Names

Operations can have names.

Unnamed:

```graphql
query{
    users{
        name
    }
}
```

Named:

```graphql
query GetUsers{
    users{
        name
    }
}
```

Naming operations makes debugging easier and is recommended in production.

The same applies to mutations:

```graphql
mutation CreateUser{
    createUser(
        name:"John"
    ){
        id
    }
}
```

And subscriptions:

```graphql
subscription NewMessages{
    newMessage{
        text
    }
}
```

---

# 10. Comments

GraphQL supports comments using `#`.

```graphql
# Fetch logged-in user
query GetProfile{

    # User details
    user(id:10){
        name
    }
}
```

Comments are ignored by the server.

---

# 11. GraphQL Playground & GraphiQL

Before integrating GraphQL into Android, developers often test queries using graphical tools.

### GraphQL Playground

Features:

* Write queries
* Execute operations
* View responses
* Documentation explorer
* Auto-completion

---

### GraphiQL

Similar capabilities:

* Query editor
* Schema explorer
* Documentation
* History

---

### Apollo Studio Explorer

Apollo provides a web interface where you can:

* Explore schemas
* Test queries
* Inspect responses
* View documentation
* Monitor GraphQL APIs (depending on your setup)

These tools are invaluable when learning or debugging GraphQL.

---

# 12. Android Request Flow

Imagine a profile screen.

```text
Profile Screen

↓

ViewModel

↓

Repository

↓

Apollo Client

↓

GraphQL Query

↓

POST /graphql

↓

GraphQL Server

↓

Response

↓

Apollo Models

↓

UI Updates
```

Unlike REST,

the ViewModel doesn't manually parse JSON.

Apollo generates strongly typed Kotlin models from your GraphQL schema and queries.

Example:

```kotlin
val response = apolloClient.query(
    GetUserQuery("10")
).execute()

val user = response.data?.user

println(user?.name)
```

Notice

No `JSONObject`.

No Gson model mapping.

No Moshi model mapping.

Apollo handles it.

---

# 13. Complete Example

Let's put everything together.

Imagine a social media app.

We want:

* User name
* Followers
* Latest posts

GraphQL Query:

```graphql
query GetProfile{
    user(id:10){
        name
        followers

        posts(limit:3){
            title
        }
    }
}
```

Server response:

```json
{
  "data": {
    "user": {
      "name": "John",
      "followers": 520,
      "posts": [
        {
          "title": "GraphQL Basics"
        },
        {
          "title": "Android Architecture"
        },
        {
          "title": "Apollo Kotlin"
        }
      ]
    }
  }
}
```

Android code:

```kotlin
val response = apolloClient.query(
    GetProfileQuery("10")
).execute()

val user = response.data?.user

println(user?.name)

println(user?.followers)

user?.posts?.forEach {
    println(it.title)
}
```

The generated `GetProfileQuery` class and nested model classes come directly from the GraphQL schema and the `GetProfile.graphql` file.

---

# 14. Summary

* GraphQL has three operation types: Query, Mutation, and Subscription.
* A GraphQL API commonly exposes a single endpoint such as `POST /graphql`.
* The query in the request body tells the server what data to return.
* Queries read data.
* Mutations modify data.
* Subscriptions receive real-time updates.
* Responses typically contain a `data` field and may also include an `errors` field.
* Apollo Kotlin generates type-safe Kotlin classes, reducing manual JSON parsing.

---

# 15. Interview Questions

### 1. What are the three GraphQL operations?

**Answer:**

* Query → Read data
* Mutation → Create, update, or delete data
* Subscription → Receive real-time updates

---

### 2. Why does GraphQL use a single endpoint?

**Answer:** The endpoint receives all GraphQL operations. The query itself specifies what data should be returned, so multiple resource-specific URLs are not needed.

---

### 3. Where is the GraphQL query sent?

**Answer:** The query is usually included in the body of an HTTP `POST` request as part of a JSON payload.

---

### 4. What is a selection set?

**Answer:** A selection set is the list of fields inside a query that specifies exactly which data should be returned.

Example:

```graphql
query {
  user(id: 10) {
    name
    email
  }
}
```

Here, `name` and `email` form the selection set.

---

### 5. Can GraphQL return both data and errors?

**Answer:** Yes. GraphQL supports partial success. A response may include successfully resolved data along with an `errors` array describing any fields that failed.

---

### 6. How does Apollo Kotlin simplify Android development?

**Answer:** Apollo Kotlin:

* Generates type-safe Kotlin models from the schema and queries.
* Executes GraphQL operations.
* Parses responses automatically.
* Integrates with coroutines, Flow, and caching features.

---

# 📌 What's Next?

**Chapter 5 – GraphQL Schema (Complete Deep Dive)**

This is one of the most important chapters in GraphQL. We'll explore:

* What a schema is
* Why every GraphQL server needs a schema
* Schema Definition Language (SDL)
* Object types
* Scalar types
* Non-null (`!`)
* Lists (`[]`)
* Input types
* Enums
* Interfaces
* Unions
* Custom scalars
* Directives
* Root types (`Query`, `Mutation`, `Subscription`)
* Relationships between types
* How Apollo Kotlin uses the schema to generate Kotlin classes

Understanding the schema is the foundation for writing correct GraphQL queries and building robust Android applications.
