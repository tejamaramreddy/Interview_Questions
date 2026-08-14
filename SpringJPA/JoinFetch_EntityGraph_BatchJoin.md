

## 1. `JOIN FETCH`

### Interview answer
"`JOIN FETCH` is a JPQL feature that allows us to fetch an entity and its associated entities in the same database query. It's commonly used to solve the N+1 query problem."

Example:

```java
@Query("""
    SELECT DISTINCT d
    FROM Department d
    LEFT JOIN FETCH d.employees
""")
List<Department> findAllWithEmployees();
```

Without `JOIN FETCH`, if I load 10 departments and then access their employees, Hibernate might execute:

```text
1 query → departments
10 queries → employees
```

So we get **N+1 queries**.

With `JOIN FETCH`, Hibernate can fetch the departments and employees together, significantly reducing the number of queries.

### When would you use it?

> "I would use `JOIN FETCH` when I know that a particular query needs the associated entities and I want explicit control over the JPQL query."

### Important caveat

> "I wouldn't blindly use `JOIN FETCH` for every relationship because joining large collections can produce a large result set, and multiple collection fetch joins can cause problems."

---

# 2. `@EntityGraph`

### Interview answer

> "`@EntityGraph` is a Spring Data JPA feature that allows us to specify which relationships should be fetched for a particular repository query. It's another way to avoid N+1 without changing the entity's default fetch strategy."

Example:

```java
@EntityGraph(attributePaths = {"employees"})
List<Department> findAll();
```

Instead of writing:

```java
@Query("""
    SELECT DISTINCT d
    FROM Department d
    LEFT JOIN FETCH d.employees
""")
```

we can simply use:

```java
@EntityGraph(attributePaths = "employees")
```

### Why is that useful?

Suppose the entity has:

```java
@OneToMany(
    mappedBy = "department",
    fetch = FetchType.LAZY
)
private List<Employee> employees;
```

We can keep it `LAZY` by default.

But for this particular query:

```java
@EntityGraph(attributePaths = "employees")
List<Department> findAll();
```

we tell Hibernate:

> "For this query, I want employees as well."

### When would you use it?

> "I prefer `@EntityGraph` when I have a straightforward Spring Data repository query and want to control which relationships are fetched without writing custom JPQL."

---

# 3. Batch Fetching

### Interview answer

> "Batch fetching is a Hibernate optimization that reduces the number of queries generated when lazy relationships are accessed. Instead of loading one relationship at a time, Hibernate loads multiple relationships in batches."

For example, without batch fetching:

```text
Department 1 → query employees
Department 2 → query employees
Department 3 → query employees
Department 4 → query employees
...
```

This can result in N+1.

We can configure:

```java
@BatchSize(size = 20)
@OneToMany(mappedBy = "department")
private List<Employee> employees;
```

Now Hibernate can group multiple department IDs into an `IN` query:

```sql
SELECT *
FROM employee
WHERE department_id IN (1, 2, 3, ..., 20);
```

Instead of:

```sql
SELECT * FROM employee WHERE department_id = 1;
SELECT * FROM employee WHERE department_id = 2;
SELECT * FROM employee WHERE department_id = 3;
...
```

### When would you use it?

> "I would use batch fetching when I want to keep relationships lazy but want to reduce the number of queries generated when multiple lazy associations are accessed."

---

# The most important comparison

If the interviewer asks:

**"What's the difference between JOIN FETCH, EntityGraph, and Batch Fetching?"**

You can answer:

> "`JOIN FETCH` and `EntityGraph` are ways of controlling what gets fetched as part of a particular query, while batch fetching is mainly an optimization for lazy loading."
>
> "`JOIN FETCH` gives me explicit JPQL control. `EntityGraph` provides a cleaner way to specify fetch relationships in Spring Data JPA. Batch fetching doesn't necessarily make everything one query; instead, it groups multiple lazy-loading queries into batches."

Then give this example:

```text
                    N+1 Problem
                         │
             ┌───────────┼───────────┐
             ↓           ↓           ↓
        JOIN FETCH   EntityGraph  Batch Fetching
             │           │           │
        One query    Fetch graph    Batch queries
             │           │           │
       Explicit JPQL  Repository    Lazy loading
                     friendly       optimization
```

---

## A strong 30-second interview answer

If they want a **short answer**, say:

> "In Spring JPA, there are several ways to address the N+1 problem. `JOIN FETCH` allows me to explicitly fetch an association using JPQL, for example `LEFT JOIN FETCH d.employees`. `@EntityGraph` provides a cleaner Spring Data JPA way to specify which associations should be fetched for a particular repository method, while keeping the default relationship lazy. Batch fetching is different: it keeps lazy loading but groups multiple lazy-loading operations into batches using Hibernate's `@BatchSize` or `hibernate.default_batch_fetch_size`. I choose between them based on the query and data size rather than making all relationships eager."

### If they ask "Which one do you prefer?"

A good answer:

> "For a simple Spring Data repository query, I generally prefer `@EntityGraph` because it's clean and readable. For a complex query where I need precise JPQL control, I use `JOIN FETCH`. If I specifically want to preserve lazy loading while reducing the number of queries generated across multiple entities, I consider batch fetching."

