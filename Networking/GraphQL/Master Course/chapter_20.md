# Chapter 20 – Apollo Kotlin Advanced Features Deep Dive

## Mastering Apollo Client Configuration for Production Android Applications

> **Learning Goal**
>
> By the end of this chapter, you will understand:
>
> * Advanced Apollo Kotlin architecture
> * Apollo Client configuration
> * HTTP engine customization
> * Interceptors
> * Authentication token handling
> * Custom Scalars
> * Custom Type Adapters
> * File Uploads
> * Multipart Requests
> * Cache persistence
> * Offline support
> * Kotlin Multiplatform support
> * Production Apollo patterns

---

# Table of Contents

1. Introduction to Apollo Kotlin Advanced Features
2. Apollo Kotlin Architecture
3. Apollo Client Creation
4. HTTP Engine Configuration
5. Apollo Interceptors
6. Authentication Interceptor
7. Logging Interceptor
8. Error Handling Interceptor
9. Retry Interceptor
10. Custom Scalars
11. Custom Scalar Adapters
12. Date and Time Handling
13. Enum Handling
14. File Uploads
15. Multipart Requests
16. Cache Persistence
17. Offline Apollo Applications
18. Kotlin Multiplatform Support
19. Dependency Injection with Apollo
20. Production Apollo Architecture
21. Complete Implementation Example
22. Best Practices
23. Summary
24. Interview Questions

---

# 1. Introduction to Apollo Kotlin Advanced Features

In previous chapters:

We learned:

```text
GraphQL Query

       ↓

Apollo Client

       ↓

Response
```

Now we go deeper.

A production application needs:

```
Authentication

Logging

Caching

Offline support

File upload

Error handling

Monitoring
```

Apollo Kotlin provides extension points for all these.

---

# 2. Apollo Kotlin Architecture

A production Apollo setup:

```
                 Android App

                     |

               Apollo Client

                     |

        -------------------------

        |          |            |

     Cache    Interceptors   Network

        |          |            |

     Storage   Headers       HTTP Engine

                     |

              GraphQL Server
```

---

## Main Components

### ApolloClient

Responsible for:

* Executing queries
* Executing mutations
* Executing subscriptions
* Cache management

---

### HttpEngine

Responsible for:

* Network communication
* HTTP requests
* Connections

---

### Interceptors

Responsible for:

* Modifying requests
* Handling responses
* Logging
* Authentication

---

### Cache

Responsible for:

* Storing GraphQL objects
* Offline data
* Faster loading

---

# 3. Apollo Client Creation

Basic Apollo Client:

```kotlin
val apolloClient =
    ApolloClient.Builder()
        .serverUrl(
            "https://api.example.com/graphql"
        )
        .build()
```

---

Production version:

```kotlin
val apolloClient =
    ApolloClient.Builder()

        .serverUrl(
            "https://api.example.com/graphql"
        )

        .addHttpInterceptor(
            AuthInterceptor()
        )

        .addHttpInterceptor(
            LoggingInterceptor()
        )

        .build()
```

---

# 4. HTTP Engine Configuration

Apollo does not directly create HTTP connections.

It uses:

```
Apollo Client

      ↓

HTTP Engine

      ↓

Network
```

---

Android commonly uses:

```
OkHttp Engine
```

---

Dependency:

```gradle
implementation(
"com.apollographql.apollo:apollo-engine-okhttp"
)
```

---

Example:

```kotlin
val okHttpClient =
    OkHttpClient.Builder()
        .connectTimeout(
            30,
            TimeUnit.SECONDS
        )
        .build()


val apolloClient =
    ApolloClient.Builder()
        .serverUrl(
            SERVER_URL
        )
        .httpEngine(
            OkHttpEngine(okHttpClient)
        )
        .build()
```

---

Benefits:

Control:

* Timeout
* SSL
* Proxy
* Network policies

---

# 5. Apollo Interceptors

Interceptors allow you to modify requests.

Flow:

```
Query

 ↓

Interceptor

 ↓

Network

 ↓

Response

 ↓

Interceptor

 ↓

App
```

---

Common interceptors:

```
Authentication

Logging

Retry

Performance tracking
```

---

# 6. Authentication Interceptor

Most GraphQL APIs use:

```
JWT Token
```

Request:

```
Authorization:
Bearer token
```

---

Example:

```kotlin
class AuthInterceptor(
    private val tokenManager:TokenManager
)
:HttpInterceptor {


override suspend fun intercept(
chain:HttpInterceptorChain
):HttpResponse {


val token =
tokenManager.getToken()


return chain.proceed(
chain.request.newBuilder()
.addHeader(
"Authorization",
"Bearer $token"
)
.build()
)


}

}
```

