# Chapter 7 – GraphQL Queries Deep Dive

## GraphQL for Android Developers

> **Learning Goal**
>
> By the end of this chapter, you will understand:
>
> * What GraphQL Queries are
> * Query structure and syntax
> * Selection sets
> * Fields and arguments
> * Variables
> * Aliases
> * Fragments
> * Inline fragments
> * Nested queries
> * Pagination
> * Filtering and sorting
> * Query execution in Apollo Kotlin
> * Query best practices

---

# Table of Contents

1. Introduction to GraphQL Queries
2. What is a Query?
3. Query Structure
4. Fields
5. Arguments
6. Variables
7. Query Operation Names
8. Selection Sets
9. Nested Queries
10. Aliases
11. Fragments
12. Inline Fragments
13. Directives in Queries
14. Pagination
15. Filtering
16. Sorting
17. Query Execution in Apollo Kotlin
18. Error Handling
19. Query Optimization
20. Summary
21. Interview Questions

---

# 1. Introduction to GraphQL Queries

A **Query** is used to fetch data from a GraphQL server.

It is similar to:

* `GET` request in REST
* `SELECT` query in SQL

Example REST:

```
GET /users/10
```

GraphQL equivalent:

```graphql
query {
    user(id:10){
        name
        email
    }
}
```

The main difference:

REST decides the response structure.

GraphQL allows the client to decide the response structure.

---

# 2. What is a Query?

A query describes:

1. What data you want
2. Which fields you need
3. Any parameters required

Example:

```graphql
query {
    user(id:10){
        id
        name
    }
}
```

Meaning:

> "Find the user with id 10 and return only id and name."

---

# 3. Query Structure

A GraphQL query has this structure:

```graphql
query OperationName($variable:Type){

    field(arguments){

        requestedFields

    }
}
```

Example:

```graphql
query GetUser($id:ID!){

    user(id:$id){

        id
        name
        email

    }
}
```

Let's break it down.

---

## Operation Type

```graphql
query
```

Means:

"We are reading data."

---

## Operation Name

```graphql
GetUser
```

Used for:

* Debugging
* Logging
* Apollo caching
* Monitoring

---

## Variable

```graphql
($id:ID!)
```

Input value.

---

## Field

```graphql
user
```

The server resolver being called.

---

## Selection Set

```graphql
{
    id
    name
    email
}
```

Fields requested from the user.

---

# 4. Fields

Fields represent the data you want.

Example:

Schema:

```graphql
type User {

    id:ID!

    name:String!

    email:String

    age:Int

}
```

Query:

```graphql
query{

    user(id:1){

        name

        email

    }
}
```

Response:

```json
{
 "data":{
    "user":{
        "name":"John",
        "email":"john@gmail.com"
    }
 }
}
```

Notice:

The server did not return:

```
id

age
```

because they were not requested.

---

# 5. Arguments

Arguments allow you to customize queries.

Example:

```graphql
query{

    user(id:10){

        name

    }

}
```

Here:

```
id:10
```

is an argument.

---

## Multiple Arguments

Example:

```graphql
query{

    products(
        category:"mobile"
        limit:10
    ){

        name
        price

    }

}
```

The server receives:

```
category = mobile

limit = 10
```

---

# 6. Variables

Hardcoding values is not recommended.

Bad:

```graphql
query{

    user(id:10){

        name

    }

}
```

Problem:

Every different user requires a new query.

---

Better:

```graphql
query GetUser($userId:ID!){

    user(id:$userId){

        name

        email

    }

}
```

Variables:

```json
{
 "userId":10
}
```

---

## Why Variables Are Better

Advantages:

* Reusable queries
* Better caching
* Safer input handling
* Cleaner code

---

# Android Example

GraphQL file:

`GetUser.graphql`

```graphql
query GetUser($id:ID!){

    user(id:$id){

        id

        name

        email

    }

}
```

Kotlin:

```kotlin
val response =
    apolloClient
        .query(
            GetUserQuery(id="10")
        )
        .execute()

val user = response.data?.user
```

