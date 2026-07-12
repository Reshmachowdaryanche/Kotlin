# Chapter 17 – GraphQL Error Handling & Debugging Deep Dive

## Building Reliable GraphQL Android Applications

> **Learning Goal**
>
> By the end of this chapter, you will understand:
>
> * Types of GraphQL errors
> * Difference between network errors and GraphQL errors
> * Apollo Kotlin error handling
> * Handling partial responses
> * Designing error models
> * Retry strategies
> * Logging GraphQL requests
> * Debugging Android GraphQL applications
> * Production monitoring
> * Real-world error handling architecture

---

# Table of Contents

1. Introduction to Error Handling
2. Why Error Handling Is Important
3. GraphQL Error Types
4. Network Errors
5. GraphQL Execution Errors
6. Validation Errors
7. Authentication Errors
8. Authorization Errors
9. Business Logic Errors
10. Partial Data and Errors
11. Apollo Kotlin Error Handling
12. Handling Query Errors
13. Handling Mutation Errors
14. Handling Subscription Errors
15. Creating UI Error States
16. Retry Strategies
17. Logging GraphQL Requests
18. Debugging Tools
19. Production Monitoring
20. Android Error Architecture
21. Complete Example
22. Best Practices
23. Summary
24. Interview Questions

---

# 1. Introduction to Error Handling

Every application can fail.

Examples:

```
No Internet

Server unavailable

Invalid input

Expired token

Permission denied

Database failure
```

A good application should:

* Detect errors
* Show meaningful messages
* Recover when possible
* Log failures
* Avoid crashes

---

# 2. Why Error Handling Is Important

Poor error handling:

```
User clicks Login

        ↓

Error occurs

        ↓

App crashes
```

Good error handling:

```
User clicks Login

        ↓

Server error

        ↓

Show:

"Unable to login. Please try again."
```

---

For Android applications:

Errors affect:

* User experience
* App stability
* Debugging speed
* Production reliability

---

# 3. GraphQL Error Types

GraphQL errors are usually divided into:

```
1. Network Errors

2. GraphQL Errors

3. Client Errors

4. Server Errors

5. Business Errors
```

---

Architecture:

```
Android App

     |
     |
Apollo Client

     |
     |
GraphQL Server

     |
     |
Database
```

Errors can happen at every layer.

---

# 4. Network Errors

Network errors happen before GraphQL execution.

Examples:

```
No Internet

Timeout

DNS failure

Connection refused

Server unreachable
```

---

Example:

Request:

```
POST /graphql
```

Response:

```
No connection
```

---

Android exception:

```kotlin
IOException
```

---

Example:

```kotlin
try {

    val response =
        apolloClient
        .query(UserQuery())
        .execute()

}
catch(e: IOException){

    println("Network error")

}
```

---

# 5. GraphQL Execution Errors

GraphQL can return errors inside a valid HTTP response.

Example:

HTTP:

```
200 OK
```

Body:

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

Important:

HTTP success does not always mean GraphQL success.

---

Example:

```json
{
 "data":null,
 "errors":[
  {
   "message":"Invalid user id"
  }
 ]
}
```

---

# 6. Validation Errors

These happen when the query is incorrect.

Example:

Client sends:

```graphql
query {

user{

username

}

}
```

Schema:

```graphql
type User {

name:String

}
```

Problem:

```
username does not exist
```

---

Usually detected during:

```
Build time
```

when using Apollo.

---

# 7. Authentication Errors

Authentication means:

"Who are you?"

Example:

Expired token:

```
Authorization:
Bearer expired_token
```

Server:

```json
{
"errors":[
{
"message":"Unauthorized"
}
]
}
```

---

Common codes:

```
UNAUTHENTICATED

TOKEN_EXPIRED

INVALID_TOKEN
```

---

Android action:

```
Refresh token

OR

Logout user
```

---

# 8. Authorization Errors

Authorization means:

"What are you allowed to do?"

Example:

Normal user:

```
Role = USER
```

Attempts:

```graphql
mutation{

deleteUser(id:10)

}
```

Server:

```json
{
"errors":[
{
"message":"Permission denied"
}
]
}
```

---

Possible codes:

