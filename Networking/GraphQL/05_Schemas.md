A **GraphQL schema** is the contract between clients and servers. It defines **what data is available, how it is structured, and what operations can be performed**. Understanding schemas is essential because every GraphQL query is validated against the schema.

---

# 1. What is a GraphQL Schema?

A GraphQL schema describes:

* Data types
* Relationships between data
* Queries
* Mutations
* Subscriptions
* Input types
* Enums
* Interfaces
* Unions
* Scalars
* Directives

Example:

```graphql
type User {
  id: ID!
  name: String!
  email: String!
  age: Int
}
```

This defines a `User` object.

---

# 2. Schema Definition Language (SDL)

GraphQL uses SDL (Schema Definition Language).

Example:

```graphql
schema {
  query: Query
  mutation: Mutation
}
```

Most GraphQL servers infer this automatically.

---

# 3. Scalar Types

Built-in scalars:

```graphql
Int
Float
String
Boolean
ID
```

Example

```graphql
type Product {
    id: ID!
    name: String!
    price: Float!
    available: Boolean!
}
```

Meaning:

```
ID!        -> Required unique identifier
String!    -> Required string
Float!     -> Required decimal
Boolean!   -> Required boolean
```

---

# 4. Non-null Types (!)

Without !

```graphql
name: String
```

Can return

```json
{
  "name": null
}
```

With !

```graphql
name: String!
```

Cannot return

```json
{
   "name": null
}
```

Instead GraphQL throws an error.

---

# 5. Lists

Single value

```graphql
friends: User
```

List

```graphql
friends: [User]
```

Required list

```graphql
friends: [User]!
```

Required items

```graphql
friends: [User!]
```

Required list with required items

```graphql
friends: [User!]!
```

Difference:

```
[User]
↓
List can be null
Items can be null

[User!]
↓
List nullable
Items cannot be null

[User]!
↓
List cannot be null
Items nullable

[User!]!
↓
Neither list nor items nullable
```

---

# 6. Object Types

Example

```graphql
type User {
    id: ID!
    name: String!
    posts: [Post!]!
}
```

```graphql
type Post {
    id: ID!
    title: String!
}
```

Relationship

```
User
 ├── id
 ├── name
 └── posts
        ├── title
        └── id
```

---

# 7. Query Type

Every GraphQL API starts here.

```graphql
type Query {
    user(id: ID!): User
    users: [User!]!
}
```

Client query

```graphql
query {
    users {
        id
        name
    }
}
```

---

# 8. Mutation Type

Used for creating/updating/deleting data.

Schema

```graphql
type Mutation {
    createUser(name: String!, age: Int!): User
}
```

Mutation

```graphql
mutation {
    createUser(name: "John", age: 25) {
        id
        name
    }
}
```

---

# 9. Subscription Type

Real-time updates.

Schema

```graphql
type Subscription {
    newUser: User
}
```

Client

```graphql
subscription {
    newUser {
        id
        name
    }
}
```

Usually implemented with WebSockets.

---

# 10. Arguments

Fields can take arguments.

```graphql
type Query {
    user(id: ID!): User
}
```

Query

```graphql
query {
    user(id: "123") {
        name
    }
}
```

---

# 11. Input Types

Instead of many parameters

Bad

```graphql
createUser(
    name: String!
    email: String!
    age: Int!
)
```

Better

```graphql
input CreateUserInput {
    name: String!
    email: String!
    age: Int!
}
```

Mutation

```graphql
type Mutation {
    createUser(input: CreateUserInput!): User
}
```

Request

```graphql
mutation {
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

---

# 12. Enums

Instead of strings

Bad

```graphql
status: String
```

Good

```graphql
enum Status {
    ACTIVE
    INACTIVE
    BLOCKED
}
```

Usage

```graphql
type User {
    status: Status!
}
```

---

# 13. Interfaces

Used for common fields.

```graphql
interface Animal {
    id: ID!
    name: String!
}
```

Implementations

```graphql
type Dog implements Animal {
    id: ID!
    name: String!
    breed: String!
}
```

```graphql
type Cat implements Animal {
    id: ID!
    name: String!
    color: String!
}
```

---

# 14. Union Types

Different from interfaces.

```graphql
union SearchResult = User | Product | Order
```

Query

```graphql
query {
    search {
        ... on User {
            name
        }

        ... on Product {
            price
        }
    }
}
```

No common fields required.

---

# 15. Custom Scalars

Create custom data types.

```graphql
scalar Date
```

Example

```graphql
type User {
    createdAt: Date!
}
```

Server implements parsing/serialization.

Common custom scalars:

```
Date