---

Flow:

```
User Login

    ↓

Receive Token

    ↓

Store Token

    ↓

Interceptor Adds Token

    ↓

GraphQL Request
```

---

# 7. Logging Interceptor

During development:

You want to see:

Request:

```graphql
query User{

user{

name

}

}
```

Response:

```json
{
"name":"John"
}
```

---

Example:

```kotlin
class LoggingInterceptor
:HttpInterceptor {


override suspend fun intercept(
chain:HttpInterceptorChain
):HttpResponse {


println(
chain.request.body
)


return chain.proceed(
chain.request
)

}

}
```

---

Important:

Never log:

```
Password

JWT Token

Personal Data
```

in production.

---

# 8. Error Handling Interceptor

Centralize errors.

Example:

Server returns:

```json
{
"errors":[
{
"message":
"Unauthorized"
}
]
}
```

Interceptor:

```kotlin
if(error.code=="UNAUTHENTICATED"){

refreshToken()

}
```

---

Benefits:

One place handles:

* Token expiration
* Global errors
* Analytics

---

# 9. Retry Interceptor

Temporary failures:

```
Timeout

Network disconnect

Server unavailable
```

can retry.

---

Example strategy:

```
Attempt 1

wait 1 second


Attempt 2

wait 2 seconds


Attempt 3

wait 4 seconds
```

Called:

```
Exponential Backoff
```

---

Do not retry:

```
Invalid password

Permission denied

Invalid query
```

---

# 10. Custom Scalars

GraphQL supports default scalars:

```graphql
String

Int

Float

Boolean

ID
```

---

Real applications need:

```
Date

UUID

Money

JSON

URL
```

---

Example schema:

```graphql
scalar DateTime


type User{

createdAt:DateTime

}
```

---

Android does not know:

```
DateTime
```

Need mapping.

---

# 11. Custom Scalar Adapters

Example:

GraphQL:

```graphql
scalar DateTime
```

Map to:

```kotlin
java.time.Instant
```

---

Adapter:

```kotlin
class DateTimeAdapter
:Adapter<Instant>{


override fun fromJson(
reader:JsonReader,
customScalarAdapters:CustomScalarAdapters
):Instant{


return Instant.parse(
reader.nextString()
)

}


override fun toJson(
writer:JsonWriter,
value:Instant,
customScalarAdapters:CustomScalarAdapters
){

writer.value(
value.toString()
)

}

}
```

---

Now:

GraphQL:

```
2026-01-01T10:00:00Z
```

becomes:

```kotlin
Instant
```

---

# 12. Date and Time Handling

Common formats:

```
ISO-8601

Unix timestamp

UTC
```

---

Recommended:

Use:

```
Instant
```

because:

* Timezone safe
* Standard format

---

Example:

```kotlin
data class User(

val createdAt:Instant

)
```

---

# 13. Enum Handling

GraphQL:

```graphql
enum Status{

ACTIVE

INACTIVE

}
```

Generated Kotlin:

```kotlin
enum class Status{

ACTIVE,

INACTIVE

}
```

---

Problem:

Server adds:

```graphql
BLOCKED
```

Old app crashes?

Apollo handles unknown values:

```kotlin
UNKNOWN__
```

---

Example:

```kotlin
when(status){

ACTIVE -> {}

INACTIVE -> {}

UNKNOWN__ -> {}

}
```

---

# 14. File Uploads

GraphQL can upload:

```
Images

Videos

Documents
```

using:

```
Multipart Requests
```

---

Example:

Profile picture:

```
Android

 ↓

Multipart Upload

 ↓

GraphQL Mutation

 ↓

Server Storage
```

---

Mutation:

```graphql
mutation UploadPhoto(
$file:Upload!
){

uploadPhoto(
file:$file
){

url

}

}
```

---

Android:

```kotlin
val upload =
Upload(
fileName,
contentType,
file
)
```

---

# 15. Multipart Requests

Normal request:

```
JSON
```

Upload request:

```
Multipart Form Data
```

Contains:

```
Operations

File Map

Binary File
```

---

Example:

```
operations:

{
mutation UploadPhoto
}


map:

{
"0":["variables.file"]
}


0:

image.jpg
```

---

# 16. Cache Persistence

Default cache:

```
Memory
```

Problem:

App closes:

```
Cache lost
```

---

Persistent cache:

```
Memory

 +

Disk
```

---

Architecture:

```
Apollo Cache

     |

SQLite/File

     |

Storage
```

---

Benefits:

* Faster startup
* Offline data
* Less network

---

# 17. Offline Apollo Applications

