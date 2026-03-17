
---

# JPA Interview Questions List


---

# 1. What is JPA and why is it used?

**JPA (Java Persistence API)** is a **Java specification for Object Relational Mapping (ORM)** that allows Java applications to interact with relational databases using objects instead of SQL-heavy code.

Instead of writing JDBC code to map tables to objects manually, JPA handles that mapping automatically through annotations like `@Entity`, `@Id`, and `@OneToMany`.

JPA itself is **not an implementation**, it's just an API specification. Implementations like **Hibernate**, **EclipseLink**, and **OpenJPA** provide the actual functionality.

We use JPA mainly to:

* reduce boilerplate database code
* manage entities automatically
* handle transactions and caching
* make applications database-independent.

---

# 2. What is Object Relational Mapping (ORM) and how does JPA implement it?

**ORM (Object Relational Mapping)** is a technique that maps **Java objects to relational database tables**.

In a relational database we have:

* tables
* rows
* columns

In Java we have:

* classes
* objects
* fields

ORM creates a mapping between these two worlds.

JPA implements ORM using **annotations and metadata**. For example:

* `@Entity` maps a class to a table
* `@Id` maps the primary key
* `@Column` maps fields to columns
* relationship annotations map table relationships.

The ORM provider like **Hibernate** then generates SQL queries automatically and manages persistence behind the scenes.

---

# 3. What is the difference between JPA and Hibernate?

The key difference is that **JPA is a specification**, while **Hibernate** is an **implementation** of that specification.

JPA defines **interfaces and rules** for ORM, such as `EntityManager`, annotations, and persistence context behavior.

Hibernate implements those interfaces and provides the actual functionality like:

* SQL generation
* caching
* dirty checking
* query execution.

In practice, when we use **Spring Data JPA**, we usually use JPA APIs while Hibernate runs underneath as the provider.

Hibernate also provides additional features beyond JPA such as:

* advanced caching
* batch fetching
* proprietary annotations.

---

# 4. What are the key components of JPA architecture?

JPA architecture mainly consists of **four major components**.

First is the **Entity**, which represents a database table mapped to a Java class.

Second is the **EntityManager**, which is responsible for performing operations like persist, update, remove, and find on entities.

Third is the **Persistence Context**, which acts as a first-level cache and tracks managed entities during a transaction.

Fourth is the **EntityManagerFactory**, which creates EntityManager instances and is usually created once per application.

Underneath these components, a JPA provider like **Hibernate** communicates with the database.

---

# 5. What is an Entity in JPA?

An **Entity** is a **Java class that represents a table in a relational database**.

Each instance of the entity corresponds to a row in the table.

To define an entity, we annotate the class with `@Entity` and specify a primary key using `@Id`.

For example, a `User` class can represent a `users` table, where class fields map to table columns.

Entities are managed by the **EntityManager**, which tracks changes and automatically synchronizes them with the database.

In real applications, entities usually contain:

* fields mapped to columns
* relationships with other entities
* business-related attributes.

---

# 6. What is the purpose of the `@Entity` annotation?

The `@Entity` annotation marks a class as a **JPA entity**.
When a class is annotated with `@Entity`, JPA treats it as a **persistent object that maps to a database table**.

During application startup, the ORM provider like **Hibernate** scans for entity classes and builds metadata for mapping them to database tables.

Without `@Entity`, the class is just a normal Java object and JPA will not persist it.

We can also optionally define a table name using the `@Table` annotation.

---

# 7. What is the difference between `@Table` and `@Entity`?

`@Entity` is mandatory for defining a **persistent class**, while `@Table` is optional and used to **customize the database table mapping**.

If we only use `@Entity`, the table name is usually inferred from the class name.

`@Table` allows us to specify additional details such as:

* table name
* schema
* unique constraints
* indexes.

For example:

```java
@Entity
@Table(name = "users")
public class User {}
```

