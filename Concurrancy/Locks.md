# Java Locks
## 1. Why Synchronization Is Needed

When multiple threads execute code concurrently, **shared mutable variables** can cause incorrect results if they are accessed without synchronization.

### Example: Incrementing a Shared Integer

Incrementing a number looks like one operation, but it actually involves **three steps**:

1. **Read** the current value.
2. **Increment** the value.
3. **Write** the new value back.

If two threads perform these steps simultaneously, their operations can overlap.

### Race Condition

A **race condition** occurs when multiple threads access and modify shared data concurrently, and the final result depends on the timing/order of thread execution.

**Example:**

Suppose the value is `0`:

```text
Thread 1: Read 0
Thread 2: Read 0
Thread 1: Increment → 1
Thread 2: Increment → 1
Thread 1: Write 1
Thread 2: Write 1
```

Expected result: `2`
Actual result: `1`

Therefore, the result can vary between executions.

---

# 2. `synchronized`

Java provides thread synchronization through the **`synchronized` keyword**.

It provides **implicit locking**, meaning Java manages the lock for you.

When a synchronized method is called:

* Threads attempting to access the same synchronized resource must coordinate.
* Only one thread can execute the protected section at a time.
* Other threads wait until the lock becomes available.

### Key idea

> `synchronized` protects shared mutable data from unsafe concurrent access.

However, Java also provides **explicit locking** through the `Lock` interface, which gives programmers more control.

---

# 3. Monitors

A **monitor** is a concurrency concept describing an object that can be safely accessed by multiple threads.

In Java, every object ultimately inherits from:

```java
java.lang.Object
```

Java objects have built-in mechanisms that allow them to act as **monitors**.

A monitor is also commonly called:

* **Monitor lock**
* **Intrinsic lock**

### Monitor and `synchronized`

When you use a synchronized method, threads accessing that method use the object's associated monitor.

### Monitor characteristics

Java monitors support concepts such as:

* Waiting threads
* Notification
* Lock ownership
* Reentrancy

---

# 4. Reentrant Locks

Java provides the `ReentrantLock` class as an explicit implementation of a lock.

It provides behavior similar to `synchronized`, but with additional capabilities and control.

### Basic usage

```java
lock.lock();

try {
    // Critical section
} finally {
    lock.unlock();
}
```

### Important rule: Always use `finally`

The `finally` block ensures that the lock is released even if an exception occurs.

```text
lock()
  ↓
critical section
  ↓
unlock() ← finally
```

### How `ReentrantLock` behaves

If one thread already owns the lock:

```text
Thread 1 → acquires lock
Thread 2 → tries to acquire lock
             ↓
          waits
```

Only after Thread 1 releases the lock can Thread 2 acquire it.

### Why "reentrant"?

A thread that already owns a reentrant lock can acquire the **same lock again** without deadlocking itself.

---

# 5. `tryLock()`

`tryLock()` attempts to acquire a lock **without blocking the current thread**.

Instead of waiting indefinitely, it returns a boolean:

```java
if (lock.tryLock()) {
    try {
        // Access shared data
    } finally {
        lock.unlock();
    }
}
```

### Return value

| Result  | Meaning                        |
| ------- | ------------------------------ |
| `true`  | Lock was successfully acquired |
| `false` | Lock was unavailable           |

This allows a program to decide what to do when the lock cannot immediately be obtained.

---

# 6. Read-Write Locks

Java's `ReadWriteLock` provides **two separate locks**:

1. **Read lock**
2. **Write lock**

The main idea is:

> Multiple threads can safely read shared data simultaneously, provided nobody is writing to it.

### Read operations

Multiple readers can operate at the same time:

```text
Reader 1 ─┐
Reader 2 ─┼──→ Read concurrently
Reader 3 ─┘
```

### Write operations

A writer requires exclusive access:

```text
Writer → exclusive access
Readers → must wait
```

### Why use ReadWriteLock?

It can improve **performance and throughput** when:

* Reads happen frequently.
* Writes happen relatively rarely.

### Example scenario

Suppose a list contains some data.

A writer:

```text
Acquire write lock
      ↓
Modify list
      ↓
Release write lock
```

While the writer holds the lock, readers must wait.

After the writer releases it:

```text
Reader 1 ─┐
Reader 2 ─┼──→ Read simultaneously
Reader 3 ─┘
```

The readers do **not** need to wait for one another.

---

# 7. Stamped Locks

`StampedLock` also supports read and write access, but it works differently from `ReadWriteLock`.

The major difference is that locking methods return a **stamp**.

A stamp is represented by a:

```java
long
```

The stamp can later be used to:

* Release the lock.
* Check whether the lock is still valid.
* Perform lock conversions.

### Obtaining locks

```java
long stamp = lock.readLock();
```

or:

```java
long stamp = lock.writeLock();
```

The stamp is then used when releasing the lock.

---

# 8. StampedLock Is Not Reentrant

An important difference:

> `StampedLock` does **not** provide reentrant locking.

With a reentrant lock, the same thread can acquire the same lock again.

With a `StampedLock`, another attempt to acquire the lock can block even if the same thread already owns a lock.

Therefore:

⚠️ **Be careful with `StampedLock` to avoid deadlocks.**

---

# 9. Optimistic Locking

One of the most useful features of `StampedLock` is **optimistic reading**.

An optimistic read is obtained using:

```java
tryOptimisticRead()
```

### Important characteristic

It **does not block the current thread**.

It immediately returns a stamp.

If a write lock is already active, the returned stamp is:

```text
0
```

### Checking validity

After reading shared data, you should check whether the optimistic read was still valid:

```java
if (lock.validate(stamp)) {
    // Read was valid
}
```

### Why is it called "optimistic"?

The thread assumes that no conflicting write will occur.

Unlike a normal read lock, an optimistic read **does not prevent a writer from acquiring the write lock**.

Example:

```text
Thread 1 → optimistic read
             ↓
Thread 2 → obtains write lock
             ↓
Thread 1's optimistic read becomes invalid
```

Even after Thread 2 releases the write lock, the old optimistic read remains invalid.

### Critical rule

> Always validate an optimistic read after accessing shared mutable data.

---

# 10. Converting a Read Lock to a Write Lock

Sometimes a thread starts with a read lock but later discovers that it needs to modify the data.

`StampedLock` provides:

```java
tryConvertToWriteLock()
```

This attempts to convert a read lock into a write lock.

### Important behavior

The method:

* Does **not block**.
* Returns a new stamp if conversion succeeds.
* Returns `0` if a write lock is not currently available.

If conversion fails, the program can obtain a write lock normally using:

```java
writeLock()
```

which may block until the lock becomes available.

### Why convert?

It avoids unnecessarily releasing the read lock and then acquiring a write lock, which can help preserve proper concurrent access behavior.

---

# 11. Locks — Quick Comparison

| Feature                 | `synchronized` | `ReentrantLock` | `ReadWriteLock`           | `StampedLock` |
| ----------------------- | -------------- | --------------- | ------------------------- | ------------- |
| Lock type               | Implicit       | Explicit        | Explicit                  | Explicit      |
| Exclusive access        | ✅              | ✅               | Write lock                | Write lock    |
| Multiple readers        | ❌              | ❌               | ✅                         | ✅             |
| Reentrant               | ✅              | ✅               | Depends on implementation | ❌             |
| `tryLock`-style control | Limited        | ✅               | ✅                         | ✅             |
| Optimistic reading      | ❌              | ❌               | ❌                         | ✅             |
| Lock conversion         | ❌              | ❌               | Limited                   | ✅             |

---

# 12. Semaphores

A **Semaphore** is different from a traditional lock.

### Lock

A lock generally provides **exclusive access**:

```text
1 permit → 1 thread at a time
```

### Semaphore

A semaphore can maintain **multiple permits**:

```text
5 permits → up to 5 threads at a time
```

This makes semaphores useful when you want to **limit the number of threads that can access a resource simultaneously**.

---

# 13. Semaphore Example

Suppose an executor can potentially run **10 tasks concurrently**.

You want only **5 tasks** to execute a particular long-running section at once.

Create a semaphore with 5 permits:

```text
Semaphore = 5 permits
Executor  = 10 possible concurrent tasks
```

The result:

```text
Task 1 ─┐
Task 2  │
Task 3  │
Task 4  ├──→ Maximum 5 concurrent tasks
Task 5 ─┘

Task 6+
   ↓
Wait for a permit
```

### Acquiring a permit

A thread attempts to acquire a permit before entering the protected section.

If all permits are already being used, subsequent threads must wait or fail, depending on the acquisition method.

### Releasing a permit

The permit must be released after the protected work is finished.

As with locks, use a `finally` block:

```java
try {
    // Long-running task
} finally {
    semaphore.release();
}
```

This guarantees that the permit is returned even if an exception occurs.

---

# 14. `tryAcquire()`

A semaphore can use `tryAcquire()` to attempt to obtain a permit without waiting indefinitely.

A timeout can also be specified.

In the example from the lecture:

```text
Maximum permits = 5
Maximum wait time = 1 second
```

If all five permits are occupied and another thread cannot obtain one within the timeout, the acquisition fails.

---

# 15. Key Concepts to Remember

