## 1. Introduction & Overview

* **** defining and using custom queries with Spring Data JPA.
* **** Key topics covered include executing JPQL and native queries, utilizing Spring Expression Language (SpEL) in queries, and executing write operations.
* **** While derived repository queries are easy for basic needs, custom queries using `@Query` become necessary when handling complex join conditions, dynamic criteria, or multiple parameters.

---

## 2. Defining JPQL Queries with `@Query`

* **** **JPQL Characteristics:** Most developers write JPQL queries based on domain model entities because JPQL is database-agnostic and supports a standard set of persistence features.
* **** **Basic Syntax:** To declare a custom query, annotate a method on your Spring Data repository interface with `@Query` and pass the JPQL query string as the value parameter.
* **** **Projections:** You can query full entity objects or select specific fields/projections if complete entity instances are not required.
* **** Spring Data JPA handles the translation of the JPQL query into SQL and executes it against the underlying database automatically.

---

## 3. Sorting & Pagination

*  Dynamic Sorting Options:
* **Static Sorting:** Include an explicit `ORDER BY` clause directly inside the JPQL query string.
* **Dynamic Sorting:** Pass a `org.springframework.data.domain.Sort` parameter into the repository method signature.


*  Pagination:
* Pass a `Pageable` object as a method parameter.
* Define page parameters using `PageRequest.of(pageNumber, pageSize)`.
* Spring Data JPA automatically generates the database-specific SQL required for paging (e.g., `LIMIT` and `OFFSET`).



---

## 4. Advanced Features & Spring Expression Language (SpEL)

*  SpEL Expressions: You can embed Spring Expression Language within `@Query` definitions.
* **** SpEL helps dynamically evaluate entity names (e.g., `#{#entityName}`) or construct dynamic expressions without hardcoding table/entity names.
* **** Promotes cleaner and more maintainable dynamic queries across shared base repositories.

---

## 5. Native SQL Queries

*  Native Queries: Used when you need database-specific SQL features, complex reporting, or performance optimizations not supported in JPQL.
*  Syntax: Add `nativeQuery = true` to the `@Query` annotation (e.g., `@Query(value = "SELECT * FROM ...", nativeQuery = true)`).
*  Portability: Native queries bypass the JPA provider parser and are sent directly to the database; they must conform strictly to your database vendor's SQL dialect.

---

## 6. Binding Query Parameters

*  Security & Efficiency: Using parameters prevents SQL injection vulnerabilities and allows database query engines to cache and reuse execution plans.
*  Parameter Types:
* **Positional Parameters:** Indicated by `?1`, `?2`, matching the order of arguments in the Java method signature.
* **Named Parameters:** Indicated by `:paramName` (matching `@Param("paramName")` annotations), making queries significantly easier to read and maintain.



---

## 7. Executing Write & Update Operations

*  Modifying Queries: Used for `UPDATE`, `INSERT`, or `DELETE` operations.
*  Key Requirement: You **must** annotate the repository method with `@Modifying` alongside `@Query` so Spring Data JPA knows to execute the query as an update statement rather than a select operation.

---

## 8. Summary & Conclusion

*  Summary: Using `@Query` gives you complete control over query execution while keeping the high-level benefits of Spring Data JPA repositories.