Apollo generates:

```
GetUserQuery
GetUserQuery.Data
User
```

automatically.

---

# 7. Query Operation Names

You can write:

Anonymous query:

```graphql
query{

    users{

        name

    }

}
```

Named query:

```graphql
query GetUsers{

    users{

        name

    }

}
```

Production applications should always use named queries.

---

## Why?

Benefits:

### Better debugging

Logs show:

```
Executing GetUsers
```

instead of:

```
Executing anonymous query
```

---

### Apollo tracking

Apollo can identify operations.

---

# 8. Selection Sets

A selection set defines fields to return.

Example:

```graphql
user{

    id

    name

    profilePicture

}
```

Selection set:

```
id

name

profilePicture
```

---

GraphQL requires selection sets.

Invalid:

```graphql
query{

    user

}
```

Error:

```
Field "user" must have a selection of subfields.
```

Because GraphQL needs to know what inside the user you want.

---

# 9. Nested Queries

GraphQL's biggest strength is fetching related data.

Example:

Schema:

```
User

 |
 |
 Posts

 |
 |
 Comments
```

Query:

```graphql
query{

    user(id:1){

        name

        posts{

            title

            comments{

                text

            }

        }

    }

}
```

Response:

```json
{
 "data":{
    "user":{
       "name":"John",
       "posts":[
          {
            "title":"GraphQL",
            "comments":[
              {
                "text":"Excellent"
              }
            ]
          }
       ]
    }
 }
}
```

One request.

Multiple related resources.

---

# 10. Aliases

Aliases allow renaming fields in the response.

Example:

You need two users.

Without alias:

```graphql
query{

    user(id:1){

        name

    }

    user(id:2){

        name

    }

}
```

Problem:

Two fields have the same name.

---

Using aliases:

```graphql
query{

    firstUser:user(id:1){

        name

    }

    secondUser:user(id:2){

        name

    }

}
```

Response:

```json
{
 "data":{
    "firstUser":{
       "name":"John"
    },
    "secondUser":{
       "name":"Alex"
    }
 }
}
```

---

# 11. Fragments

**Fragments** in GraphQL let you **define a reusable set of fields** that can be included in multiple queries, mutations, or subscriptions. They help avoid repeating the same field selections.

## Why use fragments?

Suppose you have this query:

```graphql
query {
  users {
    id
    name
    email
  }

  posts {
    id
    title
    author {
      id
      name
      email
    }
  }
}
```

Notice that the `User` fields (`id`, `name`, `email`) are repeated.

Instead, define a fragment.

---

## Basic syntax

```graphql
fragment UserFields on User {
  id
  name
  email
}
```

Here:

* `fragment UserFields` → fragment name.
* `on User` → this fragment can only be used on the `User` type.
* The fields inside are the reusable selection.

---

## Using a fragment

```graphql
query {
  users {
    ...UserFields
  }

  posts {
    id
    title
    author {
      ...UserFields
    }
  }
}

fragment UserFields on User {
  id
  name
  email
}
```

This is equivalent to writing:

```graphql
users {
  id
  name
  email
}
```

every time.

---

## Another example

Given this schema:

```graphql
type User {
  id: ID!
  name: String!
  email: String!
  posts: [Post!]!
}
```

Without fragments:

```graphql
query {
  user(id: "1") {
    id
    name
    email
  }

  users {
    id
    name
    email
  }
}
```

With fragments:

```graphql
query {
  user(id: "1") {
    ...UserInfo
  }

  users {
    ...UserInfo
  }
}

fragment UserInfo on User {
  id
  name
  email
}
```

---

## Fragments with nested fields

Fragments can include nested objects too.

```graphql
fragment PostInfo on Post {
  id
  title
  author {
    id
    name
  }
}
```

Use it like this:

```graphql
query {
  posts {
    ...PostInfo
  }
}
```

---

## Inline fragments

Inline fragments are useful when a field can return **different types**, such as a union or an interface.

