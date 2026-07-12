# Chapter 13 – GraphQL Caching Deep Dive

## Apollo Kotlin Cache Architecture for Android Applications

> **Learning Goal**
>
> By the end of this chapter, you will understand:
>
> * Why caching is important in GraphQL
> * Why GraphQL caching is different from REST caching
> * Types of caches
> * Apollo Kotlin normalized cache
> * Memory cache
> * Disk cache
> * Cache keys
> * Cache reads and writes
> * Mutation cache updates
> * Cache invalidation
> * Offline support
> * Jetpack Compose integration
> * Production caching strategies

---

# Table of Contents

1. Introduction to Caching
2. Why Mobile Apps Need Caching
3. REST Cache vs GraphQL Cache
4. Apollo Kotlin Cache Architecture
5. Normalized Cache Concept
6. How Normalized Cache Works
7. Cache Records
8. Cache Keys
9. Memory Cache
10. Disk Cache
11. Apollo Cache Policies
12. Reading Data from Cache
13. Writing Data to Cache
14. Mutation and Cache Updates
15. Cache Invalidation
16. Offline Support
17. Compose Integration
18. Real Android Examples
19. Cache Best Practices
20. Summary
21. Interview Questions

---

# 1. Introduction to Caching

Caching means storing previously fetched data so that it can be reused instead of downloading it again.

Example:

User opens profile:

First time:

```
Android App

     ↓

Network Request

     ↓

Server

     ↓

User Data
```

The data is saved locally.

Next time:

```
Android App

     ↓

Cache

     ↓

User Data
```

No network request required.

---

# 2. Why Mobile Apps Need Caching

Mobile applications have limitations:

* Slow networks
* Limited bandwidth
* Battery constraints
* Offline usage
* Expensive API calls

Example:

A social media app:

```
Home Screen
     |
     |
Fetch 50 posts
```

User opens profile and returns:

Without cache:

```
Fetch posts again
```

With cache:

```
Show existing posts instantly
```

---

Benefits:

✅ Faster UI

✅ Less network usage

✅ Better battery life

✅ Offline experience

✅ Improved user experience

---

# 3. REST Cache vs GraphQL Cache

## REST Caching

REST uses URL-based caching.

Example:

Request:

```
GET /users/10
```

Cache stores:

```
/users/10

{
 id:10,
 name:"John"
}
```

The URL identifies the resource.

---

## GraphQL Problem

GraphQL usually has one endpoint:

```
POST /graphql
```

Different queries:

Query 1:

```graphql
query{
 user(id:10){
   name
 }
}
```

Query 2:

```graphql
query{
 user(id:10){
   name
   email
 }
}
```

Same endpoint:

```
POST /graphql
```

The URL cannot identify the data.

---

Solution:

GraphQL clients use:

## Normalized Cache

---

# 4. Apollo Kotlin Cache Architecture

Apollo Kotlin provides caching:

```
              Android App

                  |

             Apollo Client

                  |

        ---------------------

        |                   |

 Memory Cache        Disk Cache

        |

        |

 GraphQL Server
```

---

Apollo supports:

1. Memory cache
2. Persistent disk cache
3. Normalized cache

---

# 5. Normalized Cache Concept

A normalized cache stores objects separately.

Example response:

```json
{
 "user":{
    "id":"10",
    "name":"John"
 }
}
```

Instead of storing:

```
User Screen Data

{
 user:{
   id:10,
   name:"John"
 }
}
```

Apollo stores:

```
User:10

{
 id:10,
 name:"John"
}
```

---

This object can be reused everywhere.

---

Example:

Profile:

```
User:10
```

Feed:

```
Post.author → User:10
```

Comments:

```
Comment.author → User:10
```

One object.

---

# 6. How Normalized Cache Works

Suppose server returns:

```json
{
 "post":{
    "id":"100",
    "title":"GraphQL",
    "author":{
        "id":"10",
        "name":"John"
    }
 }
}
```