DateTime

URL

Email

JSON
```

---

# 16. Directives

Built-in

```
@include

@skip

@deprecated
```

Example

```graphql
query {
    user {
        email @include(if:true)
    }
}
```

Deprecated

```graphql
type User {
    username: String @deprecated(reason:"Use name")
}
```

---

# 17. Root Types

```
schema
   │
   ├───────────────┐
   │               │
 Query         Mutation
   │               │
 Read         Write
```

Example

```graphql
type Query {
    users: [User!]!
}
```

```graphql
type Mutation {
    createUser(input: CreateUserInput!): User
}
```

---

# 18. Relationships

One-to-one

```graphql
type User {
    profile: Profile
}
```

One-to-many

```graphql
type User {
    posts: [Post!]!
}
```

Many-to-many

```graphql
type Student {
    courses: [Course!]!
}
```

```graphql
type Course {
    students: [Student!]!
}
```

---

# 19. Circular References

Allowed

```graphql
type User {
    posts: [Post!]!
}
```

```graphql
type Post {
    author: User!
}
```

Client

```graphql
query {
    user(id:"1"){
        posts{
            author{
                name
            }
        }
    }
}
```

---

# 20. Schema Example (Complete)

```graphql
type User {
    id: ID!
    name: String!
    email: String!
    posts: [Post!]!
}

type Post {
    id: ID!
    title: String!
    content: String!
    author: User!
}

input CreatePostInput {
    title: String!
    content: String!
    authorId: ID!
}

type Query {
    users: [User!]!
    user(id: ID!): User
    posts: [Post!]!
}

type Mutation {
    createPost(input: CreatePostInput!): Post
}

schema {
    query: Query
    mutation: Mutation
}
```

---

# 21. Introspection

GraphQL supports introspection, allowing clients and tools to discover the schema.

Example query:

```graphql
{
  __schema {
    types {
      name
    }
  }
}
```

This powers tools like GraphQL Playground and GraphiQL by enabling autocomplete and documentation.

---

# 22. Best Practices

* Use `ID` for identifiers.
* Prefer `input` objects over long argument lists.
* Use non-null (`!`) only when a field is guaranteed.
* Use enums instead of free-form strings for fixed values.
* Keep queries focused on reading and mutations on writing.
* Avoid deeply nested queries that can be expensive to resolve.
* Deprecate fields instead of removing them immediately to preserve backward compatibility.
* Document types and fields with descriptions to improve generated documentation.

---

# Visual Summary

```text
                    GraphQL Schema
                          │
        ┌─────────────────┼─────────────────┐
        │                 │                 │
     Scalars          Object Types      Input Types
        │                 │                 │
 String, Int, ID      User, Post       CreateUserInput
        │                 │
        └────────────┬────┘
                     │
             Relationships
                     │
       One-to-One / One-to-Many / Many-to-Many
                     │
        ┌────────────┼────────────┐
        │            │            │
      Query      Mutation    Subscription
        │            │            │
      Read         Write      Real-time
                     │
        ┌────────────┼────────────┐
        │            │            │
      Enums     Interfaces     Unions
                     │
               Custom Scalars
                     │
                 Directives
                     │
               Introspection
```

A solid understanding of GraphQL schemas makes it much easier to design APIs, implement resolvers, and write efficient client queries.


---

In Android, GraphQL schemas are **not used directly in your app code**. Instead, the schema is used to **generate Kotlin/Java classes** that your Android app uses to make type-safe API calls.

A common setup is:

* **Backend:** GraphQL Server (defines the schema)
* **Android App:** Apollo GraphQL client
* **Schema:** Downloaded from the server
* **Queries:** Written in `.graphql` files
* **Generated Classes:** Created automatically by Apollo

---

# Example: Fetch a User

## Step 1: Backend Schema

Suppose the backend has this schema:

```graphql
type User {
    id: ID!
    name: String!
    email: String!
}

