# JWT: Token Generation, Validation, and Refresh Tokens

These three concepts are the core of how JWT-based authentication works in a real Spring Boot application.

A good way to understand them is:

---

# 1. Token Generation

## What is Token Generation?

**Token generation is the process of creating a JWT after the user successfully authenticates.**

For example, the user sends:

```http
POST /login
Content-Type: application/json

{
    "username": "ravi",
    "password": "password123"
}
```

Spring Security first verifies the credentials.

```text
Username + Password
        ↓
Authentication
        ↓
User found?
        ↓
Password correct?
        ↓
YES
        ↓
Generate JWT
```

The server then creates a JWT and sends it back to the client.

---

## What does the JWT contain?

A JWT normally contains claims such as:

```json
{
    "sub": "ravi",
    "roles": ["USER"],
    "iat": 1723810000,
    "exp": 1723813600
}
```

For example:

* `sub` → subject/user
* `roles` → user's roles
* `iat` → issued-at time
* `exp` → expiration time

Remember that the payload is **not encrypted by normal JWT signing**. It is encoded and signed.

Therefore, don't put passwords or other secrets inside the JWT.

---

# JWT Generation Process

Suppose Ravi successfully logs in.

```text
                  Login Request
                       │
                       ▼
              Username + Password
                       │
                       ▼
              AuthenticationManager
                       │
                       ▼
             AuthenticationProvider
                       │
                       ▼
              UserDetailsService
                       │
                       ▼
                User Database
                       │
                       ▼
              Password Verification
                       │
                       ▼
                  SUCCESS
                       │
                       ▼
               JWT Generation
                       │
                       ▼
                  JWT Token
```

The server might return:

```json
{
    "accessToken": "eyJhbGciOiJIUzI1NiIs...",
    "refreshToken": "eyJhbGciOiJIUzI1NiIs..."
}
```

---

# How is a JWT generated?

Conceptually:

```text
Header
   +
Payload
   +
Secret Key / Private Key
   ↓
Signature
   ↓
JWT
```

The JWT has three parts:

```text
HEADER.PAYLOAD.SIGNATURE
```

For example:

```text
eyJhbGciOiJIUzI1NiJ9
.
eyJzdWIiOiJyYXZpIiwi...
.
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

The signature is important because it allows the server to determine whether the token has been modified.

---

# 2. Token Validation

After receiving the JWT, the client normally sends it with subsequent API requests.

For example:

```http
GET /api/accounts

Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

The server needs to determine:

> Is this token valid?

This is called **token validation**.

---

# What does JWT validation check?

A server can validate several things.

### 1. Signature

The server verifies that the token was signed using the expected key.

```text
JWT
 ↓
Extract Header + Payload
 ↓
Verify Signature
 ↓
Valid?
```

If someone changes:

```json
{
    "role": "USER"
}
```

to:

```json
{
    "role": "ADMIN"
}
```

the signature will no longer match.

The token is rejected.

---

### 2. Expiration

The server checks the `exp` claim.

For example:

```json
{
    "sub": "ravi",
    "exp": 1723813600
}
```

If the current time is after `exp`:

```text
Token expired
    ↓
Reject request
    ↓
401 Unauthorized
```

---

### 3. Issuer

The server can check the `iss` claim.

For example:

```json
{
    "iss": "my-auth-server"
}
```

The server can verify that the expected authentication server issued the token.

---

### 4. Audience

The server can check the `aud` claim.

For example:

```json
{
    "aud": "account-service"
}
```

This helps ensure that the token is intended for the particular service/API.

---

### 5. Claims

The application can inspect claims such as:

```json
{
    "sub": "ravi",
    "roles": ["USER"]
}
```

Then authorization can determine whether Ravi can perform the requested operation.

---

# JWT Validation Flow

