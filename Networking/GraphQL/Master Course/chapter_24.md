# Chapter 24 – Complete Real-World GraphQL Android Project

# Building a Production-Level Social Media Application Using Apollo Kotlin

> **Learning Goal**
>
> In this chapter, we will combine everything learned in Chapters 1–23 and design a complete production Android application using:
>
> * GraphQL API
> * Apollo Kotlin
> * Jetpack Compose
> * Clean Architecture
> * MVVM
> * Hilt Dependency Injection
> * Room Database
> * Paging 3
> * Offline-first architecture
> * GraphQL subscriptions
> * Authentication
> * Testing

---

# Table of Contents

1. Project Overview
2. Application Features
3. Technology Stack
4. High-Level Architecture
5. Project Module Structure
6. GraphQL Backend Design
7. Authentication Flow
8. Apollo Client Setup
9. Network Layer
10. GraphQL Queries
11. GraphQL Mutations
12. GraphQL Subscriptions
13. Data Layer
14. Repository Implementation
15. Domain Layer
16. ViewModel Design
17. UI Layer with Compose
18. Feed Feature
19. Pagination Implementation
20. Profile Feature
21. Comments Feature
22. Likes Feature
23. Chat Feature
24. Offline Support
25. Cache Strategy
26. Error Handling
27. Testing Strategy
28. Production Deployment
29. Complete Project Flow
30. Interview Discussion

---

# 1. Project Overview

We will build a social media application similar to:

* Instagram
* Facebook
* Twitter/X

Application name:

```
SocialGraph
```

The app allows users to:

* Create accounts
* Login
* View posts
* Like posts
* Comment
* Follow users
* Send messages
* Receive notifications

---

# 2. Application Features

## Authentication

Features:

```
Register

Login

Logout

Refresh Token

JWT Authentication
```

---

## User Profile

Features:

```
Profile image

Username

Bio

Followers

Following

Posts
```

---

## Feed

Features:

```
Infinite scrolling

Pull refresh

Like posts

Comments
```

---

## Chat

Features:

```
One-to-one messaging

Real-time messages

Typing indicators
```

---

## Notifications

Features:

```
New likes

Comments

Messages

Followers
```

---

# 3. Technology Stack

## Android

```
Kotlin

Jetpack Compose

Coroutines

Flow

ViewModel

Navigation Compose
```

---

## Architecture

```
Clean Architecture

MVVM

Repository Pattern

Multi Module
```

---

## Networking

```
Apollo Kotlin

GraphQL

WebSocket Subscription
```

---

## Storage

```
Room Database

DataStore

Apollo Cache
```

---

## Dependency Injection

```
Hilt
```

---

## Testing

```
JUnit

MockK

Turbine

Compose UI Testing
```

---

# 4. High-Level Architecture

Complete flow:

```
                Compose UI

                    |

                ViewModel

                    |

              Use Cases

                    |

              Repository

                    |

        ----------------------

        |                    |

   Local Source        Remote Source

   Room Cache          Apollo Client

        |                    |

        -------- Sync --------

                    |

             GraphQL Server
```

---

# 5. Project Module Structure

Production structure:

```
SocialGraph


app


core

 ├── network

 ├── database

 ├── designsystem

 ├── common


feature-auth

feature-feed

feature-profile

feature-chat

feature-notification


domain

data

graphql
```

---

# 6. GraphQL Backend Design

Main schema:

```graphql
type User {

 id:ID!

 username:String!

 avatar:String

 bio:String

}


type Post {

 id:ID!

 text:String

 image:String

 author:User!

 likes:Int

 comments:Int

}
```

---

Relationships:

```
User

 |

 |---- Posts

 |

 |---- Followers

 |

 |---- Messages
```

---

# 7. Authentication Flow

## Login

User enters:

```
email

password
```

---

Mutation:

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
"accessToken":"abc123",
"refreshToken":"xyz456"
}
```

---

Store:

```
Encrypted DataStore
```

---

# 8. Apollo Client Setup

Create singleton:

```kotlin
@Provides
@Singleton
fun provideApolloClient()
: ApolloClient {


return ApolloClient.Builder()

.serverUrl(
"https://api.socialgraph.com/graphql"
)

.addHttpInterceptor(
AuthInterceptor()
)

.normalizedCache(
cacheFactory
)

.build()

}
```

---

# 9. Network Layer

Structure:

```
core-network


ApolloClient

AuthInterceptor

ErrorHandler

NetworkMonitor
```

---

## Authentication Interceptor

Adds:

```
Authorization:
Bearer token
```

Example:

```kotlin
override suspend fun intercept(
chain:HttpInterceptorChain
)
:HttpResponse {


return chain.proceed(

chain.request.newBuilder()

.addHeader(
"Authorization",
"Bearer $token"
)

.build()

)

}
```

---

# 10. GraphQL Queries

## Get Feed

File:

```
Feed.graphql
```

Query:

```graphql
query Feed(
$cursor:String
){

feed(
first:20,
after:$cursor
){

edges{

node{

id

text

image

author{

username

}

}

}

pageInfo{

endCursor

hasNextPage

}

}

}
```

---

Apollo generates:

```
FeedQuery

FeedQuery.Data
```

---

# 11. GraphQL Mutations

## Like Post

```graphql
mutation LikePost(
$postId:ID!
){

likePost(
id:$postId
){

success

}

}
```

---

## Add Comment

```graphql
mutation AddComment(
$postId:ID!,
$text:String!
){

comment(

postId:$postId,

text:$text

){

id

text

}

}
```

---

# 12. GraphQL Subscriptions

For real-time updates:

Example:

New messages:

```graphql
subscription MessageSubscription{

messageReceived{

id

text

sender{

username

}

}

}
```

---

Flow:

```
Server

 |

WebSocket

 |

Apollo Subscription

 |

Flow

 |

Compose UI
```

---

# 13. Data Layer

Structure:

```
data


repository


remote

 Apollo


local

 Room
```

---

Example:

```kotlin
interface FeedRepository {


fun getFeed()
:Flow<PagingData<Post>>


}
```

---

# 14. Repository Implementation

```kotlin
class FeedRepositoryImpl(

private val apollo:ApolloClient,

private val database:PostDao

)
:FeedRepository {


override fun getFeed()
:Flow<PagingData<Post>> {


return Pager(

config = PagingConfig(
pageSize=20
)

){

FeedPagingSource(
apollo
)

}.flow


}

}
```

---

# 15. Domain Layer

Business actions:

```
domain


GetFeedUseCase

LikePostUseCase

SendMessageUseCase
```

---

Example:

```kotlin
class LikePostUseCase(

private val repository:
PostRepository

){


suspend operator fun invoke(
id:String
){

repository.like(id)

}

}
```

---

# 16. ViewModel Design

Example:

```kotlin
class FeedViewModel(

private val getFeed:
GetFeedUseCase

)
:ViewModel(){


val posts =
getFeed()

}
```

---

UI state:

```kotlin
data class FeedState(

val loading:Boolean=false,

val error:String?=null

)
```

---

# 17. UI Layer with Compose

Structure:

```
presentation


FeedScreen

ProfileScreen

ChatScreen
```

---

Example:

```kotlin
@Composable
fun FeedScreen(

posts:LazyPagingItems<Post>

){

LazyColumn{


items(
posts.itemCount
){

index ->

posts[index]?.let{

PostCard(it)

}

}

}

}
```

---

# 18. Feed Feature

Architecture:

```
Feed


FeedScreen

FeedViewModel

FeedRepository

FeedPagingSource

Feed.graphql
```

---

Features:

```
Pagination

Refresh

Like

Comment
```

---

# 19. Pagination Implementation

Using:

```
Apollo

+

Paging 3
```

Flow:

```
Open Feed

↓

Load 20 posts

↓

Scroll

↓

Load next cursor

↓

Update list
```

---

Cursor:

```
pageInfo.endCursor
```

---

# 20. Profile Feature

Query:

```graphql
query Profile{

me{

username

avatar

posts{

id

}

}

}
```

---

Cache strategy:

```
Cache First
```

because profile changes rarely.

---

# 21. Comments Feature

Architecture:

```
Comments


