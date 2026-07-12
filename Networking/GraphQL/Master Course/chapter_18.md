# Chapter 18 – GraphQL Testing Deep Dive

## Testing GraphQL Android Applications with Apollo Kotlin

> **Learning Goal**
>
> By the end of this chapter, you will understand:
>
> * Why GraphQL testing is important
> * Different levels of testing in Android GraphQL applications
> * Testing GraphQL queries and mutations
> * Mocking Apollo responses
> * Unit testing repositories
> * Testing ViewModels
> * Testing Jetpack Compose UI
> * Integration testing with GraphQL servers
> * Contract testing
> * CI/CD testing strategies
> * Production testing best practices

---

# Table of Contents

1. Introduction to GraphQL Testing
2. Why Testing GraphQL Applications Is Different
3. Testing Pyramid
4. Types of GraphQL Tests
5. Testing GraphQL Schema
6. Testing Queries
7. Testing Mutations
8. Testing Subscriptions
9. Apollo Kotlin Testing Approach
10. Mocking Apollo Responses
11. Repository Unit Testing
12. ViewModel Testing
13. Compose UI Testing
14. Integration Testing
15. Contract Testing
16. End-to-End Testing
17. Testing Authentication Flow
18. Testing Error Scenarios
19. CI/CD Pipeline Testing
20. Complete Testing Example
21. Best Practices
22. Summary
23. Interview Questions

---

# 1. Introduction to GraphQL Testing

A GraphQL Android application contains multiple layers:

```text
                UI

                 ↓

             ViewModel

                 ↓

            Repository

                 ↓

          Apollo Client

                 ↓

        GraphQL Server
```

Each layer can fail.

Examples:

UI problem:

```
Wrong data displayed
```

ViewModel problem:

```
State not updated
```

Repository problem:

```
Incorrect query handling
```

GraphQL problem:

```
Schema changed
```

Network problem:

```
Request failure
```

Testing ensures every layer works correctly.

---

# 2. Why Testing GraphQL Applications Is Different

REST API testing usually checks:

```
Endpoint

HTTP Method

Response JSON
```

Example:

```
GET /users/10
```

---

GraphQL testing checks:

```
Operation Name

Query Structure

Variables

Schema

Response Data

Errors
```

Example:

```graphql
query GetUser($id:ID!){

user(id:$id){

name

}

}
```

The query itself is part of the contract.

---

# 3. Testing Pyramid

A good Android project follows:

```
              E2E Tests
                 ▲

          Integration Tests
                 ▲

            Unit Tests
                 ▲

          Static Analysis
```

---

## Unit Tests

Test individual classes.

Examples:

```
Repository

ViewModel

Mapper
```

Fast.

---

## Integration Tests

Test multiple components.

Example:

```
Apollo Client
+
Mock Server
```

---

## End-to-End Tests

Test complete user flows.

Example:

```
Login

↓

Open Profile

↓

Load User Data
```

---

# 4. Types of GraphQL Tests

GraphQL applications usually require:

## 1. Schema Tests

Checks:

```
Types

Fields

Arguments

Permissions
```

---

## 2. Operation Tests

Checks:

```
Queries

Mutations

Subscriptions
```

---

## 3. Client Tests

Checks:

```
Apollo Client

Repository

ViewModel
```

---

## 4. UI Tests

Checks:

```
Screens

Buttons

Loading states
```

---

# 5. Testing GraphQL Schema

Schema example:

```graphql
type User {

id:ID!

name:String!

email:String

}
```

---

A schema test verifies:

```
Does User exist?

Does name field exist?

Is id required?
```

---

Why important?

Suppose backend changes:

Before:

```graphql
name:String!
```

After:

```graphql
fullName:String!
```

Android build should detect this.

---

Apollo helps because:

```
Schema

+

Queries

↓

Compile-time validation
```

---

# 6. Testing Queries

Example query:

`UserQuery.graphql`

```graphql
query UserQuery($id:ID!){

user(id:$id){

id

name

}

}
```

---

Test cases:

## Success

Input:

```
id = 10
```

Expected:

```json
{
"name":"John"
}
```

