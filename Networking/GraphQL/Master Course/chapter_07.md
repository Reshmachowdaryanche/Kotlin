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

Used with:

* Interfaces
* Unions

Example:

Schema:

```graphql
union SearchResult =
User |
Product
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

Meaning:

"If the result is User, get name."

"If the result is Product, get price."

---

# 13. Directives in Queries

Directives modify query behavior.

Two common directives:

```
@include

@skip
```

---

## @include

Include field conditionally.

Query:

```graphql
query($showEmail:Boolean!){

 user(id:1){

    name

    email @include(if:$showEmail)

 }

}
```

Variable:

```json
{
 "showEmail":true
}
```

Result:

```json
{
 "name":"John",
 "email":"john@gmail.com"
}
```

---

## @skip

Skip a field.

```graphql
query($hideEmail:Boolean!){

 user(id:1){

    name

    email @skip(if:$hideEmail)

 }

}
```

---

# 14. Pagination

Large datasets cannot be downloaded at once.

Example:

A social media app:

```
10 million posts
```

Impossible to load everything.

Pagination solves this.

---

## Offset Pagination

Similar to SQL:

```text
page=1

limit=20
```

GraphQL:

```graphql
query{

 posts(
    offset:20
    limit:10
 ){

    title

 }

}
```

---

## Cursor Pagination

Modern GraphQL APIs usually prefer cursor pagination.

Example:

```graphql
query{

 posts(
    first:10
    after:"cursor123"
 ){

    edges{

        node{

            title

        }

    }

 }

}
```

Response:

```json
{
 "edges":[
   {
    "node":{
       "title":"Post 1"
    }
   }
 ]
}
```

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

Arguments are values passed to fields. Variables are reusable parameters supplied separately from the query.

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
