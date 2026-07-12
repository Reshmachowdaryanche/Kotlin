# Chapter 11 – GraphQL Fragments & Reusable Data Models Deep Dive

## Building Maintainable GraphQL APIs for Android Applications

> **Learning Goal**
>
> By the end of this chapter, you will understand:
>
> * What GraphQL fragments are
> * Why fragments are important
> * Problems without fragments
> * Named fragments
> * Fragment composition
> * Inline fragments
> * Fragment reuse across screens
> * Apollo Kotlin fragment generation
> * Fragments with Jetpack Compose
> * Real Android project examples
> * Fragment best practices

---

# Table of Contents

1. Introduction to Fragments
2. The Problem Without Fragments
3. What is a GraphQL Fragment?
4. Fragment Syntax
5. Named Fragments
6. Using Fragments in Queries
7. Fragment on Different Types
8. Fragment Composition
9. Nested Fragments
10. Inline Fragments
11. Fragments with Interfaces and Unions
12. Apollo Kotlin Fragment Generation
13. Fragments in Android Architecture
14. Real Android Examples
15. Fragment Design Patterns
16. Best Practices
17. Summary
18. Interview Questions

---

# 1. Introduction to Fragments

In GraphQL, a **fragment** is a reusable group of fields.

It allows you to define a set of fields once and reuse them in multiple queries.

Think of a fragment as a reusable component.

---

Example:

Many screens need user information:

```text
Profile Screen

- id
- name
- photo
- email


Feed Screen

- id
- name
- photo


Comment Section

- id
- name
- photo
```

Without fragments, you repeat the same fields everywhere.

---

# 2. The Problem Without Fragments

Imagine a social media app.

## Profile Query

```graphql
query {

    user(id:1){

        id

        name

        avatar

        email

    }

}
```

---

## Feed Query

```graphql
query {

    posts{

        author{

            id

            name

            avatar

        }

    }

}
```

---

## Comment Query

```graphql
query {

    comments{

        author{

            id

            name

            avatar

        }

    }

}
```

Notice:

Repeated fields:

```text
id

name

avatar
```

Problems:

* Duplicate code
* Hard to maintain
* Easy to forget fields
* Different screens may become inconsistent

---

# 3. What is a GraphQL Fragment?

A fragment is a reusable selection of fields.

Syntax:

```graphql
fragment FragmentName on TypeName {

    fields

}
```

Example:

```graphql
fragment UserFields on User {

    id

    name

    avatar

}
```

Meaning:

> Create a reusable field group called `UserFields` for the `User` type.

---

# 4. Fragment Syntax

General structure:

```graphql
fragment Name on ObjectType {

    field1

    field2

    field3

}
```

Example:

```graphql
fragment ProductFields on Product {

    id

    title

    price

    image

}
```

Now any query can use:

```graphql
...ProductFields
```

---

# 5. Named Fragments

A named fragment has a name.

Example:

```graphql
fragment UserProfile on User {

    id

    name

    email

    profileImage

}
```

Here:

Fragment name:

```
UserProfile
```

Type:

```
User
```

Fields:

```
id
name
email
profileImage
```

---

# 6. Using Fragments in Queries

Fragment:

```graphql
fragment UserFields on User {

    id

    name

}
```

Query:

```graphql
query GetUser {

    user(id:1){

        ...UserFields

    }

}
```

The `...` means:

> Include this fragment here.

---

Equivalent query:

```graphql
query GetUser {

    user(id:1){

        id

        name

    }

}
```

The server receives the expanded version.

---

# 7. Fragment on Different Types

Fragments are connected to specific GraphQL types.

Example:

Schema:

```graphql
type User {

    id:ID!

    name:String!

}


type Product {

    id:ID!

    price:Float!

}
```

User fragment:

```graphql
fragment UserData on User {

    id

    name

}
```

Product fragment:

```graphql
fragment ProductData on Product {

    id

    price

}
```

A User fragment cannot be used on Product.

Invalid:

```graphql
product{

    ...UserData

}
```

Error:

```
Fragment UserData cannot be spread here.
```

---

# 8. Fragment Composition

Fragments can contain other fragments.

Example:

Basic user:

```graphql
fragment BasicUser on User {

    id

    name

}
```

Extended user:

```graphql
fragment FullUser on User {

    ...BasicUser

    email

    phone

}
```

Now:

```graphql
FullUser
```

contains:

```
id

name

email

phone
```

---

# 9. Nested Fragments

Real applications use nested fragments.

Example:

User:

```graphql
fragment UserCard on User {

    id

    name

    avatar

}
```

Post:

```graphql
fragment PostCard on Post {

    id

    title

    author{

        ...UserCard

    }

}
```

Query:

```graphql
query Feed {

    posts{

        ...PostCard

    }

}
```

---

Response:

```json
{
 "posts":[
    {
      "id":"10",
      "title":"GraphQL Tutorial",
      "author":{
        "id":"1",
        "name":"John",
        "avatar":"image.png"
      }
    }
 ]
}
```

---

# 10. Inline Fragments

Inline fragments do not have names.

They are used when the returned type is not known.

Commonly used with:

* Interfaces
* Unions

Example:

Schema:

```graphql
union SearchResult =
User |
Product
```

Query:

```graphql
query Search {

    search{

        ... on User {

            name

        }


        ... on Product {

            price

        }

    }

}
```

Meaning:

If result is:

```
User
```

show:

```
name
```

If result is:

```
Product
```

show:

```
price
```

---

# 11. Fragments with Interfaces and Unions