---

## User Not Found

Expected:

```json
{
"errors":[
{
"message":"User not found"
}
]
}
```

---

## Network Failure

Expected:

```
IOException
```

---

# 7. Testing Mutations

Example:

```graphql
mutation CreateUser(
$name:String!
){

createUser(
name:$name
){

id

name

}

}
```

---

Test:

Input:

```
name = "John"
```

Expected:

```
User created
```

---

Failure:

```
Duplicate email
```

Expected:

```
Business error
```

---

# 8. Testing Subscriptions

Subscription:

```graphql
subscription MessageAdded{

messageAdded{

text

}

}
```

---

Test:

Server sends:

```json
{
"text":"Hello"
}
```

Expected:

Android receives:

```
Hello
```

---

Test:

Connection lost.

Expected:

```
Reconnect
```

---

# 9. Apollo Kotlin Testing Approach

Apollo applications are normally tested by mocking:

```
Apollo Client

      ↓

Fake GraphQL Response

      ↓

Repository

      ↓

ViewModel
```

---

Do not test:

```
Real Server
```

for every unit test.

Reasons:

* Slow
* Unstable
* Depends on internet

---

# 10. Mocking Apollo Responses

Instead of:

```
Production GraphQL Server
```

Use:

```
Mock Apollo Server
```

Example:

Real:

```
https://api.company.com/graphql
```

Test:

```
Fake GraphQL Response
```

---

Example fake response:

```json
{
"data":{
"user":{
"id":"10",
"name":"John"
}
}
}
```

---

Repository thinks:

```
Server returned data
```

---

# 11. Repository Unit Testing

Repository:

```kotlin
class UserRepository(
private val client:ApolloClient
){

suspend fun getUser(id:String){

return client
.query(
UserQuery(id)
)
.execute()

}

}
```

---

Test:

```kotlin
@Test
fun getUser_success(){

// arrange

// execute

// verify

}
```

---

Verify:

```
Correct query executed

Correct data returned
```

---

Example:

```kotlin
@Test
fun userNameReturned(){

val user =
repository.getUser("10")

assertEquals(
"John",
user.name
)

}
```

---

# 12. ViewModel Testing

ViewModel:

```kotlin
class UserViewModel(
private val repository:UserRepository
)
```

---

Test:

Scenario:

```
Load User
```

Expected:

Before:

```
Loading
```

After:

```
Success(User)
```

---

Example:

```kotlin
@Test
fun loadUser_updatesState(){

viewModel.loadUser("10")

assertEquals(
Success,
viewModel.state.value
)

}
```

---

# 13. Compose UI Testing

Jetpack Compose provides UI testing APIs.

Example:

Screen:

```
Profile

John
```

---

Test:

```kotlin
@Test
fun profileNameDisplayed(){

composeTestRule
.onNodeWithText("John")
.assertExists()

}
```

---

Test cases:

## Loading

Show:

```
Progress Indicator
```

---

## Success

Show:

```
User Name
```

---

## Error

Show:

```
Retry Button
```

---

# 14. Integration Testing

Integration test checks:

```
Apollo Client

+

Repository

+

Mock GraphQL Server
```

---

Example flow:

```
Repository

      ↓

Apollo

      ↓

Mock Server

      ↓

Response
```

---

Useful for checking:

* Query correctness
* Variables
* Parsing
* Mapping

---

# 15. Contract Testing

GraphQL has a strong contract:

```
Schema = Agreement
```

Between:

```
Backend Team

        +

Android Team
```

---

Example:

Backend:

```graphql
type User{

name:String!

}
```

Android expects:

```kotlin
user.name
```

---

If backend changes:

```graphql
fullName:String!
```

Contract test fails.

---

Benefits:

* Prevent breaking changes
* Safe API evolution
* Better team communication

---

# 16. End-to-End Testing

Tests the complete application.

Example:

Login:

```
Enter email

↓

Enter password

↓

Click Login

↓

Receive Token

↓

Open Home Screen
```

---

E2E verifies:

```
UI

+

Network

+

Backend

```

---

Tools:

* Android UI Automator
* Compose Testing
* Firebase Test Lab

---

# 17. Testing Authentication Flow

Authentication contains many cases.

---

## Successful Login

Input:

```
Correct email/password
```

Expected:

```
Token saved
User logged in
```

---

## Wrong Password

Expected:

```
Invalid credentials
```

---

## Expired Token

Scenario:

```
Access token expired
```

Expected:

```
Refresh token called
New token stored
```

---

## Logout

Expected:

```
Tokens removed

Cache cleared

Login screen shown
```

---

# 18. Testing Error Scenarios

Important errors:

## Network Error

Example:

```
No internet
```

Expected:

```
Show retry
```

---

## GraphQL Error

Example:

```json
{
"errors":[
{
"message":"User not found"
}
]
}
```

Expected:

```
Display error message
```

---

## Authorization Error

Example:

```
403 Forbidden
```

Expected:

```
Logout or permission message
```

---

# 19. CI/CD Pipeline Testing

A production pipeline:

```
Developer Push Code

        ↓

Build

        ↓

Unit Tests

        ↓

GraphQL Schema Check

        ↓

Integration Tests

        ↓

UI Tests

        ↓

Release
```

---

Example checks:

## Step 1

Compile:

```
./gradlew build
```

---

## Step 2

Unit tests:

```
./gradlew test
```

---

## Step 3

Android tests:

```
./gradlew connectedAndroidTest
```

---

# 20. Complete Testing Example

## Feature

User Profile

Architecture:

```
ProfileScreen

      ↓

ProfileViewModel

      ↓

ProfileRepository

      ↓

Apollo Client
```

---

## Repository Test

Success:

```
Given:

User ID = 10


When:

Repository requests user


Then:

Return John
```

---

## ViewModel Test

Given:

```
Repository returns User
```

When:

```
loadProfile()
```

Then:

```
State = Success
```

---

## UI Test

Given:

```
Success State
```

Then:

```
John displayed
```

---

# 21. Best Practices

## 1. Test all states

Every screen should test:

```
Loading

Success

Empty

Error
```

---

## 2. Mock external systems

Avoid:

```
Real API
```

for unit tests.

---

## 3. Validate GraphQL schema

Catch breaking changes early.

---

## 4. Test failures

Do not test only happy paths.

---

## 5. Keep tests readable

Good:

```
userLogin_withInvalidPassword_showsError()
```

Bad:

```
test1()
```

---

## 6. Use test data builders

Instead of:

```kotlin
User(
"id",
"name",
"email"
)
```

Use:

```kotlin
UserFactory.create()
```

---

# 22. Summary

* GraphQL testing requires testing both API operations and Android layers.
* Apollo provides compile-time safety.
* Unit tests verify repositories and ViewModels.
* Mock servers avoid dependency on real APIs.
* Compose UI tests verify user experience.
* Integration tests verify Apollo communication.
* Contract testing prevents schema breaking changes.
* Authentication flows need dedicated tests.
* CI/CD should automatically run tests.

---

# 23. Interview Questions

### 1. Why test GraphQL applications differently from REST?

Because GraphQL operations contain:

* Queries
* Variables
* Schema contracts
* Partial responses

---

### 2. How do you test Apollo repositories?

Mock Apollo responses and verify repository behavior.

---

### 3. What should be tested in a GraphQL Android app?

* Queries
* Mutations
* Subscriptions
* Repository
* ViewModel
* UI states
* Authentication

---

### 4. Why use GraphQL contract testing?

To detect backend schema changes that can break Android clients.

---

### 5. Should unit tests call a real GraphQL server?

Usually no. Use mocks for fast and reliable tests.

---

### 6. What are important UI states to test?

```
Loading

Success

Empty

Error

Retry
```

---

# 📌 Next Chapter (Chapter 19 – GraphQL Performance Optimization Deep Dive)

Next chapter will cover:

* GraphQL performance problems
* Query optimization
* Caching strategies
* Apollo normalized cache
* Pagination
* Lazy loading
* Network optimization
* Reducing Android battery usage
* Avoiding over-fetching
* Production performance patterns.