```
FORBIDDEN

ACCESS_DENIED

INSUFFICIENT_PERMISSION
```

---

# 9. Business Logic Errors

These are valid requests but business rules reject them.

Example:

Bank app:

Request:

```
Transfer ₹10000
```

Server:

```
Balance insufficient
```

---

GraphQL response:

```json
{
"errors":[
{
"message":"Insufficient balance",
"extensions":{
"code":"LOW_BALANCE"
}
}
]
}
```

---

# 10. Partial Data and Errors

GraphQL can return:

```
data + errors
```

at the same time.

Example:

Query:

```graphql
query {

user{

name

posts{

title

}

}

}
```

Result:

```json
{
"data":{
"user":{
"name":"John",
"posts":null
}
},

"errors":[
{
"message":"Posts unavailable"
}
]
}
```

---

Meaning:

User succeeded.

Posts failed.

---

Android should handle partial data.

---

# 11. Apollo Kotlin Error Handling

Apollo response:

```kotlin
val response =
client.query(
UserQuery()
)
.execute()
```

Contains:

```kotlin
response.data

response.errors

response.exception
```

---

Example:

```kotlin
if(response.hasErrors()){

val errors =
response.errors

}
```

---

# 12. Handling Query Errors

Example:

Repository:

```kotlin
suspend fun getUser()
:Result<User>{


val response =
apolloClient
.query(
UserQuery()
)
.execute()


return when{

response.data?.user != null ->

Result.success(
response.data!!.user
)


else ->

Result.failure(
Exception(
"User unavailable"
)
)

}

}
```

---

Benefits:

Repository hides GraphQL complexity.

---

# 13. Handling Mutation Errors

Example:

Create order:

```graphql
mutation {

createOrder{

id

}

}
```

Possible errors:

```
Payment failed

Product unavailable

Invalid address
```

---

Handle:

```kotlin
when(errorCode){

"PAYMENT_FAILED" ->

showPaymentError()


"OUT_OF_STOCK" ->

showStockError()

}
```

---

# 14. Handling Subscription Errors

Subscriptions have additional problems:

```
Connection lost

Authentication expired

Server stopped

Network changed
```

---

Example:

```kotlin
apolloClient
.subscription(
MessageSubscription()
)
.toFlow()
.catch {

error ->

showConnectionError()

}
```

---

# 15. Creating UI Error States

Avoid directly showing errors from API.

Bad:

```kotlin
Text(error.message)
```

---

Create UI states:

```kotlin
sealed class UiState {


object Loading:
UiState()


data class Success(
val data:User
):UiState()


data class Error(
val message:String
):UiState()

}
```

---

Flow:

```
GraphQL Error

        ↓

Repository

        ↓

ViewModel

        ↓

UiState.Error

        ↓

Compose UI
```

---

# Example Compose

```kotlin
when(state){

is UiState.Loading -> {

CircularProgressIndicator()

}


is UiState.Success -> {

Text(
state.data.name
)

}


is UiState.Error -> {

Text(
state.message
)

}

}
```

---

# 16. Retry Strategies

Not all errors should retry.

---

## Retry Network Errors

Example:

```
Timeout
Temporary failure
```

Retry:

```
1 second

2 seconds

4 seconds
```

Called:

```
Exponential Backoff
```

---

Example:

```text
Attempt 1
wait 1 sec

Attempt 2
wait 2 sec

Attempt 3
wait 4 sec
```

---

## Do Not Retry

Examples:

```
Invalid password

Permission denied

User not found
```

---

# 17. Logging GraphQL Requests

During development, logging helps.

Example:

Request:

```graphql
query User{

user(id:10){

name

}

}
```

Variables:

```json
{
"id":10
}
```

Response:

```json
{
"name":"John"
}
```

---

Useful for:

* Debugging queries
* Finding server problems
* Checking variables

---

Be careful:

Never log:

```
Passwords

Tokens

Personal data
```

---

# 18. Debugging Tools

## 1. Apollo Logging

Example:

```kotlin
ApolloClient.Builder()

.httpEngine(
engine
)

.build()
```

---

## 2. GraphQL Playground

Used for testing:

Queries

Mutations

Subscriptions

---

## 3. Server Logs

Check:

```
Request ID

Database errors

Authentication failures
```

---

## 4. Android Studio Debugger

Inspect:

```
ViewModel state

Repository responses

Exceptions
```

---

# 19. Production Monitoring

Production apps need monitoring.

Track:

```
Error count

Failed requests

Response time

Network failures

Crash rate
```

---

Useful information:

```
Request ID

User session ID

Operation name
```

---

Example:

```
Operation:

GetUserProfile

Error:

USER_NOT_FOUND

Time:

300ms
```

---

# 20. Android Error Architecture

Recommended architecture:

```
                UI

                 |

             ViewModel

                 |

            Repository

                 |

        Error Mapper

                 |

            Apollo Client

                 |

          GraphQL Server
```

---

## Error Mapper

Converts:

Technical errors:

```
ApolloException
IOException
GraphQL error
```

into:

User errors:

```
"No internet connection"

"Session expired"

"Something went wrong"
```

---

# 21. Complete Example

## GraphQL Query

```graphql
query Profile{

viewer{

id

name

}

}
```

---

## Repository

```kotlin
class ProfileRepository(
private val client:ApolloClient
){


suspend fun getProfile()
:Result<User>{


return try{


val response =
client.query(
ProfileQuery()
)
.execute()


if(response.data?.viewer != null){


Result.success(
response.data!!.viewer
)


}
else{


Result.failure(
Exception(
"Profile unavailable"
)
)

}


}
catch(e:Exception){


Result.failure(e)


}

}

}
```

---

## ViewModel

```kotlin
class ProfileViewModel(
private val repository:ProfileRepository
)
:ViewModel(){


val state =
MutableStateFlow<UiState>(
UiState.Loading
)


fun load(){


viewModelScope.launch{


repository.getProfile()

.onSuccess{


state.value =
UiState.Success(it)

}


.onFailure{


state.value =
UiState.Error(
it.message ?: "Error"
)

}


}


}

}
```

---

# 22. Best Practices

## 1. Separate error layers

Do not mix:

```
UI

Network

GraphQL

Business logic
```

---

## 2. Create meaningful messages

Bad:

```
ApolloException
```

Good:

```
Unable to load profile
```

---

## 3. Handle partial responses

GraphQL can return:

```
data + errors
```

---

## 4. Do not expose technical errors

Avoid:

```
Database connection failed
```

Show:

```
Please try again later
```

---

## 5. Add request IDs

Helps backend debugging.

---

## 6. Secure logs

Never log:

```
JWT tokens

Passwords

Private information
```

---

# 23. Summary

* GraphQL errors are different from REST errors.
* HTTP 200 can still contain GraphQL errors.
* Network errors happen before GraphQL execution.
* Authentication errors require token handling.
* Authorization errors require permission handling.
* GraphQL supports partial data responses.
* Apollo exposes errors through response objects.
* UI should use clean error states.
* Retry only temporary failures.
* Production systems require monitoring.

---

# 24. Interview Questions

### 1. Does GraphQL return HTTP errors for all failures?

No. GraphQL often returns HTTP 200 with an `errors` field.

---

### 2. Difference between network error and GraphQL error?

Network error:

```
Request never reaches GraphQL execution.
```

GraphQL error:

```
Server executes request but something fails.
```

---

### 3. Can GraphQL return data and errors together?

Yes. GraphQL supports partial responses.

---

### 4. How should Android handle GraphQL errors?

Recommended:

```
Apollo Response

↓

Repository

↓

Error Mapper

↓

ViewModel

↓

UI State
```

---

### 5. Should every error be retried?

No.

Retry:

```
Network failures
Temporary server errors
```

Do not retry:

```
Invalid input
Authentication failure
Permission errors
```

---

### 6. How do you debug GraphQL issues?

Use:

* Apollo logs
* GraphQL playground
* Android debugger
* Server logs
* Monitoring tools

---

# 📌 Next Chapter (Chapter 18 – GraphQL Testing Deep Dive)

Next chapter will cover:

* Testing GraphQL APIs
* Unit testing Apollo repositories
* Mocking GraphQL responses
* Apollo Mock Server
* Testing ViewModels
* Testing Compose UI
* Integration testing
* Contract testing
* CI/CD testing strategies.
