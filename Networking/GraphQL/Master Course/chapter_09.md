# Chapter 9 – GraphQL Subscriptions Deep Dive

## Real-Time Data with GraphQL

## GraphQL for Android Developers

> **Learning Goal**
>
> By the end of this chapter, you will understand:
>
> * What GraphQL Subscriptions are
> * Difference between Query, Mutation, and Subscription
> * How real-time communication works
> * WebSocket architecture
> * Subscription lifecycle
> * Subscription syntax
> * Real Android use cases
> * Apollo Kotlin subscription implementation
> * Kotlin Flow integration
> * Handling connection failures
> * Best practices for production apps

---

# Table of Contents

1. Introduction to Subscriptions
2. Why Do We Need Real-Time Data?
3. Query vs Mutation vs Subscription
4. How Subscriptions Work
5. Subscription Architecture
6. WebSockets
7. GraphQL Subscription Syntax
8. Subscription Variables
9. Subscription Response
10. Real Android Use Cases
11. Apollo Kotlin Subscription Setup
12. Subscription Implementation
13. Kotlin Flow Integration
14. Handling Connection State
15. Reconnection Strategy
16. Subscription vs Polling
17. Performance Considerations
18. Best Practices
19. Summary
20. Interview Questions

---

# 1. Introduction to Subscriptions

A **Subscription** is a GraphQL operation used for receiving real-time updates from a server.

Unlike queries:

```text
Client → Server → Response
```

Subscriptions create a continuous connection:

```text
Client ←→ Server

Server can send updates anytime
```

---

Examples:

* Chat messages
* Live notifications
* Stock prices
* Sports scores
* Delivery tracking
* Multiplayer games
* Online collaboration

---

# 2. Why Do We Need Real-Time Data?

Imagine a chat application.

Without subscriptions:

```
Android App

↓

Every 5 seconds:

"Any new messages?"

↓

Server

↓

"No"

↓

Repeat
```

This is called **polling**.

Problems:

* Wastes battery
* Uses unnecessary network calls
* Delays message delivery

---

With subscriptions:

```
Android App

↓

"Notify me when a message arrives"

↓

Server keeps connection

↓

New message arrives

↓

Server pushes instantly
```

Benefits:

* Instant updates
* Less network usage
* Better user experience

---

# 3. Query vs Mutation vs Subscription

GraphQL has three operation types.

```
              GraphQL

                 |

    ----------------------------

    |             |            |

 Query       Mutation    Subscription

 Read        Write       Real-time
```

---

## Query

Purpose:

Fetch data.

Example:

```graphql
query {
    messages {
        text
    }
}
```

Flow:

```
Request
  |
  ↓
Response
```

Connection closes.

---

## Mutation

Purpose:

Modify data.

Example:

```graphql
mutation {
    sendMessage(
        text:"Hello"
    ){
        id
    }
}
```

Flow:

```
Request
  |
  ↓
Change Data
  |
  ↓
Response
```

---

## Subscription

Purpose:

Receive future updates.

Example:

```graphql
subscription {
    messageAdded {
        text
    }
}
```

Flow:

```
Connect

   ↓

Wait

   ↓

Receive updates

   ↓

Receive updates

   ↓

Receive updates
```

---

# 4. How Subscriptions Work

Let's understand the complete flow.

Example:

A user opens a chat screen.

---

## Step 1: Client Starts Subscription

Android sends:

```graphql
subscription {
    newMessage {
        id
        text
    }
}
```

---

## Step 2: Server Creates Connection

Server keeps the connection open.

```
Android

<================>

GraphQL Server
```

---

## Step 3: New Event Happens

Someone sends:

```
Hello!
```

---

## Step 4: Server Pushes Data

Server sends:

```json
{
 "data":{
    "newMessage":{
        "id":"1",
        "text":"Hello!"
    }
 }
}
```

No request was made by Android.

The server initiated the response.

---

# 5. Subscription Architecture

Complete architecture:

```
+----------------+
| Android App    |
+-------+--------+
        |
        |
        | WebSocket
        |
        ↓
+----------------+
| Apollo Client  |
+-------+--------+
        |
        |
        ↓
+----------------+
| GraphQL Server |
+-------+--------+
        |
        |
        ↓
+----------------+
| Event System   |
+----------------+
```

---

The server may listen to:

* Database changes
* Message queues
* Events
* External services

Examples:

```
New message inserted

        ↓

Publish event

        ↓

GraphQL Subscription

        ↓

Android receives update
```

---

# 6. WebSockets

Most GraphQL subscriptions use **WebSockets**.

REST normally uses:

```
HTTP Request
       |
       ↓
HTTP Response
```

Connection closes.

---

WebSocket:

```
Client
  |
  |
  |================|
                  |
                Server
```

The connection stays alive.

Both sides can send messages anytime.

---

## Why WebSocket?

Because subscriptions require:

* Persistent connection
* Low latency
* Server-to-client communication

---

# 7. GraphQL Subscription Syntax

Basic syntax:

```graphql
subscription {
    eventName {
        fields
    }
}
```

Example:

```graphql
subscription {
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

Response:

```json
{
 "data":{
    "messageAdded":{
        "id":"100",
        "text":"Hello",
        "sender":{
            "name":"John"
        }
    }
 }
}
```

---

# 8. Subscription Variables

Like queries and mutations, subscriptions can use variables.

Example:

```graphql
subscription MessageUpdates(
    $chatId:ID!
){

    messageAdded(
        chatId:$chatId
    ){

        id

        text

    }

}
```

Variables:

```json
{
 "chatId":"500"
}
```

---

Why variables?

Because the same subscription can work for:

```
Chat 1

Chat 2

Chat 3
```

without changing the query.

---

# 9. Subscription Response

A subscription can send many responses.

Example:

First message:

```json
{
 "data":{
    "messageAdded":{
       "text":"Hi"
    }
 }
}
```

Second message:

```json
{
 "data":{
    "messageAdded":{
       "text":"How are you?"
    }
 }
}
```

Third message:

```json
{
 "data":{
    "messageAdded":{
       "text":"Bye"
    }
 }
}
```

The connection remains active.

---

# 10. Real Android Use Cases

## 1. Chat Application

Examples:

* WhatsApp-style messaging
* Customer support chat
* Team communication

Subscription:

```graphql
subscription {
    messageReceived {
        id
        text
    }
}
```

---

## 2. Push Notifications

Example:

```
Someone liked your post

Someone commented

Friend request received
```

Subscription:

```graphql
subscription {
    notificationReceived {
        title
        body
    }
}
```

---

## 3. Live Sports Scores

Example:

```
India 150/4

↓

India 160/5
```

Subscription:

```graphql
subscription {
    scoreUpdated {
        team
        score
    }
}
```

---

## 4. Delivery Tracking

Example:

```
Order placed

↓

Preparing

↓

Out for delivery

↓

Delivered
```

Subscription:

```graphql
subscription {
    orderStatusChanged {
        status
    }
}
```

---

# 11. Apollo Kotlin Subscription Setup

Apollo supports subscriptions.

Typical dependencies:

```kotlin
dependencies {

    implementation(
      "com.apollographql.apollo:apollo-runtime"
    )

}
```

For WebSocket support:

```kotlin
implementation(
"com.apollographql.apollo:apollo-engine"
)
```

(The exact dependency depends on Apollo Kotlin version.)

---

# 12. Subscription Implementation

GraphQL file:

`MessageSubscription.graphql`

```graphql
subscription MessageSubscription(
    $chatId:ID!
){

    messageAdded(
        chatId:$chatId
    ){

        id

        text

        sender{
            name
        }

    }

}
```

---

Apollo Client:

```kotlin
val subscription =
    MessageSubscription(
        chatId="100"
    )


apolloClient
    .subscription(subscription)
    .execute()
```

---

However, subscriptions are streams.

Usually we collect them using Flow.

---

# 13. Kotlin Flow Integration

Apollo subscriptions work naturally with Kotlin Flow.

Example:

```kotlin
apolloClient
    .subscription(
        MessageSubscription("100")
    )
    .toFlow()
    .collect { response ->

        val message =
            response.data?.messageAdded

        println(message?.text)

    }
```

---

Flow:

```
Subscription

      ↓

Flow

      ↓

collect()

      ↓

Update UI
```

---

# Android Architecture Example

```
Compose UI

     ↓

ViewModel

     ↓

Repository

     ↓

Apollo Subscription

     ↓

Flow

     ↓

StateFlow

     ↓

