**Subscription** is the third operation type in GraphQL, after **Query** and **Mutation**.

* **Query** → Read data once
* **Mutation** → Modify data
* **Subscription** → Receive real-time updates whenever data changes

If you're interviewing for Android roles, subscriptions are often discussed in the context of **chat apps, live sports scores, stock prices, ride tracking, or notifications**.

---

# What is a Subscription?

A **subscription** establishes a long-lived connection between the client and the server. Instead of the client repeatedly asking, "Has anything changed?", the server **pushes updates** whenever relevant events occur.

Think of it as subscribing to a YouTube channel:

* You subscribe once.
* Every time a new video is uploaded, you're notified automatically.

The same idea applies to GraphQL subscriptions.

---

# Why Do We Need Subscriptions?

Without subscriptions, an app might poll the server:

```text
Android App
     |
GET /messages
     |
(wait 5 seconds)
     |
GET /messages
     |
(wait 5 seconds)
     |
GET /messages
```

Problems:

* Wastes network bandwidth
* Consumes more battery
* Increases server load
* Updates are delayed until the next poll

With a subscription:

```text
Android App
      |
      | Connect once
      ▼
GraphQL Server
      |
      | Pushes updates automatically
      ▼
Android App
```

The client receives updates immediately.

---

# Basic Subscription Syntax

```graphql
subscription {
  messageAdded {
    id
    text
    sender
  }
}
```

Meaning:

* Listen for new messages.
* Whenever a new message is created, send these fields to the client.

---

# Example: Chat Application

Schema:

```graphql
type Message {
    id: ID!
    text: String!
    sender: String!
}

type Subscription {
    messageAdded: Message!
}
```

Client subscribes:

```graphql
subscription {
    messageAdded {
        id
        text
        sender
    }
}
```

When another user sends:

```text
Hello everyone!
```

The server automatically pushes:

```json
{
  "data": {
    "messageAdded": {
      "id": "10",
      "text": "Hello everyone!",
      "sender": "Alice"
    }
  }
}
```

No additional request is required.

---

# Another Example: Stock Market

```graphql
subscription {
    stockPrice(symbol: "AAPL") {
        symbol
        price
    }
}
```

Possible updates:

```json
{
  "data": {
    "stockPrice": {
      "symbol": "AAPL",
      "price": 215.40
    }
  }
}
```

A few seconds later:

```json
{
  "data": {
    "stockPrice": {
      "symbol": "AAPL",
      "price": 215.52
    }
  }
}
```

The client keeps receiving updates while the subscription remains active.

---

# Example: Ride Tracking

```graphql
subscription {
    driverLocation(driverId: 100) {
        latitude
        longitude
    }
}
```

Every few seconds:

```json
{
  "data": {
    "driverLocation": {
      "latitude": 13.0827,
      "longitude": 80.2707
    }
  }
}
```

The map updates automatically.

---

# How Does It Work?

Unlike queries and mutations, subscriptions usually **don't use a simple request-response pattern**.

Instead:

```text
Android App
     │
     │ Open WebSocket
     ▼
GraphQL Server
     │
     │ Connection remains open
     ▼
Whenever data changes
     │
     ▼
Server pushes event
     │
     ▼
Android UI updates
```

The connection stays open until the client disconnects.

---

# Why WebSockets?

Queries and mutations generally use HTTP:

```text
Request
↓

Response

Connection closed
```

Subscriptions usually use **WebSockets**:

```text
Open Connection

↓

Keep Connection Alive

↓

Server Pushes Updates

↓

Keep Connection Alive

↓

Client Disconnects
```

A single connection can deliver many updates.

---

# Query vs Mutation vs Subscription

| Feature           | Query                | Mutation             | Subscription                                       |
| ----------------- | -------------------- | -------------------- | -------------------------------------------------- |
| Purpose           | Read data            | Modify data          | Receive live updates                               |
| Communication     | Client requests once | Client requests once | Client subscribes once; server pushes many updates |
| Typical Transport | HTTP                 | HTTP                 | Usually WebSocket                                  |
| Connection        | Short-lived          | Short-lived          | Long-lived                                         |
| Real-time         | No                   | No                   | Yes                                                |

---

# Real Android Examples

### Chat

```graphql
subscription {
    messageAdded {
        text
        sender
    }
}
```

Every new message appears instantly.

---

### Cricket Score

```graphql
subscription {
    liveScore(matchId: 5) {
        runs
        wickets
    }
}
```

The score updates ball by ball.

---

### Food Delivery

```graphql
subscription {
    orderStatus(orderId: 200) {
        status
    }
}
```

Possible updates:

```text
Preparing
↓

Out for Delivery
↓

Delivered
```

---

### Social Media

```graphql
subscription {
    newNotification {
        title
        message
    }
}
```

The user receives notifications as soon as they're available.

---

# Apollo Kotlin Example

Suppose you have:

```graphql
subscription NewMessages {
    messageAdded {
        id
        text
        sender
    }
}
```

Apollo Kotlin generates a `NewMessagesSubscription` class.

Using Kotlin Coroutines:

```kotlin
apolloClient.subscription(NewMessagesSubscription())
    .toFlow()
    .collect { response ->
        val message = response.data?.messageAdded
        println(message?.text)
    }
```

Here:

* `subscription()` starts listening.
* `toFlow()` converts incoming events into a Kotlin `Flow`.
* `collect()` receives each update as it arrives.

This integrates naturally with Android architectures that already use Kotlin Flow.

---

# What Happens Internally?

```text
Android App
      │
      │ Start subscription
      ▼
Apollo Client
      │
      │ Open WebSocket
      ▼
GraphQL Server
      │
      │ Wait for new event
      ▼
Database changes
      │
      ▼
Subscription resolver triggered
      │
      ▼
Server sends event
      │
      ▼
Apollo receives event
      │
      ▼
Flow emits new value
      │
      ▼
UI updates
```

---

# Common Interview Questions

### 1. Why use a subscription instead of polling?

Subscriptions provide real-time updates with lower latency and typically reduce unnecessary network requests compared to frequent polling.

---

### 2. Does every GraphQL application need subscriptions?

No. If your application only fetches or updates data occasionally, queries and mutations are sufficient. Subscriptions are useful when users need immediate updates.

---

### 3. Do subscriptions replace queries?

No.

A common pattern is:

* Use a **query** to load the initial screen.
* Use a **subscription** to receive future updates.
* Use a **mutation** when the user changes data.

For example, in a chat app:

1. Query the latest 50 messages.
2. Subscribe to new incoming messages.
3. Send a new message using a mutation.

---

### 4. Why are subscriptions usually implemented over WebSockets?

Because WebSockets support a persistent, two-way connection. The server can send updates at any time without waiting for a new HTTP request.

---

### 5. Are subscriptions always implemented with WebSockets?

No. While **WebSockets are the most common transport**, the GraphQL specification defines the subscription operation but does **not** require a specific transport protocol. Some implementations use alternatives such as **Server-Sent Events (SSE)**, though WebSockets remain the most widely adopted choice.

## Interview Summary

A concise answer you can give is:

> **Query** is used to fetch data once. **Mutation** is used to create, update, or delete data. **Subscription** is used for real-time updates. The client establishes a long-lived connection—typically over WebSockets—and the server pushes new data whenever relevant events occur, making it ideal for chat applications, live tracking, stock prices, and notifications.



---
# Practical Example code

I'll show a complete **Android + Kotlin + Apollo Kotlin + GraphQL Subscription** example. We'll build a simple **real-time chat message listener**.

Architecture:

```
Android App
    |
    | Apollo Kotlin Client
    |
    | WebSocket
    |
GraphQL Server
    |
Subscription: messageAdded
```

The example assumes you already have a GraphQL server exposing:

```graphql
subscription {
  messageAdded {
    id
    text
    sender
  }
}
```

---

## 1. Add Apollo Kotlin Dependencies

In your app-level `build.gradle`:

```gradle
plugins {
    id("com.apollographql.apollo3") version "4.1.1"
}

dependencies {

    implementation("com.apollographql.apollo:apollo-runtime:4.1.1")

}
```

---

## 2. Configure Apollo Plugin

In the same `build.gradle`:

```gradle
apollo {
    service("chat") {
        packageName.set("com.example.chat")
    }
}
```

Project structure:

```
app
 |
 └── src
      |
      └── main
           |
           └── graphql
                |
                └── MessageAddedSubscription.graphql
```

---

## 3. Create Subscription File

Create:

```
src/main/graphql/MessageAddedSubscription.graphql
```

Add:

```graphql
subscription MessageAddedSubscription {

    messageAdded {

        id

        text

        sender

    }

}
```

Apollo will generate:

```
MessageAddedSubscription
MessageAddedSubscription.Data
```

automatically.

---

# 4. Create Apollo Client with WebSocket

Subscriptions need WebSocket support.

Create:

`ApolloProvider.kt`

