## Isolation Levels in Spring Boot

### What Are Isolation Levels?

Isolation levels define how transactions are **isolated from each other**. When multiple transactions are running concurrently, the isolation level controls the degree to which one transaction’s operations are visible to others.


Each isolation level strikes a balance between **data consistency** and **performance**, affecting the following:

1. **Dirty Reads**: Reading uncommitted data from another transaction.
2. **Non-repeatable Reads**: A value read by a transaction changes before the transaction finishes.
3. **Phantom Reads**: New rows appear in a result set due to other transactions’ inserts.

### The Four Isolation Levels in Spring

| Isolation Level    | Dirty Reads | Non-repeatable Reads | Phantom Reads |
| ------------------ | ----------- | -------------------- | ------------- |
| `READ_UNCOMMITTED` | Allowed     | Allowed              | Allowed       |
| `READ_COMMITTED`   | Prevented   | Allowed              | Allowed       |
| `REPEATABLE_READ`  | Prevented   | Prevented            | Allowed       |
| `SERIALIZABLE`     | Prevented   | Prevented            | Prevented     |

### **How Isolation Levels Work in Spring**

Spring allows you to configure the **isolation level** for a transaction using `@Transactional(isolation = Isolation.X)`:

```java
@Transactional(isolation = Isolation.READ_COMMITTED)
public void processTransaction() {
    // Some database operation
}
```

In this example:

* The transaction will use **`READ_COMMITTED`** isolation level, which ensures that the transaction doesn’t read any uncommitted data from other transactions.

### How Each Level Affects Concurrency

**`READ_UNCOMMITTED`**

* **Dirty Reads**: Allows reading uncommitted changes from other transactions.
* **Example Issue**: A transaction reads data that another transaction hasn’t committed yet, leading to potential inconsistencies.

**`READ_COMMITTED`**

* **No Dirty Reads**: Guarantees that you will only see committed data.
* **Non-repeatable Reads**: If a transaction reads a value, another transaction can change it before the first one finishes, making the value inconsistent.

**`REPEATABLE_READ`**

* **No Dirty Reads, No Non-repeatable Reads**: Once a transaction reads a value, it will continue to see the same value (even if another transaction modifies it).
* **Phantom Reads**: New rows inserted by other transactions can still appear in queries (e.g., SELECT * WHERE id > 10).

**`SERIALIZABLE`**

* **No Dirty Reads, No Non-repeatable Reads, No Phantom Reads**: Transactions are serialized, one after another. The most isolated but slowest due to locking.
* **Example**: Two transactions modifying the same data will have to wait for the other to complete.

### **Example of Using Isolation Levels in Spring**

```java
@Transactional(isolation = Isolation.REPEATABLE_READ)
public void readData() {
    // Perform read operations
    // No non-repeatable reads, but phantom reads are possible
}
```

Here:

* The **repeatable read** level ensures that data read by this transaction cannot change during the transaction.
* This ensures more consistency but can reduce concurrency and performance.
