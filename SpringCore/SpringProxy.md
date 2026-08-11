# Spring Proxy

## 1. What is a Spring Proxy?

A **Spring Proxy** is an object created by Spring that **wraps a target Spring bean and intercepts method calls**.

The proxy allows Spring to add additional behavior without modifying the actual business logic.

### Common use cases

* AOP
* `@Transactional`
* `@Cacheable`
* `@Async`
* Spring Security
* Logging
* Retry mechanisms

### Simple Definition

> A Spring Proxy is a wrapper around a Spring bean that intercepts method calls and applies additional behavior before delegating the call to the actual target object.

---

# 2. Why Does Spring Use Proxies?

Consider:

```java
@Service
public class PaymentService {

    @Transactional
    public void makePayment() {
        // Business logic
    }
}
```

We don't manually write transaction handling:

```java
beginTransaction();

makePayment();

commit();
```

Instead, Spring creates a proxy around the bean.

### Conceptual flow

```text
Client
   |
   v
Spring Proxy
   |
   v
Transaction Interceptor
   |
   +---- BEGIN TRANSACTION
   |
   v
Target Object
   |
   v
makePayment()
   |
   v
COMMIT / ROLLBACK
```

The proxy allows Spring to execute additional logic **before and/or after the target method**.

---

# 3. Proxy vs Target Object

## Target Object

The target is the actual Spring bean containing the business logic.

```java
@Service
public class UserService {

    public void createUser() {
        System.out.println("Creating user");
    }
}
```

Here:

```text
UserService = Target Object
```

## Proxy Object

Spring can create another object that wraps the target.

```text
        Proxy
          |
          v
    Target Object
     UserService
```

When we write:

```java
UserService service = context.getBean(UserService.class);
```

the returned object may actually be a **proxy** rather than the original target object.

---

# 4. How Does a Proxy Work?

Suppose we have:

```java
@Transactional
public void transferMoney() {
    // Business logic
}
```

Conceptually, the call works like this:

```text
Client
  |
  v
Spring Proxy
  |
  v
Transaction Interceptor
  |
  +---- BEGIN TRANSACTION
  |
  v
Target.transferMoney()
  |
  +---- Business Logic
  |
  v
COMMIT / ROLLBACK
```

### Important

> The proxy intercepts the method call before it reaches the target object.

---

# 5. Types of Spring Proxies

There are two important proxy mechanisms to know:

1. JDK Dynamic Proxy
2. CGLIB/Class-based Proxy

---

# 6. JDK Dynamic Proxy

JDK Dynamic Proxy is **interface-based**.

Example:

```java
public interface UserService {
    void createUser();
}
```

Implementation:

```java
@Service
public class UserServiceImpl implements UserService {

    @Override
    public void createUser() {
        System.out.println("Creating user");
    }
}
```

Conceptually:

```text
Client
  |
  v
JDK Dynamic Proxy
  |
  v
UserServiceImpl
```

### Key Point

> JDK Dynamic Proxy works through interfaces.

---

# 7. CGLIB / Class-Based Proxy

A class-based proxy creates a **subclass of the target class**.

Example:

```java
@Service
public class UserService {

    public void createUser() {
        System.out.println("Creating user");
    }
}
```

Conceptually:

```text
      CGLIB Proxy
           |
           v
      UserService
```

The proxy is effectively a subclass of the target class and can intercept eligible method calls.

### Key Point

> CGLIB-style proxying is class/subclass-based and does not require an interface.

---

# 8. JDK Proxy vs CGLIB Proxy

| JDK Dynamic Proxy                                              | CGLIB/Class-Based Proxy                                     |
| -------------------------------------------------------------- | ----------------------------------------------------------- |
| Interface-based                                                | Class/subclass-based                                        |
| Requires an interface for the proxying approach                | Does not require an interface                               |
| Uses Java's dynamic proxy mechanism                            | Creates a subclass-based proxy                              |
| Proxy implements the interface                                 | Proxy extends the target class                              |
| Cannot proxy a class without an interface using this mechanism | Cannot normally intercept methods that cannot be overridden |

### Easy way to remember

```text
JDK Proxy  → Interface
CGLIB      → Class / Subclass
```

---

# 9. Proxy and AOP

Spring AOP is primarily implemented using proxies.

Example:

```java
@Aspect
@Component
public class LoggingAspect {

    @Before("execution(* com.example.service.*.*(..))")
    public void log() {
        System.out.println("Method called");
    }
}
```

Conceptually:

```text
Client
  |
  v
Proxy
  |
  v
Logging Advice
  |
  v
Target Method
```

### Relationship

```text
AOP
 ↓
Defines cross-cutting behavior
 ↓
Spring creates Proxy
 ↓
Proxy intercepts method
 ↓
Advice executes
 ↓
Target method executes
```

---

# 10. Proxy and @Transactional

