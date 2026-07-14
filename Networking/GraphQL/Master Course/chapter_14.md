
# Chapter: Apollo Kotlin Architecture and Complete Android GraphQL Integration

## 1. Apollo Kotlin Architecture

* What Apollo Kotlin is
* GraphQL client architecture
* How Apollo fits into an Android app
* Data flow:

```
UI (Compose/XML)
      |
ViewModel
      |
Repository
      |
Apollo Client
      |
GraphQL Server
```

* Main Apollo components:

  * ApolloClient
  * Queries
  * Mutations
  * Subscriptions
  * Generated models
  * Cache layer
  * Network layer

---

# 2. Project Setup

## Requirements

Example stack:

* Android Studio
* Kotlin
* Gradle Kotlin DSL
* Apollo Kotlin 4.x
* Coroutines
* Jetpack ViewModel
* Hilt (optional)

## Create Android Project

Structure:

```
app
 ├── src/main/java/com/example/app
 │       ├── data
 │       │    ├── repository
 │       │    └── remote
 │       │
 │       ├── domain
 │       │
 │       └── presentation
 │            ├── ui
 │            └── viewmodel
 │
 └── src/main/graphql
```

Apollo uses:

```
src/main/graphql
```

for GraphQL files.

---

# 3. Gradle Configuration

Add Apollo plugin.

`settings.gradle.kts`

```kotlin
pluginManagement {
    repositories {
        google()
        mavenCentral()
        gradlePluginPortal()
    }
}
```

Project:

```kotlin
plugins {
    id("com.apollographql.apollo3") version "4.x.x"
}
```

App:

```kotlin
plugins {
    id("com.android.application")
    id("org.jetbrains.kotlin.android")
    id("com.apollographql.apollo3")
}
```

Dependency:

```kotlin
dependencies {

    implementation(
        "com.apollographql.apollo:apollo-runtime:4.x.x"
    
    )

}
```

Configure Apollo:

```kotlin
apollo {

    service("service") {

        packageName.set(
            "com.example.graphql"
        )

    }

}
```

---

# 4. Schema Download

Apollo needs the GraphQL schema.

Schema file:

```
app/src/main/graphql/schema.graphqls
```

Download:

```bash
./gradlew :app:downloadApolloSchema \
--endpoint=https://api.example.com/graphql \
--schema=app/src/main/graphql/schema.graphqls
```

After download:

```
graphql
 |
 ├── schema.graphqls
 ├── GetUsers.graphql
 └── CreateUser.graphql
```

---

# 5. GraphQL Files

Apollo generates Kotlin classes from `.graphql` files.

Example:

`GetUsers.graphql`

```graphql
query GetUsers {

    users {

        id
        name
        email

    }

}
```

Apollo generates:

```
GetUsersQuery.kt
User.kt
```

---

# 6. Code Generation

When building:

```bash
./gradlew build
```

Apollo generates Kotlin models.

Generated:

```
build/generated/source/apollo/
```

Example:

```kotlin
GetUsersQuery.Data
```

Usage:

```kotlin
val response =
    apolloClient
        .query(GetUsersQuery())
        .execute()
```

---

# 7. Apollo Client Creation

Create client:

```kotlin
val apolloClient = ApolloClient.Builder()

    .serverUrl(
        "https://api.example.com/graphql"
    )

    .build()
```

Usually provided using Hilt:

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object NetworkModule {


@Provides
@Singleton
fun provideApolloClient():

ApolloClient {

return ApolloClient.Builder()

.serverUrl(
"https://api.example.com/graphql"
)

.build()

}

}
```

---

# 8. Queries

Queries read data.

Example:

```graphql
query GetUser {

 user(id:1){

    id
    name

 }

}
```

Android:

```kotlin
val response =
apolloClient
.query(
GetUserQuery()
)
.execute()


val user =
response.data?.user
```

---

# 9. Mutations

Mutations modify data.

Example:

```graphql
mutation CreateUser(
$name:String!
){

createUser(
name:$name
){

id
name

}

}
```

Kotlin:

```kotlin
apolloClient
.mutation(
CreateUserMutation(
"Alice"
)
)
.execute()
```

---

# 10. Subscriptions

Subscriptions receive real-time updates.

Example:

```graphql
subscription MessageAdded {


messageAdded {

id
text

}

}
```

Kotlin:

```kotlin
apolloClient
.subscription(
MessageAddedSubscription()
)
.toFlow()
.collect {

println(it.data)

}
```

Common uses:

* Chat apps
* Notifications
* Live dashboards

---

# 11. Error Handling

Apollo response:

```kotlin
val response =
apolloClient.query(
GetUsersQuery()
)
.execute()
```

Check:

```kotlin
if(response.hasErrors()){

response.errors?.forEach {

println(it.message)

}

}
```

Better approach:

```kotlin
sealed class ResultState<T>{

data class Success<T>(
val data:T
):ResultState<T>()


data class Error<T>(
val message:String
):ResultState<T>()

}
```

---

# 12. Caching Configuration

Apollo supports normalized caching.

Dependency:

```kotlin
implementation(
"com.apollographql.apollo:apollo-normalized-cache-sqlite"
)
```

Configure:

```kotlin
val apolloClient =
ApolloClient.Builder()

.serverUrl(
"https://api.example.com/graphql"
)

.normalizedCache(
SqlNormalizedCacheFactory(
context,
"apollo.db"
)

)

.build()
```

Benefits:

* Offline support
* Less network usage
* Faster UI

---

# 13. Repository Pattern

Recommended architecture:

```
ViewModel

    |
Repository

    |
Apollo Client

    |
GraphQL API
```

Example:

```kotlin
class UserRepository(
private val apollo:ApolloClient
){


suspend fun getUsers():

List<User>{

val response =
apollo.query(
GetUsersQuery()
)
.execute()


return response.data
?.users
.orEmpty()

}

}
```

---

# 14. ViewModel Integration

```kotlin
class UserViewModel(
private val repository:UserRepository
)
:ViewModel(){


val users =
MutableStateFlow<List<User>>(emptyList())


fun loadUsers(){

viewModelScope.launch{

users.value =
repository.getUsers()

}

}

}
```

---

# 15. Complete Android Project Example

Final structure:

```
GraphQLApp

app

├── graphql

│    ├── schema.graphqls
│    ├── GetUsers.graphql
│    └── CreateUser.graphql


├── data

│    ├── ApolloProvider.kt
│    └── UserRepository.kt


├── domain

│    └── User.kt


├── presentation

│    ├── UserViewModel.kt
│    └── UserScreen.kt


└── MainActivity.kt

```

Flow:

```
User taps screen

        ↓

Compose UI

        ↓

ViewModel

        ↓

Repository

        ↓

Apollo Client

        ↓

GraphQL Server

        ↓

Generated Kotlin Models

        ↓

UI Update
```

## Final Chapter Outcome

After completing this chapter, you will understand:

✅ Apollo Kotlin architecture
✅ Setting up Apollo in Android
✅ Gradle configuration
✅ Schema management
✅ Writing GraphQL queries/mutations/subscriptions
✅ Generated Kotlin code
✅ Apollo Client creation
✅ Error handling strategies
✅ Local caching
✅ Repository-based clean architecture
✅ Building a complete GraphQL Android application

I can also turn this into a **full tutorial chapter with runnable code (Apollo Kotlin 4 + Jetpack Compose + Hilt + MVVM)** if you want a book-style version.