type Query {
    user(id: ID!): User
}
```

This tells us:

* There is a `User` type.
* We can query a user by ID.

---

# Step 2: Android Creates a GraphQL Query

Instead of writing the query as a string in Kotlin, create a file:

`GetUser.graphql`

```graphql
query GetUser($id: ID!) {
    user(id: $id) {
        id
        name
        email
    }
}
```

Apollo reads this file and generates Kotlin classes.

---

# Step 3: Apollo Generates Code

Apollo automatically generates classes similar to:

```kotlin
GetUserQuery

GetUserQuery.Data

GetUserQuery.User
```

You never write these classes yourself.

---

# Step 4: Call the API

```kotlin
val response = apolloClient.query(
    GetUserQuery(id = "101")
).execute()
```

---

# Step 5: Read the Response

```kotlin
val user = response.data?.user

println(user?.name)
println(user?.email)
```

No JSON parsing is needed.

---

# What Happens Internally?

```
Android
   |
   |  GetUserQuery(id = "101")
   |
Apollo Client
   |
   | Converts to GraphQL request
   |
Server
   |
Schema
   |
Resolver
   |
Database
```

The request sent to the server looks like:

```graphql
query GetUser($id: ID!) {
    user(id: $id) {
        id
        name
        email
    }
}
```

---

# Server Response

The server returns JSON like this:

```json
{
  "data": {
    "user": {
      "id": "101",
      "name": "John",
      "email": "john@gmail.com"
    }
  }
}
```

Apollo converts it into Kotlin objects automatically.

---

# Step 6: Display in Android UI

```kotlin
lifecycleScope.launch {
    val response = apolloClient.query(
        GetUserQuery("101")
    ).execute()

    val user = response.data?.user

    binding.tvName.text = user?.name
    binding.tvEmail.text = user?.email
}
```

No manual JSON parsing with `JSONObject` or `Gson` is required.

---

# How the Schema Helps

Suppose the schema changes:

**Old schema:**

```graphql
type User {
    id: ID!
    name: String!
}
```

**New schema:**

```graphql
type User {
    id: ID!
    fullName: String!
}
```

If your query still asks for `name`:

```graphql
query GetUser {
    user(id: "1") {
        name
    }
}
```

Apollo fails during code generation with an error like:

```
Cannot query field "name" on type "User".
```

This catches the problem before your app runs, instead of causing a runtime crash.

---

# Real Android Project Structure

```
app/
 ├── src/
 │   ├── main/
 │   │   ├── java/
 │   │   │    └── com/example/app/
 │   │   │         ├── MainActivity.kt
 │   │   │         ├── UserRepository.kt
 │   │   │         └── ApolloClient.kt
 │   │   │
 │   │   ├── graphql/
 │   │   │     └── GetUser.graphql
 │   │   │
 │   │   └── graphql/schema.graphqls
 │   │
 └── build.gradle
```

---

# Complete Flow

```
1. Backend exposes schema
        │
        ▼
2. Android downloads schema
        │
        ▼
3. Developer writes GetUser.graphql
        │
        ▼
4. Apollo generates Kotlin classes
        │
        ▼
5. Android calls:

   apolloClient.query(GetUserQuery())

        │
        ▼
6. Server executes query
        │
        ▼
7. JSON response
        │
        ▼
8. Apollo converts JSON → Kotlin objects
        │
        ▼
9. Show data in RecyclerView/TextView/Compose UI
```

### Why Android developers prefer GraphQL with Apollo

Compared to REST, GraphQL with Apollo offers:

| REST                                                 | GraphQL + Apollo                                     |
| ---------------------------------------------------- | ---------------------------------------------------- |
| Multiple endpoints (`/users`, `/posts`, `/comments`) | Single GraphQL endpoint                              |
| Manual JSON model mapping                            | Auto-generated Kotlin models                         |
| Can receive unused fields                            | Request only the fields you need                     |
| Schema changes may cause runtime issues              | Schema validation catches many issues at build time  |
| Often requires multiple requests for related data    | One query can fetch related data in a single request |

This type-safe workflow is one of the biggest reasons GraphQL is popular in modern Android applications.

