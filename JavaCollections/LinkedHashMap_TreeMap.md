# LinkedHashMap

## What is LinkedHashMap?

`LinkedHashMap` is a HashMap that **maintains the insertion order** (or access order if configured). It internally uses a **Hash Table + Doubly Linked List**.

* Lookup: **O(1)** average
* Insertion: **O(1)** average
* Deletion: **O(1)** average

It extends `HashMap`.

```java
public class LinkedHashMap<K, V> extends HashMap<K, V>
```

---

## Internal Structure

It has everything a HashMap has:

* Array of buckets
* Hashing
* Collision handling (Linked List / Tree)

Additionally, every node contains two extra pointers:

```text
before
after
```

These create a **doubly linked list** connecting all entries.

```text
Hash Table

Bucket0 ---> Node(A)
                 |
                 V
Bucket3 ---> Node(C)

Bucket5 ---> Node(B)

Doubly Linked List

A <----> C <----> B
```

This linked list preserves order.

---

## How does insertion work?

Suppose:

```java
map.put(10, "A");
map.put(20, "B");
map.put(15, "C");
```

Internally

```text
Hash Table

10 -> A
20 -> B
15 -> C

Linked List

HEAD

10 -> 20 -> 15

TAIL
```

Iteration becomes

```java
for(var entry : map.entrySet())
```

Output

```
10
20
15
```

Exactly insertion order.

---

## Updating Existing Key

```java
map.put(20, "New");
```

Order does **not** change.

Still

```
10
20
15
```

Only value updates.

---

# Access Order

LinkedHashMap has another constructor.

```java
LinkedHashMap<Integer, String> map =
    new LinkedHashMap<>(16, 0.75f, true);
```

Last parameter

```java
true
```

means

> Maintain **access order**, not insertion order.

Example

```java
map.put(1, "A");
map.put(2, "B");
map.put(3, "C");
```

Current order

```
1
2
3
```

Now

```java
map.get(1);
```

becomes

```
2
3
1
```

Recently accessed entry moves to the end.

---

# Why Access Order?

Because it helps implement an **LRU (Least Recently Used) Cache**.

Whenever an entry is accessed, it becomes the newest.

Oldest entry remains at the front.

Removing first entry removes least recently used object.

---

# LRU Cache Example

```java
class LRUCache<K,V> extends LinkedHashMap<K,V>{

    private final int capacity;

    public LRUCache(int capacity){
        super(capacity,0.75f,true);
        this.capacity = capacity;
    }

    @Override
    protected boolean removeEldestEntry(Map.Entry<K,V> eldest){
        return size() > capacity;
    }
}
```

Usage

```java
LRUCache<Integer,String> cache = new LRUCache<>(3);

cache.put(1,"A");
cache.put(2,"B");
cache.put(3,"C");

cache.get(1);

cache.put(4,"D");
```

Removed

```
2
```

because it became the least recently used.

---

# Null Support

LinkedHashMap supports

* One null key ✅
* Multiple null values ✅

```java
map.put(null, "A");
map.put(1, null);
```

Perfectly valid.

---

# Time Complexity

| Operation | Complexity |
| --------- | ---------- |
| put()     | O(1)       |
| get()     | O(1)       |
| remove()  | O(1)       |
| iteration | O(n)       |

---

# Memory

Compared to HashMap,

each node stores

```text
before pointer
after pointer
```

So LinkedHashMap consumes more memory.

---

# When to use LinkedHashMap?

Use it when:

* Order matters
* Fast lookups are needed
* Building LRU cache
* Configuration files
* JSON parsing
* APIs where response order should be preserved

---

# Common Interview Questions

### 1. Difference between HashMap and LinkedHashMap?

| HashMap                 | LinkedHashMap                      |
| ----------------------- | ---------------------------------- |
| No order                | Maintains insertion/access order   |
| Less memory             | Slightly more memory               |
| Faster by a tiny margin | Slightly slower due to linked list |
| Hash Table              | Hash Table + Doubly Linked List    |

---

### 2. Why is LinkedHashMap slower?

Because every insertion/removal also updates the doubly linked list pointers (`before` and `after`), adding a small constant overhead.

---

### 3. Can LinkedHashMap implement LRU Cache?

**Yes.** It has built-in support through:

* `accessOrder = true`
* `removeEldestEntry()`

---

---

# TreeMap

## What is TreeMap?

`TreeMap` stores key-value pairs in **sorted order of keys**.

Unlike HashMap, TreeMap is implemented using a **Red-Black Tree** (a self-balancing Binary Search Tree).

* Search: **O(log n)**
* Insert: **O(log n)**
* Delete: **O(log n)**

