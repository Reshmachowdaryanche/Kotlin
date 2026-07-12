# Chapter 22 – GraphQL Offline-First Android Applications

## Building Reliable Android Apps That Work Without Internet Using Apollo Kotlin

> **Learning Goal**
>
> By the end of this chapter, you will understand:
>
> * What offline-first architecture means
> * Why offline support is important in Android apps
> * How Apollo Kotlin caching enables offline experiences
> * Difference between cache-first and offline-first
> * Apollo normalized cache persistence
> * Room + GraphQL architecture
> * Data synchronization strategies
> * Offline mutations
> * Conflict resolution
> * Network monitoring
> * Building production offline apps

---

# Table of Contents

1. Introduction to Offline-First Architecture
2. Why Offline Support Matters
3. Online-Only vs Offline-First Apps
4. Offline-First Architecture Pattern
5. Data Source Strategy
6. Apollo Kotlin Cache Deep Dive
7. Memory Cache vs Persistent Cache
8. Apollo Persistent Cache
9. Cache Policies
10. Room + Apollo Architecture
11. Repository Pattern for Offline Apps
12. Network Synchronization
13. Sync Strategies
14. Offline Queries
15. Offline Mutations
16. Mutation Queue Pattern
17. Conflict Resolution
18. Handling Deleted Data
19. Network Monitoring
20. Background Synchronization
21. Real-World Example
22. Production Architecture
23. Best Practices
24. Summary
25. Interview Questions

---

# 1. Introduction to Offline-First Architecture

Traditional apps:

```
User

 ↓

Network Request

 ↓

Server

 ↓

Display Data
```

Problem:

No internet:

```
Request failed

↓

Empty screen
```

---

Offline-first architecture:

```
User

 ↓

Local Database

 ↓

Show Data Immediately

 ↓

Sync With Server
```

The app works even when:

* Internet is slow
* Network is unavailable
* User is traveling
* Server is temporarily down

---

# 2. Why Offline Support Matters

Mobile networks are unpredictable.

Examples:

```
Subway

Airplane

Remote areas

Weak WiFi

Mobile data switching
```

Users expect:

```
Open app

↓

See previous data

↓

Continue working
```

---

Applications that benefit:

* Messaging apps
* Social media
* Banking apps
* Note apps
* Shopping apps
* Maps

---

# 3. Online-Only vs Offline-First Apps

## Online-Only

Architecture:

```
UI

↓

Repository

↓

Network

↓

Server
```

Problem:

No connection:

```
No data
```

---

## Offline-First

Architecture:

```
UI

↓

Repository

↓

Local Database

↓

Network Sync

↓

Server
```

Benefits:

* Instant UI
* Better user experience
* Lower network usage

---

# 4. Offline-First Architecture Pattern

Recommended architecture:

```
                 UI

                  |

              ViewModel

                  |

             Repository

                  |

       ---------------------

       |                   |

   Local Data          Remote Data

   Room DB             Apollo Client

       |                   |

       --------Sync---------

                  |

             GraphQL Server
```

---

The repository decides:

```
Should I read from database?

Should I call network?

Should I synchronize?
```

---

# 5. Data Source Strategy

A good offline app has:

## Local Source

Example:

```
Room Database

Apollo Cache

DataStore
```

Used for:

* Fast reads
* Offline access

---

## Remote Source

Example:

```
Apollo GraphQL Client
```

Used for:

* Latest data
* Synchronization

---

# 6. Apollo Kotlin Cache Deep Dive

Apollo provides:

```
Normalized Cache
```

It stores GraphQL objects.

Example response:

```graphql
query {
 user(id:10){
   id
   name
 }
}
```

Cache:

```
User:10

{
 id:10,
 name:"John"
}
```

---

Another screen:

```graphql
query {
 user(id:10){
   name
 }
}
```

Apollo reads:

```
User:10
```

instead of network.

---

# 7. Memory Cache vs Persistent Cache

## Memory Cache

Stored:

```
RAM
```

Lifetime:

```
While app is running
```

Example:

```
Open app

↓

Data cached

↓

Close app

↓

Cache removed
```

---

