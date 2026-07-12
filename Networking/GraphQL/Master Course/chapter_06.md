# Chapter 6 – GraphQL Types (Complete Deep Dive)

## GraphQL for Android Developers

> **Learning Goal**
>
> By the end of this chapter, you will understand:
>
> * What GraphQL Types are
> * Why GraphQL is a strongly typed language
> * Every built-in GraphQL type
> * Object Types
> * Scalar Types
> * Input Object Types
> * Enum Types
> * Interface Types
> * Union Types
> * Custom Scalars
> * Type Extensions
> * How Apollo Kotlin maps GraphQL types to Kotlin types
> * Best practices for designing GraphQL types

---

# Table of Contents

1. What are GraphQL Types?
2. Why GraphQL is Strongly Typed
3. Scalar Types
4. Object Types
5. Input Object Types
6. Enum Types
7. Interface Types
8. Union Types
9. Custom Scalars
10. Type Extensions
11. Nested Types
12. Kotlin Type Mapping (Apollo)
13. Best Practices
14. Summary
15. Interview Questions

---

# 1. What are GraphQL Types?

A **type** defines the shape of data.

Think of a type as a blueprint for an object.

For example:

```text
User

id

name

email
```

The GraphQL schema defines that every `User` has those fields.

Example:

```graphql
type User {
    id: ID!
    name: String!
    email: String!
}
```

Whenever the client asks for a `User`, GraphQL knows exactly which fields are available.

---

## Real-Life Analogy

Imagine a student ID card.

Every student ID card contains:

```text
Student

↓

ID

Name

Department

Photo
```

Every student follows the same structure.

Similarly:

```text
User

↓

ID

Name

Email
```

GraphQL types ensure consistency across the API.

---

# 2. Why GraphQL is Strongly Typed

GraphQL is a **strongly typed** language.

This means every field has a defined type.

Example:

```graphql
type Product {
    id: ID!
    name: String!
    price: Float!
    stock: Int!
    available: Boolean!
}
```

GraphQL knows:

* `id` → ID
* `name` → String
* `price` → Float
* `stock` → Int
* `available` → Boolean

Because of this, the server can validate queries before executing them.

If a client requests a field that doesn't exist, GraphQL returns an error instead of guessing.

---

# 3. Scalar Types

Scalars are the simplest GraphQL types. They represent single values.

### Built-in Scalar Types

| GraphQL Type | Description       | Example     |
| ------------ | ----------------- | ----------- |
| `Int`        | 32-bit integer    | `25`        |
| `Float`      | Decimal number    | `99.99`     |
| `String`     | Text              | `"Android"` |
| `Boolean`    | `true` or `false` | `true`      |
| `ID`         | Unique identifier | `"1001"`    |

---

## Int

```graphql
type User {
    age: Int!
}
```

Response:

```json
{
    "age": 28
}
```

---

## Float

```graphql
type Product {
    price: Float!
}
```

Response:

```json
{
    "price": 1499.50
}
```

---

## String

```graphql
type User {
    name: String!
}
```

Response:

```json
{
    "name": "John"
}
```

---

## Boolean

```graphql
type Product {
    available: Boolean!
}
```

Response:

```json
{
    "available": true
}
```

---

## ID

Many beginners ask:

**Why not use `String` instead of `ID`?**

Example:

```graphql
id: ID!
```

instead of

```graphql
id: String!
```

The reason is semantic meaning.

`ID` tells GraphQL:

> "This value uniquely identifies an object."

It may still be serialized as a string, but using `ID` makes the schema clearer.

---

# 4. Object Types

Object types represent real-world entities.

Example:

```graphql
type User {
    id: ID!
    name: String!
}
```

Another object:

```graphql
type Post {
    id: ID!
    title: String!
}
```

Objects can reference other objects.

Example:

```graphql
type User {
    id: ID!
    name: String!

    posts: [Post!]!
}
```

This creates a relationship.

---

## Object Graph

```text
User

├── id

├── name

└── posts

      ├── title

      ├── likes

      └── comments
```

This graph-like structure is one reason for the name **GraphQL**: you traverse relationships between objects.

---

# 5. Input Object Types

Object types describe **responses**.

Input object types describe **request data**.

Example:

Instead of:

```graphql
createUser(
    name: String!
    email: String!
    age: Int!
)
```

Create an input type:

```graphql
input CreateUserInput {
    name: String!
    email: String!
    age: Int!
}
```

Mutation:

```graphql
type Mutation {
    createUser(input: CreateUserInput!): User
}
```

Request:

```graphql
mutation {
    createUser(
        input: {
            name: "John"
            email: "john@example.com"
            age: 25
        }
    ) {
        id
        name
    }
}
```

### Why use Input Types?

* Cleaner APIs
* Easier to extend later
* Better organization
* Reusable structures

---

# 6. Enum Types

Enums define a fixed list of allowed values.

Instead of:

```graphql
status: String
```

Use:

```graphql
enum Status {
    ACTIVE
    INACTIVE
    BLOCKED
}
```

Usage:

```graphql
type User {
    status: Status!
}
```

Response:

```json
{
    "status": "ACTIVE"
}
```

### Why Enums?

Without enums:

```text
"active"

"ACTIVE"

"Active"

"enabled"

"running"
```

All are possible.

Enums prevent invalid values.

---

# Apollo Mapping

GraphQL:

```graphql
enum Status {
    ACTIVE
    INACTIVE
}
```

Generated Kotlin:

```kotlin
enum class Status {
    ACTIVE,
    INACTIVE
}
```

---

# 7. Interface Types

An interface defines common fields shared by multiple object types.

Example:

```graphql
interface Animal {
    id: ID!
    name: String!
}
```

