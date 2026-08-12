# AOP in Spring https://dev.to/sadiul_hakim/spring-boot-aop-programming-591p

AOP in Spring is used to handle cross-cutting concerns separately from business logic.

It uses concepts such as **aspects, advice, pointcuts, and proxies**.

Common use cases are logging, transactions, security, caching, and performance monitoring.

Spring AOP is primarily proxy-based and intercepts method calls on Spring-managed beans.

---

# How does AOP work in Spring?

Spring AOP uses **proxies (dynamic proxies or CGLIB)** to wrap beans and intercept method calls.

When a method on a bean is called, Spring checks if any advice (special code) should run before, after, or around that method.

The advice is executed, and then the target method runs.

This process is called **weaving**. In Spring, weaving is done at runtime (no need to modify class files manually).

---

# Key AOP Terminologies

| Term              | Meaning                                                                                        |
| ----------------- | ---------------------------------------------------------------------------------------------- |
| **Aspect**        | A class that contains cross-cutting logic (e.g., logging aspect).                              |
| **Join Point**    | A point in program execution (method call, exception thrown, etc.) where you can apply advice. |
| **Advice**        | The actual action taken at a join point (before, after, around).                               |
| **Pointcut**      | An expression that selects join points (e.g., all methods in a package).                       |
| **Weaving**       | Linking aspects with application code (done at runtime in Spring).                             |
| **Target Object** | The object whose method is being advised.                                                      |
| **Proxy**         | The object created by Spring AOP that wraps the target object.                                 |

---

# What is a Join Point?

A Join Point is a specific point during execution of your program where AOP code can be applied.

In Spring AOP, a join point is always a **method execution** (not field access, constructor calls, etc. — those are supported only in full AspectJ, not Spring AOP).

## Examples of Join Points

* When a method starts (`@Before`)
* When a method returns (`@AfterReturning`)
* When a method throws exception (`@AfterThrowing`)
* When a method completes (`@After`)
* Wrapping a method (`@Around`)

---

# What is a Pointcut?

A pointcut is an expression that tells Spring **where (at which JoinPoints) an advice should be applied**.

* **A JoinPoint** = any method execution in Spring-managed beans.
* **A Pointcut** = a filter that selects certain join points (based on method name, package, annotations, etc.).

So:

> **Advice = What to do, and Pointcut = Where to do it.**

---

# How to Define a Pointcut?

In Spring AOP, you can:

### Inline Pointcut Expression

Directly inside `@Before`, `@After`, etc.

### Reusable Pointcut Method

Define it once with `@Pointcut` and reuse it across advices.
