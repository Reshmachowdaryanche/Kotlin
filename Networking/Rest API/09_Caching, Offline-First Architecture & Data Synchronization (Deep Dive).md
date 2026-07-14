# Module 9 — Caching, Offline-First Architecture & Data Synchronization (Deep Dive)

This module is where networking becomes **production Android architecture**.

A beginner thinks:

> "I call API → show response."

A production app thinks:

> "What happens when the user has no internet?"

Examples:

* WhatsApp should show old chats when offline.
* Banking apps should show previous transactions.
* Shopping apps should show products even with bad connectivity.
* News apps should display cached articles instantly.

This module is heavily asked for **mid-level and senior Android interviews**.

---

# Interview Goals

After this module, you should be able to explain:

* What is caching?
* Why do we need caching?
* Types of cache
* HTTP cache vs Database cache
* OkHttp cache
* Room as local database
* Offline-first architecture
* Single Source of Truth
* Repository pattern
* Network Bound Resource pattern
* Sync strategies
* Cache invalidation
* Stale data handling

---

# Part 1 — What is Caching?

Caching means:

> Storing previously fetched data so it can be reused instead of fetching it again.

Example:

First request:

```text id="9x7s2q"
Android App

↓

Server

↓

Users Data
```

Store:

```text id="q7hzp1"
Local Cache

Users Data
```

Next time:

```text id="9b7w9k"
Android App

↓

Local Cache

↓

Instant Data
```

Benefits:

* Faster UI
* Less network usage
* Better offline experience
* Reduced server load

---

# Part 2 — Types of Cache in Android Networking

There are multiple levels.

```text id="0y3t4k"
UI Memory Cache

        ↓

Disk Cache

        ↓

Database

        ↓

Network
```

---

## Level 1 — Memory Cache

Data stored in RAM.

Example:

```kotlin id="y5y6qd"
val userList = mutableListOf<User>()
```

Very fast.

But:

* Lost when app closes
* Limited size

Used for:

* Temporary UI data
* Images
* Current screen state

---

## Level 2 — HTTP Cache (OkHttp Cache)

Handled by OkHttp.

Example:

Server response:

```http id="9j2s6r"
Cache-Control:
max-age=3600
```

Meaning:

"Client can reuse this response for 1 hour."

Flow:

```text id="x1w4m8"
Request

↓

OkHttp Cache

↓

If valid:

Return cached response


If expired:

Network request
```

---

## Level 3 — Database Cache

Usually:

* Room Database
* SQLite

Example:

```text id="q8a1f5"
Room Database

users table

id
name
email
```

This is the most important cache for offline-first apps.

---

# Part 3 — OkHttp Cache

Let's understand HTTP caching.

Example:

First request:

```http id="8w7j8s"
GET /products
```

Server response:

```http id="fd8h2j"
Cache-Control:
max-age=3600
```

OkHttp stores it.

Second request:

```http id="2q1x8r"
GET /products
```

OkHttp checks:

```
Is cached data still valid?
```

Yes:

```text id="7l9q7m"
Return cache

No network
```

---

## Enabling OkHttp Cache

Example:

```kotlin id="6h8s9c"
val cacheSize = 10L * 1024 * 1024

val cache = Cache(
    directory,
    cacheSize
)

val client =
    OkHttpClient.Builder()
        .cache(cache)
        .build()
```

---

# Interview Question

## Does Retrofit provide caching?

Correct answer:

> No. Retrofit itself does not provide caching. HTTP caching is handled by OkHttp, and application-level caching is usually implemented using local storage such as Room.

---

# Part 4 — Why OkHttp Cache Is Not Enough

Many developers think:

"Why not use OkHttp cache everywhere?"

Problem:

HTTP cache is limited.

Example:

You need:

* Modify data
* Query local data
* Search offline
* Observe changes
* Sync background updates

HTTP cache cannot do this.

Example:

User opens app offline:

Need:

```text id="8y2p5a"
Show saved products
```

Room is better.

---

# Part 5 — Database Cache with Room

Typical architecture:

```text id="k8m1x5"
             UI

              |

          ViewModel

              |

          Repository

          /          \

      Room          Retrofit

        |              |

   Local Data       Server
```

Repository decides:

* When to read database
* When to call network
* When to update database

---

# Part 6 — Single Source of Truth (SSOT)

This is a very important interview concept.

Definition:

> The UI should read data from one source, usually the local database, while other sources update that database.

Meaning:

Wrong:

```text id="2c7f5j"
UI

 ↙      ↘

Room    Network
```

Problem:

Two sources.

Data inconsistency.

---

Correct:

```text id="4f8z2k"
          UI

           |

        Room DB

           ↑

       Repository

           ↑

        Network
```

Network updates database.

UI observes database.

---

# Example

Without SSOT:

```text id="6r8z2x"
API returns users

↓

Show UI

↓

Save database
```

Problem:

UI temporarily has different data than database.

---

With SSOT:

```text id="q2n8k1"
API

↓

Update Room

↓

Room emits Flow

↓

UI updates
```

---

# Part 7 — Offline-First Architecture

Definition:

> Offline-first means the application is designed to work with local data as the primary source and synchronize with the network when available.

Flow:

```text id="6m7d2p"
User Opens App

↓

Read Room

↓

Show Cached Data

↓

Check Network

↓

Fetch Latest Data

↓

Update Room

↓

UI Automatically Refreshes
```

---

# Part 8 — Network Bound Resource Pattern

Very common interview topic.

The idea:

Repository manages:

* Local database
* Remote API

Flow:

```text id="t5z9h3"
Observe Database

        ↓

Fetch Network Data

        ↓

Save Response To Database

        ↓

Database Updates UI
```

---

Pseudo implementation:

```kotlin id="f1p7w9"
fun getUsers(): Flow<List<User>> {

    return room.observeUsers()
        .onStart {

            refreshFromNetwork()

        }

}
```

---

Detailed flow:

```
UI starts observing

↓

Database emits old data

↓

Repository calls API

↓

API success

↓

Save new data

↓

Database emits updated data

↓

UI refreshes
```

---

# Part 9 — Why Save API Response Into Database?

Suppose:

API returns:

```json id="9n6s0f"
[
 {
  "id":1,
  "name":"John"
 }
]
```

Instead of:

```text id="f7p3c9"
API → UI
```

Do:

```text id="k6m4z8"
API

↓

Room

↓

UI
```

Advantages:

* Offline support
* Better consistency
* Reactive UI
* Easier testing

---

# Part 10 — Cache Invalidation

One of the hardest computer science problems.

Question:

> How long should cached data remain valid?

Examples:

Weather:

```text id="f3n9d7"
5 minutes
```

Country list:

```text id="r7m1z2"
1 month
```

Bank balance:

```text id="0w5x8j"
Almost always fresh
```

---

Strategies:

## Time Based

Store timestamp.

Example:

```text id="5g8m3q"
lastUpdated = 10:00

Current = 10:30

Expired
```

---

## Version Based

Server sends:

```text id="a8z2k5"
version=5
```

Client has:

```text id="3m9y1f"
version=4
```

Need update.

---

## Manual Refresh

User pulls down.

Example:

```text id="4n8j3x"
Swipe Refresh
```

---

# Part 11 — Synchronization Strategies

When network returns after offline mode:

How do we sync?

---

## Full Sync

Download everything.

Example:

```text id="1w5r9m"
Delete local

↓

Download all
```

Simple but expensive.

---

## Incremental Sync

Download only changes.

Example:

Request:

```http id="d5x9n3"
GET /users?updatedAfter=10:00
```

Returns:

Only changed users.

Better for large data.

---

## Conflict Handling

What if:

Local:

```text id="6y2k9p"
Name = John
```

Server:

```text id="8w3m7a"
Name = Johnny
```

Who wins?

Strategies:

### Last Write Wins

Latest timestamp wins.

---

### Server Wins

Server data is authoritative.

---

### Client Wins

Local changes overwrite server.

---

# Part 12 — Room + Retrofit Architecture

Production architecture:

```text id="4p8w2z"

                 UI

                  |

              ViewModel

                  |

             Repository

              /       \

             /         \

          Room       Retrofit

            |            |

        SQLite        OkHttp

                         |

                       Server
```

---

# Part 13 — Example Repository Logic

Conceptually:

```kotlin id="5n8m2a"
class UserRepository(
    private val api: Api,
    private val dao: UserDao
){

    fun users(): Flow<List<User>> {

        return dao.getUsers()
            .onStart {

                val response =
                    api.getUsers()

                dao.insert(
                    response
                )

            }

    }

}
```

Important:

UI never directly talks to Retrofit.

---

# Part 14 — Paging

Large data needs pagination.

Example:

Millions of products.

Don't load:

```text id="1k8m6z"
1,000,000 products
```

Instead:

```text id="6y3m9q"
Page 1

20 items
```

Then:

```text id="2q8m7v"
Page 2

20 items
```

Android commonly uses:

Android Paging Library

It integrates:

* Remote data
* Local database
* Pagination
* Loading states

---

# Part 15 — Interview Questions

## Q1. Difference between HTTP cache and database cache?

Strong answer:

> HTTP cache stores server responses according to HTTP caching rules and is managed by OkHttp. Database cache stores application data and provides structured querying, offline access, and reactive updates.

---

## Q2. What is Single Source of Truth?

Answer:

> Single Source of Truth means the UI observes one reliable data source, usually the local database. Network responses update the database, and database changes update the UI.

---

## Q3. Explain offline-first architecture.

Answer:

> Offline-first architecture treats local storage as the primary source of data. The app displays cached data immediately and synchronizes with the server whenever network connectivity is available.

---

## Q4. Why use Room with Retrofit?

Answer:

> Retrofit provides remote data access, while Room provides persistence and offline capabilities. Together they allow apps to work reliably with unreliable networks.

---

## Q5. Where should caching logic exist?

Answer:

> Caching decisions should usually exist in the repository layer because it coordinates between local and remote data sources.

---

# Common Interview Mistakes

❌

"Cache means storing API response in memory."

Wrong.

Cache can exist at multiple layers.

---

❌

"UI should call API and database separately."

Wrong.

Creates multiple sources of truth.

---

❌

"Offline means no API calls."

Wrong.

Offline-first means the app continues working using local data and synchronizes when possible.

---

# Senior Interview Answer

If asked:

> "Design an offline-first data layer."

Answer:

> "I would use a repository as the single entry point for data. The repository coordinates between Room and Retrofit. The UI observes Room as the single source of truth through Flow or LiveData. When data needs refreshing, the repository fetches from Retrofit, updates Room, and Room automatically propagates changes to the UI. This provides offline support, consistency, and reactive updates."

---

# Module 9 Summary

You now understand:

✅ HTTP cache
✅ Database cache
✅ OkHttp caching
✅ Room caching
✅ Single Source of Truth
✅ Offline-first architecture
✅ Repository pattern
✅ Network Bound Resource
✅ Sync strategies
✅ Cache invalidation
✅ Conflict handling

---

## Next Module: Module 10 — Coroutines + Retrofit Networking Internals

This is where we connect networking with Kotlin concurrency:

* How `suspend` Retrofit APIs work internally
* Coroutine cancellation
* Dispatchers
* Structured concurrency
* `withContext`
* Parallel API calls
* Exception propagation
* Flow with networking

This module is extremely important for Android interviews because modern Android networking is almost always coroutine-based.
