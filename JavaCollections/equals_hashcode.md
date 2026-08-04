# Java Collections - `equals()` & `hashCode()`

Before learning **HashMap**, **HashSet**, or **ConcurrentHashMap**, you must understand `equals()` and `hashCode()` because almost every hash-based collection relies on them.

---

# 1. Core Concept

`equals()` and `hashCode()` work together to determine whether two objects are considered the same.

```text
                  Object Comparison
                         |
          +--------------+--------------+
          |                             |
      hashCode()                    equals()
      ----------                    --------
Generates bucket number      Checks actual equality
(Fast filtering)             (Final verification)
```

Think of them as two stages:

* **Stage 1:** `hashCode()` narrows down where to search.
* **Stage 2:** `equals()` confirms whether the object really matches.

Without `hashCode()`, Java would have to compare every object one by one.

---

# 2. Why Do We Need Them?

Imagine a library with **1 million books**.

Without hashing:

```text
Book Search

Book1
Book2
Book3
...
Book999999

Search Time = O(n)
```

With hashing:

```text
Book Title
     |
hashCode()
     |
Bucket 1254
     |
Compare only books in Bucket 1254
```

Instead of checking every object, Java checks only one bucket.

This is why **HashMap lookup is O(1) on average.**

---

# 3. What is `equals()`?

`equals()` compares the **logical/content equality** of two objects.

Default implementation from `Object`:

```java
public boolean equals(Object obj) {
    return this == obj;
}
```

By default,

* compares **memory addresses**
* not object contents

Example:

```java
String s1 = new String("Java");
String s2 = new String("Java");

System.out.println(s1 == s2);      // false
System.out.println(s1.equals(s2)); // true
```

Explanation:

```text
Heap Memory

s1 ----------> "Java"

s2 ----------> "Java"

Different Objects

==

Compare Address

false

equals()

Compare Content

true
```

---

# 4. What is `hashCode()`?

`hashCode()` returns an integer representing the object.

```java
int hash = obj.hashCode();
```

It **does not uniquely identify an object**.

Its purpose is simply to decide which bucket an object belongs to.

Example:

```java
String s = "Java";

System.out.println(s.hashCode());
```

Output (example):

```
2301506
```

HashMap converts this into a bucket index.

```text
hashCode()

2301506

↓

Bucket Index

2301506 % 16

↓

Bucket 2
```

In Java 8+, the bucket index calculation is optimized internally as:

```java
(hash) & (capacity - 1)
```

We'll discuss this in detail when covering **HashMap internals**.

---

# 5. Why Are Both Needed?

Suppose we have two students.

```java
Student s1 = new Student(101, "Ravi");
Student s2 = new Student(101, "Ravi");
```

Hash codes:

```
s1.hashCode() = 5432

s2.hashCode() = 5432
```

Does that mean they are equal?

**No.**

Hash codes only determine the bucket.

Java still calls

```java
equals()
```

to verify actual equality.

Workflow:

```text
Search Key

↓

hashCode()

↓

Find Bucket

↓

equals()

↓

Object Found
```

---

# 6. Why Can't Java Use Only `equals()`?

Suppose HashMap contains **100,000 objects**.

Without hashCode:

```text
equals()

↓

Compare Object 1

↓

Compare Object 2

↓

Compare Object 3

↓

...

↓

Compare Object 100000
```

Time Complexity

```
O(n)
```

With hashCode:

```text
hashCode()

↓

Bucket 5

↓

Compare only

Object A

Object B

Object C
```

Time Complexity

```
O(1) Average
```

---

# 7. Why Can't Java Use Only `hashCode()`?

Different objects can produce the same hash code.

Example:

```text
Object A

hashCode = 15


Object B

hashCode = 15


Different Objects

Same Bucket
```

This is called a **hash collision**.

So Java must verify using

```java
equals()
```

---

# 8. The Contract Between `equals()` and `hashCode()`

This is one of the most common interview questions.

### Rule 1

If two objects are equal,

their hash codes **must** be equal.

```java
a.equals(b) == true

↓

a.hashCode() == b.hashCode()
```

---

### Rule 2

If hash codes are equal,

objects **may or may not** be equal.

```text
hashCode()

100

↓

Student A

Student B

↓

equals()

true OR false
```

---

### Rule 3

If hash codes are different,

objects are definitely different.

```java
a.hashCode() != b.hashCode()

↓

equals()

false
```

---

# 9. How HashMap Uses Them

Suppose

```java
map.put(student, "Developer");
```

Internally:

```text
Step 1

student.hashCode()

↓

Step 2

Bucket Index

↓

Bucket Empty?

Yes

↓

Insert


No

↓

equals()

↓

Key Exists?

Yes

↓

Replace Value


No

↓

Insert New Node
```

Notice

`equals()` is **not called immediately**.

It is called **only when two keys land in the same bucket**.

This optimization makes HashMap extremely fast.

---

# 10. Example of Overriding Both Methods

```java
import java.util.Objects;

class Student {

    int id;
    String name;

    Student(int id, String name) {
        this.id = id;
        this.name = name;
    }

    @Override
    public boolean equals(Object obj) {

        if (this == obj)
            return true;

        if (!(obj instanceof Student))
            return false;

        Student other = (Student) obj;

        return id == other.id &&
               Objects.equals(name, other.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(id, name);
    }
}
```

Why use `Objects.hash(...)`?

* Simpler implementation
* Handles `null` safely
* Produces a consistent hash based on the selected fields

---

# 11. What Happens If You Override Only `equals()`?

```java
Student s1 = new Student(1,"Ravi");
Student s2 = new Student(1,"Ravi");

System.out.println(s1.equals(s2));
```

Output

```
true
```

Now

```java
HashSet<Student> set = new HashSet<>();

set.add(s1);
set.add(s2);
```

Expected

```
1
```

Actual

```
2
```

Why?

```text
equals()

true

BUT

hashCode()

Different

↓

Different Buckets

↓

HashSet Never Calls equals()

↓

Duplicate Objects Stored
```

This is a very common interview scenario.

---

# 12. What Happens If You Override Only `hashCode()`?

```java
hashCode()

Same

↓

Same Bucket

↓

equals()

false

↓

HashMap Treats Them

As Different Keys
```

The program works correctly, but performance may degrade because many objects end up in the same bucket, increasing comparisons.

---

# 13. Mutable Keys: A Dangerous Mistake

```java
class Employee {

    int id;

    // equals() & hashCode() use id
}
```

```java
Employee e = new Employee(10);

map.put(e, "Developer");

e.id = 20;
```

Now

```java
map.get(e);
```

returns

```
null
```

Why?

```text
Insertion

id = 10

↓

Bucket 5


Later

id = 20

↓

hashCode Changed

↓

Looks in Bucket 8

↓

Object Still in Bucket 5

↓

Not Found
```

**Never modify fields used by `equals()` or `hashCode()` after inserting an object into a hash-based collection.**

---

# 14. Real-World Example

Imagine an employee ID card.

```text
Employee ID Card

ID = 1001

↓

Security Gate

↓

Employee Database

↓

Verify Employee Details

↓

Allow Entry
```

Similarly,

* `hashCode()` acts like the **ID number** that directs you quickly to the right location.
* `equals()` acts like the **identity verification**, ensuring it's the correct person before granting access.

---

# 15. Common Interview Questions

### Q1. Why does HashMap use both `hashCode()` and `equals()`?

To achieve fast lookups. `hashCode()` identifies the bucket quickly, while `equals()` confirms that the matching key is actually the correct object.

---

### Q2. Can two objects have the same hash code?

Yes. This is called a **hash collision**, and `equals()` is then used to distinguish between them.

---

### Q3. Can two equal objects have different hash codes?

No. If `equals()` returns `true`, both objects **must** return the same hash code.

---

### Q4. Why should immutable objects be used as HashMap keys?

Because changing a field used in `equals()` or `hashCode()` changes the computed hash, making the object unreachable in the map.

---

### Q5. Why is `String` a perfect HashMap key?

* Immutable
* Correctly overrides `equals()`
* Correctly overrides `hashCode()`
* Frequently reused and optimized by the JVM

---

# 16. Interview Summary

```text
equals() & hashCode()

✔ equals() → Compares object content (logical equality)

✔ hashCode() → Determines bucket location

✔ HashMap first calls hashCode()

✔ equals() is called only when needed (same bucket)

✔ Equal objects must have equal hash codes

✔ Same hash code does not guarantee equality

✔ Always override equals() and hashCode() together

✔ Prefer immutable keys for HashMap and HashSet
```

---

## Key Takeaway

Remember this sequence—it explains the behavior of almost every hash-based collection in Java:

```text
                 HashMap Lookup

                      Key
                       │
                       ▼
                 hashCode()
                       │
                       ▼
                Compute Bucket
                       │
                       ▼
            Same Bucket Collision?
                 │            │
                No           Yes
                 │            ▼
          Direct Match     equals()
                               │
                               ▼
                    Logical Equality Check
```

Once you're comfortable with this, the internal implementation of **HashMap** (bucket array, collisions, resizing, and treeification) becomes much easier to understand.
