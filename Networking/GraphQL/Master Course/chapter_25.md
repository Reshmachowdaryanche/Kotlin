# Chapter 25 – GraphQL Android Interview Preparation

# 100+ Questions, Architecture Discussions, and Senior Android Scenarios

> **Learning Goal**
>
> This chapter prepares you for GraphQL Android interviews from beginner to senior level.
>
> You will learn:
>
> * Core GraphQL concepts
> * Apollo Kotlin interview questions
> * Android architecture discussions
> * Performance optimization questions
> * Debugging scenarios
> * Offline-first questions
> * System design questions
> * Real project explanation patterns

---

# Table of Contents

1. GraphQL Fundamentals Interview Questions
2. GraphQL Schema Questions
3. Query Questions
4. Mutation Questions
5. Subscription Questions
6. Apollo Kotlin Questions
7. Android Integration Questions
8. Architecture Questions
9. Cache Questions
10. Pagination Questions
11. Offline-First Questions
12. Security Questions
13. Performance Questions
14. Debugging Scenarios
15. System Design Questions
16. Senior-Level Discussion Questions
17. Project Explanation Template

---

# 1. GraphQL Fundamentals Interview Questions

---

## Q1. What is GraphQL?

### Answer:

GraphQL is a query language and runtime for APIs that allows clients to request exactly the data they need.

It provides:

* Strongly typed schema
* Single API endpoint
* Flexible queries
* Client-controlled responses

Example:

```graphql
query {
 user(id:10){
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

---

# Q2. Why was GraphQL created?

### Answer:

GraphQL was created to solve REST API problems:

## Over-fetching

REST:

```
GET /users/10
```

Returns:

```json
{
"name":"John",
"email":"john@gmail.com",
"age":30,
"address":"USA"
}
```

Android needs:

```
name
```

Extra data is downloaded.

---

## Under-fetching

Multiple REST calls:

```
GET /user

GET /posts

GET /comments
```

GraphQL:

```graphql
query {

user{

name

posts{

title

comments

}

}

}
```

One request.

---

# Q3. Difference between REST and GraphQL?

| REST                     | GraphQL                |
| ------------------------ | ---------------------- |
| Multiple endpoints       | Single endpoint        |
| Server controls response | Client controls fields |
| Fixed response           | Flexible response      |
| HTTP methods             | Query/Mutation         |
| API versions             | Schema evolution       |
| Simple caching           | Advanced caching       |

---

# Q4. Is GraphQL a replacement for REST?

Answer:

No.

Both solve API communication problems.

REST is still useful for:

* File uploads
* Simple APIs
* Public APIs

GraphQL is useful for:

* Mobile apps
* Complex data relationships
* Multiple clients

---

# Q5. What are the main components of GraphQL?

Answer:

Three operations:

## Query

Read data.

```graphql
query{
 users{
   name
 }
}
```

---

## Mutation

Change data.

```graphql
mutation{
 createUser(
 name:"John"
 )
}
```

---

## Subscription

Real-time updates.

```graphql
subscription{
 messageReceived{
   text
 }
}
```

---

# 2. GraphQL Schema Questions

---

# Q6. What is a GraphQL schema?

Answer:

A schema defines:

* Available operations
* Object types
* Fields
* Relationships
* Data types

Example:

```graphql
type User{

id:ID!

name:String!

email:String

}
```

---

# Q7. What does ! mean in GraphQL?

Example:

```graphql
name:String!
```

Means:

```
name cannot be null
```

Without:

```graphql
name:String
```

Value can be null.

---

# Q8. What are GraphQL scalar types?

Built-in types:

```
String

Int

Float

Boolean

ID
```

Example:

```graphql
age:Int
```

---

# Q9. What are custom types?

Example:

```graphql
type Product{

id:ID

name:String

price:Float

}
```

---

# Q10. What is schema stitching?

Combining multiple GraphQL schemas into one.

Example:

```
User Service

      +

Payment Service

      +

Product Service


        |

 Unified GraphQL API