## Persistent Cache

Stored:

```
Disk
```

Lifetime:

```
Survives app restart
```

Example:

```
Open app

↓

Load cache

↓

Show old data

↓

Sync
```

---

# 8. Apollo Persistent Cache

Architecture:

```
Apollo Client

      |

Normalized Cache

      |

Persistent Storage

      |

Disk
```

---

Example:

```kotlin
val apolloClient =
ApolloClient.Builder()
    .serverUrl(URL)
    .normalizedCache(
        SqlNormalizedCacheFactory(
            database
        )
    )
    .build()
```

---

Benefits:

* Faster startup
* Offline reads
* Reduced API calls

---

# 9. Cache Policies

Apollo supports different strategies.

---

## Cache First

Flow:

```
Check Cache

↓

If available

Return Data

↓

Otherwise Network
```

Best for:

```
Profile

Settings

Product details
```

---

## Network Only

Always fetch:

```
Network

↓

Server
```

Best for:

```
Payment

Live information
```

---

## Cache And Network

Flow:

```
Cache

↓

Display

↓

Network

↓

Update UI
```

Best for:

```
Feeds

Dashboards
```

---

## Cache Only

Only local data.

Best for:

```
Offline mode
```

---

# 10. Room + Apollo Architecture

For complex apps:

Use:

```
Room

+

Apollo Cache
```

Architecture:

```
                UI

                 |

             Repository

                 |

       --------------------

       |                  |

     Room             Apollo

       |                  |

       -------- Sync ------

                 |

             GraphQL API
```

---

Example:

Social media app:

Room stores:

```
Posts

Messages

User profile
```

Apollo handles:

```
Server communication
```

---

# 11. Repository Pattern for Offline Apps

Repository hides data source.

Example:

```kotlin
class UserRepository(
private val room:UserDao,
private val apollo:ApolloClient
)
{


suspend fun getUser()
:User {


val local =
room.getUser()


if(local != null)
return local


val remote =
apollo.query(
UserQuery()
)
.execute()


room.insert(remote)


return remote

}

}
```

---

Flow:

```
UI

↓

Repository

↓

Check Local

↓

Return Data

↓

Sync Network
```

---

# 12. Network Synchronization

Synchronization means:

```
Local Data

+

Server Data

=

Same State
```

---

Example:

Offline:

User creates:

```
New Note
```

Stored:

```
Local Database

status=PENDING
```

Later:

Internet returns.

Sync:

```
Send Note

↓

Server saves

↓

Update local status
```

---

# 13. Sync Strategies

## Strategy 1: Pull Sync

Server → Client

Example:

```
Download latest messages
```

---

## Strategy 2: Push Sync

Client → Server

Example:

```
Upload offline changes
```

---

## Strategy 3: Two-Way Sync

Most common:

```
Client

 ↕

Server
```

---

# 14. Offline Queries

Scenario:

User opens profile.

No internet.

Flow:

```
UI

↓

Apollo Cache

↓

Persistent Storage

↓

Return User Data
```

---

Example:

```kotlin
apolloClient
.query(
UserQuery()
)
.fetchPolicy(
FetchPolicy.CacheOnly
)
.execute()
```

---

# 15. Offline Mutations

Problem:

User performs action:

```
Like Post
```

No internet.

Need:

```
Save locally

↓

Send later
```

---

Example:

Local:

```
Like

postId=100

status=PENDING
```

---

When online:

```
Send Mutation

↓

Update Status
```

---

# 16. Mutation Queue Pattern

Very common production pattern.

Architecture:

```
User Action

↓

Mutation Queue

↓

Local Database

↓

Network Available

↓

Execute Mutation

↓

Remove Queue Item
```

---

Example table:

```
MutationQueue

id

operation

payload

status
```

---

Example:

```
1

LIKE_POST

post=100

PENDING
```

---

# 17. Conflict Resolution

Problem:

Same data changed in two places.

Example:

Server:

```
Name = John
```

Offline device:

```
Name = Johnny
```

Another device:

```
Name = Jonathan
```

Which one wins?

---

Common strategies:

---

## Last Write Wins

