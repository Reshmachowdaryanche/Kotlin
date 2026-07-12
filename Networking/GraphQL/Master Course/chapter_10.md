# Chapter 10 – GraphQL Variables, Arguments & Directives Deep Dive

## Making GraphQL Operations Dynamic and Powerful

## GraphQL for Android Developers

> **Learning Goal**
>
> By the end of this chapter, you will understand:
>
> * What GraphQL arguments are
> * What GraphQL variables are
> * Difference between arguments and variables
> * How variables work internally
> * Variable types and nullability
> * Default variable values
> * Input validation
> * GraphQL directives
> * Built-in directives
> * Custom directives
> * How Apollo Kotlin handles variables
> * Real Android examples

---

# Table of Contents

1. Introduction
2. Arguments in GraphQL
3. Why Arguments Are Needed
4. Variables in GraphQL
5. Arguments vs Variables
6. Variable Declaration
7. Variable Types
8. Non-Null Variables
9. Nullable Variables
10. Default Values
11. Multiple Variables
12. Input Objects with Variables
13. Directives Introduction
14. @include Directive
15. @skip Directive
16. @deprecated Directive
17. Custom Directives
18. Apollo Kotlin Variable Handling
19. Real Android Examples
20. Best Practices
21. Summary
22. Interview Questions

---

# 1. Introduction

GraphQL operations need dynamic values.

Example:

A user profile screen:

```text
User A opens profile

↓

Load user id = 10


User B opens profile

↓

Load user id = 20
```

The query structure is the same.

Only the value changes.

GraphQL provides two mechanisms:

1. Arguments
2. Variables

---

# 2. Arguments in GraphQL

Arguments are values passed directly to fields.

Example:

Schema:

```graphql
type Query {

    user(
        id: ID!
    ): User

}
```

Query:

```graphql
query {

    user(id:10){

        name

    }

}
```

Here:

```graphql
id:10
```

is an argument.

---

## Real Example

Product search:

```graphql
query {

    products(
        category:"mobile"
    ){

        name

        price

    }

}
```

Argument:

```text
category = mobile
```

The server uses this value to filter data.

---

# 3. Why Arguments Are Needed

Without arguments:

```graphql
query {

    users {

        name

    }

}
```

The server returns:

```text
All users
```

Maybe:

```
1 million users
```

Not practical.

---

With arguments:

```graphql
query {

    users(
        limit:10
    ){

        name

    }

}
```

Server returns:

```
Only 10 users
```

---

Arguments are used for:

* Filtering
* Searching
* Pagination
* Sorting
* Selecting specific objects

---

# 4. Variables in GraphQL

Variables separate data from the query.

Instead of:

```graphql
query {

    user(id:10){

        name

    }

}
```

Use:

```graphql
query GetUser($id:ID!){

    user(id:$id){

        name

    }

}
```

Variable:

```json
{
 "id":10
}
```

---

## Why Use Variables?

Advantages:

### 1. Reusable Queries

Same query:

```
GetUser
```

works for:

```
User 1

User 2

User 3
```

---

### 2. Better Security

Avoid building query strings manually.

Bad:

```kotlin
"user(id=$id)"
```

---

Good:

```kotlin
GetUserQuery(id)
```

---

### 3. Better Caching

Apollo can identify:

```
GetUser(id=10)

GetUser(id=20)
```

as different operations.

---

# 5. Arguments vs Variables

Many developers confuse these.

## Arguments

Values inside the query.

Example:

```graphql
user(id:10)
```

`id:10` is an argument.

---

## Variables

Values provided separately.

Example:

Query:

```graphql
query($id:ID!){
    user(id:$id)
}
```

Variables:

```json
{
"id":10
}
```

---

Comparison:

| Feature      | Arguments             | Variables            |
| ------------ | --------------------- | -------------------- |
| Location     | Inside query          | Outside query        |
| Purpose      | Pass values to fields | Store dynamic values |
| Reusable     | No                    | Yes                  |
| Mobile usage | Less common           | Very common          |

---

# 6. Variable Declaration

Syntax:

```graphql
query OperationName(
    $variableName: Type
)
```