Dog:

```graphql
type Dog implements Animal {
    id: ID!
    name: String!
    breed: String!
}
```

Cat:

```graphql
type Cat implements Animal {
    id: ID!
    name: String!
    color: String!
}
```

---

## Why Interfaces?

Suppose your app displays a list of animals.

You know every animal has:

* id
* name

You don't care whether it's a Dog or Cat.

Interfaces allow this shared behavior.

---

# 8. Union Types

Unions are different.

Example:

```graphql
union SearchResult = User | Product | Order
```

A search API can return:

```text
User

or

Product

or

Order
```

No shared fields required.

Query:

```graphql
query {
    search {
        ... on User {
            name
        }

        ... on Product {
            price
        }

        ... on Order {
            total
        }
    }
}
```

---

## Interface vs Union

| Interface                   | Union                    |
| --------------------------- | ------------------------ |
| Common fields required      | No common fields         |
| Objects implement interface | Objects belong to union  |
| Better for inheritance      | Better for mixed results |

---

# 9. Custom Scalars

Sometimes built-in scalars are not enough.

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
type User {
    createdAt: DateTime!
}
```

Apollo can map custom scalars to Kotlin types such as `Instant`, `LocalDateTime`, or custom classes through adapters.

---

# 10. Type Extensions

Large schemas are often split across multiple files.

Example:

```graphql
type User {
    id: ID!
    name: String!
}
```

Later:

```graphql
extend type User {
    posts: [Post!]!
}
```

Instead of rewriting the entire type, you extend it.

This is especially useful in modular or federated GraphQL services.

---

# 11. Nested Types

GraphQL types naturally form a graph.

Example:

```graphql
type User {
    name: String!
    posts: [Post!]!
}

type Post {
    title: String!
    comments: [Comment!]!
}

type Comment {
    text: String!
}
```

Query:

```graphql
query {
    user(id: 1) {
        name
        posts {
            title
            comments {
                text
            }
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
          "title": "Learning GraphQL",
          "comments": [
            {
              "text": "Great article!"
            }
          ]
        }
      ]
    }
  }
}
```

The response mirrors the structure of the query.

---

# 12. Kotlin Type Mapping (Apollo)

Apollo generates Kotlin models based on both the schema and your query.

| GraphQL    | Kotlin                                  |
| ---------- | --------------------------------------- |
| `String`   | `String?`                               |
| `String!`  | `String`                                |
| `Int`      | `Int?`                                  |
| `Int!`     | `Int`                                   |
| `Float`    | `Double?` (by default in Apollo Kotlin) |
| `Float!`   | `Double`                                |
| `Boolean`  | `Boolean?`                              |
| `Boolean!` | `Boolean`                               |
| `ID`       | `String?`                               |
| `ID!`      | `String`                                |

Example schema:

```graphql
type User {
    id: ID!
    name: String!
    age: Int
}
```

Generated model (simplified):

```kotlin
data class User(
    val id: String,
    val name: String,
    val age: Int?
)
```

Apollo uses the schema's nullability rules to generate safe Kotlin types.

---

# 13. Best Practices

### Use `ID` for identifiers

```graphql
id: ID!
```

instead of

```graphql
id: String!
```

---

### Use Enums for fixed values

Good:

```graphql
status: Status!
```

Avoid:

```graphql
status: String
```

---

### Group mutation arguments

Prefer:

```graphql
input CreateUserInput
```

instead of a long list of parameters.

---

### Use Non-Null Carefully

Only mark fields as non-null if you can always provide a value.

Incorrectly marking optional data as non-null can cause unnecessary GraphQL errors.

---

### Keep Types Focused

A `User` type should represent a user.

Avoid adding unrelated fields that belong to another domain.

---

# 14. Summary

* Types define the shape of data in GraphQL.
* GraphQL is strongly typed, enabling early validation.
* Scalars represent primitive values.
* Object types model real-world entities.
* Input types are designed for request payloads.
* Enums restrict values to predefined options.
* Interfaces define shared contracts.
* Unions allow multiple unrelated return types.
* Custom scalars extend GraphQL beyond the built-in types.
* Apollo Kotlin maps GraphQL types into type-safe Kotlin classes.

---

# 15. Interview Questions

### 1. What is a GraphQL type?

**Answer:** A GraphQL type defines the structure and data available for an object or value in the schema.

---

### 2. Why is GraphQL called strongly typed?

**Answer:** Every field has a defined type, allowing the server to validate queries before execution and enabling tools like Apollo to generate type-safe client code.

---

### 3. What is the difference between an object type and an input type?

**Answer:** Object types describe data returned by the server, while input types describe structured data sent by the client, typically in mutations.

---

### 4. When should you use an enum?

**Answer:** Use enums when a field should only allow a fixed set of values, such as `ACTIVE`, `INACTIVE`, or `BLOCKED`.

---

### 5. What is the difference between an interface and a union?

**Answer:** Interfaces define common fields that implementing types must have. Unions combine different object types without requiring shared fields.

---

### 6. How does Apollo Kotlin use GraphQL types?

**Answer:** Apollo reads the schema and your `.graphql` files to generate Kotlin models with the correct field types and nullability, providing compile-time type safety.

---

# 📌 Next Chapter (Chapter 7 – GraphQL Queries Deep Dive)

In the next chapter, we'll focus entirely on **Queries**, including:

* Query syntax in detail
* Selection sets
* Field arguments
* Variables
* Aliases
* Fragments
* Inline fragments
* Nested queries
* Pagination
* Filtering and sorting
* How Apollo Kotlin executes queries
* Performance considerations
* Common query design mistakes

This is the chapter where you'll begin writing production-quality GraphQL queries for Android applications.