```

---

# 3. GraphQL Query Questions

---

# Q11. What is a GraphQL query?

A query fetches data.

Example:

```graphql
query{

user(id:1){

name

}

}
```

---

# Q12. What are variables?

Instead of:

```graphql
user(id:10)
```

Use:

```graphql
query User(
$id:ID!
){

user(id:$id){

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

Benefits:

* Security
* Reusability
* Caching

---

# Q13. What are fragments?

Reusable fields.

Without:

```graphql
user{

id

name

email

}


friend{

id

name

email

}
```

With:

```graphql
fragment UserFields on User{

id

name

email

}
```

---

# Q14. What are aliases?

Rename fields.

Example:

```graphql
query{

first:user(id:1){

name

}


second:user(id:2){

name

}

}
```

---

# Q15. What is introspection?

Ability to query schema information.

Example:

```graphql
{
 __schema{
   types{
     name
   }
 }
}
```

Used by:

* IDE tools
* Documentation
* Code generation

---

# 4. Mutation Questions

---

# Q16. What is mutation?

Mutation changes server data.

Examples:

* Create
* Update
* Delete

Example:

```graphql
mutation{

createPost(
text:"Hello"
){

id

}

}
```

---

# Q17. Difference between Query and Mutation?

| Query          | Mutation              |
| -------------- | --------------------- |
| Read           | Write                 |
| Safe operation | Changes data          |
| Cache friendly | Needs update handling |

---

# Q18. What is optimistic update?

Updating UI before server response.

Example:

User clicks:

```
Like
```

Immediately:

```
❤️ 1 Like
```

Then:

```
Mutation sent
```

If failed:

```
Rollback
```

---

# 5. Subscription Questions

---

# Q19. What is GraphQL Subscription?

Used for real-time data.

Examples:

* Chat
* Notifications
* Live score

Uses:

```
WebSocket
```

---

# Q20. Query vs Subscription?

Query:

```
Request once
```

Subscription:

```
Continuous updates
```

---

# 6. Apollo Kotlin Questions

---

# Q21. What is Apollo Kotlin?

Apollo Kotlin is a GraphQL client library for Kotlin applications.

It provides:

* Query execution
* Code generation
* Type safety
* Cache
* Coroutines support

---

# Q22. Why use Apollo instead of manual Retrofit?

Manual Retrofit:

```
JSON

↓

DTO

↓

Manual mapping
```

Apollo:

```
GraphQL Query

↓

Generated Kotlin Models
```

Advantages:

* Type safety
* Less boilerplate
* Schema validation

---

# Q23. How does Apollo generate models?

You create:

```
User.graphql
```

Example:

```graphql
query User{

user{

id

name

}

}
```

Apollo generates:

```
UserQuery.kt
```

---

# Q24. Where should Apollo Client be placed?

Answer:

Data layer.

Architecture:

```
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

---

# Q25. How do you create Apollo Client?

Example:

```kotlin
val apolloClient =
ApolloClient.Builder()

.serverUrl(
"https://api.example.com/graphql"
)

.build()
```

---

# 7. Android Integration Questions

---

# Q26. Explain GraphQL Android architecture.

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

Apollo

↓

GraphQL Server
```

---

# Q27. How do you handle loading states?

Using StateFlow:

```kotlin
data class UiState(

val loading:Boolean,

val data:Data?,

val error:String?

)
```

---

# Q28. How do you handle GraphQL errors?

Example:

```kotlin
if(response.hasErrors()){

showError()

}
```

Better:

```
GraphQL Error

↓

Mapper

↓

Domain Error

↓

UI
```

---

# 8. Architecture Questions

---

# Q29. Why use Repository Pattern?

Repository hides data sources.

Example:

ViewModel does not know:

```
Apollo

Room

REST
```

---

# Q30. Why use Clean Architecture?

Separates:

```
Presentation

Domain

Data
```

Benefits:

* Testing
* Maintainability
* Scalability

---

# Q31. Explain MVVM.

```
View

↓

ViewModel

↓

Model/Data
```

ViewModel:

* Holds UI state
* Calls business logic

---

# Q32. Why use Hilt?

Dependency injection.

Instead of:

```kotlin
val repository =
Repository()
```

Hilt provides:

```kotlin
@Inject
Repository
```

Benefits:

* Testability
* Less coupling

---

# 9. Cache Questions

---

# Q33. What is Apollo normalized cache?

Apollo stores objects by ID.

Example:

```json
User:10

{
"name":"John"
}
```

Any query requesting:

```
User 10
```

can reuse cached data.

---

# Q34. Cache First vs Network Only?

Cache First:

```
Cache

↓

Network if missing
```

Network Only:

```
Always server
```

---

# Q35. How do you implement offline support?

Architecture:

```
UI

↓

Repository

↓

Room/Apollo Cache

↓

Sync

↓

Server
```

---

# 10. Pagination Questions

---

# Q36. Why pagination?

Avoid:

* Large responses
* Memory problems
* Slow UI

---

# Q37. Offset vs Cursor pagination?

Offset:

```
page=2
```

Cursor:

```
after=abc123
```

Cursor is better for dynamic feeds.

---

# Q38. How do you implement pagination in Android?

Use:

```
Apollo

+

Paging 3
```

---

# 11. Offline Questions

---

# Q39. What is offline-first architecture?

Application works without internet.

Flow:

```
Local Database

↓

UI

↓

Sync Server
```

---

# Q40. How do offline mutations work?

Use:

```
Mutation Queue

↓

WorkManager

↓

Apollo Mutation

↓

Server
```

---

# 12. Security Questions

---

# Q41. How do you authenticate GraphQL requests?

Using:

```
JWT Token
```

Header:

```
Authorization:
Bearer token
```

---

# Q42. Where store tokens?

Use:

```
Encrypted DataStore
```

Avoid:

```
Plain SharedPreferences
```

---

# Q43. How do you protect GraphQL APIs?

Server-side:

* Authentication
* Authorization
* Query depth limits
* Rate limiting

---

# 13. Performance Questions

---

# Q44. How to optimize GraphQL Android apps?

Techniques:

* Pagination
* Cache
* Query only required fields
* Compression
* Lazy loading
* Background sync

---

# Q45. What causes GraphQL performance problems?

Examples:

* Huge queries
* Deep relationships
* Missing pagination
* Expensive server resolvers

---

# 14. Debugging Scenarios

---

# Q46. API works in Postman but Android fails. Why?

Check:

* Headers
* Authentication
* Query format
* Variables
* Network security config

---

# Q47. Apollo returns null data. Why?

Possible reasons:

* GraphQL errors
* Nullable fields
* Incorrect query
* Authentication issue

---

# Q48. Cache returns old data. Solution?

Options:

* Clear cache
* Change fetch policy
* Refresh network

---

# 15. System Design Questions

---

# Q49. Design Instagram feed using GraphQL.

Architecture:

```
Compose

↓

FeedViewModel

↓

Repository

↓

Apollo

↓

GraphQL Feed API
```

Features:

* Cursor pagination
* Cache
* Offline support
* Optimistic likes

---

# Q50. Design WhatsApp-like chat.

Use:

```
GraphQL Mutation

+

Subscription

+

Room

+

Sync Worker
```

---

# 16. Senior-Level Discussion Questions

---

# Q51. How do you handle schema changes?

Use:

* Backward compatibility
* Deprecation
* Schema validation

Example:

```graphql
oldField:String
@deprecated
```

---

# Q52. How do you manage multiple environments?

Use:

```
dev

qa

production
```

with:

```
BuildConfig
```

---

# Q53. How do you monitor GraphQL performance?

Track:

* Query duration
* Error rate
* Cache hit ratio
* Network failures

---

# 17. Project Explanation Template

When interviewer asks:

> Explain your GraphQL Android project.

Answer structure:

---

## Project

"I worked on a social media application using GraphQL."

---

## Architecture

"We followed Clean Architecture with MVVM."

```
Compose

↓

ViewModel

↓

UseCase

↓

Repository

↓

Apollo Client
```

---

## Networking

"We used Apollo Kotlin for GraphQL communication."

---

## Features

Implemented:

* Authentication
* Feed pagination
* Comments
* Likes
* Chat
* Offline support

---

## Performance

Used:

* Apollo cache
* Paging 3
* LazyColumn
* Query optimization

---

## Challenges

Examples:

* Cache invalidation
* Offline sync
* Pagination handling
* Error management

---

# Final GraphQL Android Interview Checklist

Before interview, know:

✅ GraphQL fundamentals
✅ Schema design
✅ Query/Mutation/Subscription
✅ Apollo Kotlin
✅ Code generation
✅ Cache
✅ Pagination
✅ Offline-first architecture
✅ Clean Architecture
✅ MVVM
✅ Hilt
✅ Paging 3
✅ Error handling
✅ Security
✅ System design

---

# Complete GraphQL Android Learning Path Completed

You have now covered:

1. GraphQL Introduction
2. Schema Deep Dive
3. Types and Operations
4. Queries
5. Mutations
6. Subscriptions
7. Apollo Kotlin Basics
8. Apollo Advanced Usage
9. Caching
10. Error Handling
11. Authentication
12. Security
13. Testing
14. Performance Optimization
15. Android Architecture
16. Advanced Apollo
17. Real-Time Applications
18. Offline Applications
19. Pagination
20. Production Patterns
21. Pagination Deep Dive
22. Offline-First Apps
23. Enterprise Architecture
24. Complete Real Project
25. Interview Preparation

**End of GraphQL Android Mastery Course.**
