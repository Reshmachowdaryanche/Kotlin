# Chapter 15 – GraphQL Authentication & Security Deep Dive

## Building Secure GraphQL Android Applications

> **Learning Goal**
>
> By the end of this chapter, you will understand:
>
> * How authentication works in GraphQL
> * Difference between authentication and authorization
> * JWT-based authentication
> * Access tokens and refresh tokens
> * Adding authentication headers in Apollo Kotlin
> * Secure token storage in Android
> * Handling token expiration
> * Role-based access control
> * Field-level security
> * Query security
> * Mutation security
> * Preventing common GraphQL attacks
> * Production security architecture

---

# Table of Contents

1. Introduction to GraphQL Security
2. Authentication vs Authorization
3. GraphQL Authentication Flow
4. Login Architecture
5. JWT Authentication
6. Access Token
7. Refresh Token
8. Token Storage in Android
9. Apollo Authentication Interceptor
10. Adding Authorization Headers
11. Handling Token Expiration
12. Refresh Token Flow
13. Logout Implementation
14. Role-Based Authorization
15. Field-Level Security
16. Query Security
17. Mutation Security
18. GraphQL Security Attacks
19. Rate Limiting
20. HTTPS and Network Security
21. Android Security Architecture
22. Complete Authentication Example
23. Best Practices
24. Summary
25. Interview Questions

---

# 1. Introduction to GraphQL Security

A GraphQL API exposes many operations through one endpoint:

```
POST /graphql
```

Example:

```graphql
query {
    user(id:10){
        name
    }
}
```

Because everything goes through one endpoint, security becomes very important.

A secure GraphQL system must answer:

Questions:

```
Who is this user?

What can this user access?

What operations can this user perform?

How much data can this user request?
```

---

# 2. Authentication vs Authorization

These two concepts are different.

---

## Authentication

Authentication answers:

> "Who are you?"

Example:

Login:

```
Email:
john@gmail.com

Password:
******
```

Server verifies identity.

Response:

```json
{
 "accessToken":"abc123"
}
```

---

## Authorization

Authorization answers:

> "What are you allowed to do?"

Example:

User:

```
Role: CUSTOMER
```

Can:

```
View products
Create orders
```

Cannot:

```
Delete users
Access admin dashboard
```

---

Flow:

```
Authentication

        ↓

Identity Created

        ↓

Authorization

        ↓

Permission Check
```

---

# 3. GraphQL Authentication Flow

Typical mobile flow:

```
Android App

     |
     |
 Login Mutation

     |
     ↓

GraphQL Server

     |
     |
 Validate User

     |
     ↓

Return Tokens

     |
     ↓

Store Token

     |
     ↓

Future GraphQL Requests
include token
```

---

Example login mutation:

```graphql
mutation Login(
$email:String!,
$password:String!
){

login(
email:$email,
password:$password
){

accessToken

refreshToken

}

}
```

---

Response:

```json
{
 "data":{
   "login":{
     "accessToken":"eyJhb...",
     "refreshToken":"xyz..."
   }
 }
}
```

---

# 4. Login Architecture

Android architecture:

```
Login Screen

      ↓

LoginViewModel

      ↓

AuthRepository

      ↓

Apollo Client

      ↓

GraphQL Login Mutation

      ↓

Save Tokens
```

---

Components:

## UI

Collects:

```
Email

Password
```

---

## ViewModel

Handles:

```
Loading

Success

Error
```

---

## Repository

Calls GraphQL.

---

## Token Manager

Stores tokens securely.

---

# 5. JWT Authentication

JWT means:

```
JSON Web Token
```

A JWT contains information about the user.

Example:

```
xxxxx.yyyyy.zzzzz
```

Structure:

```
Header.Payload.Signature
```

---

Example payload:

```json
{
 "userId":10,
 "role":"USER",
 "exp":1720000000
}
```

---

Server uses the token to identify the user.

---

# 6. Access Token

Access token is used for API requests.

Example:

Request:

```
POST /graphql
```

Headers:

```
Authorization:
Bearer eyJhbGciOiJIUzI1
```

---

GraphQL query:

```graphql
query {
 viewer {
    name
 }
}
```

Server:

```
Read Token

      ↓

Identify User

      ↓

Return Data
```

---

Access token characteristics:

Usually:

```
Short lifetime

5 minutes

15 minutes

1 hour
```

---

Why short?