`@Transactional` is one of the most important examples of Spring proxy usage.

Example:

```java
@Service
public class PaymentService {

    @Transactional
    public void transferMoney() {
        // Business logic
    }
}
```

Conceptually:

```text
Client
   |
   v
Transaction Proxy
   |
   v
Transaction Interceptor
   |
   +---- BEGIN
   |
   v
transferMoney()
   |
   +---- Business Logic
   |
   v
COMMIT / ROLLBACK
```

The proxy/interceptor is responsible for applying the transaction behavior around the method.

---

# 11. Self-Invocation

Self-invocation is one of the most important Spring Proxy interview topics.

Example:

```java
@Service
public class UserService {

    public void methodA() {
        methodB();
    }

    @Transactional
    public void methodB() {
        // Transactional logic
    }
}
```

Suppose the client calls:

```java
userService.methodA();
```

The flow is:

```text
Client
  |
  v
Proxy
  |
  v
methodA()
  |
  v
this.methodB()
  |
  v
methodB()
```

The call from `methodA()` to `methodB()` happens directly inside the target object.

It does **not** go back through the Spring proxy.

Therefore, the proxy does not get an opportunity to apply the `@Transactional` interceptor to that internal call.

### Interview Definition

> Self-invocation occurs when one method in a Spring bean directly calls another method on the same object. Since the internal call bypasses the Spring proxy, proxy-based features such as `@Transactional` may not be applied to the called method.

---

# 12. Methods That Cannot Normally Be Intercepted

Proxy-based interception has limitations.

Important methods to remember:

```text
private
final
static
```

### Private methods

Private methods cannot be overridden by a subclass proxy.

```java
private void process() {
}
```

### Final methods

Final methods cannot be overridden.

```java
public final void process() {
}
```

Therefore, subclass-based proxying cannot normally intercept them.

### Static methods

Static methods belong to the class rather than an object instance, so normal instance-method proxy interception does not apply.

---

# 13. Checking Whether an Object Is a Proxy

You can inspect the runtime class:

```java
System.out.println(bean.getClass());
```

You might see something similar to:

```text
UserService$$SpringCGLIB$$...
```

Spring also provides:

```java
AopUtils.isAopProxy(bean);
```

Example:

```java
boolean proxy = AopUtils.isAopProxy(bean);

System.out.println(proxy);
```

---

# 14. Multiple Interceptors

A bean can have multiple cross-cutting behaviors.

Example:

```java
@Transactional
@Cacheable
@PreAuthorize("hasRole('ADMIN')")
public User getUser() {
    // Business logic
}
```

Conceptually:

```text
Client
  |
  v
Proxy / Interceptors
  |
  +-- Security
  |
  +-- Transaction
  |
  +-- Cache
  |
  v
Target Method
```

The exact execution order depends on Spring's advisor/interceptor ordering.

---

# 15. Complete Proxy Flow

```text
                  CLIENT
                     |
                     v
              SPRING PROXY
                     |
          +----------+----------+
          |          |          |
          v          v          v
        AOP      Security   Transaction
          |          |          |
          +----------+----------+
                     |
                     v
              TARGET OBJECT
                     |
                     v
              BUSINESS LOGIC
```

---

# 16. Important Interview Questions

### Q1. What is a Spring Proxy?

> A Spring Proxy is a wrapper around a target Spring bean that intercepts method calls and applies cross-cutting behavior such as transactions, AOP, security, and caching.

### Q2. Why does Spring use proxies?

> Spring uses proxies to apply cross-cutting behavior around method execution without modifying the business logic.

### Q3. What are the two main proxy mechanisms?

> JDK Dynamic Proxy and class-based CGLIB-style proxying.

### Q4. What is the difference between JDK Proxy and CGLIB?

> JDK Proxy is interface-based, while CGLIB-style proxying creates a subclass of the target class.

### Q5. Does `@Transactional` use a proxy?

> Yes. In the standard proxy-based Spring transaction model, Spring uses an interceptor through a proxy to apply transaction behavior.

### Q6. What is self-invocation?

> Self-invocation occurs when one method calls another method within the same object. The internal call bypasses the Spring proxy, so proxy-based features such as `@Transactional` may not be triggered.

### Q7. Why can't a final method normally be intercepted by a class-based proxy?

> Because a class-based proxy relies on subclassing and overriding methods, while a final method cannot be overridden.

---

# 17. Key Points to Remember

```text
1. Proxy wraps the target bean.
2. Proxy intercepts method calls.
3. AOP uses proxies in Spring's proxy-based model.
4. @Transactional commonly works through a proxy/interceptor.
5. JDK Proxy → Interface-based.
6. CGLIB → Class/subclass-based.
7. Self-invocation bypasses the proxy.
8. private/final/static methods have proxy-interception limitations.
9. The object returned by Spring may be a proxy.
10. Proxy mechanism is the foundation for understanding @Transactional internals.
```