Query Comments

Mutation AddComment

Pagination
```

---

Query:

```graphql
query Comments(
$postId:ID!
){

comments(
postId:$postId
){

text

author{

username

}

}

}
```

---

# 22. Likes Feature

Optimistic Update:

User clicks:

```
Like button
```

Immediately:

```
UI changes
```

Then:

```
Mutation sent
```

---

Benefits:

* Faster UX
* Better responsiveness

---

# 23. Chat Feature

Architecture:

```
ChatScreen

↓

ChatViewModel

↓

ChatRepository

↓

Apollo Subscription

↓

WebSocket
```

---

Message sending:

```
User sends message

↓

Mutation

↓

Server

↓

Subscription

↓

Receiver
```

---

# 24. Offline Support

Architecture:

```
UI

↓

Room Database

↓

Sync Manager

↓

Apollo

↓

Server
```

---

Offline message:

```
Save locally

status=PENDING

↓

Internet returns

↓

Upload

↓

Mark SENT
```

---

# 25. Cache Strategy

Different features use different policies.

| Feature  | Strategy        |
| -------- | --------------- |
| Profile  | Cache First     |
| Feed     | Cache + Network |
| Payment  | Network Only    |
| Chat     | Subscription    |
| Settings | Cache First     |

---

# 26. Error Handling

Central error mapping:

```
GraphQL Error

↓

Error Mapper

↓

Domain Error

↓

UI Message
```

---

Example:

Server:

```
UNAUTHENTICATED
```

Converted:

```
SessionExpired
```

UI:

```
Please login again
```

---

# 27. Testing Strategy

## Unit Tests

Test:

```
UseCases

Repositories

ViewModels
```

---

## GraphQL Tests

Validate:

```
Queries

Mutations

Schema
```

---

## UI Tests

Test:

```
Login Screen

Feed Screen

Chat Screen
```

---

# 28. Production Deployment

Pipeline:

```
Developer Push

↓

CI Build

↓

Run Tests

↓

Generate APK

↓

Deploy
```

---

Monitor:

```
Crash Reports

API Errors

Slow Queries

Network Failures
```

---

# 29. Complete Project Flow

Example:

User opens Feed:

```
Compose Screen

↓

FeedViewModel

↓

GetFeedUseCase

↓

FeedRepository

↓

Apollo Client

↓

GraphQL Server

↓

Response

↓

Apollo Cache

↓

UI Update
```

---

Offline:

```
Compose Screen

↓

Repository

↓

Room Cache

↓

Display Data

↓

Sync Later
```

---

# 30. Interview Discussion

## Question 1

### How would you design a GraphQL Android application?

Answer:

```
Compose UI

↓

ViewModel

↓

UseCase

↓

Repository

↓

Apollo Client

↓

GraphQL API
```

with:

* Hilt
* Cache
* Paging
* Offline support

---

## Question 2

### Where should Apollo Client exist?

Answer:

```
Data Layer
```

Never directly inside UI.

---

## Question 3

### How do you handle offline mutations?

Answer:

```
Local Queue

↓

WorkManager

↓

Apollo Mutation

↓

Sync Result
```

---

## Question 4

### How do you optimize GraphQL Android apps?

Answer:

* Pagination
* Cache
* Query optimization
* Lazy loading
* Compression
* Proper architecture

---

# Chapter 24 Summary

You now have a complete production-level GraphQL Android architecture:

```
Jetpack Compose

        ↓

ViewModel

        ↓

Clean Architecture

        ↓

Repository

        ↓

Apollo Kotlin

        ↓

GraphQL API
```

With:

✅ Authentication
✅ Pagination
✅ Offline support
✅ Cache
✅ Real-time updates
✅ Testing
✅ Production architecture

---

# Next Chapter

## Chapter 25 – GraphQL Android Interview Preparation

Topics:

* 100+ GraphQL interview questions
* Apollo Kotlin interview questions
* Architecture discussions
* Senior Android scenarios
* Debugging questions
* Performance questions
* Real project explanations
* System design questions for Android engineers
