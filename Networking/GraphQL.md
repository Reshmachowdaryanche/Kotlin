# What is GraphQL?

**GraphQL** is a **query language for APIs** and a **runtime** for executing those queries.

It was developed by Meta Platforms (formerly Facebook) in 2012 and open-sourced in 2015.

Unlike REST, where the server exposes multiple endpoints, GraphQL usually exposes **a single endpoint**, and the client specifies exactly what data it wants.

Example endpoint:

```
POST /graphql
```

---

# Why was GraphQL created?

REST APIs often have problems like:

* Over-fetching data
* Under-fetching data
* Multiple network requests
* Different endpoints for related resources

Example:

Suppose you need:

* User name
* User profile picture
* Last 5 posts

With REST:

```
GET /users/10

GET /users/10/posts

GET /posts/101
GET /posts/102
```

Multiple API calls.

With GraphQL:

```graphql
query {
  user(id: 10) {
    name
    profilePicture
    posts(limit:5){
      title
    }
  }
}
```

One request.

---

# REST vs GraphQL

| REST                                       | GraphQL                    |
| ------------------------------------------ | -------------------------- |
| Multiple endpoints                         | Usually one endpoint       |
| Server decides response                    | Client decides response    |
| Can over-fetch                             | Fetch only required fields |
| Versioning required                        | Usually no versioning      |
| Uses HTTP methods (GET, POST, PUT, DELETE) | Mostly POST                |
| Fixed response                             | Flexible response          |

---

# Example

Suppose server contains

```
User

id
name
email
age
phone
address
profilePic
```

In REST

```
GET /users/10
```

returns

```json
{
 "id":10,
 "name":"John",
 "email":"john@gmail.com",
 "age":30,
 "phone":"12345",
 "address":"USA",
 "profilePic":"..."
}
```

Even if Android only needs

```
name
profilePic
```

all fields come.

GraphQL:

```graphql
query {
  user(id:10){
    name
    profilePic
  }
}
```

Response

```json
{
  "data": {
    "user": {
      "name": "John",
      "profilePic": "..."
    }
  }
}
```

Only requested fields are returned.

---

# GraphQL Components

## 1. Query

Used to fetch data.

```graphql
query {
  user(id:1){
    id
    name
  }
}
```

Equivalent to

```
GET
```

---

## 2. Mutation

Used to change data.

```graphql
mutation {
  createUser(name:"John"){
      id
      name
  }
}
```

Equivalent to

```
POST
PUT
DELETE
```

---

## 3. Subscription

Used for real-time updates.

Example:

```
Chat messages

Stock prices

Live cricket score

Notifications
```

Whenever server changes data, clients automatically receive updates.

---

# GraphQL Schema

Every GraphQL server exposes a schema.

Example

```graphql
type User{
   id: ID!
   name: String!
   email: String
}
```

Schema defines

* objects
* fields
* relationships
* types
* available queries
* mutations

Think of it as a contract between client and server.

---

# GraphQL Types

Scalar types

```
String

Int

Float

Boolean

ID
```

Custom types

```graphql
type User{
   id:ID
   name:String
}
```

Lists

```graphql
posts:[Post]
```

Non-null

```
String!
```

means value cannot be null.

---

# Variables

Instead of hardcoding values

```graphql
query{
 user(id:10){
   name
 }
}
```

Use variables

```graphql
query GetUser($id:ID!){
   user(id:$id){
      name
   }
}
```

Variables

```json
{
 "id":10
}
```

---

# Fragments

Avoid repeating fields.

Without fragment

```graphql
user{
 id
 name
 email
}

friend{
 id
 name
 email
}
```

With fragment

```graphql
fragment UserFields on User{
 id
 name
 email
}

user{
 ...UserFields
}

friend{
 ...UserFields
}
```

---

# Aliases

Rename fields.

```graphql
query{

 first:user(id:1){
   name
 }

 second:user(id:2){
   name
 }

}
```

---

# GraphQL Response

```json
{
  "data": {
    "user": {
      "name": "John"
    }
  }
}
```

If error

```json
{
  "errors":[
      {
         "message":"User not found"
      }
  ]
}
```

Notice that GraphQL may return both `data` and `errors` together if some fields succeed while others fail.

---

# GraphQL in Android

The most popular library is [Apollo Kotlin](https://www.apollographql.com/docs/kotlin?utm_source=chatgpt.com).

Flow:

```
Android App

↓

Apollo Client

↓

GraphQL Server

↓

Response

↓

Kotlin Models
```

---

# Apollo Kotlin Example

GraphQL query

```graphql
query GetUser($id: ID!) {
  user(id: $id) {
    id
    name
    email
  }
}
```

Calling it

```kotlin
val response = apolloClient.query(
    GetUserQuery(id)
).execute()

val user = response.data?.user
```

Apollo generates Kotlin models automatically from your GraphQL schema and queries, providing type safety and reducing manual JSON parsing.

---

# Advantages

✅ Fetch only required fields

✅ Single endpoint

✅ Less network usage

✅ Strong typing

✅ Auto-generated models

✅ Better for mobile apps

✅ Easier API evolution (often without versioning)

---

# Disadvantages

❌ More complex server implementation

❌ Query complexity can affect performance

❌ Caching is more challenging than REST because everything typically goes through one endpoint

❌ Learning curve

---

# Android Interview Questions

### 1. What is GraphQL?

A query language for APIs where the client specifies exactly what data it needs.

---

### 2. Difference between REST and GraphQL?

Main points:

* REST has multiple endpoints.
* GraphQL usually has one endpoint.
* GraphQL avoids over-fetching.
* Client controls the response.
* REST commonly uses API versioning; GraphQL often evolves the schema instead.

---

### 3. Why is GraphQL good for Android?

* Saves bandwidth
* Fewer API calls
* Better battery usage
* Faster screens
* Strongly typed models
* Automatic code generation

---

### 4. What are Query, Mutation, and Subscription?

* Query → Read data
* Mutation → Create/update/delete data
* Subscription → Real-time updates

---

### 5. What is Apollo Kotlin?

A GraphQL client for Kotlin/Android that:

* Executes GraphQL operations
* Generates Kotlin models
* Supports normalized caching
* Integrates well with coroutines and Flow

---

### 6. Does GraphQL replace REST?

No. Both are API styles. Many companies use both:
