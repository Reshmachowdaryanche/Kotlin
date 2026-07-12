# Chapter 21 – GraphQL Pagination Deep Dive

## Building Efficient Infinite Scrolling Android Applications with Apollo Kotlin and Paging

> **Learning Goal**
>
> By the end of this chapter, you will understand:
>
> * Why pagination is required in GraphQL applications
> * Different pagination strategies
> * Offset pagination
> * Cursor-based pagination
> * Relay-style pagination
> * GraphQL Connection pattern
> * Apollo Kotlin pagination handling
> * Android Paging 3 integration
> * Infinite scrolling implementation
> * Cache management during pagination
> * Real-world feed architecture

---

# Table of Contents

1. Introduction to Pagination
2. Why Pagination Matters in Mobile Apps
3. Problems Without Pagination
4. Pagination Architecture
5. Types of GraphQL Pagination
6. Offset-Based Pagination
7. Cursor-Based Pagination
8. Relay Cursor Pagination
9. GraphQL Connection Pattern
10. PageInfo Object
11. Implementing Cursor Pagination
12. Apollo Kotlin Pagination
13. Paging 3 Integration
14. Infinite Scroll Implementation
15. Cache and Pagination
16. Pull-to-Refresh with Pagination
17. Handling Pagination Errors
18. Optimizing Large Lists
19. Real-World Feed Example
20. Production Architecture
21. Best Practices
22. Summary
23. Interview Questions

---

# 1. Introduction to Pagination

Pagination means:

> Loading data in smaller chunks instead of loading everything at once.

Example:

A social media app has:

```
10,000 posts
```

Loading all:

```
Request

↓

10,000 records

↓

Memory problem
```

Instead:

```
Page 1

20 posts


Page 2

20 posts


Page 3

20 posts
```

---

# 2. Why Pagination Matters in Mobile Apps

Mobile devices have limitations:

```
Limited RAM

Limited Battery

Slow Networks

Limited Storage
```

Without pagination:

```text
Open Feed

↓

Download 500 MB

↓

App becomes slow
```

With pagination:

```text
Open Feed

↓

Download first 20 items

↓

User scrolls

↓

Load next page
```

---

# 3. Problems Without Pagination

## Memory Problem

Example:

```graphql
query {

posts {

id

title

image

comments

}

}
```

Response:

```json
{
"posts":[
100000 items
]
}
```

Problems:

* Large JSON parsing
* High memory usage
* UI freezing

---

## Network Problem

Large responses cause:

```
Slow download

High data usage

Battery drain
```

---

## Database Problem

Server must:

```
Fetch thousands of records

Serialize response

Send huge payload
```

---

# 4. Pagination Architecture

A typical Android architecture:

```
                UI

                 |

          LazyColumn

                 |

             Paging 3

                 |

           ViewModel

                 |

          Repository

                 |

          Apollo Client

                 |

          GraphQL API
```

---

# 5. Types of GraphQL Pagination

Common strategies:

## 1. Offset Pagination

Example:

```
page=1

limit=20
```

---

## 2. Cursor Pagination

Example:

```
after=abc123
```

---

## 3. Relay Pagination

Advanced cursor specification.

---

# 6. Offset-Based Pagination

## Concept

Uses:

```
offset

limit
```

Example:

First request:

```graphql
query {

posts(
limit:20,
offset:0
){

id

title

}

}
```

---

Response:

```json
{
"posts":[
1,
2,
3,
...
20
]
}
```

---

Next page:

```graphql
query {

posts(
limit:20,
offset:20
)

}
```

Returns:

```
21-40
```

---

# Offset Pagination Flow

```
offset = 0

↓

Load 20 items

↓

offset = 20

↓

Load next 20

↓

offset = 40

↓

Continue
```

---

# Advantages

Simple:

```
Easy implementation

Easy debugging
```

---

# Disadvantages

Problem:

## Data Changes

Example:

Initial:

```
Posts:

1
2
3
4
5
```

Request:

```
offset=5
```

Before response:

New post added:

```
0
1
2
3
4
5
```

Now:

```
offset=5
```

returns wrong data.

---

# 7. Cursor-Based Pagination

Modern GraphQL systems prefer cursor pagination.

Instead of:

```
offset=20
```

Use:

```
cursor=abc123
```

---

Example:

First request:

```graphql
query {

posts(
first:20
){

edges{

node{

id

title

}

}

}

}
```

---

Response:

```json
{
"edges":[

{
"node":{
"id":"1",
"title":"Post 1"
}
}

],

"pageInfo":{

"endCursor":"abc123",

"hasNextPage":true

}

}
```

---

Next request:

```graphql
query {

posts(
first:20,
after:"abc123"
)

}
```

---

# Cursor Flow

```
Request

↓

20 items

↓

Receive cursor

↓

Send cursor

↓

Next 20 items
```

---

# Advantages

More reliable:

```
Works with changing data

Better performance

Used by large apps
```

---

# Disadvantages

More complex.

---

# 8. Relay Cursor Pagination

Relay is a GraphQL pagination specification.

Used by:

* Facebook-style systems
* Large GraphQL APIs

Structure:

```
Connection

   |

Edges

   |

Nodes

   |

PageInfo
```

---

Example:

```graphql
query {

users{

edges{

cursor

node{

id

name

}

}

pageInfo{

hasNextPage

}

}

}
```

---

# 9. GraphQL Connection Pattern

A connection contains:

## Edge

Contains:

```
cursor

node
```

Example:

```json
{
"cursor":"abc",
"node":{
"id":10
}
}
```

---

## Node

Actual object:

```json
{
"id":10,
"name":"John"
}
```

---

## PageInfo

Pagination information:

```json
{
"hasNextPage":true,
"endCursor":"xyz"
}
```

---

# 10. PageInfo Object

Common fields:

```graphql
type PageInfo {

hasNextPage:Boolean!

hasPreviousPage:Boolean!

startCursor:String

endCursor:String

}
```

---

Example:

```json
{
"hasNextPage":true,
"endCursor":"cursor100"
}
```

Meaning:

```
More data exists.

Use cursor100 for next request.
```

---

# 11. Implementing Cursor Pagination

Schema:

```graphql
type Query {

posts(
first:Int
after:String
):PostConnection

}
```

---

Query:

```graphql
query Feed(
$cursor:String
){

posts(
first:20,
after:$cursor
){

edges{

node{

id

title

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

Variables:

First request:

```json
{
"cursor":null
}
```

Next:

```json
{
"cursor":"abc123"
}
```

---

# 12. Apollo Kotlin Pagination

Apollo returns:

```kotlin
response.data?.posts
```

Example:

```kotlin
val posts =
response.data
?.posts
?.edges
```

---

Store cursor:

```kotlin
var cursor:String? = null
```

After response:

```kotlin
cursor =
response.data
?.posts
?.pageInfo
?.endCursor
```

---

Next request:

```kotlin
FeedQuery(
cursor
)
```

---

# 13. Paging 3 Integration

Android Paging library handles:

```
Loading pages

Retry

Refresh

Memory management
```

---

Architecture:

```
PagingSource

      |

Repository

      |

Apollo Client

      |

GraphQL API
```

---

Create PagingSource:

```kotlin
class FeedPagingSource(

private val client:ApolloClient

)
:PagingSource<String,Post>(){


override suspend fun load(
params:LoadParams<String>
)
:LoadResult<String,Post>{


val cursor =
params.key


val response =
client.query(
FeedQuery(cursor)
)
.execute()


val posts =
response.data
?.posts
?.edges
?.map{
it.node
}


return LoadResult.Page(

data = posts ?: emptyList(),

prevKey = null,

nextKey =
response.data
?.posts
?.pageInfo
?.endCursor

)


}

}
```

---

# 14. Infinite Scroll Implementation

Compose:

```kotlin
LazyColumn{

items(
items = posts
){post ->


PostItem(post)


}

}
```

---

Paging collects:

```kotlin
val posts =
viewModel.posts.collectAsLazyPagingItems()
```

---

UI:

```kotlin
LazyColumn{


items(
posts.itemCount
){index ->


posts[index]?.let{

PostItem(it)

}


}

}
```

---

# 15. Cache and Pagination

Pagination must work with cache.

Example:

Page 1:

```
Posts 1-20
```

Cache:

```
Post:1
Post:2
...
```

Page 2:

```
Posts 21-40
```

Cache:

```
Post:21
Post:22
...
```

---

Apollo normalized cache combines them.

---

# 16. Pull-to-Refresh with Pagination

Common requirement:

User pulls screen:

```
Refresh
```

Flow:

```
Clear Paging State

↓

Load first page

↓

Update UI
```

---

Example:

```kotlin
posts.refresh()
```

---

# 17. Handling Pagination Errors

Possible errors:

## Network Failure

Example:

```
Page 3 failed
```

Show:

```
Retry
```

---

## End of Data

Example:

```json
{
"hasNextPage":false
}
```

Show:

```
No more posts
```

---

## Server Error

Example:

```
500 Internal Server Error
```

Retry later.

---

# 18. Optimizing Large Lists

## Use LazyColumn

Instead of:

```
Column
```

because:

```
Column loads everything
```

---

## Use Stable Keys

Bad:

```kotlin
items(posts)
```

Good:

```kotlin
items(
posts,
key={
it.id
}
)
```

---

## Avoid Heavy UI Work

Bad:

```
Every item loads image repeatedly
```

Good:

```
Image caching
```

---

# 19. Real-World Feed Example

Feature:

Social Media Feed

Requirements:

```
10 million posts

Infinite scroll

Offline support

Fast loading
```

---

Solution:

## GraphQL

```graphql
query Feed(
$cursor:String
){

feed(
first:20,
after:$cursor
)

}
```

---

## Apollo

```
Normalized Cache
```

---

## Android

```
Paging 3

LazyColumn
```

---

Architecture:

```
Compose

↓

Paging Adapter

↓

ViewModel

↓

Repository

↓

Apollo

↓

GraphQL
```

---

# 20. Production Architecture

Recommended:

```
feature/feed

 ├── FeedScreen

 ├── FeedViewModel

 ├── FeedRepository

 ├── FeedPagingSource

 └── FeedQuery.graphql
```

---

Benefits:

* Feature isolation
* Easy testing
* Maintainable code

---

# 21. Best Practices

## 1. Prefer Cursor Pagination

For:

```
Feeds

Messages

Large lists
```

---

## 2. Always return PageInfo

Client needs:

```
hasNextPage

cursor
```

---

## 3. Use reasonable page size

Usually:

```
10-50 items
```

---

## 4. Cache paginated data

Improves:

```
Scrolling

Offline support
```

---

## 5. Handle failures gracefully

Every page can fail independently.

---

## 6. Avoid loading everything

Never:

```
Return all records
```

---

# 22. Summary

In this chapter:

* Pagination concepts were explained.
* Offset pagination was covered.
* Cursor pagination was explained.
* Relay connection model was introduced.
* Apollo pagination handling was discussed.
* Paging 3 integration was covered.
* Infinite scrolling architecture was explained.
* Cache behavior was discussed.
* Production feed architecture was designed.

---

# 23. Interview Questions

### 1. Why is pagination important in GraphQL?

Because large responses cause:

* Memory issues
* Slow networks
* Poor performance

---

### 2. Difference between offset and cursor pagination?

Offset:

```
page number based
```

Cursor:

```
position based
```

Cursor works better for changing data.

---

### 3. What is a GraphQL cursor?

A pointer representing a position in a dataset.

Example:

```
abc123
```

---

### 4. What is PageInfo?

An object containing pagination metadata.

Example:

```graphql
hasNextPage

endCursor
```

---

### 5. How do you implement infinite scrolling in Android GraphQL?

Using:

```
Apollo Kotlin

+

Paging 3

+

LazyColumn
```

---

### 6. Why use cursor pagination for feeds?

Because feeds change frequently. Cursor pagination avoids duplicate or missing items when new data appears.

---

# 📌 Next Chapter (Chapter 22 – GraphQL Offline-First Android Applications)

Next chapter will cover:

* Offline-first architecture
* Apollo persistent cache
* Room + GraphQL integration
* Sync strategies
* Conflict resolution
* Offline mutations
* Network monitoring
* Building apps that work without internet.
