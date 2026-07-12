I assume you meant **Chapter 5**.

---

# Chapter 5 – GraphQL Schema (Deep Dive)

## GraphQL for Android Developers

> **Learning Goal**
>
> By the end of this chapter, you will understand:
>
> * What a GraphQL Schema is
> * Why schemas are important
> * Schema Definition Language (SDL)
> * Object Types
> * Scalar Types
> * Non-Null Types
> * Lists
> * Input Types
> * Enums
> * Interfaces
> * Unions
> * Custom Scalars
> * Directives
> * Root Types
> * How Apollo Kotlin uses the schema

---

# Table of Contents

1. What is a GraphQL Schema?
2. Why Do We Need a Schema?
3. Schema Definition Language (SDL)
4. Scalar Types
5. Object Types
6. Relationships Between Types
7. Non-Null Types
8. Lists
9. Input Types
10. Enums
11. Interfaces
12. Unions
13. Custom Scalars
14. Directives
15. Root Types
16. Complete Schema Example
17. How Apollo Uses the Schema
18. Summary
19. Interview Questions

---

# 1. What is a GraphQL Schema?

A **GraphQL Schema** is the **blueprint** of a GraphQL API.

Think of it as the API's contract.

It defines:

* What data exists
* What operations clients can perform
* The relationships between data
* Which fields are required
* Which fields are optional

Without a schema, a GraphQL server doesn't know:

* Which queries are valid
* Which mutations are allowed
* What data types exist

---

## Real-Life Analogy

Imagine building a house.

Before construction begins, an architect creates a blueprint.

```text
Blueprint

↓

Rooms

Doors

Windows

Kitchen

Bathroom
```

The builder follows the blueprint.

Similarly,

```text
GraphQL Schema

↓

Users

Products

Orders

Comments

Reviews
```

The GraphQL server follows the schema.

---

# 2. Why Do We Need a Schema?

Suppose Android sends:

```graphql
query {
    user(id:1){
        name
        email
    }
}
```

How does the server know:

* Does `user` exist?
* Does `email` exist?
* Is `id` an Integer?
* Is `name` nullable?

It checks the schema.

Without a schema,

the server cannot validate the query.

---

# Schema Validation

Imagine the schema says:

```graphql
type User{
    id:ID!
    name:String!
}
```

Android sends:

```graphql
query{
    user(id:1){
        age
    }
}
```

The server immediately rejects it.

Error:

```text
Cannot query field "age" on type "User".
```

The schema prevents invalid queries before execution.

---

# 3. Schema Definition Language (SDL)

Schemas are written using the **Schema Definition Language (SDL)**.

Example:

```graphql
type User{
    id:ID!
    name:String!
    email:String!
}
```

Let's understand each part.

```graphql
type
```

Creates a new object type.

```graphql
User
```

The object's name.

Fields:

```graphql
id

name

email
```

Field types:

```graphql
ID!

String!

String!
```

---

# 4. Scalar Types

Scalars are the simplest data types.

GraphQL provides five built-in scalars.

| Scalar    | Description       | Example |
| --------- | ----------------- | ------- |
| `Int`     | Whole number      | 25      |
| `Float`   | Decimal number    | 12.5    |
| `String`  | Text              | "John"  |
| `Boolean` | True/False        | true    |
| `ID`      | Unique identifier | "1001"  |

---

## Example

```graphql
type Product{
    id:ID!
    name:String!
    price:Float!
    quantity:Int!
    available:Boolean!
}
```

Response:

```json
{
  "id":"101",
  "name":"Laptop",
  "price":59999.99,
  "quantity":20,
  "available":true
}
```

---

# 5. Object Types

Object types represent real-world entities.

Example:

```graphql
type User{
    id:ID!
    name:String!
    email:String!
}
```

Another object:

```graphql
type Post{
    id:ID!
    title:String!
}
```

Objects can reference other objects.

---

# 6. Relationships Between Types

Suppose one user has many posts.

```graphql
type User{
    id:ID!
    name:String!

    posts:[Post!]!
}
```