Example schema:

```graphql
union SearchResult = User | Post
```

Query:

```graphql
query {
  search {
    ... on User {
      id
      name
      email
    }

    ... on Post {
      id
      title
      content
    }
  }
}
```

Here:

* If the result is a `User`, GraphQL returns `id`, `name`, and `email`.
* If the result is a `Post`, GraphQL returns `id`, `title`, and `content`.

---

## Fragment spreads

The syntax:

```graphql
...UserFields
```

is called a **fragment spread** because it "spreads" the fields defined in the fragment into the current selection.

---

## Benefits of fragments

* **Avoid duplication:** Define common fields once and reuse them.
* **Improve maintainability:** Update a field selection in one place instead of many.
* **Keep queries readable:** Large queries become easier to understand.
* **Support type-specific fields:** Inline fragments let you query different fields based on the actual returned type.

### Summary

| Concept         | Example                                                                 |
| --------------- | ----------------------------------------------------------------------- |
| Named fragment  | `fragment UserFields on User { id name email }`                         |
| Use a fragment  | `...UserFields`                                                         |
| Inline fragment | `... on User { id name }`                                               |
| Main purpose    | Reuse field selections and handle polymorphic types (interfaces/unions) |

Fragments are especially valuable in larger applications, where the same object (such as `User` or `Post`) is queried in many places and you want a single, consistent definition of the fields to fetch.

---

# 12. Inline Fragments

**Inline fragments** are used when you want to select fields **only if the returned object is a particular type**. They are most useful with **interfaces** and **unions**, where a field can return different object types.

Unlike named fragments, inline fragments are written directly inside the query and don't have a separate name.

## Syntax

```graphql
... on TypeName {
  field1
  field2
}
```

* `...` indicates a fragment.
* `on TypeName` specifies the type it applies to.
* The fields inside are returned only if the object is of that type.

---

## Example with a Union

Suppose your schema has:

```graphql
union SearchResult = User | Post
```

And the types:

```graphql
type User {
  id: ID!
  name: String!
  email: String!
}

type Post {
  id: ID!
  title: String!
  content: String!
}
```

The query:

```graphql
query {
  search {
    ... on User {
      id
      name
      email
    }

    ... on Post {
      id
      title
      content
    }
  }
}
```

### What happens?

Imagine `search` returns:

* A `User`
* A `Post`

The response could be:

```json
{
  "data": {
    "search": [
      {
        "id": "1",
        "name": "Alice",
        "email": "alice@example.com"
      },
      {
        "id": "10",
        "title": "Learning GraphQL",
        "content": "..."
      }
    ]
  }
}
```

For the first item (a `User`), GraphQL executes the `... on User` block.

For the second item (a `Post`), GraphQL executes the `... on Post` block.

---

## Example with an Interface

Suppose you have:

```graphql
interface Animal {
  id: ID!
  name: String!
}

type Dog implements Animal {
  id: ID!
  name: String!
  breed: String!
}

type Cat implements Animal {
  id: ID!
  name: String!
  livesLeft: Int!
}
```

Query:

```graphql
query {
  animals {
    id
    name

    ... on Dog {
      breed
    }

    ... on Cat {
      livesLeft
    }
  }
}
```

Here:

* Every `Animal` has `id` and `name`, so those are queried normally.
* Only `Dog` objects have `breed`.
* Only `Cat` objects have `livesLeft`.

---

## Why not just request every field?

This would be invalid:

```graphql
query {
  animals {
    id
    name
    breed
    livesLeft
  }
}
```

because the `Animal` interface doesn't define `breed` or `livesLeft`. GraphQL doesn't know whether every `Animal` has those fields.

Inline fragments tell GraphQL:

> "If this object is a `Dog`, then fetch `breed`."

---

## Named fragment vs. inline fragment

### Named fragment

Reusable:

```graphql
fragment UserFields on User {
  id
  name
}

query {
  users {
    ...UserFields
  }
}
```

---

### Inline fragment

Not reusable; written where it's needed:

```graphql
query {
  search {
    ... on User {
      id
      name
    }

    ... on Post {
      id
      title
    }
  }
}
```

---

## Inline fragment on the same type

You can technically write:

```graphql
query {
  user(id: "1") {
    ... on User {
      id
      name
      email
    }
  }
}
```

But this is unnecessary because `user` already returns a `User`. It's equivalent to:

```graphql
query {
  user(id: "1") {
    id
    name
    email
  }
}
```

Inline fragments are most useful when the exact runtime type **isn't known in advance**, such as with interfaces or unions.

---

## Summary

| Named Fragment                              | Inline Fragment                                     |
| ------------------------------------------- | --------------------------------------------------- |
| Has a name                                  | No name                                             |
| Can be reused                               | Used only where written                             |
| Good for avoiding repeated field selections | Good for selecting fields based on the runtime type |
| Example: `...UserFields`                    | Example: `... on User { name }`                     |

**Rule of thumb:**

* Use **named fragments** to reuse the same field selections in multiple places.
* Use **inline fragments** when a field may return different object types (typically interfaces or unions), and you need different fields depending on the actual type returned.

---

# 13. Directives in Queries

Great! Let's move on to **directives**.

A **directive** is an instruction you give to GraphQL that tells it **how to execute a query or interpret part of the schema**.

Think of them as **modifiers** that change GraphQL's behavior.

They always start with `@`.

---

# Example 1: `@include`

Suppose you have this query:

```graphql
query {
  user(id: "1") {
    id
    name
    email
  }
}
```

Normally, GraphQL always returns all three fields.

Now suppose your application has a checkbox:

> Show email? ✅ / ❌

If the user doesn't want to see the email, you don't want to request it.

You can use the `@include` directive.

```graphql
query GetUser($showEmail: Boolean!) {
  user(id: "1") {
    id
    name
    email @include(if: $showEmail)
  }
}
```

Variables:

```json
{
  "showEmail": true
}
```

Response:

```json
{
  "data": {
    "user": {
      "id": "1",
      "name": "Alice",
      "email": "alice@example.com"
    }
  }
}
```

If

```json
{
  "showEmail": false
}
```

Response:

```json
{
  "data": {
    "user": {
      "id": "1",
      "name": "Alice"
    }
  }
}
```

Notice the `email` field is omitted.

---

# Example 2: `@skip`

This is the opposite of `@include`.

```graphql
query GetUser($hideEmail: Boolean!) {
  user(id: "1") {
    id
    name
    email @skip(if: $hideEmail)
  }
}
```

If

```json
{
  "hideEmail": true
}
```

GraphQL skips the field.

Response

```json
{
  "data": {
    "user": {
      "id": "1",
      "name": "Alice"
    }
  }
}
```

If

```json
{
  "hideEmail": false
}
```

Then GraphQL returns email.

---

# Difference

`@include`

```graphql
email @include(if: true)
```

means

> Include this field.

---

`@skip`

```graphql
email @skip(if: true)
```

means

> Skip this field.

---

# Why are directives useful?

Without directives you might write two separate queries.

Query 1

```graphql
query {
  user(id: "1") {
    id
    name
  }
}
```

Query 2

```graphql
query {
  user(id: "1") {
    id
    name
    email
  }
}
```

Instead, write one query and control it with a variable.

---

# Schema directives

Directives are also used in schema definitions.

Example:

```graphql
type User {
  id: ID!
  email: String! @deprecated(reason: "Use primaryEmail instead")
}
```

The `@deprecated` directive tells clients:

> This field still exists, but avoid using it in new code.

Instead use another field.

---

# Custom directives

GraphQL also lets you create your own directives.

Example:

```graphql
type User {
  password: String! @masked
}
```

Here `@masked` isn't built into GraphQL.

Your server can define what it means.

For example:

* Hide the password
* Return `******`
* Log access
* Check permissions

Another example:

```graphql
type Query {
  users: [User] @auth
}
```

A custom `@auth` directive might mean:

> Only authenticated users can access this field.

---

# Where can directives be used?

They can be applied to:

* Fields

```graphql
email @skip(if: true)
```

* Fragment spreads

```graphql
...UserFields @include(if: $showUser)
```

* Inline fragments

```graphql
... on User @include(if: $isAdmin) {
  email
}
```

* Schema definitions (such as `@deprecated` or custom directives)

---

# Summary

| Directive                        | Purpose                                                                    |
| -------------------------------- | -------------------------------------------------------------------------- |
| `@include(if: Boolean)`          | Include a field only if the condition is `true`.                           |
| `@skip(if: Boolean)`             | Skip a field if the condition is `true`.                                   |
| `@deprecated(reason: "...")`     | Mark a schema field or enum value as deprecated.                           |
| Custom directives (e.g. `@auth`) | Add server-defined behavior such as authorization, logging, or validation. |

**In simple terms:** a directive is an instruction attached to part of a GraphQL query or schema that tells GraphQL (or your server) to do something special with that field, fragment, or type.

---

# 14. Pagination

Pagination in GraphQL is a way to **fetch data in small chunks (pages)** instead of loading everything at once.

For example, imagine you have **10,000 posts**.

Without pagination:

```graphql
query {
  posts {
    id
    title
  }
}
```

The server tries to return all 10,000 posts, which is slow and wastes bandwidth.

Instead, you fetch only a few at a time.

---

# Method 1: Offset Pagination (Simple)

This is similar to SQL's `LIMIT` and `OFFSET`.

Schema:

```graphql
type Query {
  posts(limit: Int!, offset: Int!): [Post!]!
}
```

Query:

```graphql
query {
  posts(limit: 5, offset: 0) {
    id
    title
  }
}
```

Meaning:

* `limit: 5` → return 5 posts.
* `offset: 0` → start from the beginning.

Suppose your posts are:

```text
1
2
3
4
5
6
7
8
9
10
```

### Page 1

```graphql
posts(limit: 5, offset: 0)
```

Returns:

```text
1
2
3
4
5
```

### Page 2

```graphql
posts(limit: 5, offset: 5)
```

Returns:

```text
6
7
8
9
10
```

### Pros

* Easy to understand.
* Easy to implement.

### Cons

Imagine someone inserts a new post while you're paging:

Before:

```text
1 2 3 4 5 6 7 8
```

You fetch page 1:

```text
1 2 3
```

Now a new post is inserted at the top:

```text
NEW 1 2 3 4 5 6 7 8
```

Then you fetch:

```graphql
offset: 3
```

Now you get:

```text
3 4 5
```

Notice `3` appears twice because the list shifted.

This is why large applications often avoid offset pagination.

---

# Method 2: Cursor Pagination (Recommended)

Instead of saying:

> "Skip 20 rows."

You say:

> "Start after this specific item."

Schema:

```graphql
type Query {
  posts(first: Int!, after: String): PostConnection!
}
```

Here:

* `first` = how many items to fetch.
* `after` = the cursor from the last item of the previous page.

Suppose the server returns:

```json
{
  "posts": {
    "edges": [
      {
        "cursor": "abc123",
        "node": {
          "id": "1",
          "title": "GraphQL"
        }
      },
      {
        "cursor": "def456",
        "node": {
          "id": "2",
          "title": "Apollo"
        }
      }
    ],
    "pageInfo": {
      "hasNextPage": true,
      "endCursor": "def456"
    }
  }
}
```

The client stores:

```text
endCursor = "def456"
```

Next request:

```graphql
query {
  posts(first: 2, after: "def456") {
    edges {
      node {
        id
        title
      }
    }
    pageInfo {
      hasNextPage
      endCursor
    }
  }
}
```

The server returns the next two posts.

---

# Why use a cursor?

Suppose the database changes.

Before:

```text
1 2 3 4 5
```

You read:

```text
1 2
```

Cursor = item 2.

Now someone inserts:

```text
NEW 1 2 3 4 5
```