Example:

Schema:

```graphql
interface Character {

    id:ID!

    name:String!

}
```

Types:

```graphql
type Human implements Character {

    id:ID!

    name:String!

    homePlanet:String

}
```

```graphql
type Droid implements Character {

    id:ID!

    name:String!

    primaryFunction:String

}
```

Query:

```graphql
query Characters {

characters{

    name


    ... on Human {

        homePlanet

    }


    ... on Droid {

        primaryFunction

    }

}

}
```

---

# 12. Apollo Kotlin Fragment Generation

Apollo Kotlin reads fragments and generates Kotlin models.

Example:

Fragment:

```graphql
fragment UserFields on User {

    id

    name

    avatar

}
```

Generated Kotlin:

```kotlin
data class UserFields(
    val id:String,
    val name:String,
    val avatar:String?
)
```

---

Query:

```graphql
query Profile {

viewer{

    ...UserFields

}

}
```

Generated:

```kotlin
ProfileQuery.Data.Viewer
```

contains:

```kotlin
viewer.userFields
```

---

# 13. Fragments in Android Architecture

A common Android architecture:

```
Compose UI

     ↓

ViewModel

     ↓

Repository

     ↓

Apollo Client

     ↓

GraphQL Query

```

Fragments help define UI requirements.

---

Example:

## User Card Component

Compose:

```kotlin
@Composable
fun UserCard(
    user:UserCardFragment
){

Text(user.name)

}
```

The UI component receives exactly what it needs.

---

# 14. Real Android Examples

## Example 1: Social Media Feed

Different screens:

```
Home Feed

Profile

Search Results

Comments
```

All need:

```
User name
Avatar
```

Create:

`UserCard.graphql`

```graphql
fragment UserCard on User {

    id

    name

    avatar

}
```

---

Feed:

```graphql
query Feed {

posts{

    title

    author{

        ...UserCard

    }

}

}
```

---

Profile:

```graphql
query Profile {

viewer{

    ...UserCard

}

}
```

---

# Example 2: Product App

Product card:

```graphql
fragment ProductCard on Product {

    id

    title

    price

    image

}
```

Used in:

```
Home Screen

Search Screen

Category Screen

Wishlist Screen
```

---

# Example 3: Chat Application

Message:

```graphql
fragment MessageItem on Message {

    id

    text

    createdAt

    sender{

        ...UserCard

    }

}
```

Used in:

```
Chat Screen

Message Search

Notification Preview
```

---

# 15. Fragment Design Patterns

## Pattern 1: Component-Based Fragments

Create fragments based on UI components.

Example:

```
UserAvatarFragment

UserCardFragment

PostCardFragment

CommentFragment
```

---

## Pattern 2: Screen-Level Fragments

Create fragments for screens.

Example:

```
ProfileScreenFragment

HomeScreenFragment

SettingsScreenFragment
```

---

## Pattern 3: Layered Fragments

Example:

```
UserBasic

       ↓

UserCard

       ↓

UserProfile

```

Reuse smaller fragments.

---

# 16. Best Practices

## 1. Create fragments around UI needs

Good:

```
UserCardFragment
```

because a component needs it.

---

Avoid:

```
EverythingUserFragment
```

with:

```
50 fields
```

---

# 2. Keep fragments small

Good:

```graphql
fragment Avatar on User {

image

}
```

Bad:

```graphql
fragment UserEverything on User {

id

name

email

phone

address

posts

comments

friends

}

```

---

# 3. Avoid unnecessary nesting

Bad:

```
User

 → Posts

    → Comments

       → Author

          → Friends

```

Can create large responses.

---

# 4. Match fragments to UI components

Example:

Compose:

```
PostCard()
```

should use:

```
PostCardFragment
```

---

# 5. Share fragments across features carefully

Avoid creating dependencies between unrelated modules.

---

# 17. Summary

* Fragments are reusable GraphQL field groups.
* They reduce duplicate query code.
* They improve consistency across Android screens.
* Named fragments work with specific object types.
* Inline fragments are used with interfaces and unions.
* Apollo Kotlin generates fragment models automatically.
* UI components can directly consume fragment models.
* Good fragments are small and focused around UI requirements.

---

# 18. Interview Questions

### 1. What is a GraphQL fragment?

A fragment is a reusable collection of fields that can be included in multiple queries or mutations.

---

### 2. Why do we use fragments?

Fragments reduce duplication, improve maintainability, and keep data requirements consistent across components.

---

### 3. What is the difference between named and inline fragments?

Named fragments have a reusable name:

```graphql
fragment UserData on User
```

Inline fragments are written directly:

```graphql
... on User
```

and are commonly used with interfaces and unions.

---

### 4. How does Apollo Kotlin handle fragments?

Apollo generates strongly typed Kotlin classes for fragments, allowing Android components to consume reusable GraphQL data models.

---

### 5. Can a fragment be used on multiple types?

No. A fragment belongs to one GraphQL type. For different types, create separate fragments or use inline fragments.

---

### 6. Why are fragments useful in Jetpack Compose?

Compose components usually need specific data. Fragments allow each component to define and reuse its exact GraphQL data requirements.

---

# 📌 Next Chapter (Chapter 12 – GraphQL Schema Design Deep Dive)

Next chapter will cover:

* GraphQL schema architecture
* Schema Definition Language (SDL)
* Designing scalable schemas
* Object relationships
* Query and Mutation design
* Naming conventions
* Schema evolution
* Deprecation strategies
* Federation basics
* Backend and Android team collaboration
* Production schema examples.
