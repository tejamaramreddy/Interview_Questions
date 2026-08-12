# Transaction Propagation in Spring

Propagation defines **how transactions behave when a method is called inside an existing transaction**. Imagine one service calling another how should Spring handle the transaction?

Spring provides **7 propagation behaviours** through the `Propagation` enum.

---

## Transaction Propagation Summary

| Propagation     | Existing Transaction                 | No Existing Transaction  | Key Idea                   |
| --------------- | ------------------------------------ | ------------------------ | -------------------------- |
| `REQUIRED`      | Joins existing                       | Creates new              | Join or create             |
| `REQUIRES_NEW`  | Suspends existing and creates new    | Creates new              | Independent transaction    |
| `NESTED`        | Creates nested transaction/savepoint | Creates new*             | Partial rollback           |
| `NEVER`         | Throws exception                     | Runs without transaction | Transaction must not exist |
| `NOT_SUPPORTED` | Suspends existing                    | Runs without transaction | Run without transaction    |
| `SUPPORTS`      | Joins existing                       | Runs without transaction | Optional transaction       |
| `MANDATORY`     | Joins existing                       | Throws exception         | Transaction must exist     |

> **Note:** `NESTED` behavior depends on the transaction manager and database support for savepoints.

---

## Most Common Propagation Types

### Required (Default)

**Joins the existing transaction** if present; otherwise, creates a new one.

**Use Case:** Simple service methods that should always run inside a transaction.

```java
@Service
public class OrderService {

    @Transactional
    public void placeOrder() {
        // Starts a new transaction
        paymentService.processPayment();
    }
}

@Service
public class PaymentService {

    @Transactional(propagation = Propagation.REQUIRED)
    public void processPayment() {
        // Joins the existing transaction from OrderService
    }
}
```

---

### REQUIRES_NEW

Suspends any existing transaction and starts a new independent one.

**Use Case:** You want to **commit a sub-operation** regardless of the outer transaction's success/failure.

```java
@Service
public class OrderService {

    @Transactional
    public void placeOrder() {
        orderRepository.save(order);

        try {
            paymentService.logPayment(); // Runs in a separate transaction
        } catch (Exception e) {
            // handle but don't rollback order
        }
    }
}

@Service
public class PaymentService {

    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void logPayment() {
        // Runs in a new, independent transaction
        // Will commit even if placeOrder() fails
    }
}
```

If `serviceA()` throws an exception, only `serviceA` rolls back. `serviceB()` may still commit because it's using a new transaction.

> Think of **`REQUIRED`** as **adding passengers to an ongoing bus ride**, while **`REQUIRES_NEW`** means **getting off and starting a brand new ride**.

---

### NESTED

Executes in a **nested transaction** if an existing one exists (uses **savepoints**).

**Use Case:** You want to **partially rollback** a sub-task without affecting the main transaction.

```java
@Service
public class OrderService {

    @Transactional
    public void placeOrder() {
        orderRepository.save(order);

        try {
            paymentService.chargeCard();
        } catch (Exception e) {
            // Only rollbacks chargeCard(); order is still saved
        }
    }
}

@Service
public class PaymentService {

    @Transactional(propagation = Propagation.NESTED)
    public void chargeCard() {
        // Fails and rolls back to savepoint
        throw new RuntimeException("Card failed");
    }
}
```

**Note:** Only works if your database supports savepoints (like PostgreSQL, H2, Oracle).

---

### NEVER

**Fails** if there’s already an existing transaction.

**Use Case:** You want to make sure a method is called **outside of a transaction**, e.g., read-only audit logs.

```java
@Service
public class AuditService {

    @Transactional(propagation = Propagation.NEVER)
    public void logReadOperation() {
        // Will throw exception if called from within a transaction
    }
}
```

---

### NOT_SUPPORTED

**Suspends the current transaction** and runs **non-transactionally**.

**Use Case:** You want to perform a task that must **not participate in a transaction**, like logging or caching.

```java
@Service
public class ReportService {

    @Transactional(propagation = Propagation.NOT_SUPPORTED)
    public void generateReport() {
        // Suspends any active transaction
        // Useful to prevent long transaction holding locks
    }
}
```

---

### SUPPORTS

**Uses current transaction** if one exists; otherwise, runs non-transactionally.

**Use Case:** Read-only methods that can optionally participate in a transaction if called from one.