You ask:

> Give me items **after cursor 2**.

The server still returns:

```text
3 4
```

No duplicates and no skipped items.

---

# Connection Pattern

Cursor pagination often follows the Relay Connection specification.

Schema:

```graphql
type PostConnection {
  edges: [PostEdge!]!
  pageInfo: PageInfo!
}

type PostEdge {
  cursor: String!
  node: Post!
}

type PageInfo {
  hasNextPage: Boolean!
  endCursor: String
}
```

* **`node`** is the actual data (`Post`).
* **`cursor`** identifies where that item is in the sequence.
* **`pageInfo`** tells the client whether more pages exist and what cursor to use next.

---

# Example flow

1. First request:

```graphql
query {
  posts(first: 2) {
    edges {
      cursor
      node {
        id
        title
      }
    }
    pageInfo {
      hasNextPage
      endCursor
    }
  }
}
```

2. Response includes:

```text
Cursor of last post = xyz789
```

3. Next request:

```graphql
query {
  posts(first: 2, after: "xyz789") {
    ...
  }
}
```

This continues until:

```json
{
  "pageInfo": {
    "hasNextPage": false
  }
}
```

which tells the client there are no more pages.

---

# Offset vs. Cursor

| Feature                                | Offset Pagination  | Cursor Pagination                        |
| -------------------------------------- | ------------------ | ---------------------------------------- |
| Uses                                   | `limit` + `offset` | `first` + `after` (or `last` + `before`) |
| Easy to implement                      | ✅                  | Slightly more involved                   |
| Works well for small/static datasets   | ✅                  | ✅                                        |
| Stable if rows are inserted or deleted | ❌                  | ✅                                        |
| Preferred for feeds and large datasets | ❌                  | ✅                                        |

### Rule of thumb

* For a small admin dashboard or internal tool, **offset pagination** is often sufficient.
* For production APIs with frequently changing data (social feeds, comments, products, messages), **cursor pagination** is generally the preferred approach because it provides more stable navigation through the data.


---

# Android Infinite Scroll Example

Flow:

```
RecyclerView

↓

Load next page

↓

Apollo fetchMore()

↓

Append items

↓

Update UI
```

Common in:

* Instagram
* Twitter
* Facebook feeds

---

# 15. Filtering

Filtering reduces unnecessary data.

Example:

Without filter:

```graphql
query{

 products{

    name

 }

}
```

Returns everything.

---

With filter:

```graphql
query{

 products(
    category:"Laptop"
 ){

    name

 }

}
```

Only laptops.

---

# 16. Sorting

Example:

Newest products first:

```graphql
query{

 products(
    sortBy:CREATED_DATE
 ){

    name

 }

}
```

Common sorting options:

```
PRICE_ASC

PRICE_DESC

NEWEST

POPULAR
```

---

# 17. Query Execution in Apollo Kotlin

Architecture:

```
Compose UI

↓

ViewModel

↓

Repository

↓

Apollo Client

↓

GraphQL Server

↓

Response

↓

Apollo Models

↓

UI
```

---

Example:

```kotlin
class UserRepository(
    private val apollo:ApolloClient
){

 suspend fun getUser(id:String){

    return apollo
        .query(
            GetUserQuery(id)
        )
        .execute()

 }

}
```

---

ViewModel:

```kotlin
viewModelScope.launch {

 val response =
 repository.getUser("10")

 user.value =
 response.data?.user

}
```

---

# 18. Error Handling

Apollo response:

```kotlin
val response =
apolloClient
.query(GetUserQuery("10"))
.execute()
```

Check:

```kotlin
if(response.hasErrors()){

    println(response.errors)

}
```

Data:

```kotlin
response.data?.user
```

Errors:

```kotlin
response.errors
```

---

# 19. Query Optimization

## Avoid requesting unused fields

Bad:

```graphql
user{

 id

 name

 email

 address

 phone

}
```

If UI only needs:

```
name
```

request only:

```graphql
user{

 name

}
```

---

