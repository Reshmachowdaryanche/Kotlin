# Chapter 8 – GraphQL Mutations Deep Dive

## Creating, Updating, and Deleting Data in GraphQL

## GraphQL for Android Developers

> **Learning Goal**
>
> By the end of this chapter, you will understand:
>
> * What GraphQL Mutations are
> * Difference between Queries and Mutations
> * Mutation syntax
> * Creating data
> * Updating data
> * Deleting data
> * Input objects in mutations
> * Mutation variables
> * Mutation responses
> * Mutation errors
> * Optimistic updates
> * Cache updates with Apollo Kotlin
> * Real Android mutation examples

---

# Table of Contents

1. Introduction to Mutations
2. Query vs Mutation
3. Mutation Structure
4. Creating Data
5. Updating Data
6. Deleting Data
7. Mutation Arguments
8. Input Types
9. Mutation Variables
10. Mutation Responses
11. Error Handling
12. Optimistic Updates
13. Apollo Kotlin Mutation Implementation
14. Real Android Examples
15. Mutation Best Practices
16. Summary
17. Interview Questions

---

# 1. Introduction to Mutations

A **Mutation** is a GraphQL operation used to modify server data.

Mutation operations include:

* Create
* Update
* Delete

Examples:

```
Create User

Update Profile

Delete Post

Add Comment

Like Post

Upload Image
```

In REST APIs, these operations use:

```
POST
PUT
PATCH
DELETE
```

In GraphQL, all of them are represented using:

```graphql
mutation
```

---

# 2. Query vs Mutation

## Query

Used for reading data.

Example:

```graphql
query {
    user(id:10){
        name
    }
}
```

Equivalent REST:

```
GET /users/10
```

---

## Mutation

Used for changing data.

Example:

```graphql
mutation {
    createUser(
        name:"John"
    ){
        id
        name
    }
}
```

Equivalent REST:

```
POST /users
```

---

## Comparison

| Feature         | Query         | Mutation        |
| --------------- | ------------- | --------------- |
| Purpose         | Read data     | Change data     |
| Database effect | No change     | Changes data    |
| REST equivalent | GET           | POST/PUT/DELETE |
| Side effects    | No            | Yes             |
| Example         | Fetch profile | Create account  |

---

# 3. Mutation Structure

Basic mutation syntax:

```graphql
mutation {
    operationName(arguments){
        returnedFields
    }
}
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

Let's break it down.

---

## Mutation Keyword

```graphql
mutation
```

Tells GraphQL:

> "This request will modify data."

---

## Mutation Field

```graphql
createUser
```

This maps to a server resolver.

---

## Arguments

```graphql
name:"John"
email:"john@gmail.com"
```

Data sent to the server.

---

## Selection Set

```graphql
{
    id
    name
}
```

Data returned after modification.

---

# 4. Creating Data

The most common mutation is creating new data.

Example:

User registration.

Schema:

```graphql
type Mutation {

    createUser(
        name:String!
        email:String!
    ):User

}
```

Mutation:

```graphql
mutation {

    createUser(
        name:"John"
        email:"john@gmail.com"
    ){

        id

        name

        email

    }

}
```

Response:

```json
{
 "data":{
    "createUser":{
        "id":"101",
        "name":"John",
        "email":"john@gmail.com"
    }
 }
}
```

---

# 5. Updating Data

Updating existing information.

Example:

Change profile name.

Schema:

```graphql
type Mutation {

    updateUser(
        id:ID!
        name:String!
    ):User

}
```

Mutation:

```graphql
mutation {

    updateUser(
        id:"101"
        name:"Peter"
    ){

        id

        name

    }

}
```

Response:

```json
{
 "data":{
    "updateUser":{
        "id":"101",
        "name":"Peter"
    }
 }
}
```

---

# 6. Deleting Data

Deleting records.

Example:

Delete a post.

Schema:

```graphql
type Mutation {

    deletePost(
        id:ID!
    ):Boolean

}
```

Mutation:

```graphql
mutation {

    deletePost(
        id:"500"
    )

}
```

Response:

```json
{
 "data":{
    "deletePost":true
 }
}
```

---

# 7. Mutation Arguments

A mutation can receive multiple arguments.

Example:

```graphql
mutation {

    updateProfile(

        name:"John"

        age:30

        city:"London"

    ){

        id

        name

    }

}
```

This works, but it becomes difficult when there are many fields.

Imagine:

```text
name