```kotlin
package com.example.chat

import com.apollographql.apollo.ApolloClient
import com.apollographql.apollo.network.ws.WebSocketNetworkTransport
import okhttp3.OkHttpClient


object ApolloProvider {


    private const val HTTP_URL =
        "https://example.com/graphql"


    private const val WS_URL =
        "wss://example.com/graphql"


    val client: ApolloClient by lazy {


        ApolloClient.Builder()

            .serverUrl(HTTP_URL)

            .subscriptionNetworkTransport(

                WebSocketNetworkTransport.Builder()

                    .serverUrl(WS_URL)

                    .build()

            )

            .build()

    }

}
```

Explanation:

```kotlin
.serverUrl()
```

handles:

```
Queries
Mutations
```

Example:

```
https://example.com/graphql
```

---

```kotlin
.subscriptionNetworkTransport()
```

handles:

```
Subscriptions
```

Example:

```
wss://example.com/graphql
```

---

# 5. Create Repository

`ChatRepository.kt`

```kotlin
package com.example.chat


import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.map



class ChatRepository {


    private val apolloClient =
        ApolloProvider.client



    fun observeMessages():

            Flow<MessageAddedSubscription.Data?>


    {


        return apolloClient

            .subscription(
                MessageAddedSubscription()
            )

            .toFlow()

            .map {

                response ->

                response.data

            }

    }


}
```

What happens:

```
Subscription starts
        |
        |
WebSocket opens
        |
        |
Server pushes message
        |
        |
Flow emits data
```

---

# 6. Create ViewModel

`ChatViewModel.kt`

```kotlin
package com.example.chat


import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.launch



class ChatViewModel : ViewModel() {


    private val repository =
        ChatRepository()



    private val _messages =
        MutableStateFlow<List<MessageAddedSubscription.MessageAdded>>(emptyList())


    val messages: StateFlow<List<MessageAddedSubscription.MessageAdded>>
        = _messages



    fun startListening() {


        viewModelScope.launch {


            repository
                .observeMessages()
                .collect { data ->


                    val newMessage =
                        data?.messageAdded


                    if(newMessage != null){


                        _messages.value =
                            _messages.value + newMessage


                    }

                }

        }


    }


}
```

---

# 7. Collect Subscription Data in Jetpack Compose

`ChatScreen.kt`

```kotlin
@Composable
fun ChatScreen(
    viewModel: ChatViewModel =
        viewModel()
) {


    val messages by
        viewModel.messages.collectAsState()



    LaunchedEffect(Unit) {

        viewModel.startListening()

    }



    LazyColumn {


        items(messages){ message ->


            Text(

                text =
                "${message.sender}: ${message.text}"

            )


        }


    }

}
```

---

# Complete Flow

Imagine Alice sends:

```
Hello
```

Server event:

```json
{
 "data":{
   "messageAdded":{
      "id":"1",
      "text":"Hello",
      "sender":"Alice"
   }
 }
}
```

Apollo receives it:

```
WebSocket
    |
    |
Apollo Client
    |
    |
Flow emits
    |
    |
ViewModel updates StateFlow
    |
    |
Compose recomposes
    |
    |
UI shows:

Alice: Hello
```

---

# Handling Errors

A production app should handle errors:

```kotlin
apolloClient
    .subscription(
        MessageAddedSubscription()
    )
    .toFlow()
    .collect { response ->


        response.errors?.forEach {

            println(
              it.message
            )

        }


        response.data?.let {

            println(
              it.messageAdded.text
            )

        }

    }
```

---

# Authentication Example

Most apps need a token.

Example:

```kotlin
ApolloClient.Builder()

    .serverUrl(
        "https://example.com/graphql"
    )

    .addHttpHeader(
        "Authorization",
        "Bearer $token"
    )

    .build()
```

For WebSockets, the token is usually sent as connection parameters:

```kotlin
WebSocketNetworkTransport.Builder()

    .serverUrl(
        "wss://example.com/graphql"
    )

    .addHeader(
        "Authorization",
        "Bearer $token"
    )

    .build()
```

(The exact API depends on Apollo Kotlin version.)

---

# Real Production Pattern

A typical Android app uses all three:

```
Opening Chat Screen

        |
        |
Query
(fetch old messages)

        |
        |
Subscription
(listen for new messages)

        |
        |
Mutation
(send message)
```

Example:

```
Query:
GET previous 50 messages

Subscription:
Receive new messages instantly

Mutation:
Send a new message
```

---

# Interview Explanation

If asked **"How do you implement GraphQL subscription in Android?"**, a strong answer:

> "I use Apollo Kotlin as the GraphQL client. I define a subscription operation in a `.graphql` file, Apollo generates type-safe Kotlin classes, configure a WebSocket network transport, execute the subscription using `apolloClient.subscription()`, convert the response stream into Kotlin Flow, and collect updates in a ViewModel to update the UI using StateFlow."


