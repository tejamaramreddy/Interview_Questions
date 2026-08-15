# Spring Security — Authentication Flow

## Introduction

Have you ever had a case with Spring Security where you tried to copy and paste some code, but it didn't work? You spent hours trying to figure out what the issue was, only to realize that there were many different components involved.

Finally, once you solve the issue, you hope you never have to deal with it again because you're not entirely sure what happened.

The goal here is to give you a better understanding of **why Spring Security has so many components, how they work, and most importantly, why each one is needed**.

Once you understand the overall flow, the next time you make a change, you'll have a much better idea of what you're actually changing.

---

# 1. Security Filters

A good place to start is by adding Spring Security to our application.

The first thing you'll notice is that Spring Security will block requests by default and apply a certain configuration structure involving several security components.

Part of that structure is a **login page** with default credentials.

For example, Spring Security can generate a default username such as:

```text
username: user
password: <generated-password>
```

The generated password can be seen in the application console.

Although you're unlikely to use these defaults in production, they provide a good starting point for understanding Spring Security.

Once we enter our credentials, the request is sent through a series of filters.

There are many pre-configured filters, but the ones we're interested in are the **security filters**.

### Security Filter Chain

A **Security Filter Chain** is essentially a collection of security filters.

One of the important filters in a username/password authentication scenario is:

```text
UsernamePasswordAuthenticationFilter
```

Its responsibility is to process credentials submitted through the login form.

We don't necessarily have to use Spring Security's default login page. We can use any form as long as we provide a username and password and send a `POST` request to the appropriate login endpoint.

The filter extracts the credentials from the request and starts the authentication process.

---

# 2. Authentication Provider

There are many ways usernames and passwords can be stored and verified.

For example:

* In-memory storage
* Database
* LDAP
* OAuth/OpenID Connect
* Operating-system authentication
* Custom authentication mechanisms

The question is:

> Should every authentication filter contain logic for all these different authentication mechanisms?

Obviously, that wouldn't be a good design.

Instead, Spring Security uses the concept of an **AuthenticationProvider**.

An `AuthenticationProvider` is responsible for performing a particular type of authentication.

There are multiple authentication providers available, and you can also implement your own.

However, two questions immediately arise:

1. How does Spring Security know which provider to use?
2. How does the authentication filter know which providers are available?

This is where the **AuthenticationManager** comes in.

---

# 3. Authentication Manager

The `AuthenticationManager` sits between the security filters and authentication providers.

```text
Security Filter
      ↓
AuthenticationManager
      ↓
AuthenticationProvider
```

`AuthenticationManager` is an interface.

Its most common implementation is:

```text
ProviderManager
```

The `ProviderManager` maintains a list of authentication providers.

It goes through the providers and checks which one supports the particular type of authentication being requested.

Each provider implements a method called:

```java
supports()
```

Conceptually, the process looks like this:

```text
Authentication Request
        ↓
ProviderManager
        ↓
Provider 1 → supports? No
        ↓
Provider 2 → supports? Yes
        ↓
Provider 2 performs authentication
```

This allows Spring Security to support multiple authentication mechanisms without putting all the logic into a single filter.

---

# 4. DaoAuthenticationProvider

One of the most commonly used authentication providers is:

```text
DaoAuthenticationProvider
```

It is commonly used to authenticate users against a database or another persistent user store.

For example:

```text
Username + Password
        ↓
DaoAuthenticationProvider
        ↓
User Store
        ↓
User Details
        ↓
Password Verification
```

For simple applications, the user information can also be stored in memory.

But what happens if:

* We use one data source locally for testing?
* We use a different database in production?
* Our database has a custom schema?
* Our user table doesn't match Spring Security's default structure?

Should we extend the authentication provider and add our own user-loading logic?

Spring Security provides a better abstraction for this.

That abstraction is called **UserDetailsService**.

---

# 5. UserDetailsService

`UserDetailsService` is an interface used to load user information from a user store.

Its main method is:

```java
UserDetails loadUserByUsername(String username)
```

Its responsibility is to:

> Load a user by username and return the user's details.

For example:

```text
AuthenticationProvider
        ↓
UserDetailsService
        ↓
Database
        ↓
UserDetails
```

The returned `UserDetails` object contains information about the user that the authentication provider needs to perform authentication.

This abstraction is useful because the authentication logic remains separate from the logic responsible for retrieving users.

For example, you can implement `UserDetailsService` to load users from:

* MySQL
* PostgreSQL
* MongoDB
* LDAP
* A custom user store