email

phone

address

city

country

profileImage

birthday

gender
```

The mutation becomes huge.

Solution:

Input Types.

---

# 8. Input Types

Input types organize mutation data.

Instead of:

```graphql
mutation {

 createUser(
    name:String
    email:String
    age:Int
 )

}
```

Use:

```graphql
input CreateUserInput {

    name:String!

    email:String!

    age:Int!

}
```

Mutation:

```graphql
type Mutation {

    createUser(
        input:CreateUserInput!
    ):User

}
```

Now request:

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

## Why Input Types Are Better

Advantages:

### Cleaner mutations

Before:

```graphql
createUser(
name
email
phone
address
)
```

After:

```graphql
createUser(
input:CreateUserInput
)
```

---

### Easier API changes

Later you add:

```graphql
input CreateUserInput{

    name:String!

    email:String!

    age:Int!

    country:String

}
```

Existing clients continue working.

---

# 9. Mutation Variables

Avoid hardcoding values.

Bad:

```graphql
mutation{

 createUser(
    name:"John"
 )

}
```

---

Better:

```graphql
mutation CreateUser(
    $input:CreateUserInput!
){

    createUser(
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
    "age":25
 }
}
```

---

## Android Example

GraphQL file:

`CreateUser.graphql`

```graphql
mutation CreateUser(
    $input:CreateUserInput!
){

    createUser(
        input:$input
    ){

        id

        name

        email

    }

}
```

Kotlin:

```kotlin
val input =
    CreateUserInput(
        name="John",
        email="john@gmail.com",
        age=25
    )


val response =
    apolloClient
        .mutation(
            CreateUserMutation(input)
        )
        .execute()
```

---

# 10. Mutation Responses

After changing data, the server usually returns the updated object.

Example:

Mutation:

```graphql
mutation{

 updateUser(
    id:"10"
    name:"Alex"
 ){

    id

    name

 }

}
```

Response:

```json
{
 "data":{
    "updateUser":{
        "id":"10",
        "name":"Alex"
    }
 }
}
```

---

## Why Return Updated Data?

Because the client can immediately update UI.

Example:

User edits profile:

```
Before:

John


After:

Alex
```

The server returns:

```json
{
"name":"Alex"
}
```

Android updates screen instantly.

---

# 11. Mutation Error Handling

Mutations can fail.

Examples:

```
Email already exists

Permission denied

Invalid input

Server failure
```

Example:

```json
{
 "errors":[
    {
      "message":"Email already exists"
    }
 ]
}
```

---

Apollo Kotlin:

```kotlin
val response =
    apolloClient
        .mutation(
            CreateUserMutation(input)
        )
        .execute()


if(response.hasErrors()){

    println(response.errors)

}
```

---

# 12. Optimistic Updates

One important mobile concept.

Imagine:

User likes a post.

Without optimization:

```
Tap Like

↓

Network request

↓

Server response

↓

Update UI
```

The user waits.

---

With optimistic update:

```
Tap Like

↓

Immediately update UI

↓

Send request

↓

Confirm with server
```

The app feels instant.

---

Example:

Before:

```
Likes: 100
```

User taps:

```
Likes: 101
```

Immediately.

Later server confirms.

---

## Apollo Kotlin Optimistic Updates

Apollo supports optimistic cache updates.

Example:

```kotlin
apolloClient.mutation(
    LikePostMutation(postId)
)
```

The cache can temporarily store:

```text
likes = 101
```

until the server response arrives.

---

# 13. Apollo Kotlin Mutation Implementation

Architecture:

```
Compose Screen

        ↓

ViewModel

        ↓

Repository

        ↓

