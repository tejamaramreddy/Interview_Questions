# Java Collections - HashMap

---

# 1. Core Concept: Hashing

A **HashMap** is a hash table-based implementation of the `Map` interface that stores data as **key-value pairs**.

Its main goal is **fast insertion, retrieval, and deletion**.

```text
                     HASHMAP
                        |
        +---------------+---------------+
        |                               |
   Hash Function                 Bucket Array
        |                               |
 Generates Bucket Index        Stores Nodes (Key, Value)
```

Instead of searching every element, HashMap calculates where the key should be stored.

---

# 2. Why Do We Need HashMap?

Suppose we have employee records.

Without HashMap:

```text
Employee List

101 -> Ravi
102 -> John
103 -> Alex
104 -> David
105 -> Sam

Search 104

101
↓

102
↓

103
↓

104

Time Complexity = O(n)
```

With HashMap:

```text
Key = 104

↓

hashCode()

↓

Bucket 4

↓

Employee Found

Time Complexity = O(1) Average
```

---

# 3. Internal Structure

HashMap internally contains an array called the **bucket array**.

```text
Bucket Array

Index

0 ---> null

1 ---> Node

2 ---> Node -> Node

3 ---> TreeNode

4 ---> null

5 ---> Node

...
15 ---> Node
```

Each bucket stores a `Node`.

```java
static class Node<K,V>{

    final int hash;
    final K key;
    V value;
    Node<K,V> next;
}
```

Visualization

```text
+----------------------+
| hash                 |
| key                  |
| value                |
| next -------------->
+----------------------+
```

---

# 4. Internal Working of `put()`

Suppose

```java
map.put("Ravi",100);
```

### Step 1: Calculate hash

```java
hash = key.hashCode();
```

Example

```text
"Ravi"

↓

hashCode()

↓

981245
```

---

### Step 2: Calculate bucket index

Java doesn't use `%`.

Instead,

```java
index = (n - 1) & hash;
```

Example

```text
Capacity = 16

hash = 981245

↓

15 & 981245

↓

Bucket 13
```

Using bitwise AND is faster than modulo.

---

### Step 3: Bucket Empty?

```text
Bucket 13

↓

Empty ?

↓

YES

↓

Insert Node
```

---

### Step 4: Bucket Already Has Data

```text
Bucket

Node1

↓

Node2

↓

Node3
```

HashMap checks

```java
equals()
```

If key already exists

```text
equals()

↓

true

↓

Replace Old Value
```

Else

```text
equals()

↓

false

↓

Insert New Node
```

---

# 5. Collision Handling

Two different keys can generate the same bucket.

Example

```text
hashCode()

John -> Bucket 5

Alex -> Bucket 5
```

Collision

```text
Bucket 5

John

↓

Alex

↓

David
```

This is called **Separate Chaining**.

---

# 6. Treeification (Java 8)

Before Java 8

```text
Bucket

Node

↓

Node

↓

Node

↓

Node

↓

Node
```

Searching becomes

```text
O(n)
```

Java 8 introduced **Red-Black Trees**.

After treeification

```text
Bucket

        Node
       /    \
   Node      Node
   /            \
Node            Node
```

Search becomes

```text
O(log n)
```

Treeification happens only when

* Bucket size ≥ **8**
* Table capacity ≥ **64**

Otherwise, HashMap prefers resizing.

---

# 7. Resize Mechanism

Default Capacity

```text
16
```

Default Load Factor

```text
0.75
```

Threshold

```text
16 × 0.75 = 12
```

When the 13th element is inserted

```text
Capacity

16

↓

32

↓

Recalculate Bucket

↓

Move Nodes
```

Why resize?

Because too many collisions reduce performance.

---

# 8. Internal Flow of `get()`

Suppose

```java
map.get("Ravi");
```

Internally

```text
Key

↓

hashCode()

↓

Bucket Index

↓

Bucket Found

↓

First Node

↓

equals()

↓

Found?

↓

Return Value
```

---

# 9. Time Complexity

| Operation | Average | Worst Case |
| --------- | ------- | ---------- |
| put()     | O(1)    | O(log n)   |
| get()     | O(1)    | O(log n)   |
| remove()  | O(1)    | O(log n)   |

Worst case occurs after treeification. Before Java 8 it was O(n).

---

# 10. Important Characteristics

* Allows **one null key**
* Allows multiple null values
* Not synchronized
* Not thread-safe
* No ordering guarantee
* Uses Array + Linked List + Red-Black Tree

---

# 11. Real-World Example

```text
Student Roll Number

↓

HashMap

101 -> Ravi

102 -> John

103 -> Alex

104 -> Sam

Search Roll No 104

↓

Direct Bucket Lookup
```

---

# 12. Common Interview Questions

### Why is HashMap O(1)?

Because hashing directly computes the bucket instead of scanning every element.

---

### Why power-of-two capacity?

Allows

```java
(hash) & (capacity - 1)
```

instead of modulo, which is faster.

---

### Why immutable keys?

Changing the key changes its hash code, making the entry unreachable.

---

### Why only one null key?

All null keys have the same hash (0), so only one unique null key can exist.

---

### Does HashMap preserve insertion order?

No. Use **LinkedHashMap** if insertion order matters.

---

# 13. Interview Summary

```text
HashMap

✔ Array + Linked List + Red-Black Tree

✔ Hash Table

✔ O(1) Average

✔ O(log n) Worst

✔ One Null Key

✔ Multiple Null Values

✔ Not Thread Safe

✔ Default Capacity = 16

✔ Load Factor = 0.75

✔ Treeify Threshold = 8

✔ Resize Threshold = Capacity × Load Factor
```

---

# Java Collections - ConcurrentHashMap

---

# 1. Core Concept