So essentially, `@Entity` tells JPA the class is persistent, while `@Table` controls how it maps to the database table.

---

# 8. What is the role of EntityManager in JPA?

The **EntityManager** is the **core interface in JPA responsible for managing entity lifecycle and database operations**.

It performs operations such as:

* `persist()` to insert entities
* `find()` to retrieve entities
* `merge()` to update detached entities
* `remove()` to delete entities.

It also manages the **persistence context**, which keeps track of managed entities and their changes.

Another important responsibility is **dirty checking**, where it automatically detects changes in managed entities and updates the database during transaction commit.

In most Spring Boot applications, EntityManager is used internally by **Spring Data JPA repositories**.

---

# 9. What is the difference between EntityManagerFactory and EntityManager?

**EntityManagerFactory** is responsible for **creating EntityManager instances**.

It is a **heavyweight object** that is typically created once during application startup and shared across the application.

On the other hand, **EntityManager** is a **lightweight object used to interact with the persistence context and perform CRUD operations**.

Each transaction or request typically uses its own EntityManager instance.

So in simple terms:

* EntityManagerFactory → creates EntityManagers
* EntityManager → performs database operations on entities.

---

# 10. What are the responsibilities of EntityManager?

The EntityManager handles several important responsibilities in JPA.

First, it **manages the lifecycle of entities**, including persisting, updating, and deleting them.

Second, it **maintains the persistence context**, which tracks all managed entities and ensures only one instance of an entity exists per context.

Third, it performs **automatic dirty checking**, meaning any changes to managed entities are automatically synchronized with the database.

It also provides APIs for executing **JPQL queries, native SQL queries, and criteria queries**.

In most modern applications using Spring Boot, developers interact with repositories while EntityManager operates behind the scenes.

---

# 11. What are the different lifecycle states of a JPA entity?

A JPA entity goes through **four lifecycle states**.

1️⃣ **Transient (New)** – The object is created using `new`, but it is not yet associated with the persistence context and is not stored in the database.

2️⃣ **Managed (Persistent)** – Once we call `persist()` or retrieve an entity using `find()`, it becomes managed by the persistence context. Any changes to the entity are automatically tracked.

3️⃣ **Detached** – When the persistence context is closed or the entity is detached manually, it becomes detached and changes are no longer tracked.

4️⃣ **Removed** – When `remove()` is called, the entity is marked for deletion and will be deleted from the database when the transaction commits.

ORM providers like **Hibernate** manage these states internally.

---

# 12. What is the persistence context?

The **persistence context** is a **cache-like environment where JPA manages entity instances during a transaction**.

It ensures that for a given entity identity, only `one instance exists within the context`.

For example, if we fetch the same entity multiple times within a transaction, JPA will return the same object from the persistence context instead of querying the database again.

This mechanism is also known as the **first-level cache**.

The persistence context is managed by the **EntityManager**, and it automatically synchronizes changes with the database when a transaction is committed.

---

# 13. What is dirty checking in JPA?

**Dirty checking** is a mechanism where JPA automatically detects changes made to **managed entities** and updates the database accordingly.

When an entity is in the managed state inside the persistence context, JPA keeps track of its original state. During transaction commit or flush, it compares the current state with the original state.

If it detects changes, it automatically generates an **UPDATE SQL statement**.

For example, if we modify a field like:

```java
user.setName("John");
```

We don't need to explicitly call an update method. The ORM provider such as **Hibernate** will detect the change and update the database automatically.

---

# 14. How does JPA detect changes in entities automatically?

JPA detects changes using the **dirty checking mechanism combined with the persistence context**.

When an entity becomes managed, JPA stores a snapshot of its original state in memory. During the flush phase, it compares the current object state with the snapshot.

If differences are detected, JPA generates the required SQL update statement.

Some providers like **Hibernate** also use **bytecode enhancement or proxies** to optimize change detection.