```java
public class TreeMap<K,V>
        extends AbstractMap<K,V>
        implements NavigableMap<K,V>
```

---

## Internal Structure

```text
           40(B)
          /     \
      20(R)     60(R)
      /   \      /  \
   10(B)30(B)50(B)70(B)
```

* Every node has at most 2 children.
* Keys are always kept sorted.
* Tree remains balanced using Red-Black Tree rules.

---

## Example

```java
TreeMap<Integer,String> map = new TreeMap<>();

map.put(50,"A");
map.put(20,"B");
map.put(80,"C");
map.put(10,"D");
```

Iteration

```java
for(var e : map.entrySet())
```

Output

```
10
20
50
80
```

Regardless of insertion order.

---

# Why Red-Black Tree?

A normal Binary Search Tree can become skewed:

```text
1
 \
  2
   \
    3
     \
      4
```

Searching becomes

```
O(n)
```

Red-Black Tree performs rotations and recoloring to keep height approximately `log n`, ensuring `put()`, `get()`, and `remove()` remain `O(log n)`.

---

# Natural Ordering

By default,

TreeMap sorts using

```java
Comparable
```

Example

```java
TreeMap<Integer,String> map = new TreeMap<>();
```

Numbers

```
1
2
5
9
```

Strings

```
Apple
Banana
Cat
Dog
```

---

# Custom Sorting

Using Comparator

```java
TreeMap<Integer,String> map =
    new TreeMap<>(Comparator.reverseOrder());

map.put(1,"A");
map.put(3,"B");
map.put(2,"C");
```

Output

```
3
2
1
```

---

# NavigableMap Features

TreeMap provides powerful navigation methods:

| Method          | Description                           |
| --------------- | ------------------------------------- |
| `firstKey()`    | Smallest key                          |
| `lastKey()`     | Largest key                           |
| `higherKey(k)`  | Next greater key                      |
| `lowerKey(k)`   | Previous smaller key                  |
| `ceilingKey(k)` | Smallest key ≥ k                      |
| `floorKey(k)`   | Largest key ≤ k                       |
| `subMap()`      | Keys within a range                   |
| `headMap()`     | Keys less than a value                |
| `tailMap()`     | Keys greater than or equal to a value |

Example:

```java
TreeMap<Integer,String> map = new TreeMap<>();

map.put(10,"A");
map.put(20,"B");
map.put(30,"C");

System.out.println(map.higherKey(20));   // 30
System.out.println(map.lowerKey(20));    // 10
System.out.println(map.ceilingKey(25));  // 30
System.out.println(map.floorKey(25));    // 20
```

---

# Null Support

* **Null keys:** ❌ Not allowed (throws `NullPointerException` with natural ordering).
* **Null values:** ✅ Allowed.

```java
map.put(null, "A"); // Exception
map.put(1, null);   // Valid
```

---

# Time Complexity

| Operation  | Complexity |
| ---------- | ---------- |
| put()      | O(log n)   |
| get()      | O(log n)   |
| remove()   | O(log n)   |
| firstKey() | O(log n)   |
| lastKey()  | O(log n)   |
| iteration  | O(n)       |

---

# When to use TreeMap?

Use TreeMap when:

* Keys must always remain sorted.
* You need range queries (`subMap`, `headMap`, `tailMap`).
* You need nearest-key lookups (`floorKey`, `ceilingKey`, etc.).
* You need ordered reports or leaderboards.

Avoid TreeMap when ordering is not required and you need maximum performance—`HashMap` is typically faster.

---

# Common Interview Questions

### 1. Difference between HashMap and TreeMap?

| HashMap                 | TreeMap                  |
| ----------------------- | ------------------------ |
| Unordered               | Sorted by key            |
| Hash Table              | Red-Black Tree           |
| O(1) average operations | O(log n) operations      |
| Allows one null key     | Does not allow null keys |
| Best for fast lookups   | Best for sorted data     |

---

### 2. Difference between LinkedHashMap and TreeMap?

| LinkedHashMap                                     | TreeMap                                 |
| ------------------------------------------------- | --------------------------------------- |
| Maintains insertion/access order                  | Maintains sorted key order              |
| Hash Table + Doubly Linked List                   | Red-Black Tree                          |
| O(1) average operations                           | O(log n) operations                     |
| One null key allowed                              | Null keys not allowed                   |
| Ideal for preserving insertion order or LRU cache | Ideal for sorted data and range queries |

---

### 3. When would you choose TreeMap over HashMap?

Choose `TreeMap` when your application needs keys to stay sorted or requires operations like `firstKey()`, `lastKey()`, `floorKey()`, `ceilingKey()`, or efficient range queries. Otherwise, `HashMap` is usually the better choice for performance.
