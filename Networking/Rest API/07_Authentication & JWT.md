Excellent. **Module 7 — Authentication & JWT** is one of the highest-value topics for Android interviews.

Almost every real production Android app needs authentication:

* Banking apps
* E-commerce apps
* Social apps
* Enterprise apps

Interviewers love this topic because it tests whether you understand:

* Security
* HTTP
* Interceptors
* Token lifecycle
* Concurrency problems
* Production architecture

---

# Module 7 — Authentication, JWT & Token Management (Deep Dive)

## Interview Goals

After this module, you should be able to answer:

* What is Authentication?
* What is Authorization?
* Difference between them?
* How login works?
* What is a session?
* What is JWT?
* Access Token vs Refresh Token?
* Where should tokens be stored?
* How does OkHttp add tokens?
* Interceptor vs Authenticator?
* How token refresh works?
* How to handle multiple failed requests?
* Common security mistakes

---

# Part 1 — Authentication vs Authorization

This is a very common interview question.

Many developers confuse them.

---

# Authentication

Authentication answers:

> "Who are you?"

Example:

You enter:

```
Email
Password
```

Server checks:

```
Is this really John?
```

If yes:

```
Authentication successful
```

---

# Authorization

Authorization answers:

> "What are you allowed to do?"

Example:

User logs in successfully.

Server knows:

```
Role = USER
```

Can access:

```
View Profile
Place Order
```

Cannot access:

```
Delete All Users
```

because that requires:

```
Role = ADMIN
```

---

## Simple Difference

| Authentication        | Authorization                |
| --------------------- | ---------------------------- |
| Identity verification | Permission checking          |
| Who are you?          | What can you do?             |
| Login                 | Access control               |
| Happens first         | Happens after authentication |

Flow:

```
User

↓

Authentication

↓

Authorization

↓

Resource Access
```

---

# Part 2 — Basic Login Flow

Let's understand a normal login.

User enters:

```
email
password
```

Android sends:

```http
POST /login
```

Body:

```json
{
 "email":"john@gmail.com",
 "password":"123456"
}
```

---

Server:

1. Checks credentials.
2. Creates identity.
3. Generates tokens.

Response:

```json
{
 "accessToken":"abc123",
 "refreshToken":"xyz789"
}
```

Android stores tokens.

Now future API calls:

```http
GET /profile

Authorization: Bearer abc123
```

---

# Part 3 — Why Do We Need Tokens?

Why not send username/password every time?

Example:

```
GET /profile

email=john@gmail.com
password=123456
```

Problems:

* Password exposure risk
* Repeated authentication
* Difficult session management

Instead:

Login once:

```
Username + Password

↓

Token

↓

Use Token
```

---

# Part 4 — Session Based Authentication

Before JWT, many systems used sessions.

Flow:

Login:

```
Username
Password

↓

Server creates session

↓

sessionId = 12345
```

Client stores:

```
Cookie:
sessionId=12345
```

Next request:

```
GET /profile

Cookie:
sessionId=12345
```

Server checks:

```
Is sessionId valid?
```

---

Problem:

The server must store sessions.

Example:

```
10 million users

↓

10 million sessions
```

Scaling becomes harder.

---

# Part 5 — JWT (JSON Web Token)

JWT solves some of these problems.

A JWT is a digitally signed token containing information.

Example:

```
xxxxx.yyyyy.zzzzz
```

Three parts:

```
Header.Payload.Signature
```

---

Example:

## Header

```json
{
 "alg":"HS256",
 "typ":"JWT"
}
```

Contains:

* Algorithm
* Token type

---

## Payload

```json
{
 "userId":123,
 "role":"USER",
 "exp":1720000000
}
```

Contains:

* User information
* Expiration time

---

## Signature

Used to verify:

* Token was not modified
* Token was created by trusted server

---

# Important Interview Point

JWT is:

* Encoded
* Signed

It is **not encrypted by default**.

Anyone can decode the payload.

Example:

You should NOT put:

```
password
credit card number
secret data
```

inside JWT payload.

---

# Part 6 — Access Token

Access token is used for API access.

Example:

```
Authorization:
Bearer access_token
```

Characteristics:

* Short lifetime
* Used frequently
* Sent with every API request

Example:

```
Expires:
15 minutes
```

---

# Part 7 — Refresh Token

Problem:

Access tokens expire.

Why?

Security.

If a token is stolen:

Long lifetime = dangerous.

Solution:

Short access token.

Long refresh token.

Example:

```
Access Token:

15 minutes


Refresh Token:

30 days
```

---

Flow:

```
Access Token Expired

↓

Send Refresh Token

↓

Server Gives New Access Token

↓

Continue API Calls
```

---

# Part 8 — Where Store Tokens in Android?

Interview question.

Bad:

```kotlin
SharedPreferences
```

for sensitive tokens.

Better:

Use encrypted storage.

Options:

* EncryptedSharedPreferences
* Android Keystore
* Secure storage solutions

---

Why?

Normal SharedPreferences:

```
plain text XML file
```

Rooted devices or compromised environments may expose it.

---

# Part 9 — Adding Token Using Interceptor

Now connect this with Module 6.

Without interceptor:

Every API:

```kotlin
@GET("profile")
suspend fun profile(
 @Header("Authorization")
 token:String
)
```

Terrible.

Every endpoint repeats authentication.

---

Better:

Create interceptor.

Flow:

```
Request

↓

Auth Interceptor

↓

Add Token

↓

Server
```

Example:

```kotlin
class AuthInterceptor(
    private val tokenProvider: TokenProvider
): Interceptor {

    override fun intercept(
        chain: Interceptor.Chain
    ): Response {

        val token = tokenProvider.getToken()

        val request =
            chain.request()
                .newBuilder()
                .addHeader(
                    "Authorization",
                    "Bearer $token"
                )
                .build()

        return chain.proceed(request)
    }
}
```

Now every request automatically contains the token.

---

# Part 10 — Token Expiration Problem

Imagine:

```
Access Token expired
```

Server responds:

```http
401 Unauthorized
```

Now what?

The app needs to:

1. Detect 401
2. Get a new access token
3. Retry original request

This is where many developers make mistakes.

---

# Part 11 — Interceptor vs Authenticator

Very important.

## Interceptor

Runs before request.

Purpose:

Modify outgoing requests.

Example:

```
Add token
Add headers
Logging
```

Flow:

```
Request

↓

Interceptor

↓

Server
```

---

## Authenticator

Runs after authentication failure.

Example:

```
Request

↓

Server

↓

401

↓

Authenticator
```

Purpose:

Recover authentication.

Example:

```
Refresh token

↓

Get new access token

↓

Retry request
```

---

# Interview Question

## Why not refresh token inside interceptor?

This is a common trap.

You can technically do it, but it creates problems:

* Interceptors are called for every request.
* Multiple requests may refresh simultaneously.
* Risk of infinite loops.
* Harder concurrency management.

Authenticator exists specifically for authentication challenges like 401.

---

# Part 12 — Complete Token Refresh Flow

Let's see production flow.

Initial request:

```
GET /profile

Authorization:
Bearer old_token
```

---

Server:

```
401 Unauthorized
```

---

Authenticator:

```
Refresh Token

↓

POST /refresh

Body:
{
 refreshToken:"xyz"
}

↓

Receive new access token

↓

Retry original request
```

Retry:

```
GET /profile

Authorization:
Bearer new_token
```

---

# Part 13 — The Multiple Request Problem

Real scenario:

Your app makes:

```
GET /profile

GET /orders

GET /messages
```

At the same time.

Token expires.

All receive:

```
401
```

Without control:

```
Request 1
    ↓
Refresh token


Request 2
    ↓
Refresh token


Request 3
    ↓
Refresh token
```

Three refresh calls.

Bad.

---

Solution:

Only one refresh should happen.

Other requests wait.

Concept:

```
Request 1

↓

Refresh Lock

↓

New Token


Request 2

↓

Wait


Request 3

↓

Wait
```

This is a common senior-level interview discussion.

---

# Part 14 — Infinite Loop Problem

Bad implementation:

```
API Request

↓

401

↓

Refresh Token

↓

Retry

↓

401

↓

Refresh Token

↓

Retry

↓

Infinite Loop
```

Solution:

Limit retries.

Example:

```
Only refresh once.
```

If refresh fails:

```
Logout user
Clear tokens
Navigate to Login
```

---

# Part 15 — Complete Authentication Architecture

A clean Android architecture:

```
                 UI

                  |

              ViewModel

                  |

             Repository

                  |

             Retrofit

                  |

             OkHttp

                  |

       Auth Interceptor

                  |

              Server

                  |

        Authenticator

                  |

          Token Manager

                  |

       Secure Storage
```

---

# Interview Questions

## Q1. Difference between Authentication and Authorization?

Answer:

Authentication verifies identity. Authorization determines permissions after identity is verified.

---

## Q2. What is JWT?

Answer:

JWT is a signed token format containing claims that can be used to securely transmit identity information between parties. It consists of header, payload, and signature.

---

## Q3. Why should access tokens expire?

Answer:

To reduce security risk if a token is compromised. Short-lived tokens limit the damage window.

---

## Q4. Access token vs Refresh token?

| Access Token            | Refresh Token                    |
| ----------------------- | -------------------------------- |
| Used for API access     | Used to obtain new access tokens |
| Short lifetime          | Longer lifetime                  |
| Sent frequently         | Stored securely                  |
| Less sensitive duration | More sensitive                   |

---

## Q5. Interceptor vs Authenticator?

Strong answer:

> Interceptors modify requests before they are executed, such as adding authorization headers. Authenticators handle authentication challenges from the server, such as a 401 response, and can obtain new credentials and retry the request.

---

## Q6. Where store JWT tokens?

Answer:

Use secure encrypted storage backed by Android Keystore. Avoid storing sensitive tokens in plain SharedPreferences.

---

# Common Interview Mistakes

❌

"JWT is encrypted."

Correct:

JWT is usually signed, not encrypted.

---

❌

"401 means user doesn't have permission."

Correct:

401 is authentication failure.

403 is authorization failure.

---

❌

"Interceptor refreshes token."

Better:

Authenticator is designed for handling authentication failures and refreshing credentials.

---

# Final Interview Answer

If asked:

> "Explain JWT authentication flow in Android."

A strong answer:

> "During login, the user credentials are sent to the server. If valid, the server returns an access token and refresh token. The access token is attached to subsequent API requests using an OkHttp interceptor. When the access token expires, the server returns 401. OkHttp's Authenticator can handle this challenge by using the refresh token to obtain a new access token and retry the original request. Tokens should be stored securely using encrypted storage backed by Android Keystore. The implementation should also handle concurrent refresh requests and avoid infinite retry loops."

---

# Next Module Preview: Error Handling + Network State

Module 8 will cover a topic that separates production Android apps from demo apps:

* HTTP errors vs network errors
* IOException
* Timeout errors
* Parsing failures
* Creating `NetworkResult`
* Sealed classes for UI states
* Retry strategies
* Handling no internet
* Offline-first architecture basics

This is heavily asked in Android interviews because real apps fail more often due to networking problems than successful responses.