without having to rewrite the entire authentication mechanism.

---

# 6. Password Encoder

Another important component is the **PasswordEncoder**.

Passwords should generally **not be stored as plain text** in a database.

Instead, passwords are stored using a password hashing mechanism.

For example:

```text
Plain Password
      ↓
PasswordEncoder
      ↓
Hashed Password
      ↓
Database
```

When the user logs in, they provide their plain-text password.

Spring Security then uses the configured `PasswordEncoder` to verify that password against the stored password hash.

Conceptually:

```text
Password entered by user
          ↓
PasswordEncoder
          ↓
Compare with stored password hash
          ↓
Match?
   ↓             ↓
  Yes            No
   ↓             ↓
Success        Failure
```

If the password verification succeeds, authentication is successful.

> Important: modern password encoders such as BCrypt are designed for password hashing and verification. You don't decrypt the stored password.

---

# 7. Authentication Object

Once the authentication provider successfully authenticates the user, it returns an **Authentication** object.

The `Authentication` object represents the authenticated user and contains information such as:

* Principal
* Authorities/roles
* Authentication status

The password is not retained in the authenticated object because there is no reason to keep sensitive credential information around after authentication.

The flow now looks like:

```text
Security Filter
      ↓
AuthenticationManager
      ↓
AuthenticationProvider
      ↓
UserDetailsService
      ↓
UserDetails
      ↓
PasswordEncoder
      ↓
Authentication
```

The `Authentication` object is then returned back through the `AuthenticationManager` to the security filter.

---

# 8. SecurityContext

Now that the user has successfully authenticated, Spring Security needs somewhere to store the authenticated user's information so that other parts of the application can access it.

This is where the **SecurityContext** comes in.

The security filter places the authenticated user into the `SecurityContext`.

Conceptually:

```text
Authentication
      ↓
SecurityContext
      ↓
Current authenticated user
```

The currently authenticated user is often referred to as the **principal**.

Spring Security provides `SecurityContextHolder` as a way to access the current `SecurityContext`.

For example:

```java
SecurityContext context =
        SecurityContextHolder.getContext();

Authentication authentication =
        context.getAuthentication();
```

You can then access information about the authenticated user.

---

# 9. Subsequent Requests

What happens when the user makes another request?

There are multiple possibilities depending on the authentication mechanism.

## Session-Based Authentication

If the application uses a session, after successful authentication Spring Security associates the authenticated user with the session.

For subsequent requests, Spring Security can retrieve the authentication information from the session.

Therefore, the user doesn't have to authenticate again on every request.

Conceptually:

```text
First Request
     ↓
Authentication
     ↓
SecurityContext
     ↓
Session
```

Then:

```text
Second Request
     ↓
Session
     ↓
Existing Authentication
     ↓
SecurityContext
```

---

# 10. Basic Authentication

With Basic Authentication, credentials are typically sent with each request.

Therefore, authentication processing can occur for every request.

Conceptually:

```text
Request
   ↓
Basic Authentication Filter
   ↓
AuthenticationManager
   ↓
AuthenticationProvider
   ↓
Authentication
```

---

# 11. JWT Authentication

Another common approach is **JWT-based authentication**.

JWT authentication is commonly used for stateless applications.

The client sends a JWT with the request:

```text
Request
  +
Authorization: Bearer <JWT>
        ↓
JWT Authentication
        ↓
Validate JWT
        ↓
Create Authentication
        ↓
SecurityContext
```

Because the token contains the required claims and is cryptographically signed, the server can validate the token without maintaining a traditional server-side session.

The server primarily needs to verify the token's validity and signature and then establish the authenticated principal.

---

# 12. Authentication Errors

What happens if authentication fails?

For example:

* Username doesn't exist
* Password is incorrect
* Credentials are invalid
* Authentication cannot be performed

Spring Security has mechanisms for handling authentication and authorization exceptions.

One important component involved in exception handling is the:

```text
ExceptionTranslationFilter
```

It helps translate security exceptions into appropriate responses based on the application's configuration.

For example, depending on the situation, the client may receive:

```text
401 Unauthorized
```

or:

```text
403 Forbidden
```

These two responses have different meanings.

### 401 Unauthorized

Generally means:

> The request does not have valid authentication credentials.

### 403 Forbidden

Generally means:

> The user is authenticated but does not have permission to access the resource.

---

# 13. Why Are There So Many Security Filters?

A common question is:

> Why does Spring Security need so many filters?

Different filters are responsible for different security concerns.

For example:

```text
CSRF Filter
Logout Filter
Authentication Filters
Exception Translation Filter
Authorization-related filters
```