UI Update
```

---

# ViewModel Example

```kotlin
class ChatViewModel(
    private val repository:ChatRepository
):ViewModel(){


val messages =
    MutableStateFlow<List<Message>>(emptyList())


fun observeMessages(){

viewModelScope.launch {


repository
.observeMessages()
.collect{message ->


messages.value += message


}


}

}

}
```

---

# 14. Handling Connection State

Real apps need to handle:

* Connecting
* Connected
* Disconnected
* Failed

Example:

```kotlin
sealed class ConnectionState{

object Connecting:
ConnectionState()

object Connected:
ConnectionState()

object Disconnected:
ConnectionState()

}
```

UI:

```
Connecting...

Connected ✓

Offline
```

---

# 15. Reconnection Strategy

Mobile networks are unstable.

Examples:

```
User enters elevator

WiFi changes

Mobile data switches

Phone sleeps
```

The subscription may disconnect.

A production app should:

1. Detect disconnect
2. Retry connection
3. Restore subscription

---

Example strategy:

```
Connection Lost

↓

Wait 2 seconds

↓

Reconnect

↓

Subscribe again
```

---

# 16. Subscription vs Polling

## Polling

Example:

```
Every 5 seconds:

GET /messages
```

Problems:

* Extra requests
* Battery drain
* Delayed updates

---

## Subscription

```
Open connection

Server pushes updates
```

Advantages:

* Real-time
* Efficient
* Lower latency

---

Comparison:

| Feature       | Polling           | Subscription |
| ------------- | ----------------- | ------------ |
| Connection    | Repeated requests | Persistent   |
| Real-time     | No                | Yes          |
| Battery usage | Higher            | Lower        |
| Complexity    | Simple            | Higher       |
| Network usage | More              | Less         |

---

# 17. Performance Considerations

## Avoid Too Many Subscriptions

Bad:

```
1000 screens

↓

1000 WebSocket connections
```

Better:

Use one subscription manager.

---

## Close Unused Subscriptions

Example:

User leaves chat screen.

Stop:

```kotlin
job.cancel()
```

---

## Limit Data

Avoid:

```graphql
subscription {

    messages{

        everything

    }

}
```

Request only:

```graphql
subscription {

    messages{

        id

        text

    }

}
```

---

# 18. Best Practices

## 1. Use subscriptions only for real-time needs

Do not replace all queries with subscriptions.

Good:

```
Chat messages
Live tracking
Notifications
```

Bad:

```
User profile
Product list
Settings
```

---

## 2. Combine Query + Subscription

Common pattern:

Initial data:

```
Query

↓

Load existing messages
```

Then:

```
Subscription

↓

Receive new messages
```

Example:

Chat screen:

```
Open chat

↓

Query previous messages

↓

Subscribe for new messages
```

---

## 3. Handle Lifecycle Properly

Android:

```
onStart

↓

Start subscription


onStop

↓

Cancel subscription
```

Avoid memory leaks.

---

# 19. Summary

* Subscriptions provide real-time GraphQL updates.
* They maintain a persistent connection between client and server.
* WebSockets are commonly used for GraphQL subscriptions.
* Queries fetch data once.
* Mutations modify data.
* Subscriptions continuously receive updates.
* Apollo Kotlin integrates subscriptions with Kotlin Flow.
* Android apps should combine queries and subscriptions for real-time screens.
* Connection handling and lifecycle management are essential.

---

# 20. Interview Questions

### 1. What is a GraphQL Subscription?

A subscription is a GraphQL operation that allows clients to receive real-time updates from a server.

---

### 2. How are GraphQL subscriptions implemented?

Most GraphQL subscriptions use WebSockets to maintain a persistent connection between client and server.

---

### 3. Difference between Query and Subscription?

Query returns data once. Subscription keeps listening and receives multiple updates over time.

---

### 4. Why are subscriptions useful in Android apps?

They provide instant updates for features like chat, notifications, live tracking, and real-time dashboards.

---

### 5. How does Apollo Kotlin handle subscriptions?

Apollo Kotlin converts subscriptions into Kotlin Flow streams, allowing Android developers to collect updates using coroutines.

---

### 6. Should every screen use subscriptions?

No. Use subscriptions only where real-time updates are required. Normal screens should use queries because subscriptions consume more resources.

---

# 📌 Next Chapter (Chapter 10 – GraphQL Variables, Arguments & Directives Deep Dive)

Next chapter will cover:

* Variables in detail
* Arguments
* Input validation
* Default values
* Query directives
* `@include`
* `@skip`
* `@deprecated`
* Custom directives
* Android Apollo variable handling
* Practical examples with filters, search, and pagination.