Example:

```graphql
query GetUser(
    $id:ID!
){

    user(id:$id){

        name

    }

}
```

Breakdown:

```text
$id

↓

Variable name


ID!

↓

Variable type
```

---

# 7. Variable Types

GraphQL variables use GraphQL types.

Example:

```graphql
query(
    $id:ID!
    $age:Int
    $name:String
){

}
```

Available types:

```
String

Int

Float

Boolean

ID
```

---

Lists:

```graphql
$ids:[ID!]
```

Example:

```json
{
 "ids":[1,2,3]
}
```

---

Custom types:

```graphql
$input:CreateUserInput!
```

---

# 8. Non-Null Variables

The `!` means required.

Example:

```graphql
$id:ID!
```

Means:

```
id must exist
```

Valid:

```json
{
"id":10
}
```

Invalid:

```json
{}
```

Error:

```
Variable "$id" of required type "ID!" was not provided.
```

---

# 9. Nullable Variables

Without `!`:

```graphql
$id:ID
```

The value can be missing.

Example:

```graphql
query Search(
    $keyword:String
){

products(
    search:$keyword
){

name

}

}
```

Possible variables:

```json
{}
```

or

```json
{
"keyword":"phone"
}
```

---

# 10. Default Values

Variables can have default values.

Example:

```graphql
query Products(
    $limit:Int = 10
){

products(
    limit:$limit
){

name

}

}
```

If no value is provided:

```json
{}
```

GraphQL uses:

```
limit = 10
```

---

# 11. Multiple Variables

Real applications usually have many variables.

Example:

Product listing:

```graphql
query Products(
    $category:String!
    $limit:Int!
    $page:Int!
){

products(
 category:$category
 limit:$limit
 page:$page
){

id

name

price

}

}
```

Variables:

```json
{
 "category":"mobile",
 "limit":20,
 "page":1
}
```

---

# 12. Input Objects with Variables

Large mutations use input objects.

Example:

Schema:

```graphql
input RegisterInput{

name:String!

email:String!

password:String!

}
```

Mutation:

```graphql
mutation Register(
    $input:RegisterInput!
){

register(
    input:$input
){

id

name

}

}
```

Variables:

```json
{
 "input":{
    "name":"John",
    "email":"john@gmail.com",
    "password":"123456"
 }
}
```

---

# 13. Directives Introduction

Directives modify the behavior of GraphQL operations.

Syntax:

```graphql
field @directive
```

Example:

```graphql
email @skip(if:true)
```

Meaning:

```
Do not include email
```

---

Common built-in directives:

| Directive    | Purpose                     |
| ------------ | --------------------------- |
| @include     | Include field conditionally |
| @skip        | Skip field conditionally    |
| @deprecated  | Mark old fields             |
| @specifiedBy | Define scalar specification |

---

# 14. @include Directive

`@include` returns a field only when a condition is true.

Example:

```graphql
query User(
    $showEmail:Boolean!
){

user(id:10){

name

email @include(
    if:$showEmail
)

}

}
```

Variables:

```json
{
"showEmail":true
}
```

Response:

```json
{
"name":"John",
"email":"john@gmail.com"
}
```

---

If:

```json
{
"showEmail":false
}
```

Response:

```json
{
"name":"John"
}
```

---

## Android Example

Profile screen:

```text
Basic profile:

Name

Photo


Detailed profile:

Name

Photo

Email

Phone
```

Use:

```graphql
@include
```

instead of creating two queries.

---

# 15. @skip Directive

Opposite of include.

Example:

```graphql
query User(
    $hideEmail:Boolean!
){

user(id:10){

name

email @skip(
    if:$hideEmail
)

}

}
```

Variable:

```json
{
"hideEmail":true
}
```

Result:

```json
{
"name":"John"
}
```

---

# 16. @deprecated Directive

Used in schemas.

Example:

```graphql
type User{

id:ID!

name:String!

username:String
@deprecated(
reason:"Use email instead"
)

email:String!

}
```

Meaning:

The field still works, but clients should stop using it.

---

Example:

Old:

```graphql
username
```

New:

```graphql
email
```

---