This mechanism allows developers to work with entities as normal Java objects while the framework handles synchronization with the database.

---

# 15. What happens when an entity becomes detached?

When an entity becomes **detached**, it is no longer associated with the persistence context.

This usually happens when:

* the transaction ends
* the EntityManager is closed
* `detach()` is called manually
* `clear()` is invoked.

Once detached, any modifications made to the entity **will not be tracked or persisted to the database**.

If we want to save changes made to a detached entity, we must reattach it using the `merge()` method, which returns a new managed instance.

---

# 16. What is the difference between managed and detached entities?

A **managed entity** is associated with the persistence context, while a **detached entity** is not.

For managed entities:

* JPA tracks all changes automatically.
* Dirty checking applies.
* Updates are automatically flushed to the database.

For detached entities:

* They exist outside the persistence context.
* JPA does not track modifications.
* Any updates must be explicitly merged back using `EntityManager.merge()`.

In real-world applications, entities often become detached when they move outside the transactional scope, such as returning them from service layers.

---

# 17. What happens internally when `EntityManager.clear()` is called?

When `EntityManager.clear()` is called, the **entire persistence context is cleared**.

This means:

* all managed entities become **detached**
* the first-level cache is emptied
* JPA stops tracking changes for those entities.

After this operation, if we access an entity again, JPA will need to **query the database again** because the cached objects are no longer available.

This method is often used in **batch processing** to prevent memory issues when processing large numbers of entities.

---

# 18. What is the role of the `@Id` annotation?

The `@Id` annotation marks a field as the **primary key of the entity**.

Every entity must have a unique identifier so that JPA can track it in the persistence context.

The primary key allows JPA to:

* uniquely identify rows in the database
* maintain entity identity
* manage caching and updates efficiently.

Typically, the `@Id` field is combined with `@GeneratedValue` to allow the database or ORM provider to generate unique IDs automatically.

---

# 19. What are the different primary key generation strategies in JPA?

JPA provides several strategies to automatically generate primary keys using the `@GeneratedValue` annotation.

The main strategies include:

* **IDENTITY** – The database auto-increment column generates the ID.
* **SEQUENCE** – Uses a database sequence object to generate IDs.
* **TABLE** – Uses a separate table to generate IDs.
* **AUTO** – JPA automatically chooses the appropriate strategy based on the database.

For example:

```java
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;
```

Different ORM providers like **Hibernate** may optimize these strategies internally.

---

# 20. What is the difference between `GenerationType.IDENTITY`, `SEQUENCE`, `AUTO`, and `TABLE`?

These strategies determine **how primary keys are generated**.

**IDENTITY**

* Uses database auto-increment.
* ID generated during insert.
* Common in databases like MySQL.

**SEQUENCE**

* Uses database sequence objects.
* More efficient for batch inserts.
* Common in databases like PostgreSQL or Oracle.

**TABLE**

* Uses a separate table to store ID values.
* Database independent but slower.

**AUTO**

* Lets the JPA provider automatically select the best strategy depending on the database dialect.

In most production systems, **SEQUENCE is preferred for performance**, especially with providers like **Hibernate**.

---

# 22. What is the difference between **persist()**, **merge()**, and **save()**?

The main difference is **how they handle entity state and persistence context**.

**`persist()`** is a JPA method used to **store a new entity**. It makes the entity **managed** and inserts it into the database when the transaction is flushed or committed. It only works for **new/transient entities**.

**`merge()`** is used for **detached entities**. It copies the state of the detached entity into a **new managed instance** inside the persistence context and returns that managed instance.

**`save()`** is not part of JPA. It is a method provided by frameworks like **Spring Data JPA**. Internally it decides whether to call `persist()` or `merge()` depending on whether the entity already has an ID.

So in practice:

* `persist()` → new entity
* `merge()` → update detached entity
* `save()` → convenience method used in repositories.

---