A **ConcurrentHashMap** is a thread-safe implementation of `Map` designed for **high concurrency**.

Unlike `HashMap`, it allows **multiple threads to read and update the map simultaneously** without locking the entire map.

```text
               ConcurrentHashMap
                        |
       +----------------+----------------+
       |                                 |
  Thread Safety                  High Performance
       |                                 |
 Multiple Readers          Multiple Writers
```

---

# 2. Why Do We Need ConcurrentHashMap?

Imagine an online shopping application:

* Thousands of users are viewing products.
* Hundreds are updating stock.
* Multiple services are reading prices.

Using a regular `HashMap`:

```text
Thread 1

put()

↓

Thread 2

put()

↓

Race Condition

↓

Data Corruption
```

With `Collections.synchronizedMap()`:

```text
Entire Map Locked

↓

One Thread Works

↓

Others Wait
```

This is safe but slow.

ConcurrentHashMap improves this by allowing much more parallelism.

---

# 3. Internal Evolution

### Java 7

Used **Segment Locking**.

```text
ConcurrentHashMap

Segment 1

Segment 2

Segment 3

Segment 4
```

Only one thread could modify a segment at a time.

---

### Java 8+

No more segments.

Uses

* CAS (Compare-And-Swap)
* synchronized on individual bins only when necessary
* volatile fields for visibility

```text
Bucket Array

0

1

2

3

4

Only Bucket 2 Locked

Other Buckets Continue
```

This greatly increases throughput under contention.

---

# 4. Internal Working of `put()`

Suppose

```java
map.put("Apple",100);
```

Flow

```text
Key

↓

hashCode()

↓

Bucket Index

↓

Bucket Empty?

Yes

↓

CAS Insert

No

↓

Lock Only That Bucket

↓

Update

↓

Unlock
```

Most insertions into empty buckets complete without locking using CAS.

---

# 5. Internal Working of `get()`

```java
map.get("Apple");
```

Flow

```text
Key

↓

hashCode()

↓

Bucket

↓

Read Node

↓

Return Value
```

Reads generally do not require locking because of `volatile` semantics.

---

# 6. Resize Mechanism

Like HashMap, it resizes when the load grows.

Difference:

```text
HashMap

One Thread

↓

Resize


ConcurrentHashMap

Multiple Threads

↓

Help Transfer Data

↓

Resize Completes Faster
```

During resizing, threads can cooperate in moving buckets to the new table.

---

# 7. Time Complexity

| Operation | Average |
| --------- | ------- |
| put()     | O(1)    |
| get()     | O(1)    |
| remove()  | O(1)    |

Under heavy collisions, tree bins keep operations efficient, similar to `HashMap`.

---

# 8. Why Doesn't It Allow `null`?

HashMap

```java
map.put(null,100);
```

Allowed.

ConcurrentHashMap

```java
map.put(null,100);
```

Throws

```java
NullPointerException
```

Reason:

```text
map.get(key)

↓

Returns null

Does that mean

1. Key absent?

OR

2. Value stored is null?

Impossible to distinguish safely in concurrent access.
```

Disallowing `null` avoids this ambiguity.

---

# 9. Atomic Operations

ConcurrentHashMap provides atomic methods that help avoid race conditions.

```java
putIfAbsent()

replace()

remove(key, value)

compute()

computeIfAbsent()

computeIfPresent()

merge()
```

Example:

```java
map.computeIfAbsent("Ravi", k -> new ArrayList<>());
```

Only one thread computes and inserts the value for a missing key.

---

# 10. HashMap vs ConcurrentHashMap

| Feature     | HashMap              | ConcurrentHashMap      |
| ----------- | -------------------- | ---------------------- |
| Thread Safe | ❌ No                 | ✅ Yes                  |
| Null Key    | ✅ One                | ❌ No                   |
| Null Values | ✅ Yes                | ❌ No                   |
| Read Lock   | ❌                    | Usually Lock-Free      |
| Write Lock  | ❌                    | Bucket-Level Lock/CAS  |
| Performance | Fast (Single Thread) | Fast Under Concurrency |
| Iteration   | Fail-Fast            | Weakly Consistent      |

---

# 11. Real-World Example

```text
Inventory Service

Product ID

↓

ConcurrentHashMap

101 -> Stock = 20

102 -> Stock = 35

103 -> Stock = 12

Thousands of users

↓

Read & Update Concurrently

↓

No Data Corruption
```

---

# 12. Common Interview Questions

### Why is ConcurrentHashMap faster than `Collections.synchronizedMap()`?

`Collections.synchronizedMap()` locks the entire map for each operation, while ConcurrentHashMap minimizes contention by locking only the affected bucket when necessary and using CAS for many updates.

---

### Why are reads usually lock-free?

The internal nodes use `volatile` fields, ensuring visibility without locking for most read operations.

---

### Why are `compute()` methods useful?

They perform read-modify-write operations atomically, preventing race conditions when multiple threads update the same key.

---

### Can multiple threads write simultaneously?

Yes, as long as they are operating on different buckets. Threads contending for the same bucket may synchronize briefly.

---

### Is ConcurrentHashMap completely lock-free?

No. It is **not fully lock-free**. It combines CAS, volatile fields, and fine-grained synchronization to achieve high concurrency.

---

# 13. Interview Summary

```text
ConcurrentHashMap

✔ Thread Safe

✔ High Concurrency

✔ Lock-Free Reads (mostly)

✔ CAS + Bucket-Level Locking

✔ Java 8 Removed Segment Locking

✔ No Null Keys

✔ No Null Values

✔ Weakly Consistent Iterator

✔ Atomic compute()/merge()/putIfAbsent()

✔ Best Choice for Shared Maps in Multi-Threaded Applications
```
