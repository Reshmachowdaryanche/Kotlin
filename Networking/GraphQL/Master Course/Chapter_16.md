# Chapter 16 – GraphQL Subscriptions & Real-Time Data Deep Dive

## Building Real-Time Android Applications with GraphQL Subscriptions and Apollo Kotlin

> **Learning Goal**
>
> By the end of this chapter, you will understand:
>
> * What GraphQL subscriptions are
> * Difference between Query, Mutation, and Subscription
> * How real-time communication works
> * WebSocket architecture
> * Subscription lifecycle
> * Apollo Kotlin subscription implementation
> * Handling live data using Kotlin Flow
> * Authentication with subscriptions
> * Reconnection handling
> * Real-world examples
> * Building chat and notification systems
> * Production considerations

---

# Table of Contents

1. Introduction to Real-Time Applications
2. What is GraphQL Subscription?
3. Query vs Mutation vs Subscription
4. Why HTTP Request/Response Is Not Enough
5. WebSocket Architecture
6. GraphQL Subscription Flow
7. Subscription Schema Design
8. Creating a Subscription Operation
9. Apollo Kotlin Subscription Setup
10. Executing Subscriptions
11. Using Kotlin Flow
12. Subscription Lifecycle
13. Authentication with Subscriptions
14. Reconnection Handling
15. Error Handling
16. Real-Time Chat Example
17. Live Notification Example
18. Live Sports Example
19. Subscription and Apollo Cache
20. Android Architecture
21. Best Practices
22. Summary
23. Interview Questions

---

# 1. Introduction to Real-Time Applications

Modern applications often need data instantly.

Examples:

## Messaging Apps

When someone sends:

```
Hello
```

The receiver should immediately see:

```
New message received
```

No refresh button.

---

## Other real-time examples

* Chat applications
* Stock prices
* Delivery tracking
* Multiplayer games
* Live comments
* Notifications
* Sports scores
* Collaboration tools

---

Traditional approach:

```
Android App

   |
   |
Every 5 seconds ask server

   |
   |
Any new data?
```

This is called:

## Polling

---

Problem:

* More network usage
* Battery consumption
* Delayed updates

---

GraphQL Subscriptions solve this.

---

# 2. What is GraphQL Subscription?

A subscription is a GraphQL operation used for receiving real-time updates from the server.

Unlike queries:

```
Client → Server
Request

Server → Client
Response
```

Subscription keeps a connection open.

---

Normal Query:

```graphql
query {
    user(id:10){
        name
    }
}
```

One response.

---

Subscription:

```graphql
subscription {
    messageAdded {
        id
        text
    }
}
```

Many responses over time.

---

Example:

Server sends:

First event:

```json
{
 "messageAdded":{
    "text":"Hello"
 }
}
```

Later:

```json
{
 "messageAdded":{
    "text":"How are you?"
 }
}
```

The connection remains active.

---

# 3. Query vs Mutation vs Subscription

| Operation    | Purpose         | Connection            |
| ------------ | --------------- | --------------------- |
| Query        | Read data       | Single request        |
| Mutation     | Modify data     | Single request        |
| Subscription | Receive updates | Long-lived connection |

---

Example:

## Query

Get messages:

```graphql
query {
 messages {
   text
 }
}
```

---

## Mutation

Send message:

```graphql
mutation {
 sendMessage(
   text:"Hello"
 )
}
```

---

## Subscription

Receive messages:

```graphql
subscription {
 messageAdded {
   text
 }
}
```

---

Complete chat flow:

```
User A

Mutation
(send message)

        ↓

Server

        ↓

Subscription

        ↓

User B receives message
```

---

# 4. Why HTTP Request/Response Is Not Enough

HTTP:

```
Client
 |
 |
Request
 |
 |
Server
 |
 |
Response
```

Connection ends.

---

For real-time:

Server needs to push data:

```
Server
 |
 |
New Event
 |
 |
Client
```

HTTP cannot easily do this.

---

Solutions:

## Polling

Client repeatedly asks.

Example:

```
Every 5 seconds:

Any updates?
```

---

## WebSocket

Persistent two-way connection.

```
Client  <==========> Server
```

---

GraphQL subscriptions usually use WebSockets.

---

# 5. WebSocket Architecture

Architecture:

```
Android App

      |
      |
 WebSocket Connection

      |
      |
GraphQL Server

      |
      |
Database Events
```