# 23. What is the difference between **remove()** and **delete()**?

**`remove()`** is a **JPA EntityManager method** used to delete an entity from the database. It only works with **managed entities**, meaning the entity must first exist in the persistence context.

For example, if the entity is detached, we usually retrieve it first or merge it before calling remove.

**`delete()`** is typically a method provided by frameworks like **Spring Data JPA** repositories. It internally uses the EntityManager and eventually calls `remove()`.

So the key difference is:

* `remove()` → low-level JPA API
* `delete()` → higher-level repository abstraction.

Both ultimately result in a **DELETE SQL query** being executed during transaction commit.

---

# 24. What does **flush()** do in JPA?

The **`flush()`** method synchronizes the **persistence context with the database**.

When we modify managed entities, the changes are stored in memory inside the persistence context. Calling `flush()` forces JPA to **generate and execute the SQL statements immediately**, such as INSERT, UPDATE, or DELETE.

However, it **does not commit the transaction**. The transaction can still be rolled back after a flush.

ORM providers like **Hibernate** automatically call flush before certain operations like query execution or transaction commit, but developers can call it manually when they need early synchronization.

---

# 25. What is the difference between **flush()** and **commit()**?

The main difference is **scope and finality**.

**`flush()`** synchronizes changes from the persistence context to the database by executing SQL statements, but the **transaction is still active** and changes can still be rolled back.

**`commit()`**, on the other hand, **finalizes the transaction**. Once commit happens, all changes are permanently saved in the database and cannot be rolled back.

So the sequence typically looks like this:

1. Entity changes happen in memory
2. `flush()` sends SQL statements to the database
3. `commit()` permanently saves those changes.

In frameworks like **Spring Data JPA**, flushing usually happens automatically during transaction commit.


---


## 26. Relationship mappings in JPA

JPA supports four relationship types:
`@OneToOne`, `@OneToMany`, `@ManyToOne`, and `@ManyToMany`.

They map object relationships to database associations using foreign keys or join tables.
In real-world applications, **`@ManyToOne` and `@OneToMany` are the most commonly used**.

---

## 27. `@OneToMany` vs `@ManyToOne`

`@ManyToOne` is the **owning side** and holds the foreign key, so it directly controls the relationship.

`@OneToMany` is usually the **inverse side** and uses `mappedBy`.

In practice, we prefer **`@ManyToOne` for better performance and simpler mapping**, and use `@OneToMany` only when we need parent-to-child navigation.

---

## 28. `@OneToOne` vs `@ManyToOne`

`@OneToOne` represents a strict **1:1 relationship** with a unique constraint.

`@ManyToOne` represents **many-to-one**, where multiple records can reference a single parent.

In production, `@ManyToOne` is more common, while `@OneToOne` is used for tightly coupled entities like User–Profile.

---

## 29. Unidirectional vs Bidirectional

Unidirectional means only one entity references the other.

Bidirectional means both entities reference each other, with one **owning side** and one **inverse side** using `mappedBy`.

We use bidirectional mapping when we need navigation in both directions, but keep it minimal to avoid complexity.

---

## 30. Owning side of a relationship

The owning side is the entity that **contains the foreign key and controls the relationship**.

It’s typically the side with `@JoinColumn`.

Only changes made on the owning side are persisted to the database.

---

## 31. `mappedBy` in JPA

`mappedBy` defines the **inverse side** of a bidirectional relationship.

It tells JPA that the other entity owns the relationship, preventing duplicate mappings or extra join tables.

---

## 32. When to use `@OneToMany` vs `@ManyToOne`

Prefer `@ManyToOne` for **performance and simplicity**, especially in large-scale systems.

Use `@OneToMany` only when you need to access child collections from the parent.

Best practice is to use **`@ManyToOne` as the owning side and optionally add `@OneToMany` for read/navigation**.

---


## 33. What is cascade type in JPA?