Some filters protect the application from specific attacks or perform specific security-related operations.

Other filters are responsible for authentication.

There can also be multiple authentication filters because credentials can arrive in different ways.

For example:

### Form Login

Credentials can be sent as form fields:

```text
username=user
password=password
```

### Basic Authentication

Credentials are typically sent using the `Authorization` header.

Therefore, different authentication mechanisms can require different filters.

---

# 14. Authentication Token

Although different authentication mechanisms may use different filters, they need a common way to communicate with the `AuthenticationManager`.

This is where an **Authentication token** is used.

For username/password authentication, Spring Security creates a:

```text
UsernamePasswordAuthenticationToken
```

The flow becomes:

```text
HTTP Request
     ↓
Authentication Filter
     ↓
Extract username/password
     ↓
UsernamePasswordAuthenticationToken
     ↓
AuthenticationManager
     ↓
AuthenticationProvider
```

The authentication token acts as a representation of the authentication request.

---

# 15. Overall Spring Security Authentication Flow

Putting everything together:

```text
                HTTP Request
                     │
                     ▼
            Security Filter Chain
                     │
                     ▼
       UsernamePasswordAuthenticationFilter
                     │
                     ▼
    UsernamePasswordAuthenticationToken
                     │
                     ▼
           AuthenticationManager
                     │
                     ▼
              ProviderManager
                     │
                     ▼
        AuthenticationProvider
                     │
                     ▼
          UserDetailsService
                     │
                     ▼
                 User Store
                     │
                     ▼
               UserDetails
                     │
                     ▼
              PasswordEncoder
                     │
                     ▼
            Authentication
                     │
                     ▼
              SecurityContext
                     │
                     ▼
          SecurityContextHolder
                     │
                     ▼
              Authenticated
                  Request
```

---

# 16. Why Spring Security Has So Many Components

Spring Security has many components because it follows a highly modular design.

Each component has a specific responsibility:

| Component                      | Responsibility                                       |
| ------------------------------ | ---------------------------------------------------- |
| **Security Filter Chain**      | Processes incoming requests through security filters |
| **Authentication Filter**      | Extracts authentication credentials from the request |
| **Authentication Token**       | Represents the authentication request                |
| **AuthenticationManager**      | Coordinates authentication                           |
| **ProviderManager**            | Selects an appropriate authentication provider       |
| **AuthenticationProvider**     | Performs the actual authentication                   |
| **UserDetailsService**         | Loads user information                               |
| **UserDetails**                | Represents user information                          |
| **PasswordEncoder**            | Verifies passwords securely                          |
| **Authentication**             | Represents the authenticated user                    |
| **SecurityContext**            | Stores the current authentication                    |
| **SecurityContextHolder**      | Provides access to the security context              |
| **ExceptionTranslationFilter** | Handles security exceptions                          |

The advantage of this architecture is that you usually don't need to replace the entire security system.

Instead, you can replace or customize only the component you need.

For example:

```text
Need custom users?
        ↓
Customize UserDetailsService

Need different password hashing?
        ↓
Configure PasswordEncoder

Need a different authentication mechanism?
        ↓
Configure AuthenticationProvider

Need JWT authentication?
        ↓
Configure JWT-based authentication
```

---

# 17. Debugging Spring Security

One useful trick for understanding what Spring Security is doing is to enable detailed Spring Security logging.

For example, you can enable debug/trace logging for the Spring Security packages in your application configuration.

Then, when you make an authenticated request, the logs can show which security filters are being invoked.

You might see a sequence similar to:

```text
CSRF Filter
     ↓
Logout Filter
     ↓
UsernamePasswordAuthenticationFilter
     ↓
ProviderManager
     ↓
AuthenticationProvider
     ↓
Authentication successful
     ↓
SecurityContext
```

This is extremely useful when debugging Spring Security because instead of guessing which component is causing a problem, you can observe the actual filter chain and authentication flow.

---

# Key Takeaway

The most important idea is to understand the **responsibility of each component** rather than memorizing the entire Spring Security configuration.

The core authentication flow is:

```text
Request
  ↓
Security Filter
  ↓
Authentication Token
  ↓
AuthenticationManager
  ↓
AuthenticationProvider
  ↓
UserDetailsService
  ↓
UserDetails
  ↓
PasswordEncoder
  ↓
Authentication
  ↓
SecurityContext
  ↓
Authenticated Request
```

Once you understand this flow, Spring Security becomes much easier to reason about because each component has a clear responsibility.
