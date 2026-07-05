
## What is Android Modularization?

Android modularization is the process of dividing an Android application into multiple independent Gradle modules, where each module has a specific responsibility and communicates with other modules through well-defined APIs.

Instead of this:

```
App
├── UI
├── Login
├── Home
├── Profile
├── Network
├── Database
└── Utils
```

You organize it like this:

```
app
│
├── feature-login
├── feature-home
├── feature-profile
│
├── core-ui
├── core-network
├── core-database
├── core-common
│
└── domain
```

---

# Why do we use modularization?

This is the first interview question.

### 1. Faster Build Time

If you modify only the Login module, Gradle recompiles only that module instead of the whole application.

**Example**

Change:

```
feature-login
```

Only

```
feature-login
```

gets rebuilt.

---

### 2. Better Code Organization

Each module has one responsibility.

Example:

```
feature-login
```

contains only

* Login Screen
* Login ViewModel
* Login Repository

No Home or Profile code.

---

### 3. Multiple Developers Can Work Together

Developer A

```
feature-login
```

Developer B

```
feature-home
```

Developer C

```
core-network
```

Minimal merge conflicts.

---

### 4. Reusability

A module can be reused.

Example

```
core-network
```

contains

* Retrofit
* OkHttp
* API Interceptors

Every feature uses it.

---

### 5. Better Testing

You can test modules independently.

Example

```
feature-login
```

without launching the entire app.

---

### 6. Better Separation of Concerns

UI logic

↓

Domain logic

↓

Data layer

remain separated.

---

# Types of Modules

## 1. App Module

```
app
```

This is the launcher module.

Responsibilities

* Navigation
* Dependency Injection setup
* App configuration

---

## 2. Feature Module

Contains one feature.

Examples

```
feature-login

feature-home

feature-profile

feature-settings
```

Each feature contains

* UI
* ViewModel
* Use Cases

---

## 3. Core Module

Shared by all features.

Example

```
core-network

core-ui

core-common

core-utils

core-database
```

---

## 4. Domain Module

Contains business logic.

Example

```
domain

├── UseCases
├── Repository Interfaces
├── Models
```

The domain module should not depend on Android framework classes.

---

## 5. Data Module

Contains

* Repository implementations
* Remote APIs
* Local database
* Room
* Retrofit

---

# Typical Dependency Flow

```
          app
           |
   -----------------
   |       |       |
 login   home   profile
     \      |      /
      ----------
      domain
         |
      data
         |
   ----------------
   |      |      |
network database common
```

A common rule is:

* Feature modules depend on `domain`.
* `data` implements repository interfaces defined in `domain`.
* Shared functionality lives in `core` modules.

---

# Common Interview Questions

### Q1. Why modularization?

**Answer**

* Faster builds
* Better scalability
* Easier maintenance
* Independent testing
* Team collaboration
* Reusable code

---

### Q2. What is the difference between Feature Module and Core Module?

**Feature Module**

Contains business feature.

Example

```
feature-login
```

**Core Module**

Contains shared code.

Example

```
core-network
```

---

### Q3. Why should the Domain module not depend on Android?

Because business logic should be platform-independent and easy to unit test.

---

### Q4. Can one feature depend on another feature?

Generally, **no**. Features should be independent.

Instead of:

```
Login
   ↓
Profile
```

Use shared abstractions:

```
Login
    ↓
Domain

Profile
    ↓
Domain
```

This reduces coupling and improves maintainability.

---

### Q5. What are Dynamic Feature Modules?

Modules that are downloaded only when needed.

Example:

* Payment
* AR feature
* Chat
* Premium feature

Benefits:

* Smaller initial APK
* Faster installation
* On-demand delivery

---

### Q6. What are the disadvantages of modularization?

* Initial setup is more complex.
* Dependency management requires discipline.
* Too many small modules can increase maintenance overhead.
* Build configuration becomes more involved.

---

# Sample Interview Answer (1–2 minutes)

> "In Android, modularization means splitting the application into multiple Gradle modules, where each module has a single responsibility. We usually organize projects into an app module, feature modules like Login and Home, domain modules for business logic, data modules for repository implementations, and core modules for shared components such as networking and UI. This approach improves build times, enables parallel development by multiple teams, makes testing easier, and keeps the codebase scalable and maintainable. We also avoid direct dependencies between feature modules, using shared interfaces in the domain layer instead."

This is the level of detail most Android interviewers expect from candidates with around **2–5 years of Android development experience**.