```text
Client
  │
  │ Authorization: Bearer <JWT>
  ▼
API
  │
  ▼
Extract JWT
  │
  ▼
Verify Signature
  │
  ▼
Check Expiration
  │
  ▼
Check Issuer/Audience
  │
  ▼
Read Claims
  │
  ▼
Create Authentication
  │
  ▼
SecurityContext
  │
  ▼
Authorization
  │
  ├── Allowed → Controller
  │
  └── Denied → 403
```

If the token itself is invalid or missing where authentication is required, the request will commonly result in **401 Unauthorized**.

---

# Token Validation vs Authentication

This is an important distinction.

### Authentication

Determines:

> Who is this user?

### Token Validation

Determines:

> Can I trust this JWT as a valid representation of this authenticated identity?

For example:

```text
JWT
 ↓
Signature valid?
 ↓
Not expired?
 ↓
Correct issuer?
 ↓
Correct audience?
 ↓
Valid claims?
 ↓
Authenticated identity
```

---

# 3. Refresh Tokens

The biggest problem with access tokens is **expiration**.

You generally don't want an access token to remain valid forever.

For example:

```text
Access Token
Expiration: 15 minutes
```

After 15 minutes:

```text
Access Token
      ↓
Expired
      ↓
API rejects request
```

The user could log in again, but that would be a bad user experience.

This is where **refresh tokens** come in.

---

# What is a Refresh Token?

A refresh token is a credential used to obtain a **new access token** after the current access token expires.

The basic idea is:

```text
Access Token
    ↓
Short lifetime
    ↓
Expires
    ↓
Refresh Token
    ↓
New Access Token
```

The user doesn't have to enter their username and password again.

---

# Access Token vs Refresh Token

| Access Token                              | Refresh Token                                             |
| ----------------------------------------- | --------------------------------------------------------- |
| Used to access APIs                       | Used to obtain new access tokens                          |
| Short-lived                               | Usually longer-lived                                      |
| Sent with API requests                    | Sent to token/refresh endpoint                            |
| More frequently exposed                   | Should be protected carefully                             |
| Contains authorization information/claims | Primarily represents ability to obtain a new access token |
| Example: 15 minutes                       | Example: days/weeks, depending on design                  |

The exact lifetimes depend on the application's security requirements.

---

# Complete Refresh Token Flow

Suppose Ravi logs in.

### Step 1 — Login

```http
POST /auth/login
```

```text
Username + Password
        ↓
Authentication
        ↓
Success
        ↓
Generate:
    Access Token
    Refresh Token
```

Server returns:

```json
{
    "accessToken": "ACCESS_TOKEN",
    "refreshToken": "REFRESH_TOKEN"
}
```

---

### Step 2 — Access API

Client sends:

```http
GET /api/accounts

Authorization: Bearer ACCESS_TOKEN
```

Server validates the access token.

```text
Valid
 ↓
Allow request
```

---

### Step 3 — Access Token Expires

Eventually:

```text
Access Token
      ↓
Expired
```

The API returns:

```text
401 Unauthorized
```

---

### Step 4 — Client Uses Refresh Token

The client sends the refresh token to a refresh endpoint:

```http
POST /auth/refresh
```

For example:

```json
{
    "refreshToken": "REFRESH_TOKEN"
}
```

---

### Step 5 — Server Validates Refresh Token

The server verifies that the refresh token is:

* Valid
* Not expired
* Associated with the appropriate client/user
* Not revoked, depending on the application's design

If valid:

```text
Refresh Token
      ↓
Valid
      ↓
Generate New Access Token
```

---

### Step 6 — New Access Token

The server returns:

```json
{
    "accessToken": "NEW_ACCESS_TOKEN"
}
```

The client can now continue making API requests.

```text
NEW_ACCESS_TOKEN
       ↓
API requests
       ↓
Validation
       ↓
Access
```

The user didn't have to log in again.

---

# Complete JWT Authentication Architecture

Putting everything together:

```text
                     LOGIN
                       │
                       ▼
              Username + Password
                       │
                       ▼
                Authentication
                       │
                       ▼
                     SUCCESS
                       │
              ┌────────┴────────┐
              ▼                 ▼
        Access Token       Refresh Token
        short-lived         long-lived
              │                 │
              │                 │
              ▼                 │
        API Requests            │
              │                 │
              ▼                 │
       Validate JWT             │
              │                 │
       ┌──────┴──────┐          │
       │             │          │
     Valid         Expired      │
       │             │          │
       ▼             ▼          │
    Access         401          │
                     │          │
                     ▼          │
              Send Refresh Token
                     │          │
                     └──────────┘
                          │
                          ▼
                 Validate Refresh Token
                          │
                          ▼
                 Generate New Access Token
                          │
                          ▼
                    API Requests
```

# Spring Boot Perspective

In a Spring Boot application, you typically have components that handle these responsibilities:

```text
Security Filter Chain
        ↓
JWT Authentication Filter
        ↓
Extract Bearer Token
        ↓
JWT Service
        ↓
Validate JWT
        ↓
Create Authentication
        ↓
SecurityContext
        ↓
Authorization
        ↓
Controller
```

For login:

```text
Controller
    ↓
AuthenticationManager
    ↓
AuthenticationProvider
    ↓
UserDetailsService
    ↓
PasswordEncoder
    ↓
Successful Authentication
    ↓
JWT Service
    ↓
Generate Access Token
    ↓
Return Token
```

For refresh:

```text
Refresh Endpoint
      ↓
Receive Refresh Token
      ↓
Validate Refresh Token
      ↓
Find/verify user/session
      ↓
Generate New Access Token
      ↓
Return New Access Token
```

---

# Important Interview Question: Why Make Access Tokens Short-Lived?

Because if an access token is stolen, an attacker can potentially use it until it expires.

For example:

```text
Access Token Lifetime = 15 minutes
```

If stolen, its normal validity window is limited.

Compare that with:

```text
Access Token Lifetime = 30 days
```

A stolen token could potentially be used for a much longer period.

Refresh tokens allow you to keep access tokens short-lived without forcing the user to log in repeatedly.

---

# Access Token + Refresh Token Analogy

Think of a hotel.

### Access Token

Your room key:

```text
Room Key
   ↓
Access your room
```

It is short-lived.

### Refresh Token

A longer-lived credential that lets the system issue a new room key under the appropriate conditions.

So:

```text
Refresh Token
      ↓
New Access Token
      ↓
Access APIs
```

---

# One Important Security Point

A refresh token is powerful because whoever possesses it may be able to obtain new access tokens.

Therefore, refresh tokens need stronger protection than ordinary API data.

Depending on the application architecture, refresh tokens may be:

* Stored securely on the client
* Stored in an `HttpOnly`, `Secure` cookie for browser applications
* Stored server-side or tracked server-side
* Rotated when used
* Revoked when necessary

The exact strategy depends on your application and threat model.

---

# The Three Concepts to Remember

### 1. Token Generation

> After successful authentication, the server creates a signed JWT containing the necessary claims and gives it to the client.

```text
Login
 ↓
Authenticate
 ↓
Generate JWT
 ↓
Client
```

### 2. Token Validation

> For each protected request, the server validates the JWT's signature and relevant claims such as expiration, issuer, and audience before establishing the authenticated identity.

```text
Request
 ↓
JWT
 ↓
Validate
 ↓
Authentication
 ↓
Authorization
```

### 3. Refresh Tokens

> A refresh token allows the client to obtain a new access token after the access token expires, without requiring the user to authenticate again.

```text
Access Token
 ↓
Expires
 ↓
Refresh Token
 ↓
New Access Token
```

## Interview-ready answer

> **In JWT-based authentication, token generation happens after successful user authentication. The server creates a signed access token containing claims such as the subject, roles, issued-at time, and expiration time. For subsequent requests, the client sends the access token, usually as a Bearer token. The server validates its signature and relevant claims such as expiration, issuer, and audience, and then establishes the authenticated user in the security context. Because access tokens should generally be short-lived, a refresh token can be used to obtain a new access token when the original expires, allowing the user to remain logged in without submitting their credentials again.**