## Avoid Deep Nesting

Bad:

```graphql
user{

 posts{

  comments{

   replies{

    author{

     friends{

      posts{

      }

     }

    }

   }

  }

 }

}
```

Very expensive.

---

## Use Pagination

Never:

```graphql
posts{

 title

}
```

for millions of records.

Use:

```graphql
posts(first:20)
```

---

# 20. Summary

* Queries retrieve data from GraphQL servers.
* Queries contain operation names, fields, arguments, variables, and selection sets.
* Fields determine exactly what data is returned.
* Variables make queries reusable.
* Aliases allow multiple requests for the same field.
* Fragments reduce duplication.
* Inline fragments work with interfaces and unions.
* Pagination prevents loading huge datasets.
* Apollo Kotlin converts GraphQL queries into type-safe Kotlin code.

---

# 21. Interview Questions

### 1. What is a GraphQL Query?

A query is a read operation used to request data from a GraphQL server.

---

### 2. Why does GraphQL require a selection set?

Because GraphQL needs to know exactly which fields of an object should be returned.

---

### 3. Difference between arguments and variables?

This is one of the most important GraphQL concepts. The difference is:

* **Arguments** are the **inputs that a field accepts**.
* **Variables** are **values supplied separately** and used to fill those arguments.

Think of it like a function in JavaScript.

## JavaScript analogy

```javascript
function getUser(id) {
  // ...
}

getUser(1);
```

Here:

* `id` is the **parameter** (similar to a GraphQL argument).
* `1` is the **value** you pass (similar to a GraphQL variable or literal value).

---

## Arguments

Suppose your schema has:

```graphql
type Query {
  user(id: ID!): User
}
```

Here,

```graphql
user(id: ID!)
```

means the `user` field accepts an argument called `id`.

You can call it directly:

```graphql
query {
  user(id: "1") {
    name
    email
  }
}
```

Here:

* `user` → field
* `id` → argument
* `"1"` → literal value passed to the argument

---

## Variables

Instead of hardcoding `"1"` into the query, you can use a variable.

```graphql
query GetUser($userId: ID!) {
  user(id: $userId) {
    name
    email
  }
}
```

Notice:

```graphql
$userId
```

This is a **variable**.

Then you send the value separately:

```json
{
  "userId": "1"
}
```

GraphQL substitutes:

```graphql
user(id: $userId)
```

becomes

```graphql
user(id: "1")
```

before executing the query.

---

## Why use variables?

Without variables:

```graphql
query {
  user(id: "1") {
    name
  }
}
```

To fetch another user, you must change the query:

```graphql
query {
  user(id: "2") {
    name
  }
}
```

With variables, the query stays the same:

```graphql
query GetUser($userId: ID!) {
  user(id: $userId) {
    name
  }
}
```

Only the variables change:

User 1:

```json
{
  "userId": "1"
}
```

User 2:

```json
{
  "userId": "2"
}
```

This is especially useful because clients can reuse the same query while sending different values.

---

## Another example

Schema:

```graphql
type Query {
  posts(limit: Int!, offset: Int!): [Post!]!
}
```

### Using arguments directly

```graphql
query {
  posts(limit: 5, offset: 0) {
    title
  }
}
```

Here:

* `limit` and `offset` are arguments.
* `5` and `0` are literal values.

---

### Using variables

```graphql
query GetPosts($limit: Int!, $offset: Int!) {
  posts(limit: $limit, offset: $offset) {
    title
  }
}
```

Variables:

```json
{
  "limit": 5,
  "offset": 0
}
```

Again, the variables provide the values for the arguments.

---

## Relationship

You can think of it like this:

```text
Schema
------
user(id: ID!)

        ▲
        │
     Argument

Query
-----
user(id: $userId)

        ▲
        │
     Variable

Variables JSON
--------------
{
  "userId": "1"
}
```

---

## Summary

