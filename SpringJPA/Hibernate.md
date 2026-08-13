# JPA Entity Lifecycle States & Persistence Context

A **persistence context** is a set of entity instances in which, for any persistent entity identity, there is a unique entity instance. Within the persistence context, the entity instances and their lifecycle are managed.

---

## JPA’s 4 Lifecycle States

The lifecycle model consists of four states: **transient**, **managed**, **detached**, and **removed**.

---

### 1. Transient

The lifecycle state of a newly instantiated entity object is called **transient**. The entity hasn’t been persisted yet, so it doesn’t represent any database record.

Your persistence context doesn’t know about your newly instantiated object. Because of that, it doesn’t automatically perform an SQL `INSERT` statement or track any changes. As long as your entity object is in the lifecycle state *transient*, you can think of it as a basic Java object without any connection to the database and any JPA-specific functionality.

```java
Author author = new Author();
author.setFirstName("Thorben");
author.setLastName("Janssen");

```

That changes when you provide it to the `EntityManager.persist` method *(note: often referenced via `persist` or when fetched via `find`)*. The entity object then changes its lifecycle state to **managed** and gets attached to the current persistence context.

---

### 2. Managed

All entity objects attached to the current persistence context are in the lifecycle state **managed**. That means that your persistence provider (e.g., Hibernate) will detect any changes on the objects and generate the required SQL `INSERT` or `UPDATE` statements when it flushes the persistence context.

There are different ways to get an entity to the lifecycle state **managed**:

1. **Call `EntityManager.persist` with a new entity object:**
```java
Author author = new Author();
author.setFirstName("Thorben");
author.setLastName("Janssen");
em.persist(author);

```


2. **Load an entity object from the database** using `EntityManager.find`, a JPQL query, a `CriteriaQuery`, or a native SQL query:
```java
Author author = em.find(Author.class, 1L);

```


3. **Merge a detached entity** by calling `EntityManager.merge` or updating it via the `update` method on a Hibernate `Session`:
```java
em.merge(author);

```



---

### 3. Detached

An entity that was previously managed but is no longer attached to the current persistence context is in the lifecycle state **detached**.

An entity gets detached when you close the persistence context. That typically happens after a request gets processed. Then the database transaction gets committed, the persistence context gets closed, and the entity object gets returned to the caller. The caller then retrieves an entity object in the lifecycle state *detached*.

You can also programmatically detach an entity by calling the `detach` method on the `EntityManager`:

```java
em.detach(author);

```

> **Note:** There are only very few performance tuning reasons to detach a managed entity. If you decide to detach an entity, you should first flush the persistence context to avoid losing any pending changes.

#### Reattaching an entity

You can reattach an entity by calling the `update` method on your Hibernate `Session` or the `merge` method on the `EntityManager`. In both cases, the entity changes its lifecycle state back to **managed**.

---

### 4. Removed

When you call the `remove` method on your `EntityManager`, the mapped database record doesn’t get removed immediately. The entity object only changes its lifecycle state to **removed**.

During the next flush operation, Hibernate will generate an SQL `DELETE` statement to remove the record from the database table.

```java
em.remove(author);

```

---

## Conclusion

All entity operations are based on JPA’s lifecycle model. It consists of 4 states, which define how your persistence provider handles the entity object:

* **Transient:** New entities that are not attached to the current persistence context.
* **Managed:** Entities attached to the current persistence context (via `persist`, reading from DB, or `merge`). The persistence context generates the required SQL `INSERT` and `UPDATE` statements to persist changes.
* **Detached:** Entities previously associated with an active persistence context that are no longer attached. Changes to these objects will not be persisted automatically.
* **Removed:** Entities scheduled for removal. The persistence provider will generate and execute the required SQL `DELETE` statement during the next flush operation.

# Hibernate — Flush & Caching

## 1. Flush

**Flush** synchronizes the **persistence context with the database** by executing pending SQL statements.

```text
Entity Change
     ↓
Persistence Context
     ↓
   Flush
     ↓
SQL → Database
```

Example:

```java
user.setName("Ravi");
entityManager.flush();
```

Hibernate generates:

```sql
UPDATE users SET name = 'Ravi' WHERE id = ?;
```

**Important:** Flush ≠ Commit. A transaction can still be rolled back after flush.

**Interview:**

> Flush synchronizes the persistence context with the database. It executes pending SQL but does not commit the transaction.

---

## 2. First-Level Cache

The **First-Level Cache** is Hibernate's default cache associated with the **Persistence Context / Session**.

```java
User user1 = entityManager.find(User.class, 101L);
User user2 = entityManager.find(User.class, 101L);
```

The second `find()` can use the cached entity instead of querying the database again.

**Key points:**

* Enabled by default
* Per EntityManager/Session
* Not shared between sessions
* Helps reduce database queries

**Interview:**

> First-Level Cache is Hibernate's default persistence-context cache. It stores managed entities and avoids repeated database queries within the same session.

---

## 3. Second-Level Cache

The **Second-Level Cache** is associated with the **SessionFactory/EntityManagerFactory** and can be shared across multiple persistence contexts.

```text
Session 1 ──┐
            ├──→ Second-Level Cache
Session 2 ──┘
```

**Key points:**

* Not enabled/configured by default
* Shared across sessions
* Requires cache configuration/provider
* Reduces database load

**Interview:**

> Second-Level Cache is a Hibernate cache shared across persistence contexts. Unlike First-Level Cache, it can reuse cached data between different sessions.

---

## Quick Comparison

|          | First-Level              | Second-Level          |
| -------- | ------------------------ | --------------------- |
| Scope    | Session / EntityManager  | SessionFactory        |
| Shared?  | ❌                        | ✅                     |
| Default? | ✅                        | ❌                     |
| Purpose  | Cache within one session | Cache across sessions |

### Remember

**Flush → Sync with DB**
**1st Level → One Session**
**2nd Level → Multiple Sessions**