### Race Condition

Multiple threads access shared mutable data in an unsafe way, producing unpredictable results.

### Synchronization

A mechanism that coordinates access to shared resources.

### Monitor

An object's built-in mechanism for coordinating synchronized access.

### Reentrant Lock

A lock that the same thread can acquire multiple times safely.

### Read Lock

Allows multiple threads to read concurrently when no writer holds the lock.

### Write Lock

Provides exclusive access for modifying shared data.

### Stamped Lock

A lock that returns a `long` stamp and supports normal, optimistic, and conversion-based locking.

### Optimistic Lock

Allows a thread to read without blocking writers, but the read must later be validated.

### Semaphore

Controls **how many threads** can access a resource simultaneously by maintaining multiple permits.

---

# 16. Big Picture

The main progression in the lecture is:

```text
Shared Mutable Data
        ↓
   Race Conditions
        ↓
 Synchronization
        ↓
 ┌───────────────────────┐
 │       Java Locks      │
 └───────────────────────┘
        ↓
 ┌─────────────┬──────────────┬───────────────┐
 │ synchronized│ ReentrantLock│ ReadWriteLock │
 └─────────────┴──────────────┴───────────────┘
                                      ↓
                                StampedLock
                                      ↓
                           Optimistic Locking
                                      ↓
                                  Semaphore
                                      ↓
                     Limit concurrent access
```

### Core takeaway

**Locks** are primarily about controlling access to shared resources, while **semaphores** are about controlling the **number of threads** that can access a resource at the same time.

For exams/interviews, the most important distinctions are:

* **`synchronized`** → simple implicit locking.
* **`ReentrantLock`** → explicit, more controllable locking.
* **`ReadWriteLock`** → many readers, one writer.
* **`StampedLock`** → read/write locks + optimistic reads + lock conversion, but **not reentrant**.
* **`Semaphore`** → multiple permits; limits concurrent access rather than enforcing single-thread ownership.


Here are **10 interview questions** covering the most important concepts from the lecture, arranged from basic to more advanced.

## 10 Interview Questions

### 1. What is a race condition?

Explain what happens when multiple threads access and modify the same shared mutable variable concurrently. Why can a simple `count++` operation cause incorrect results?

**Key points:** shared mutable state, read-modify-write, non-atomic operation.

---

### 2. What is a monitor in Java?

What is an intrinsic/monitor lock, and how does the `synchronized` keyword use an object's monitor?

**Key points:** object monitor, intrinsic lock, `synchronized`, mutual exclusion.

---

### 3. What does "reentrant" mean in `ReentrantLock`?

Why can a thread acquire the same `ReentrantLock` multiple times without deadlocking itself?

**Key points:** lock ownership, same thread, repeated acquisition, reentrancy.

---

### 4. What is the difference between `synchronized` and `ReentrantLock`?

When would you choose `ReentrantLock` instead of `synchronized`?

**Key points:** implicit vs explicit locking, `tryLock()`, timeout, more control, `finally`/`unlock()`.

---

### 5. Why should `unlock()` be placed inside a `finally` block?

Consider:

```java
lock.lock();

try {
    // critical section
} finally {
    lock.unlock();
}
```

What problem does the `finally` block prevent?

**Key points:** exceptions, guaranteed lock release, avoiding permanently locked resources.

---

### 6. What is the difference between `ReentrantLock` and `ReadWriteLock`?

Why can a `ReadWriteLock` provide better performance in a read-heavy application?

**Key points:** exclusive lock vs multiple readers, concurrent reads, exclusive writes, throughput.

---

### 7. What is a `StampedLock`, and how is it different from `ReadWriteLock`?

What is the purpose of the `long` stamp returned by `StampedLock` methods?

**Key points:** read/write locks, stamps, validation, conversion, optimistic locking, non-reentrant behavior.

---

### 8. What is optimistic locking in `StampedLock`?

Explain how `tryOptimisticRead()` works and why you must call `validate()` after reading shared data.

**Key points:** non-blocking read, writer can proceed, stamp, validation, invalid read.

---

### 9. Can you convert a read lock into a write lock with `StampedLock`?

Explain how `tryConvertToWriteLock()` works. What does a return value of `0` mean?

**Key points:** lock conversion, non-blocking attempt, `0` means conversion failed, fallback to `writeLock()`.

---

### 10. What is the difference between a Lock and a Semaphore?

Suppose you have **20 threads**, but only **5 threads should be allowed to access a resource simultaneously**. Which concurrency mechanism would you use and why?

**Key points:** lock → generally exclusive access; semaphore → multiple permits; `Semaphore(5)` → maximum 5 concurrent accesses.

---