| Arguments                                   | Variables                                                       |
| ------------------------------------------- | --------------------------------------------------------------- |
| Defined by the schema as inputs to a field. | Defined in the query and supplied separately at execution time. |
| Example: `user(id: ID!)`                    | Example: `$userId: ID!`                                         |
| Used when calling a field: `user(id: ...)`  | Used to provide the value: `user(id: $userId)`                  |
| Part of the schema and query syntax.        | Not part of the schema; they are part of the request.           |

### A simple analogy

Imagine an online food ordering app:

* The restaurant menu says: **"Choose a spice level."** → That's the **argument**.
* You select **"Medium"** when placing your order. → That's the **variable value**.

The menu (schema) defines what can be chosen, while your order (request) supplies the actual choice.

 The confusing part is that **a GraphQL request actually has two parts**:

1. The **query** (the structure of what you want).
2. The **variables** (the values to use).

Think of it like filling out a form.

---

## Part 1: The query

```graphql
query GetUser($userId: ID!) {
  user(id: $userId) {
    name
    email
  }
}
```

Notice that `$userId` is just a **placeholder**. It doesn't have a value yet.

---

## Part 2: The variables

Along with the query, the client sends:

```json
{
  "userId": "1"
}
```

So the complete request looks something like this:

```json
{
  "query": "query GetUser($userId: ID!) { user(id: $userId) { name email } }",
  "variables": {
    "userId": "1"
  }
}
```

The GraphQL server receives **both** the query and the variables.

---

## What the server does

The server sees:

Query:

```graphql
user(id: $userId)
```

Variables:

```json
{
  "userId": "1"
}
```

So it knows:

> `$userId` has the value `"1"`.

It then executes the query **as if** it were:

```graphql
user(id: "1")
```

You don't actually write or send this second version. It's just a way to understand what happens internally.

---

## JavaScript analogy

Imagine this function:

```javascript
function getUser(id) {
  console.log(id);
}

const userId = "1";

getUser(userId);
```

Here:

```javascript
getUser(userId);
```

is **not** literally:

```javascript
getUser("1");
```

But since:

```javascript
userId = "1";
```

JavaScript passes `"1"` to the function.

GraphQL variables work the same way.

---

## Another example

Query:

```graphql
query GetPosts($limit: Int!) {
  posts(limit: $limit) {
    title
  }
}
```

Variables:

```json
{
  "limit": 5
}
```

The server executes it as if you had written:

```graphql
query {
  posts(limit: 5) {
    title
  }
}
```

---

### Key idea

* **Argument** = the field's input.

  ```graphql
  user(id: ...)
  ```

* **Variable** = the value supplied for that input.

  ```graphql
  $userId
  ```

* **Variables JSON** = where the actual value comes from.

  ```json
  {
    "userId": "1"
  }
  ```

So the flow is:

```text
Query
-----
user(id: $userId)
          │
          ▼
Variables
---------
{
  "userId": "1"
}
          │
          ▼
Execution
---------
user(id: "1")
```

**Question for you:** Which part is confusing?

1. Why the query and variables are sent separately?
2. Why we write `$userId` in the query?
3. How the server matches `"userId"` in the JSON with `$userId` in the query?


---

### 4. What are fragments?

Fragments are reusable pieces of query fields that reduce duplication.

---

### 5. What are aliases?

Aliases allow the same field to be requested multiple times with different names in the response.

---

### 6. What pagination methods are common in GraphQL?

Two common approaches:

* Offset pagination
* Cursor-based pagination

Cursor pagination is preferred for large datasets.

---

### 7. How does Apollo Kotlin execute queries?

Apollo Kotlin generates query classes from `.graphql` files and executes them through `ApolloClient`, converting responses into strongly typed Kotlin models.

---

# 📌 Next Chapter (Chapter 8 – GraphQL Mutations Deep Dive)

Next chapter covers:

* Creating data
* Updating data
* Deleting data
* Mutation syntax
* Input objects
* Optimistic updates
* Mutation error handling
* Cache updates
* Apollo Kotlin mutation examples
* Real Android examples (Login, Signup, Create Post, Update Profile)