---

Example:

Database:

```
New message inserted
```

↓

GraphQL server detects event

↓

Pushes:

```json
{
 "messageAdded":{
   "text":"Hi"
 }
}
```

↓

Android receives instantly.

---

# 6. GraphQL Subscription Flow

Complete flow:

```
1. Android opens WebSocket

          ↓

2. Client sends subscription

          ↓

3. Server registers listener

          ↓

4. Database event occurs

          ↓

5. Server pushes data

          ↓

6. Apollo receives response

          ↓

7. UI updates
```

---

# 7. Subscription Schema Design

Subscriptions are defined in schema.

Example:

```graphql
type Subscription {

    messageAdded: Message

}
```

---

Message type:

```graphql
type Message {

    id:ID!

    text:String!

    sender:User!

    createdAt:String!

}
```

---

Another example:

Notifications:

```graphql
type Subscription {

    notificationReceived:
    Notification

}
```

---

# 8. Creating a Subscription Operation

Create:

```
MessageSubscription.graphql
```

---

Example:

```graphql
subscription MessageSubscription {

    messageAdded {

        id

        text

        sender {

            name

        }

    }

}
```

---

Apollo generates:

```
MessageSubscription

MessageSubscription.Data
```

---

# 9. Apollo Kotlin Subscription Setup

Apollo requires WebSocket transport.

Dependency:

```kotlin
implementation(
"com.apollographql.apollo:apollo-runtime"
)
```

---

Create WebSocket engine:

```kotlin
val apolloClient =
ApolloClient.Builder()

.serverUrl(
"https://api.example.com/graphql"
)

.webSocketServerUrl(
"wss://api.example.com/graphql"
)

.build()
```

---

Notice:

HTTP:

```
https://
```

WebSocket:

```
wss://
```

---

# 10. Executing Subscriptions

Example:

```kotlin
apolloClient
.subscription(
MessageSubscription()
)
.toFlow()
.collect {

response ->

println(
response.data
)

}
```

---

The subscription continues running.

Example output:

```
Message 1 received

Message 2 received

Message 3 received
```

---

# 11. Using Kotlin Flow

Apollo converts subscriptions into Flow.

Architecture:

```
GraphQL Subscription

        ↓

Kotlin Flow

        ↓

StateFlow

        ↓

Compose UI
```

---

Example:

Repository:

```kotlin
class ChatRepository(
private val client:ApolloClient
){

fun messages() =

client
.subscription(
MessageSubscription()
)
.toFlow()

}
```

---

ViewModel:

```kotlin
class ChatViewModel(
private val repository:ChatRepository
)
:ViewModel(){


val messages =
MutableStateFlow<List<Message>>(emptyList())


fun start(){

viewModelScope.launch{


repository.messages()
.collect{

response ->

val message =
response.data?.messageAdded


}

}

}

}
```

---

# 12. Subscription Lifecycle

Subscriptions consume resources.

Lifecycle:

```
Screen Opens

      ↓

Start Subscription

      ↓

Receive Events

      ↓

Screen Closed

      ↓

Cancel Subscription
```

---

Android example:

```kotlin
viewModelScope.launch {

subscription.collect {

}

}
```

When ViewModel clears:

```
Coroutine cancelled
```

Subscription closes.

---

# 13. Authentication with Subscriptions

Normal API:

Header:

```
Authorization:
Bearer TOKEN
```

---

WebSocket also needs authentication.

Example:

Connection parameters:

```json
{
 "Authorization":
 "Bearer eyJ..."
}
```

---

Flow:

```
Open WebSocket

       ↓

Send Token

       ↓

Server Validates

       ↓

Allow Subscription
```

---

# 14. Reconnection Handling

Mobile networks change frequently.

Example:

```
WiFi

  ↓

Disconnected

  ↓

Mobile Data

  ↓

Reconnect
```

---

Good subscription systems handle:

* Connection loss
* Retry
* Backoff
* Token refresh

---

Example:

```
Connection lost

       ↓

Wait 2 seconds

       ↓

Reconnect

       ↓

Resume subscription
```

---

# 15. Error Handling

Possible errors:

## Network Error

Example:

```
No Internet
```

---

## Authentication Error

Example:

```
Token expired
```

---

## Server Error

Example:

```
Subscription unavailable
```

---

Handle:

```kotlin
subscription
.toFlow()
.catch {

error ->

showError()

}
```