Post:

```graphql
type Post{
    id:ID!
    title:String!
}
```

Relationship:

```text
User

├── id

├── name

└── posts

     ├── title

     └── id
```

The client can navigate this relationship naturally:

```graphql
query{
    user(id:1){
        name

        posts{
            title
        }
    }
}
```

---

# 7. Non-Null Types (`!`)

This is one of GraphQL's most important features.

Without `!`

```graphql
name:String
```

The value may be:

```json
{
   "name":null
}
```

With `!`

```graphql
name:String!
```

The server guarantees that `name` will never be `null`.

If it cannot provide a value, GraphQL returns an error instead of silently returning `null`.

---

## Android Benefit

Apollo generates:

Nullable:

```kotlin
val name: String?
```

Non-null:

```kotlin
val name: String
```

The schema directly influences the generated Kotlin types, reducing unnecessary null checks.

---

# 8. Lists

Suppose one user has multiple posts.

```graphql
posts:[Post]
```

This means:

* The list may be `null`
* Items may also be `null`

GraphQL supports several combinations.

```graphql
[Post]
```

List nullable

Items nullable

---

```graphql
[Post]!
```

List cannot be null

Items may be null

---

```graphql
[Post!]
```

List nullable

Items cannot be null

---

```graphql
[Post!]!
```

Neither the list nor its items can be null.

This last form is commonly used because it gives clients stronger guarantees.

---

# 9. Input Types

Queries read data.

Mutations usually send data.

Instead of:

```graphql
createUser(
    name:String!
    email:String!
    age:Int!
)
```

GraphQL encourages grouping related values.

```graphql
input CreateUserInput{

    name:String!

    email:String!

    age:Int!
}
```

Mutation:

```graphql
type Mutation{

    createUser(
        input:CreateUserInput!
    ):User
}
```

Request:

```graphql
mutation{

    createUser(

        input:{
            name:"John"
            email:"john@gmail.com"
            age:25
        }

    ){

        id

        name
    }
}
```

Input types improve readability and make APIs easier to evolve.

---

# 10. Enums

Instead of free-form strings:

```graphql
status:String
```

Use an enum.

```graphql
enum Status{

    ACTIVE

    INACTIVE

    BLOCKED
}
```

Usage:

```graphql
type User{

    status:Status!
}
```

Response:

```json
{
    "status":"ACTIVE"
}
```

Enums prevent invalid values like `"Running"` or `"Unknown"` if those aren't supported.

---

# 11. Interfaces

Interfaces define common fields shared by multiple types.

```graphql
interface Animal{

    id:ID!

    name:String!
}
```

Dog:

```graphql
type Dog implements Animal{

    id:ID!

    name:String!

    breed:String!
}
```

Cat:

```graphql
type Cat implements Animal{

    id:ID!

    name:String!

    color:String!
}
```

Both types guarantee that `id` and `name` exist.

---

# 12. Unions

Unions group different types without requiring common fields.

```graphql
union SearchResult = User | Product | Order
```

Query:

```graphql
query{

    search{

        ... on User{
            name
        }

        ... on Product{
            price
        }

    }

}
```

A search result could be any supported type.

---

# 13. Custom Scalars

Sometimes the built-in scalars aren't enough.

Examples:

```graphql
scalar Date

scalar DateTime

scalar URL

scalar Email

scalar JSON
```

Usage:

```graphql
type User{

    createdAt:DateTime!
}
```

The server defines how these values are parsed and serialized.

---

# 14. Directives

Directives modify the behavior of a query or schema.

Built-in directives:

```graphql
@include

@skip

@deprecated
```

Example:

```graphql
query{

    user{

        email @include(if:true)

    }

}
```

Deprecation:

```graphql
type User{

    username:String
        @deprecated(reason:"Use name")
}
```

This lets API providers phase out fields without immediately breaking clients.

---

# 15. Root Types

Every GraphQL API starts from one of three root types.

```text
Schema

│

├── Query

├── Mutation

└── Subscription
```

Example:

```graphql
type Query{

    users:[User!]!

    user(id:ID!):User
}
```

Mutation:

```graphql
type Mutation{

    createUser(
        input:CreateUserInput!
    ):User
}
```

Subscription:

```graphql
type Subscription{

    newUser:User
}
```

These root types define all the operations clients can perform.

---

# 16. Complete Schema Example

```graphql
type User{

    id:ID!

    name:String!

    email:String!

    posts:[Post!]!
}

type Post{

    id:ID!

    title:String!

    content:String!

    author:User!
}

input CreatePostInput{

    title:String!

    content:String!

    authorId:ID!
}

type Query{

    users:[User!]!

    user(id:ID!):User

    posts:[Post!]!
}

type Mutation{

    createPost(
        input:CreatePostInput!
    ):Post
}

schema{

    query:Query

    mutation:Mutation
}
```

This schema describes:

* Two object types (`User` and `Post`)
* One input type (`CreatePostInput`)
* A query root
* A mutation root
* Relationships between users and posts

---

# 17. How Apollo Uses the Schema

One of Apollo Kotlin's biggest advantages is **code generation**.

The workflow looks like this:

```text
GraphQL Server

↓

Schema

↓

Android Project

↓

Apollo Plugin

↓

Generated Kotlin Classes

↓

Your App
```

Suppose the schema contains:

```graphql
type User{

    id:ID!

    name:String!

    email:String!
}
```

And your query is:

```graphql
query GetUser{

    user(id:"1"){

        id

        name
    }
}
```

Apollo generates Kotlin models similar to:

```kotlin
data class User(
    val id: String,
    val name: String
)
```

Notice:

* `email` isn't generated because the query didn't request it.
* `id` is `String`, not `String?`, because the schema marked it as non-null (`ID!`).
* `name` is also non-null because it's `String!`.

This is why the schema is so important: it gives Apollo enough information to generate **type-safe** Kotlin code.

---

# 18. Summary

* A schema is the contract between the client and the server.
* Schemas are written using SDL.
* Scalars represent primitive values.
* Object types model real-world entities.
* Non-null types (`!`) define required fields.
* Lists (`[]`) represent collections.
* Input types are used for mutations.
* Enums provide fixed sets of values.
* Interfaces define shared fields.
* Unions allow fields to return different object types.
* Root types (`Query`, `Mutation`, `Subscription`) define available operations.
* Apollo Kotlin uses the schema to generate strongly typed Kotlin classes.

---

# 19. Interview Questions

### 1. What is a GraphQL schema?

**Answer:** A GraphQL schema is the contract that defines the available types, fields, relationships, queries, mutations, and subscriptions in a GraphQL API.

---

### 2. What is SDL?

**Answer:** SDL (Schema Definition Language) is the language used to define GraphQL schemas in a human-readable format.

---

### 3. What is the difference between `String` and `String!`?

**Answer:** `String` can be `null`, while `String!` is guaranteed to have a value. Apollo Kotlin generates `String?` for nullable fields and `String` for non-null fields.

---

### 4. Why are input types used?

**Answer:** Input types group related arguments into a single object, making mutations easier to read, maintain, and extend.

---

### 5. What is the difference between an interface and a union?

**Answer:** An **interface** defines common fields that implementing types must provide. A **union** groups different object types without requiring any shared fields.

---

### 6. Why is the GraphQL schema important for Android developers?

**Answer:** Apollo Kotlin uses the schema to validate queries and generate type-safe Kotlin classes, reducing runtime errors and eliminating much of the manual JSON parsing.

---

## 📌 Next Chapter (Chapter 6)

The next chapter will focus entirely on **GraphQL Types**, with an even deeper exploration of:

* Scalar types (internal representation)
* Object types
* Nested object graphs
* Input object types
* Enum design
* Interfaces vs. Unions (when to use each)
* Custom scalar implementation
* Type extensions
* Advanced schema design patterns

This chapter will include Android-focused examples showing how each GraphQL type maps to generated Kotlin models in Apollo Kotlin.