Cascade type defines how operations performed on a **parent entity** are automatically propagated to its **child entities**.

For example, if we save or delete a parent, cascading ensures the same operation happens on related entities without writing extra code.

It’s mainly used to maintain **relationship consistency and reduce boilerplate**.

---

## 34. What are the different types of `CascadeType`?

JPA provides these cascade types:

* `PERSIST` → saves child entities
* `MERGE` → updates detached child entities
* `REMOVE` → deletes child entities
* `REFRESH` → reloads from database
* `DETACH` → detaches entities from persistence context
* `ALL` → applies all of the above

In real-world use, **PERSIST, MERGE, and REMOVE are the most commonly used**.

---

## 35. Difference between `CascadeType.PERSIST` and `CascadeType.MERGE`

`PERSIST` is used for **new entities**. When we save a parent, all new child entities are also inserted.

`MERGE` is used for **detached entities**. It updates existing child entities when the parent is merged.

👉 Key difference:

* `PERSIST` → insert new records
* `MERGE` → update existing records

---

## 36. When should you use `CascadeType.ALL`?

`CascadeType.ALL` should be used when the child entity’s lifecycle is **completely dependent on the parent**.

For example, in a **Order–OrderItems** relationship where child records should always be created, updated, and deleted along with the parent.

Avoid using it in shared relationships like **ManyToOne**, as it may cause unintended deletes or updates.

---

## 37. What is `orphanRemoval` in JPA?

`orphanRemoval = true` ensures that if a child entity is **removed from the parent’s collection**, it is automatically **deleted from the database**.

It’s useful in scenarios where child entities should not exist independently.

👉 Difference from cascade remove:

* Cascade REMOVE → deletes when parent is deleted
* orphanRemoval → deletes when child is removed from collection

---

## 38. Difference between `FetchType.LAZY` and `FetchType.EAGER`

`LAZY` means data is **loaded only when accessed**, while `EAGER` means data is **loaded immediately with the parent entity**.

`LAZY` improves performance by avoiding unnecessary queries, whereas `EAGER` can lead to **extra joins and over-fetching**.

👉 In production, **LAZY is preferred by default**.

---

## 39. Default fetch type for `@OneToMany` and `@ManyToOne`

* `@OneToMany` → **LAZY by default**
* `@ManyToOne` → **EAGER by default**

This is important because `EAGER` on `ManyToOne` can unintentionally cause **performance issues** if not handled carefully.

---

## 40. Why is LAZY loading recommended?

LAZY loading is recommended because it **fetches data only when needed**, reducing memory usage and unnecessary database queries.

It helps avoid **over-fetching and improves scalability**, especially in large applications with complex relationships.

---

## 41. Problems with EAGER fetching

EAGER fetching can cause:

* **Performance issues** due to unnecessary joins
* **N+1 query problems**
* Loading large object graphs unintentionally
* Increased memory consumption

👉 It can severely impact performance in production if used carelessly.

---

## 42. What is `LazyInitializationException`?

`LazyInitializationException` occurs when a **LAZY-loaded entity is accessed outside the persistence context**, usually after the transaction is closed.

Since the session is no longer active, JPA cannot fetch the data.

👉 Common fixes:

* Use **JOIN FETCH**
* Use **DTO projections**
* Ensure access happens within a transaction

---

## 43. What is JPQL (Java Persistence Query Language)?

JPQL is an **object-oriented query language** used to query **entities instead of database tables**.

It works on entity classes and their fields, not directly on table/column names, making it **database-independent**.

---

## 44. Difference between JPQL and SQL

JPQL operates on **entities and object models**, while SQL operates on **tables and columns**.

JPQL is **database-agnostic**, whereas SQL is **database-specific**.

👉 Example: JPQL uses `User`, SQL uses `users` table.

---

## 45. Difference between JPQL and native queries

JPQL is **portable and object-oriented**, while native queries are **raw SQL** written for a specific database.