Apollo stores:

Post:

```
Post:100

{
 title:"GraphQL",
 author:"User:10"
}
```

User:

```
User:10

{
 name:"John"
}
```

---

Relationship:

```
Post:100

    |
    |
    ↓

User:10
```

---

# 7. Cache Records

A cache contains records.

Example:

```
Cache

-----------------

User:10

id
name
avatar


-----------------

Post:100

title
content
author


-----------------

Comment:500

text
author

```

---

Each record has:

* Object ID
* Fields
* References

---

# 8. Cache Keys

Cache keys identify objects.

Default:

```
TypeName:id
```

Example:

GraphQL:

```graphql
type User{
 id:ID!
}
```

Object:

```json
{
"id":"10"
}
```

Cache key:

```
User:10
```

---

## Why Cache Keys Matter

Without unique keys:

Two users:

```
User

{
name:"John"
}
```

and

```
User

{
name:"Alex"
}
```

Cannot be separated.

---

# 9. Memory Cache

Memory cache stores data in RAM.

Flow:

```
App Running

      |

Memory Cache

      |

Instant Data
```

Advantages:

* Very fast
* No disk access

Disadvantage:

Cleared when app closes.

---

Example:

User opens profile:

First time:

```
Network
  |
Cache Memory
```

Second time:

```
Memory Cache
  |
Instant
```

---

# 10. Disk Cache

Disk cache stores data permanently.

Example:

```
Phone Storage

Apollo Cache Database

     |
     |
User Data
```

Advantages:

* Survives app restart
* Supports offline

---

Example:

User opens app offline:

```
No Internet

     |

Disk Cache

     |

Show Previous Data
```

---

# 11. Apollo Cache Policies

Cache policy controls where data comes from.

Apollo provides:

## Cache First

Default.

Flow:

```
Check Cache

   |

Found?

   |

Return Cache

```

Otherwise:

```
Network Request
```

---

Example:

```kotlin
apolloClient
.query(UserQuery())
.fetchPolicy(
FetchPolicy.CacheFirst
)
```

---

## Network Only

Always call server.

```
Network

↓

Ignore Cache
```

Use for:

* Banking balance
* Live data

---

## Cache Only

Only use cache.

```
Cache

↓

No Network
```

Useful offline.

---

## Cache and Network

Returns cache first, then refreshes.

Flow:

```
Cache Result

      +

Network Result
```

Great for feeds.

---

# 12. Reading Data from Cache

Example:

Query:

```graphql
query User{

user(id:10){

name

}

}
```

Apollo checks:

```
User:10
```

If exists:

```
Return cached name
```

---

# 13. Writing Data to Cache

After network response:

Server:

```json
{
"user":{
"id":"10",
"name":"John"
}
}
```

Apollo:

```
Updates:

User:10
```

Now future queries use cache.

---

# 14. Mutation and Cache Updates

Important scenario:

User changes profile name.

Before:

```
User:10

name = John
```

Mutation:

```graphql
mutation{

updateUser(
name:"Alex"
)

}
```

Server returns:

```json
{
"id":"10",
"name":"Alex"
}
```

Apollo updates:

```
User:10

name = Alex
```

Now every screen updates.

---

# Example

Profile screen:

```
John
```

Feed:

```
John posted
```

After mutation:

```
Alex
```

Both update automatically.

---

# 15. Cache Invalidation

Sometimes cached data becomes invalid.

Example:

Product price:

Cache:

```
Phone

price=$500
```

Server:

```
price=$450
```

Old cache is wrong.

---

Solutions:

## 1. Remove Cache Entry

Example:

```
Delete Product:10
```

---

## 2. Refetch Query

After mutation:

```
Update Product

     |

Fetch Product Again
```

---

## 3. Update Cache Manually

Modify:

```
Product:10

price=450
```

---

# 16. Offline Support

Apollo cache can support offline apps.

Architecture:

```
Android App

     |

Apollo Client

     |

Disk Cache

     |

Network (when available)

```

---

Example:

Travel app:

Internet available:

```
Download trips
```

Later:

```
No Internet

     |

Show cached trips
```

---

# 17. Compose Integration

Modern Android:

```
GraphQL

   |

Apollo

   |

ViewModel

   |

StateFlow

   |

Jetpack Compose
```

---

Example:

ViewModel:

```kotlin
class UserViewModel(
private val repository:UserRepository
):ViewModel(){


val user =
    repository.getUser()


}
```

---

Compose:

```kotlin
@Composable
fun ProfileScreen(
user:User
){

Text(
text=user.name
)

}
```

---

When Apollo cache changes:

```
Cache Update

      |

Flow emits

      |

Compose recomposes
```

---

# 18. Real Android Examples

## Example 1: Social Media Feed

Requirements:

* Fast scrolling
* Offline viewing

Strategy:

```
Cache First

+

Background Refresh
```

Flow:

```
Open App

↓

Show cached posts

↓

Fetch latest posts

↓

Update cache

↓

Refresh UI
```

---

# Example 2: Chat Application

Messages:

Use:

```
Network + Subscription
```

Flow:

```
Query old messages

↓

Cache messages

↓

Subscription receives new messages

↓

Update cache

```

---

# Example 3: E-Commerce App

Product details:

Cache:

```
Product:100
```

Cart update:

Mutation:

```
Add Product
```

Cache:

```
Cart updated
```

---

# 19. Cache Best Practices

## 1. Define Good IDs

Every object should have:

```
id
```

Example:

Good:

```graphql
type User{

id:ID!

}
```

---

## 2. Avoid Huge Cache Objects

Bad:

```
User

1000 fields
```

---

Good:

```
User

Only required fields
```

---

## 3. Choose Correct Fetch Policy

Examples:

Profile:

```
Cache First
```

Stock price:

```
Network Only
```

Feed:

```
Cache and Network
```

---

## 4. Update Cache After Mutations

Avoid:

```
Mutation

↓

Reload Everything
```

Prefer:

```
Mutation

↓

Update Cache

↓

UI updates
```

---

## 5. Handle Cache Migration

When app updates:

Old cache:

```
User{name}
```

New:

```
User{firstName,lastName}
```

Handle migration carefully.

---

# 20. Summary

* GraphQL caching is different from REST because many queries use one endpoint.
* Apollo Kotlin uses normalized caching.
* Objects are stored separately using cache keys.
* Memory cache provides speed.
* Disk cache provides persistence.
* Cache policies control data sources.
* Mutations can automatically update cached objects.
* Proper cache design improves Android performance.
* Offline support is possible with persistent cache.

---

# 21. Interview Questions

### 1. Why is GraphQL caching different from REST caching?

REST uses URL-based caching. GraphQL uses a single endpoint, so clients use normalized caches based on object identity.

---

### 2. What is normalized caching?

Normalized caching stores objects separately and references them by unique IDs.

Example:

```
User:10
```

---

### 3. How does Apollo identify objects?

Usually using:

```
typename + id
```

Example:

```
User:10
```

---

### 4. What happens after a mutation?

Apollo can automatically update cache records if returned objects contain IDs.

---

### 5. Difference between Cache First and Network Only?

Cache First checks local cache before network.

Network Only always requests fresh server data.

---

### 6. How does GraphQL support offline Android apps?

Apollo Kotlin can store normalized cache data on disk and serve it when the network is unavailable.

---

# 📌 Next Chapter (Chapter 14 – Apollo Kotlin Deep Dive)

Next chapter will cover:

* Apollo Kotlin architecture
* Project setup
* Gradle configuration
* Schema download
* GraphQL files
* Code generation
* Apollo Client creation
* Queries
* Mutations
* Subscriptions
* Error handling
* Caching configuration
* Repository pattern
* Complete Android project example.