Apollo Client

        ↓

GraphQL Mutation

        ↓

Server

        ↓

Response

```

---

## Repository

```kotlin
class UserRepository(
    private val apolloClient: ApolloClient
){

suspend fun createUser(
    input:CreateUserInput
)
=
apolloClient
    .mutation(
        CreateUserMutation(input)
    )
    .execute()

}
```

---

## ViewModel

```kotlin
fun createUser(){

viewModelScope.launch {


val response =
repository.createUser(input)


if(response.data != null){

    _user.value =
       response.data?.createUser

}


}

}
```

---

# 14. Real Android Examples

## Example 1: Login

Mutation:

```graphql
mutation Login(
$username:String!
$password:String!
){

login(
username:$username
password:$password
){

token

user{

id

name

}

}

}
```

Response:

```json
{
"token":"abc123",
"user":{
"id":"10",
"name":"John"
}
}
```

---

# Example 2: Create Post

Mutation:

```graphql
mutation CreatePost(
$input:CreatePostInput!
){

createPost(
input:$input
){

id

title

createdAt

}

}
```

---

# Example 3: Add Comment

```graphql
mutation AddComment(
$postId:ID!
$text:String!
){

addComment(
postId:$postId
text:$text
){

id

text

author{

name

}

}

}
```

---

# Example 4: Update Profile Image

```graphql
mutation UpdateAvatar(
$userId:ID!
$url:String!
){

updateAvatar(
userId:$userId
url:$url
){

profileImage

}

}
```

---

# 15. Mutation Best Practices

## 1. Use meaningful names

Good:

```graphql
createUser

updateProfile

deletePost
```

Avoid:

```graphql
change1

action2
```

---

## 2. Use Input Objects

Good:

```graphql
createUser(input:CreateUserInput)
```

Avoid:

```graphql
createUser(
name,
email,
phone,
address
)
```

---

## 3. Return updated objects

Good:

```graphql
updateUser{

id

name

}
```

Avoid:

```graphql
updateUser:Boolean
```

Returning the object allows the UI to update immediately.

---

## 4. Validate on Server

Never trust client input.

Example:

Android sends:

```text
age=-100
```

Server must reject it.

---

## 5. Handle Loading States

Android UI:

```
Button clicked

↓

Show loading

↓

Mutation running

↓

Success/Error
```

---

# 16. Summary

* Mutations modify GraphQL data.
* They are used for create, update, and delete operations.
* Mutation structure contains:

  * Mutation name
  * Arguments
  * Returned fields
* Input objects organize mutation parameters.
* Variables make mutations reusable.
* Returning updated objects helps update UI quickly.
* Apollo Kotlin provides type-safe mutation execution.
* Optimistic updates improve mobile user experience.

---

# 17. Interview Questions

### 1. What is a GraphQL Mutation?

A mutation is a GraphQL operation used to modify server-side data such as creating, updating, or deleting resources.

---

### 2. Difference between Query and Mutation?

Query reads data without changing it. Mutation changes data and may have side effects.

---

### 3. Why use Input Types in mutations?

Input types group mutation arguments into structured objects, making APIs cleaner and easier to extend.

---

### 4. What are optimistic updates?

Optimistic updates update the UI immediately before receiving server confirmation, improving perceived performance.

---

### 5. Why should mutations return updated data?

Returning updated objects allows clients like Android apps to refresh UI state immediately without another query.

---

### 6. How does Apollo Kotlin execute mutations?

Apollo generates mutation classes from `.graphql` files. The Android app passes variables to the generated mutation class, executes it through `ApolloClient`, and receives strongly typed Kotlin responses.

---

# 📌 Next Chapter (Chapter 9 – GraphQL Subscriptions Deep Dive)

Next chapter will cover:

* Real-time communication in GraphQL
* Subscription architecture
* WebSockets
* Live chat example
* Notifications
* Live sports updates
* Apollo Kotlin subscription implementation
* Flow and coroutine integration
* Subscription lifecycle management
* Handling reconnects and failures
* Real Android project example (Chat App)
