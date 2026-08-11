# `@Transactional` in Spring

## What is `@Transactional`?

`@Transactional` is a Spring annotation used to **wrap a method or class with a database transaction**.

It ensures that operations inside the method are executed within a **single transaction context**.

If something fails and the transaction is rolled back, the operations are treated as a **single unit of work**.

```java
@Transactional
public void createUser() {
    // Database operations
}
```

---

# Why Transactions?

Transactions help maintain **data integrity** by following the **ACID principles**.

### ACID

| Principle       | Description                                                                                                                                                        |
| --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Atomicity**   | All operations within a transaction are treated as a single unit. If one part fails and the transaction is rolled back, the transaction's changes are rolled back. |
| **Consistency** | A transaction takes the database from one valid state to another while maintaining rules such as constraints and other database integrity rules.                   |
| **Isolation**   | Controls how concurrent transactions interact with each other and what changes are visible between transactions.                                                   |
| **Durability**  | Once a transaction is committed, the changes are persisted even if a system failure occurs afterward.                                                              |

---

# `@Transactional` — Complete Internal Flow

```text
Client
  ↓
Spring Proxy
  ↓
TransactionInterceptor
  ↓
Read @Transactional attributes
  ↓
TransactionManager
  ↓
Begin / Join Transaction
  ↓
Target Method
  ↓
   ┌───────────────┐
   │               │
Success         Exception
   │               │
   ↓               ↓
 Commit          Rollback
```

## Step-by-Step

### 1. Client calls the method

```java
service.createOrder();
```

### 2. Spring Proxy intercepts the call

The method call first goes through the **Spring Proxy**.

### 3. TransactionInterceptor processes the transaction

The `TransactionInterceptor` reads transaction settings such as:

* Propagation
* Isolation
* Timeout
* Read-only
* Rollback rules

### 4. TransactionManager manages the transaction

The `TransactionManager`:

* Starts a new transaction, or
* Joins an existing transaction

### 5. Target method executes

The actual business logic is executed.

### 6. Successful execution

If the method completes successfully:

```text
Target Method
     ↓
  Success
     ↓
   Commit
```

### 7. Exception

If a rollback-triggering exception occurs:

```text
Target Method
     ↓
 Exception
     ↓
  Rollback
```

---

# Where to Use `@Transactional`

`@Transactional` is typically placed on the **Service Layer**.

Example:

```java
@Service
public class UserService {

    private final UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    @Transactional
    public void createUser(User user) {
        userRepository.save(user);

        // Other database operations
    }
}
```

If a rollback-triggering failure occurs during `createUser()`, the transaction can be rolled back as a unit.

---

# Class-Level `@Transactional`

You can place `@Transactional` at the class level:

```java
@Service
@Transactional
public class UserService {

    public void createUser() {
    }

    public void updateUser() {
    }
}
```

This makes the class's methods transactional according to Spring's transaction/proxy rules.

A method-level annotation can override the class-level configuration.

---

# Important Notes

### 1. Service Layer

`@Transactional` is generally best placed at the **Service Layer**, where a business operation may involve multiple repository/database operations.

### 2. Proxy-Based Behavior

In Spring's default proxy-based transaction model, the transactional method must be invoked **through the Spring proxy** for the transaction interceptor to run.

### 3. Self-Invocation

A call from one method to another method within the same object can bypass the proxy:

```java
public void methodA() {
    methodB();
}

@Transactional
public void methodB() {
}
```

Therefore, `@Transactional` may not be applied to `methodB()` in this scenario.

### 4. Public Methods

For the standard proxy-based approach, transaction interception is normally applied to methods invoked through the Spring proxy. In typical Spring applications, `@Transactional` is therefore placed on **public service methods**.

---

# Key Interview Points

```text
@Transactional
      ↓
Spring Proxy
      ↓
TransactionInterceptor
      ↓
TransactionManager
      ↓
Begin / Join Transaction
      ↓
Target Method
      ↓
Commit / Rollback
```

### Interview Answer

> **`@Transactional` is a Spring annotation used to define a transaction boundary around a method or class. Spring uses a proxy and `TransactionInterceptor` to intercept the method call, read the transaction configuration, and use a transaction manager to begin or join a transaction. After the method executes, Spring commits the transaction on success or rolls it back according to the configured rollback rules.**
