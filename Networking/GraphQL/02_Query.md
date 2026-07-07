The **Query** component is the most important part of GraphQL because it's used to **read or fetch data** from the server. If you're preparing for Android interviews, you should understand not just the syntax but also how queries are executed and used in an Android app.

---

# What is a Query?

A **Query** is an operation that asks the GraphQL server for specific data.

Think of it like an SQL `SELECT` statement.

**SQL**

```sql
SELECT name, email
FROM users
WHERE id = 1;
```

**GraphQL**

```graphql
query {
  user(id: 1) {
    name
    email
  }
}
```

The client explicitly specifies the fields it wants.

---

# Basic Query Structure

```graphql
query {
  user(id: 1) {
    id
    name
    email
  }
}
```

Breakdown:

```graphql
query {
```

* Indicates a read operation.
* The `query` keyword is optional in many simple cases.

```graphql
user(id: 1)
```

* `user` is a field defined in the GraphQL schema.
* `id: 1` is an argument passed to that field.

```graphql
{
    id
    name
    email
}
```

* This is the **selection set**.
* It specifies exactly which fields should be returned.

---

# Query Without the `query` Keyword

These are equivalent:

```graphql
query {
  user(id: 1) {
    name
  }
}
```

and

```graphql
{
  user(id: 1) {
    name
  }
}
```

The explicit `query` keyword is often used when naming operations or using variables.

---

# Nested Queries

One of GraphQL's strengths is fetching related data in a single request.

Suppose the schema is:

```graphql
type User {
    id: ID!
    name: String!
    posts: [Post]
}

type Post {
    id: ID!
    title: String!
}
```

Query:

```graphql
query {
  user(id: 1) {
    name
    posts {
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
          "title": "Android Interview"
        }
      ]
    }
  }
}
```

A single request retrieves both the user and their posts.

---

# Selecting Only Required Fields

Suppose the server has:

```text
User
 ├── id
 ├── name
 ├── email
 ├── age
 ├── phone
 ├── address
 ├── profilePic
```

If your Android screen only displays:

* Name
* Profile picture

You can write:

```graphql
query {
  user(id: 1) {
    name
    profilePic
  }
}
```

Only those two fields are returned.

This reduces network usage and parsing work.

---

# Query Arguments

Arguments filter or customize the returned data.

Example:

```graphql
query {
  product(id: 100) {
    name
    price
  }
}
```

Another example:

```graphql
query {
  posts(limit: 5) {
    title
  }
}
```

Or multiple arguments:

```graphql
query {
  users(country: "India", age: 25) {
    name
  }
}
```

---

# Query Variables

Hardcoding values isn't ideal.

Instead:

```graphql
query GetUser($id: ID!) {
  user(id: $id) {
    id
    name
  }
}
```

Variables:

```json
{
  "id": 5
}
```

Benefits:

* Reusable query
* Dynamic values
* Better caching
* Cleaner code

---

# Named Queries

Instead of:

```graphql
query {
    user(id:1){
        name
    }
}
```

Use:

```graphql
query GetUser {
    user(id:1){
        name
    }
}
```

Naming operations helps with:

* Logging
* Debugging
* Analytics
* Error reporting

---

# Multiple Root Fields

A query can fetch multiple unrelated objects.

```graphql
query {
  user(id: 1) {
    name
  }

  products {
    name
    price
  }

  categories {
    title
  }
}
```

Response:

```json
{
  "data": {
    "user": {
      "name": "John"
    },
    "products": [
      {
        "name": "Laptop",
        "price": 50000
      }
    ],
    "categories": [
      {
        "title": "Electronics"
      }
    ]
  }
}
```

One request replaces several REST API calls.

---

# Aliases

Suppose you need two users.

This won't work:

```graphql
query {
  user(id:1) {
    name
  }

  user(id:2) {
    name
  }
}
```

Use aliases:

```graphql
query {
  firstUser: user(id:1) {
    name
  }

  secondUser: user(id:2) {
    name
  }
}
```

Response:

```json
{
  "data": {
    "firstUser": {
      "name": "John"
    },
    "secondUser": {
      "name": "Alice"
    }
  }
}
```

---

# Fragments

Suppose many queries need:

```graphql
id
name
email
profilePic
```

Instead of repeating:

```graphql
query {
  user(id:1){
      id
      name
      email
      profilePic
  }

  manager(id:2){
      id
      name
      email
      profilePic
  }
}
```

Use a fragment:

```graphql
fragment UserInfo on User {
    id
    name
    email
    profilePic
}

query {
    user(id:1){
        ...UserInfo
    }

    manager(id:2){
        ...UserInfo
    }
}
```

Fragments improve reuse and consistency.

---

# Response Format

Every GraphQL response has a `data` field.

Example:

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

If part of the query fails:

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

This is called **partial success**.

---

# How Apollo Kotlin Executes a Query

Suppose you have a GraphQL file:

```graphql
query GetUser($id: ID!) {
  user(id: $id) {
    id
    name
    email
  }
}
```

Apollo Kotlin generates a `GetUserQuery` class.

You can execute it like this:

```kotlin
val response = apolloClient
    .query(GetUserQuery(id))
    .execute()

val user = response.data?.user
```

Apollo parses the JSON response into generated Kotlin types, giving you compile-time type safety instead of manually reading JSON fields.

---

# How Query Execution Works Internally

```
Android App
     │
     │ GraphQL Query
     ▼
Apollo Client
     │
     │ HTTP POST
     ▼
GraphQL Server
     │
     │ Resolver for "user"
     ▼
Database
     │
     ▼
JSON Response
     │
     ▼
Apollo parses response
     │
     ▼
Generated Kotlin Objects
```

The GraphQL server uses **resolvers** to determine how to fetch data for each field. For example, the `user` field might query a database, while the `posts` field could call another service. The client doesn't need to know where the data comes from.

---

# Common Android Interview Questions About Queries

1. **Why is a GraphQL query better than multiple REST calls?**

   * It can fetch related data in one request, reducing network round trips.

2. **Why use variables instead of hardcoding values?**

   * Queries become reusable, easier to maintain, and can improve caching.

3. **What is a selection set?**

   * The list of fields inside a query that specifies exactly what data should be returned.

4. **Can a GraphQL query fetch nested objects?**

   * Yes. That's one of GraphQL's core features.

5. **Can one query fetch multiple resources?**

   * Yes. A single query can include multiple root fields (for example, `user`, `products`, and `categories`) and retrieve them in one request.

These are the concepts interviewers frequently explore because they demonstrate an understanding of why GraphQL is valuable for mobile applications.
