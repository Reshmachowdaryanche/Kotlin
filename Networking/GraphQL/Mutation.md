A **Mutation** is the GraphQL operation used to **create, update, or delete data**. If a **Query** is similar to an SQL `SELECT`, then a **Mutation** is like `INSERT`, `UPDATE`, or `DELETE`.

---

# What is a Mutation?

A mutation changes data on the server.

Examples:

* Create a new user
* Update a user's profile
* Delete a post
* Like a post
* Add an item to a shopping cart
* Place an order

Unlike queries, mutations have **side effects** because they modify data.

---

# Basic Mutation Structure

```graphql
mutation {
  createUser(name: "John", email: "john@example.com") {
    id
    name
    email
  }
}
```

Let's break it down:

```graphql
mutation {
```

* Indicates this is a write operation.

```graphql
createUser(...)
```

* Calls a mutation defined in the GraphQL schema.

```graphql
{
    id
    name
    email
}
```

* Specifies which fields should be returned after the mutation succeeds.

---

# Example 1: Create a User

Schema:

```graphql
type Mutation {
    createUser(name: String!, email: String!): User
}
```

Mutation:

```graphql
mutation {
  createUser(
    name: "Alice",
    email: "alice@example.com"
  ) {
    id
    name
    email
  }
}
```

Response:

```json
{
  "data": {
    "createUser": {
      "id": "101",
      "name": "Alice",
      "email": "alice@example.com"
    }
  }
}
```

Notice that the server returns the newly created object.

---

# Example 2: Update a User

Schema:

```graphql
type Mutation {
    updateUser(
        id: ID!,
        name: String
    ): User
}
```

Mutation:

```graphql
mutation {
  updateUser(
    id: 1,
    name: "John Smith"
  ) {
    id
    name
  }
}
```

Response:

```json
{
  "data": {
    "updateUser": {
      "id": "1",
      "name": "John Smith"
    }
  }
}
```

---

# Example 3: Delete a User

Schema:

```graphql
type Mutation {
    deleteUser(id: ID!): Boolean
}
```

Mutation:

```graphql
mutation {
    deleteUser(id: 1)
}
```

Response:

```json
{
  "data": {
    "deleteUser": true
  }
}
```

---

# Why Return Data After a Mutation?

In REST:

```http
POST /users
```

You might need another API call to fetch the created user.

GraphQL lets you request the data immediately.

```graphql
mutation {
    createUser(name: "Bob") {
        id
        name
        profilePicture
    }
}
```

The response already contains everything your UI needs.

This avoids an additional network request.

---

# Using Variables

Instead of hardcoding values:

```graphql
mutation {
    createUser(
        name: "Alice",
        email: "alice@example.com"
    ) {
        id
    }
}
```

Use variables:

```graphql
mutation CreateUser(
    $name: String!,
    $email: String!
) {
    createUser(
        name: $name,
        email: $email
    ) {
        id
        name
    }
}
```

Variables:

```json
{
    "name": "Alice",
    "email": "alice@example.com"
}
```

Benefits:

* Reusable operations
* Cleaner code
* Better security (avoids string interpolation)
* Easier testing

---

# Multiple Mutations

You can write:

```graphql
mutation {

    createUser(name:"John") {
        id
    }

    createPost(
        title:"GraphQL"
    ) {
        id
    }

}
```

Unlike query fields, **mutation fields are executed sequentially** (in the order they appear). This helps maintain predictable behavior when operations depend on each other.

---

# Mutation Response

Success:

```json
{
  "data": {
    "updateUser": {
      "id": "1",
      "name": "John"
    }
  }
}
```

Failure:

```json
{
  "data": {
    "updateUser": null
  },
  "errors": [
    {
      "message": "User not found"
    }
  ]
}
```

---

# Apollo Kotlin Example

Suppose you have:

```graphql
mutation CreateUser(
    $name: String!
) {
    createUser(name: $name) {
        id
        name
    }
}
```

Apollo generates a `CreateUserMutation` class.

Android code:

```kotlin
val response = apolloClient
    .mutation(CreateUserMutation(name))
    .execute()

val user = response.data?.createUser
```

This is very similar to executing a query:

```kotlin
apolloClient.query(...)
```

The only difference is:

```kotlin
apolloClient.mutation(...)
```

---

# Query vs Mutation

| Query                   | Mutation                                           |
| ----------------------- | -------------------------------------------------- |
| Reads data              | Changes data                                       |
| No side effects         | Has side effects                                   |
| Similar to SQL `SELECT` | Similar to SQL `INSERT`, `UPDATE`, `DELETE`        |
| Used for fetching       | Used for create, update, delete                    |
| Can often be cached     | Usually not cached because it changes server state |

---

# Real Android Examples

### User Login

```graphql
mutation Login(
    $email: String!,
    $password: String!
) {
    login(
        email: $email,
        password: $password
    ) {
        token
        user {
            id
            name
        }
    }
}
```

---

### Add to Cart

```graphql
mutation {
    addToCart(
        productId: 100,
        quantity: 2
    ) {
        id
        totalPrice
    }
}
```

---

### Like a Post

```graphql
mutation {
    likePost(postId: 50) {
        likesCount
    }
}
```

---

### Update Profile

```graphql
mutation {
    updateProfile(
        name: "John",
        city: "Chennai"
    ) {
        id
        name
        city
    }
}
```

---

# How a Mutation Flows

```
Android App
     │
     │ Mutation
     ▼
Apollo Client
     │
     │ HTTP POST
     ▼
GraphQL Server
     │
     │ Executes mutation resolver
     ▼
Database
     │
     │ Updates data
     ▼
Returns updated object
     │
     ▼
Apollo parses response
     │
     ▼
UI updates
```

---

# Common Android Interview Questions

### 1. Why use a mutation instead of a query?

Because a mutation is specifically designed to **modify server-side data**, while a query should only read data.

---

### 2. Why does a mutation return data?

To avoid making another request after creating or updating data. The client can request exactly the fields it needs in the mutation response.

---

### 3. Are mutations executed in parallel?

No. Fields within a single mutation operation are executed **sequentially**, ensuring predictable updates. Query fields, on the other hand, may be resolved independently.

---

### 4. Can a mutation return nested objects?

Yes.

Example:

```graphql
mutation {
    createOrder {
        id
        customer {
            name
        }
        items {
            productName
            price
        }
    }
}
```

---

### 5. Can a mutation use variables?

Yes. In fact, using variables is the recommended approach because it makes operations reusable, easier to maintain, and avoids embedding dynamic values directly in the GraphQL document.

> **Interview tip:** A strong answer is: "A mutation changes server state. After the change, I can request exactly the fields my Android screen needs, allowing the UI to update immediately without making an additional fetch request."