Offline flow:

```
Open App

    ↓

Read Cache

    ↓

Display Data

    ↓

Network Available

    ↓

Sync Data
```

---

Example:

User opens:

```
Profile Screen
```

No internet:

Apollo returns:

```
Cached Profile
```

---

# 18. Kotlin Multiplatform Support

Apollo Kotlin supports:

```
Android

iOS

Desktop

Backend
```

---

Architecture:

```
Shared Module

      |

Apollo Client

      |

GraphQL API
```

---

Example:

```
commonMain

   |
   |
GraphQL queries

Models

Repository


androidMain

   |
Android UI


iosMain

   |
iOS UI
```

---

Benefits:

* Shared networking
* Shared models
* Less duplicate code

---

# 19. Dependency Injection with Apollo

Recommended:

Use:

```
Hilt
```

---

Module:

```kotlin
@Module
@InstallIn(
SingletonComponent::class
)
object NetworkModule{


@Provides
@Singleton
fun provideApolloClient()
:ApolloClient{


return ApolloClient.Builder()

.serverUrl(
URL
)

.build()

}

}
```

---

Usage:

```kotlin
class UserRepository @Inject constructor(

private val apolloClient:ApolloClient

)
```

---

# 20. Production Apollo Architecture

Recommended structure:

```
app

 ├── graphql

 │     ├── queries

 │     ├── mutations

 │     └── fragments


 ├── data

 │     └── repository


 ├── network

 │     ├── ApolloClient

 │     ├── Interceptors


 ├── cache


 └── ui

       └── Compose
```

---

# 21. Complete Implementation Example

## Apollo Client

```kotlin
object ApolloProvider {


fun create():ApolloClient {


return ApolloClient.Builder()

.serverUrl(
"https://api.company.com/graphql"
)

.addHttpInterceptor(
AuthInterceptor()
)

.addHttpInterceptor(
LoggingInterceptor()
)

.build()

}

}
```

---

## Repository

```kotlin
class UserRepository(

private val client:ApolloClient

){


suspend fun getUser(){


return client

.query(
UserQuery()
)

.execute()

}

}
```

---

## ViewModel

```kotlin
class UserViewModel(

private val repository:UserRepository

):ViewModel(){


fun load(){


viewModelScope.launch{


repository.getUser()


}


}

}
```

---

# 22. Best Practices

## 1. Keep Apollo configuration centralized

Bad:

```
Create ApolloClient everywhere
```

Good:

```
Single instance
```

---

## 2. Use interceptors for common logic

Example:

```
Authentication

Logging

Analytics
```

---

## 3. Use custom scalars

Avoid:

```
String everywhere
```

Prefer:

```
Instant

UUID

Money
```

---

## 4. Enable caching

Improves:

```
Speed

Offline support
```

---

## 5. Secure logs

Never expose:

```
Tokens

Passwords
```

---

## 6. Use dependency injection

Makes:

```
Testing easier
```

---

# 23. Summary

In this chapter:

* Apollo Client architecture was explained.
* HTTP engines were discussed.
* Interceptors were implemented.
* Authentication handling was covered.
* Custom Scalars were explained.
* File uploads and multipart requests were introduced.
* Persistent caching was covered.
* Offline support was explained.
* Kotlin Multiplatform integration was discussed.
* Production architecture patterns were shown.

---

# 24. Interview Questions

### 1. What is Apollo Client?

Apollo Client is a GraphQL client that executes operations, manages caching, and generates Kotlin models.

---

### 2. Why use Apollo Interceptors?

For:

* Authentication headers
* Logging
* Retry logic
* Error handling

---

### 3. What are custom scalars?

Custom GraphQL types mapped to application-specific types.

Example:

```
DateTime → Instant
```

---

### 4. How do you handle JWT authentication in Apollo?

Using an HTTP interceptor that adds:

```
Authorization: Bearer token
```

---

### 5. How does Apollo support offline mode?

Using:

* Normalized cache
* Persistent cache
* Cache-first policies

---

### 6. What is Apollo normalized cache?

A cache that stores GraphQL objects by unique IDs and shares them across queries.

Example:

```
User:10
```

---

### 7. Can Apollo Kotlin be used with Kotlin Multiplatform?

Yes. Apollo Kotlin supports shared GraphQL networking across platforms.

---

# 📌 Next Chapter (Chapter 21 – GraphQL Pagination Deep Dive)

Next chapter will cover:

* Offset pagination
* Cursor pagination
* Relay-style pagination
* GraphQL connections
* Apollo pagination cache
* Android Paging 3 integration
* Infinite scrolling feeds
* Real-world implementation patterns.