If stolen, damage is limited.

---

# 7. Refresh Token

Refresh token creates a new access token.

Example:

Access token:

```
Expires in 15 minutes
```

Refresh token:

```
Expires in 30 days
```

---

Flow:

```
Access Token Expired

          ↓

Send Refresh Token

          ↓

Server Generates New Token

          ↓

Continue API Calls
```

---

Example:

```graphql
mutation RefreshToken(
$token:String!
){

refreshToken(
token:$token
){

accessToken

}

}
```

---

# 8. Token Storage in Android

Never store tokens in:

❌ SharedPreferences plain text

❌ Database without encryption

❌ Files

---

Use:

## Android Keystore

Secure storage:

```
Android Keystore

       |

Encrypted Key

       |

Encrypted Token
```

---

Common approach:

```
EncryptedSharedPreferences
```

Example:

```kotlin
val prefs =
EncryptedSharedPreferences.create(
    context,
    "secure_storage",
    masterKey,
    AES256_SIV,
    AES256_GCM
)
```

---

Store:

```
accessToken

refreshToken
```

---

# 9. Apollo Authentication Interceptor

Apollo allows adding headers automatically.

Requirement:

Every request:

```
Authorization:
Bearer TOKEN
```

---

Architecture:

```
Apollo Client

      |

Interceptor

      |

Add Token

      |

GraphQL Server
```

---

# 10. Adding Authorization Headers

Example:

```kotlin
val apolloClient =
ApolloClient.Builder()

.serverUrl(
"https://api.example.com/graphql"
)

.addHttpHeader(
"Authorization",
"Bearer $token"
)

.build()
```

---

Problem:

Token can change.

Better:

Use interceptor.

---

Example:

```kotlin
class AuthInterceptor(
private val tokenManager:TokenManager
){

fun getToken():String?{

return tokenManager.getToken()

}

}
```

---

Every request gets the latest token.

---

# 11. Handling Token Expiration

Scenario:

```
User opens app

       ↓

Access token expired

       ↓

API returns 401

       ↓

Refresh token request

       ↓

New access token

       ↓

Retry original request
```

---

Example server response:

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

Client checks:

```
Is token expired?

YES

↓

Refresh
```

---

# 12. Refresh Token Flow

Complete example:

```
GraphQL Request

        ↓

Server

        ↓

401 Unauthorized

        ↓

Apollo Interceptor

        ↓

Refresh Token Mutation

        ↓

Receive New Access Token

        ↓

Save Token

        ↓

Retry Request
```

---

Pseudo Kotlin:

```kotlin
if(response.isUnauthorized()){

 val newToken =
 refreshToken()

 saveToken(newToken)

 retry()

}
```

---

# 13. Logout Implementation

Logout should:

1. Remove tokens
2. Clear cache
3. Reset user state

---

Flow:

```
Logout Button

       ↓

Delete Tokens

       ↓

Clear Apollo Cache

       ↓

Navigate Login Screen
```

---

Example:

```kotlin
fun logout(){

tokenManager.clear()

apolloClient.clearCache()

}
```

---

# 14. Role-Based Authorization

Users have roles.

Example:

```
ADMIN

CUSTOMER

SELLER
```

---

JWT:

```json
{
"userId":10,
"role":"ADMIN"
}
```

---

Schema:

```graphql
type Mutation{

deleteUser(
id:ID!
):Boolean

}
```

Server checks:

```
Is role ADMIN?

YES → Allow

NO → Reject
```

---

# 15. Field-Level Security

Sometimes users can access an object but not every field.

Example:

User object:

```graphql
type User{

id:ID!

name:String!

salary:Float

}
```

---

Normal user:

Can see:

```
id

name
```

Cannot see:

```
salary
```

---

Admin:

Can see:

```
salary
```

---

Security happens on server.

Never depend on Android hiding fields.

---

# 16. Query Security

GraphQL allows clients to request arbitrary fields.

Example:

Malicious query:

```graphql
query{

users{

id

name

password

}

}
```

Server must prevent sensitive fields.

---

Rules:

Never expose:

```
password

private keys

internal data

```

---

# 17. Mutation Security

Mutations modify data.

Example:

```graphql
mutation{

deleteUser(
id:10
)

}
```

Dangerous.

Server should check:

```
Authentication

Authorization

Validation

Business rules
```

---

Example:

Delete user:

```
Is logged in?

↓

Is admin?

↓

Can delete this user?

```

---

# 18. GraphQL Security Attacks

## 1. Query Depth Attack

Example:

```graphql
user{

friends{

friends{

friends{

friends{

}

}

}

}

}
```

Problem:

Huge nested request.

Solution:

Limit depth.

Example:

Maximum:

```
Depth = 5
```

---

# 2. Query Complexity Attack

Example:

Request:

```
10000 users

each with:

1000 posts

each with:

100 comments
```

Solution:

Assign costs.

Example:

```
users = 10 points

posts = 5 points

comments = 2 points
```

Maximum:

```
1000 points
```

---

# 3. Introspection Exposure

GraphQL allows:

```graphql
__schema
```

to inspect API.

Production systems may disable it.

---

# 4. Injection Attacks

Example:

Bad input:

```
' OR 1=1
```

Solution:

* Validate inputs
* Use parameterized queries
* Sanitize data

---

# 19. Rate Limiting

Protect API from abuse.

Example:

Limit:

```
100 requests/minute/user
```

---

Applied at:

```
API Gateway

Server

Authentication Layer
```

---

# 20. HTTPS and Network Security

Always use:

```
HTTPS
```

Never:

```
http://api.com/graphql
```

---

Use:

```
https://api.com/graphql
```

---

Android:

Enable:

```
Network Security Config
```

---

# 21. Android Security Architecture

Recommended:

```
Compose UI

     ↓

ViewModel

     ↓

Repository

     ↓

Apollo Client

     ↓

Auth Interceptor

     ↓

Token Manager

     ↓

Encrypted Storage

```

---

# 22. Complete Authentication Example

## Login Mutation

```graphql
mutation Login(
$email:String!,
$password:String!
){

login(
email:$email,
password:$password
){

accessToken

refreshToken

}

}
```

---

## Repository

```kotlin
class AuthRepository(
private val client:ApolloClient
){

suspend fun login(
email:String,
password:String
){

val response =
client.mutation(
LoginMutation(
email,
password
)
)
.execute()


return response.data?.login

}

}
```

---

## Save Token

```kotlin
tokenManager.save(
accessToken
)
```

---

## Future Request

Header:

```
Authorization:
Bearer token
```

---

# 23. Best Practices

## 1. Never store plain tokens

Use:

```
EncryptedSharedPreferences

Android Keystore
```

---

## 2. Use short-lived access tokens

Example:

```
15 minutes
```

---

## 3. Rotate refresh tokens

Avoid permanent refresh tokens.

---

## 4. Validate everything on server

Never trust Android clients.

---

## 5. Clear cache on logout

Remove:

```
Tokens

Apollo cache

User data
```

---

## 6. Limit query complexity

Prevent expensive requests.

---

# 24. Summary

* Authentication identifies users.
* Authorization controls permissions.
* JWT tokens are commonly used with GraphQL.
* Access tokens authenticate API calls.
* Refresh tokens generate new access tokens.
* Apollo interceptors add authentication headers.
* Tokens should be stored securely.
* Server must enforce permissions.
* Query depth and complexity limits protect GraphQL APIs.
* HTTPS is mandatory.
* Apollo cache should be cleared during logout.

---

# 25. Interview Questions

### 1. How does authentication work in GraphQL?

The client sends credentials using a mutation, receives tokens, and sends the access token in the Authorization header for future requests.

---

### 2. Difference between authentication and authorization?

Authentication:

```
Who are you?
```

Authorization:

```
What can you do?
```

---

### 3. Where should Android store JWT tokens?

Use secure storage such as Android Keystore-backed encrypted storage.

---

### 4. How does Apollo add authentication headers?

Using HTTP headers or authentication interceptors.

---

### 5. What happens when an access token expires?

The client uses the refresh token to obtain a new access token and retries the failed request.

---

### 6. Why should GraphQL queries have complexity limits?

To prevent attackers from sending extremely expensive queries that consume server resources.

---

# 📌 Next Chapter (Chapter 16 – GraphQL Subscriptions & Real-Time Data Deep Dive)

Next chapter will cover:

* Real-time communication
* Query vs Mutation vs Subscription
* WebSocket architecture
* Apollo Kotlin subscriptions
* Chat application example
* Live notifications
* Live sports updates
* Subscription authentication
* Reconnection handling
* Flow integration
* Production real-time architecture.
