# ArrayList

## What is ArrayList?

`ArrayList` is a **resizable array** implementation of the `List` interface.

Unlike arrays, its size grows automatically when elements are added.

```java
List<String> list = new ArrayList<>();
```

### Characteristics

* Maintains insertion order ✅
* Allows duplicate elements ✅
* Allows multiple `null` values ✅
* Not synchronized (not thread-safe) ❌
* Backed by a dynamic array

---

# Internal Structure

```text
Index

0      1      2      3      4
+------+------+------+------+------+
| A    | B    | C    | D    |      |
+------+------+------+------+------+
```

Internally, `ArrayList` stores elements in an array (`Object[] elementData`).

---

# How does it grow?

Suppose capacity is 10.

After adding the 11th element:

```text
Old Array (10)

[A][B][C][D][E][F][G][H][I][J]

↓

Create New Array (~1.5x larger)

[A][B][C][D][E][F][G][H][I][J][ ][ ][ ][ ]
```

Steps:

1. Create a larger array (typically **1.5×** the old capacity).
2. Copy existing elements to the new array.
3. Add the new element.
4. Replace the old array.

This resizing is expensive, but it happens infrequently, so appending is **amortized O(1)**.

---

# Time Complexity

| Operation     | Complexity     |
| ------------- | -------------- |
| get(index)    | O(1)           |
| set(index)    | O(1)           |
| add(element)  | O(1) amortized |
| add(index)    | O(n)           |
| remove(index) | O(n)           |
| contains()    | O(n)           |

---

# Example

```java
List<String> list = new ArrayList<>();

list.add("Java");
list.add("Spring");
list.add("Kafka");

System.out.println(list.get(1)); // Spring
```

---

# Advantages

* Very fast random access (`get(index)`).
* Less memory overhead.
* Cache-friendly due to contiguous memory.
* Best for read-heavy applications.

---

# Disadvantages

* Inserting/removing in the middle requires shifting elements.
* Resizing involves copying the array.
* Slow for frequent insertions/deletions.

---

# Real-World Uses

* Displaying product lists.
* Student records.
* Search results.
* Read-heavy applications.

---

# Common Interview Questions

### Why is `get()` O(1)?

Because the address is calculated directly:

```text
Address = Base + (index × elementSize)
```

No traversal is required.

---

### Why is insertion in the middle O(n)?

Example:

```text
Before

A B C D E

Insert X at index 2

A B X C D E
```

`C`, `D`, and `E` must be shifted one position to the right.

---

## LinkedList

## What is LinkedList?

`LinkedList` is a **doubly linked list** implementation of the `List` and `Deque` interfaces.

Each element (node) stores:

* Data
* Reference to previous node
* Reference to next node

```java
List<String> list = new LinkedList<>();
```

---

# Internal Structure

```text
null <- [A] <-> [B] <-> [C] <-> [D] -> null
```

Each node contains:

```text
Previous | Data | Next
```

Unlike `ArrayList`, elements are **not stored contiguously** in memory.

---

# Insertion

Insert `X` between `B` and `C`.

Before:

```text
A <-> B <-> C <-> D
```

After:

```text
A <-> B <-> X <-> C <-> D
```

Only pointers are updated.

No shifting of elements.

---

# Time Complexity

| Operation     | Complexity |
| ------------- | ---------- |
| addFirst()    | O(1)       |
| addLast()     | O(1)       |
| removeFirst() | O(1)       |
| removeLast()  | O(1)       |
| get(index)    | O(n)       |
| add(index)    | O(n)       |
| remove(index) | O(n)       |
| contains()    | O(n)       |

> Although insertion/deletion itself is O(1), finding the node at a given index takes O(n), so `add(index)` and `remove(index)` are O(n).

---

# Example

```java
LinkedList<String> list = new LinkedList<>();

list.add("Java");
list.add("Spring");
list.addFirst("Kafka");

System.out.println(list);
```

Output:

```text
[Kafka, Java, Spring]
```

---

# Queue Support

Because it implements `Deque`, `LinkedList` can be used as:

### Queue

```java
Queue<Integer> q = new LinkedList<>();

q.offer(10);
q.offer(20);

System.out.println(q.poll()); // 10
```

### Stack

```java
Deque<Integer> stack = new LinkedList<>();

stack.push(10);
stack.push(20);

System.out.println(stack.pop()); // 20
```

---

# Advantages

* Fast insertion/removal at the beginning or end.
* No resizing required.
* Good for queue and deque operations.

---

# Disadvantages

* Slow random access (`get(index)`).
* Higher memory usage due to `prev` and `next` pointers.
* Poor cache locality because nodes are scattered in memory.

---

# Real-World Uses

* Queue implementations.
* Undo/redo functionality.
* Browser history navigation.
* LRU-style structures (often combined with a `HashMap`).

---

# Common Interview Questions

### Why is `get(index)` O(n)?

To reach an element, `LinkedList` must traverse node by node from the head or tail until it reaches the desired index.

---

### Why is insertion at the beginning O(1)?

Only the head pointer and a few node references need to be updated.

---

# ArrayList vs LinkedList

| Feature                 | ArrayList                        | LinkedList                                          |
| ----------------------- | -------------------------------- | --------------------------------------------------- |
| Internal structure      | Dynamic Array                    | Doubly Linked List                                  |
| Random access (`get`)   | **O(1)**                         | **O(n)**                                            |
| Add at end              | O(1) amortized                   | O(1)                                                |
| Insert/delete in middle | O(n) (shifting)                  | O(n) (traversal, then O(1) link update)             |
| Memory usage            | Lower                            | Higher                                              |
| Cache locality          | Excellent                        | Poor                                                |
| Best use case           | Frequent reads and random access | Frequent insertions/removals at ends, queues/deques |

---

# Interview Answer (30 seconds)

> **ArrayList** is backed by a dynamic array, so it provides **O(1)** random access and is ideal for read-heavy applications. However, inserting or deleting elements in the middle is **O(n)** because elements must be shifted.
>
> **LinkedList** is implemented as a doubly linked list. It provides **O(1)** insertion and deletion at the beginning or end, but random access is **O(n)** because it must traverse the list. It's commonly used for queues, deques, and workloads with frequent end insertions/removals.