```java
@Service
public class ProductService {

    @Transactional(propagation = Propagation.SUPPORTS, readOnly = true)
    public Product getProductDetails(Long id) {
        // Will join transaction if exists; else run without one
        return productRepository.findById(id).orElse(null);
    }
}
```

---

### MANDATORY

**Requires an existing transaction**; throws exception if none is present.

**Use Case:** Enforces that the method must always be called within a transaction (usually nested service logic).

```java
@Service
public class ShippingService {

    @Transactional(propagation = Propagation.MANDATORY)
    public void updateShippingStatus() {
        // Will throw IllegalTransactionStateException if no transaction
    }
}
```

---

# Rollback Rules in Spring’s `@Transactional`

Spring **only rolls back on unchecked exceptions**, i.e.:

* **RuntimeException** and its subclasses.
* **Error** and its subclasses.

It does **not roll back on checked exceptions** like `IOException`, `SQLException`, etc.

```java
@Transactional
public void process() throws IOException {
    repository.save(entity);
    throw new IOException("This won't trigger rollback!");
}
```

In the example above, Spring won’t rollback because `IOException` is a **checked exception**.

---

## How to Customize Rollback Behavior

You can explicitly control rollback using the `rollbackFor` or `noRollbackFor` attributes.

### `rollbackFor`

```java
@Transactional(rollbackFor = IOException.class)
public void process() throws IOException {
    repository.save(entity);
    throw new IOException("This will now roll back!");
}
```

### `noRollbackFor`

```java
@Transactional(noRollbackFor = IllegalArgumentException.class)
public void process() {
    repository.save(entity);
    throw new IllegalArgumentException("This will NOT roll back!");
}
```

### Best Practices

Use `rollbackFor` **when you're dealing with checked exceptions** (like those thrown by JDBC, file I/O, etc.). This helps ensure that failures trigger full rollbacks.

---

# Summary

| Topic              | Key Point                                                     |
| ------------------ | ------------------------------------------------------------- |
| `REQUIRED`         | Join existing transaction or create a new one                 |
| `REQUIRES_NEW`     | Suspend existing transaction and create a new independent one |
| `NESTED`           | Use savepoints for partial rollback                           |
| `NEVER`            | Fail if a transaction already exists                          |
| `NOT_SUPPORTED`    | Suspend transaction and run without one                       |
| `SUPPORTS`         | Join transaction if one exists                                |
| `MANDATORY`        | Require an existing transaction                               |
| Default rollback   | `RuntimeException` and `Error`                                |
| Checked exceptions | No rollback by default                                        |
| `rollbackFor`      | Explicitly configure exceptions that trigger rollback         |
| `noRollbackFor`    | Prevent rollback for specific exceptions                      |

---

# Read-Only Transactions in Spring Boot

## What Are Read-Only Transactions?

When you annotate a method with `@Transactional(readOnly = true)`, you are indicating that the transaction is intended **only for reading data** — no modifications to the database should be made. This can help optimize the performance of queries, as the database may adjust its internal behaviour (such as disabling certain locking mechanisms) for read-only transactions.

---

## Benefits of Read-Only Transactions

* **Performance Improvement**: For databases that support it (e.g., PostgreSQL, MySQL), enabling read-only mode can speed up query execution by removing certain transaction overheads related to write operations.
* **Optimization**: The database can leverage **optimistic locking** and **MVCC (Multi-Version Concurrency Control)**, as it doesn’t need to track changes.
* **Prevents Accidental Writes**: It acts as a safeguard, ensuring that no modification of data occurs in methods marked as read-only.

Here’s how to use `@Transactional(readOnly = true)`:

```java
@Transactional(readOnly = true)
public List<User> getAllUsers() {
    return userRepository.findAll();  // Only reads data, no writes
}
```

In this example:

* The transaction is marked as **read-only**.
* Spring will **optimize** the transaction, signalling the database that no modifications will happen.
* Any attempt to perform a write operation (e.g., `save()`, `delete()`) inside this method will trigger an exception.

---

## Important Notes

* **JDBC/Database Support**: Not all databases or JDBC drivers may take advantage of the `readOnly` flag. Some databases will ignore this flag, while others (e.g., PostgreSQL) may optimize queries.
* **Only for Select Queries**: Use it only for **select operations** (reading data), as attempting any write operations inside a read-only transaction will result in an exception.

---

## Best Practices

Use `readOnly = true` in methods that only perform **queries** to:

* Enhance performance.
* Ensure no accidental updates or deletions are made in methods meant only for fetching data.