Native queries are used when:

* we need **complex queries**
* database-specific features (like window functions)

👉 Trade-off: flexibility vs portability.

---

## 46. What is the Criteria API?

Criteria API is a **type-safe, programmatic way to build queries** in JPA.

Instead of writing strings like JPQL, we construct queries using Java objects.

👉 Useful for **dynamic queries and compile-time safety**.

---

## 47. When to use Criteria API instead of JPQL

Use Criteria API when:

* queries are **dynamic (conditions change at runtime)**
* multiple optional filters are involved
* type safety is required

For simple queries, JPQL is more readable and preferred.

---

## 48. What are named queries?

Named queries are **predefined, static queries** defined using annotations like `@NamedQuery`.

They are compiled at startup, which helps in **early validation and better performance**.

---

## 49. Difference between named query and dynamic query

Named queries are:

* **static**
* defined at compile time
* reusable

Dynamic queries are:

* built at runtime
* flexible for changing conditions

👉 Use named queries for fixed logic, dynamic queries for flexible filtering.

---

## 50. Purpose of `@Query` in Spring Data JPA

`@Query` allows us to define **custom JPQL or native SQL queries directly in repository methods**.

It is useful when:

* derived query methods are not sufficient
* we need custom joins or filters

👉 It provides **more control while still using repository abstraction**.

---


51. What is the **N+1 select problem**?
52. Why does the **N+1 problem occur in Hibernate**?
53. How can the **N+1 problem be solved**?
54. What is **JOIN FETCH in JPQL**?
55. What is **EntityGraph** and when should it be used?

---

56. What is **pagination in JPA**?
57. How do you implement pagination using **Pageable**?
58. What is the difference between **Page, Slice, and List** return types?
59. What is **sorting** in JPA repositories?
60. What problems occur if pagination is not implemented?

---

61. What is **projection in Spring Data JPA**?
62. How do you fetch only selected columns instead of the full entity?
63. How do you implement **DTO projections in Spring Data JPA**?

---

64. How does transaction management work in JPA?
65. What is the role of **@Transactional** in the service layer?
66. What happens if a transaction fails in JPA?
67. What is the difference between **read-only and read-write transactions**?
68. How does **rollback work in JPA transactions**?
69. What happens if a transaction is not defined in a service method?

---

70. What is **optimistic locking**?
71. What is **pessimistic locking**?
72. What is the purpose of the **@Version** annotation?
73. When should optimistic locking be preferred over pessimistic locking?
74. What happens when **OptimisticLockException** occurs?

---

75. What is the **first-level cache in JPA**?
76. What is the **second-level cache**?
77. How does caching work in Hibernate with JPA?
78. What are common second-level cache providers?
79. What are the benefits and drawbacks of caching?

---

80. What is **batch processing in JPA**?
81. How can you optimize **bulk insert or update operations**?
82. What is **batch fetching**?
83. What is the difference between **List and Set in OneToMany mapping**?

---

84. What is **@MappedSuperclass**?
85. What is the difference between **@Embeddable and @Embedded**?
86. What is **inheritance mapping in JPA**?
87. What are the different **inheritance strategies in JPA**?

---

88. How do you map a **Many-to-Many relationship**?
89. How do you map a **composite primary key in JPA**?
90. How do you avoid **infinite recursion in bidirectional relationships when returning JSON**?

---

91. How do you write **dynamic queries with multiple filters**?
92. How would you implement **search with optional parameters**?

---

93. How would you optimize **large data fetch operations**?
94. How would you update **only a few fields of an entity**?
95. How would you implement **soft delete in JPA**?
96. How do you update **millions of records efficiently**?

---

97. How would you debug **unexpected extra SQL queries in Hibernate**?
98. How do you enable **SQL logging in Spring Boot**?
99. How would you analyze **slow queries generated by JPA**?
100. How do you monitor JPA queries in production?

---