# 17. Custom Directives

Servers can create custom directives.

Example:

Authentication:

```graphql
type Query{

adminData:String
@auth

}
```

Meaning:

Only authenticated users can access.

---

Another example:

Permission:

```graphql
deleteUser:
User
@requiresRole(
role:"ADMIN"
)
```

---

# 18. Apollo Kotlin Variable Handling

Apollo generates Kotlin classes from variables.

GraphQL:

```graphql
query GetUser(
$id:ID!
){

user(id:$id){

name

}

}
```

Generated Kotlin:

```kotlin
GetUserQuery(
    id = "10"
)
```

---

Example:

```kotlin
val query =
    GetUserQuery(
        id="100"
    )


val response =
    apolloClient
        .query(query)
        .execute()
```

---

# 19. Real Android Examples

## Example 1: Search Screen

User types:

```
phone
```

Query:

```graphql
query Search(
$keyword:String!
){

search(
keyword:$keyword
){

id

title

}

}
```

Variables:

```json
{
"keyword":"phone"
}
```

---

Android:

```kotlin
fun search(text:String){

viewModelScope.launch{

apolloClient
.query(
SearchQuery(text)
)
.execute()

}

}
```

---

# Example 2: Pagination

Query:

```graphql
query Products(
$page:Int!
$limit:Int!
){

products(
page:$page
limit:$limit
){

name

}

}
```

Variables:

```json
{
"page":1,
"limit":20
}
```

---

# Example 3: Conditional UI Data

Profile:

```graphql
query Profile(
$loadPrivate:Boolean!
){

viewer{

name

email @include(
if:$loadPrivate
)

}

}
```

---

# 20. Best Practices

## 1. Always use variables

Avoid:

```graphql
user(id:10)
```

Prefer:

```graphql
user(id:$id)
```

---

## 2. Use meaningful names

Good:

```graphql
$userId

$productId

$pageSize
```

Avoid:

```graphql
$x

$a
```

---

## 3. Use Non-Null Carefully

Good:

```graphql
$id:ID!
```

when required.

Avoid:

```graphql
$email:String!
```

if email is optional.

---

## 4. Use Directives for UI variations

Good:

```graphql
email @include
```

Instead of:

```
Create many similar queries
```

---

## 5. Keep Variables Close to Operation

Example:

```graphql
query ProductDetails(
$productId:ID!
){

product(id:$productId)

}
```

Easy to understand.

---

# 21. Summary

* Arguments pass values directly to GraphQL fields.
* Variables make operations dynamic and reusable.
* Variables improve caching, security, and maintainability.
* `!` means required/non-null.
* Input objects organize complex data.
* Directives modify query behavior.
* `@include` conditionally includes fields.
* `@skip` conditionally removes fields.
* `@deprecated` helps evolve APIs safely.
* Apollo Kotlin generates type-safe variable classes automatically.

---

# 22. Interview Questions

### 1. What is the difference between arguments and variables?

Arguments are values passed to fields. Variables are reusable parameters defined outside the query and supplied separately.

---

### 2. Why should we use GraphQL variables?

Variables make queries reusable, safer, easier to cache, and easier to maintain.

---

### 3. What does `!` mean in GraphQL?

It represents a non-null type. The value is required and cannot be null.

---

### 4. What are GraphQL directives?

Directives modify the behavior of fields or operations.

Examples:

* `@include`
* `@skip`
* `@deprecated`

---

### 5. Difference between @include and @skip?

`@include` adds a field when a condition is true.

`@skip` removes a field when a condition is true.

---

### 6. How does Apollo Kotlin handle variables?

Apollo generates strongly typed Kotlin classes for GraphQL operations. Variables are passed through these generated classes when executing queries or mutations.

---

# 📌 Next Chapter (Chapter 11 – GraphQL Fragments & Reusable Data Models Deep Dive)

Next chapter will cover:

* What fragments are
* Why fragments are important in Android
* Fragment composition
* Named fragments
* Inline fragments
* Fragment reuse across screens
* Apollo Kotlin fragment generation
* Compose UI + fragments architecture
* Real project examples (User profile, Feed, Chat UI)