---

# 16. Real-Time Chat Example

## Requirement

Chat screen:

```
Messages list

+
New messages instantly
```

---

Schema:

```graphql
type Subscription {

messageAdded:
Message

}
```

---

Subscription:

```graphql
subscription {

messageAdded{

id

text

sender{

name

}

}

}
```

---

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

When new message arrives:

```
Server

 ↓

Apollo

 ↓

Flow

 ↓

StateFlow

 ↓

Compose

 ↓

New Message
```

---

# 17. Live Notification Example

Example:

Banking app:

Transaction occurs:

```
Payment received
```

Server:

```
notificationReceived
```

Subscription:

```graphql
subscription {

notificationReceived{

title

message

}

}
```

Android:

Shows:

```
₹500 received
```

---

# 18. Live Sports Example

Example:

Cricket score:

Initial query:

```graphql
query {

match(id:10){

score

}

}
```

---

Live updates:

```graphql
subscription {

scoreUpdated{

runs

wickets

}

}
```

---

Flow:

```
Query

↓

Current Score


Subscription

↓

Future Updates

```

---

# 19. Subscription and Apollo Cache

When subscription receives data:

Example:

New message:

```json
{
"id":"100",
"text":"Hello"
}
```

Apollo can update cache.

Cache:

Before:

```
Messages

1
2
3
```

After:

```
Messages

1
2
3
4
```

---

Benefits:

* UI updates automatically
* No manual refresh

---

# 20. Android Architecture

Recommended:

```
                 Compose UI

                     |

                ViewModel

                     |

              Repository

                     |

          Apollo Subscription

                     |

                WebSocket

                     |

             GraphQL Server

```

---

Responsibilities:

## Compose

Display events.

---

## ViewModel

Maintain UI state.

---

## Repository

Manage subscription.

---

## Apollo

Manage WebSocket.

---

# 21. Best Practices

## 1. Use subscriptions only when needed

Do not use subscriptions for:

```
Static profile data
Product details
Settings
```

Use queries.

---

## 2. Close subscriptions properly

Avoid:

```
Memory leaks
Battery drain
```

---

## 3. Handle reconnects

Mobile networks are unreliable.

---

## 4. Secure WebSocket connections

Use:

```
wss://
```

Not:

```
ws://
```

---

## 5. Combine Query + Subscription

Common pattern:

```
Open Screen

↓

Query existing data

↓

Subscribe for updates

↓

Update UI
```

Example:

Chat:

```
Query old messages

+

Subscription new messages
```

---

# 22. Summary

* GraphQL subscriptions provide real-time updates.
* Queries fetch data once.
* Mutations modify data.
* Subscriptions maintain a live connection.
* WebSockets are commonly used.
* Apollo Kotlin converts subscriptions into Kotlin Flow.
* Flow integrates naturally with ViewModel and Compose.
* Authentication is required for secure subscriptions.
* Reconnection handling is important for mobile apps.
* Query + Subscription combination is a common production pattern.

---

# 23. Interview Questions

### 1. What is a GraphQL subscription?

A subscription is a GraphQL operation that allows clients to receive real-time updates from the server.

---

### 2. Difference between query and subscription?

Query:

```
Request → Response → Connection closes
```

Subscription:

```
Connection remains open
Server pushes updates
```

---

### 3. Which protocol is used for GraphQL subscriptions?

Usually:

```
WebSocket
```

---

### 4. How does Apollo Kotlin handle subscriptions?

Apollo converts subscription results into Kotlin Flow, which can be collected using coroutines.

---

### 5. How do you use subscriptions with Jetpack Compose?

Architecture:

```
Subscription

↓

Flow

↓

StateFlow

↓

Compose State

↓

UI Update
```

---

### 6. Should every GraphQL operation be a subscription?

No. Use subscriptions only for real-time data. Normal data fetching should use queries.

---

### 7. How do you handle lost WebSocket connections?

Implement:

* Retry logic
* Reconnection
* Token refresh
* Lifecycle-aware cancellation

---

# 📌 Next Chapter (Chapter 17 – GraphQL Error Handling & Debugging Deep Dive)

Next chapter will cover:

* GraphQL error types
* Network errors vs GraphQL errors
* Apollo error handling
* Logging
* Debugging tools
* Server error responses
* Retry strategies
* Monitoring production GraphQL apps
* Android debugging techniques
* Real project examples.