Newest update wins.

```
Latest timestamp
```

---

## Server Wins

Server data always wins.

Used in:

```
Banking
```

---

## Client Wins

Local changes override server.

Used in:

```
Notes apps
```

---

## Manual Merge

User chooses.

Used in:

```
Documents
```

---

# 18. Handling Deleted Data

Deletion is difficult offline.

Example:

Device A:

```
Delete message
```

Device B:

```
Edit message
```

---

Solution:

Use soft delete.

Instead of:

```
DELETE row
```

Use:

```
deleted=true
```

---

Sync later:

```
Server resolves conflict
```

---

# 19. Network Monitoring

Android provides:

```
ConnectivityManager
```

Example:

```kotlin
class NetworkMonitor {


fun isOnline():Boolean{

return connectivityManager
.activeNetwork != null

}

}
```

---

Flow:

```
Network Lost

↓

Pause Sync


Network Available

↓

Start Sync
```

---

# 20. Background Synchronization

Use:

```
WorkManager
```

Architecture:

```
WorkManager

↓

Sync Worker

↓

Repository

↓

Apollo Mutation

↓

Server
```

---

Example:

```kotlin
class SyncWorker(
context:Context,
params:WorkerParameters
)
:CoroutineWorker(context,params)
{

override suspend fun doWork()
:Result{

syncData()

return Result.success()

}

}
```

---

# 21. Real-World Example

## Offline Chat Application

Requirements:

```
Send messages offline

Receive messages online

Sync automatically
```

Architecture:

```
Compose UI

↓

ChatViewModel

↓

ChatRepository

↓

----------------

Room

Apollo Client

----------------

↓

GraphQL Server
```

---

Sending message:

Offline:

```
User types message

↓

Save Room

↓

status=PENDING
```

---

Online:

```
Network Available

↓

Apollo Mutation

↓

Server

↓

Update Room
```

---

# 22. Production Architecture

Recommended project structure:

```
feature/chat

 ├── ChatScreen

 ├── ChatViewModel


data

 ├── repository

 ├── local

 │    └── Room

 ├── remote

 │    └── Apollo


sync

 ├── SyncWorker

 └── MutationQueue
```

---

# 23. Best Practices

## 1. Always design offline behavior

Decide:

```
What happens without internet?
```

---

## 2. Keep one source of truth

Usually:

```
Local Database
```

---

## 3. Make sync reliable

Handle:

* Retry
* Failure
* Conflicts

---

## 4. Use background sync

Do not block UI.

---

## 5. Store pending operations

Never lose user actions.

---

## 6. Use timestamps

Important for conflict handling.

---

# 24. Summary

In this chapter:

* Offline-first architecture was explained.
* Apollo cache behavior was discussed.
* Persistent cache was introduced.
* Room + Apollo architecture was covered.
* Repository pattern was explained.
* Sync strategies were discussed.
* Offline mutations were implemented conceptually.
* Mutation queues were introduced.
* Conflict resolution strategies were explained.
* WorkManager synchronization was covered.

---

# 25. Interview Questions

### 1. What is offline-first architecture?

An architecture where the application reads from local storage first and synchronizes with the server when possible.

---

### 2. Difference between cache-first and offline-first?

Cache-first:

```
Optimization strategy
```

Offline-first:

```
Complete application architecture
```

---

### 3. How do you handle offline mutations?

Using:

```
Local database

+

Mutation queue

+

Background sync
```

---

### 4. Why use Room with Apollo?

Apollo handles GraphQL communication.

Room provides:

* Complex local storage
* Queries
* Offline persistence

---

### 5. What is conflict resolution?

The process of deciding what happens when local and server data differ.

---

### 6. How do you sync data automatically?

Using:

```
Network monitoring

+

WorkManager

+

Repository sync logic
```

---

# 📌 Next Chapter (Chapter 23 – Production GraphQL Android Architecture)

Next chapter will cover:

* Enterprise-level GraphQL architecture
* Multi-module Android projects
* Feature-based organization
* Repository patterns
* Hilt dependency injection
* Build optimization
* Large team development practices
* Production-ready GraphQL Android structure.
