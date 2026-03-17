# Core Spring & Spring Boot 

### 1. What is Spring Framework?

Spring Framework is a **lightweight Java framework** used for building enterprise applications. It provides features like **Dependency Injection, Aspect-Oriented Programming, and modular architecture**, which make applications easier to develop, test, and maintain.

---

### 2. What is Spring Boot and how is it different from Spring?

Spring Boot is an extension of the Spring Framework that simplifies application setup. Unlike traditional Spring, it provides **auto-configuration, embedded servers like Tomcat, and minimal configuration**, allowing developers to build and run applications quickly.

---

### 3. What are the advantages of using Spring Boot?

Spring Boot enables **rapid development with minimal configuration**. It reduces boilerplate code, includes **embedded servers**, provides **production-ready features like monitoring and metrics**, and integrates easily with databases, security, and microservices.
**The Spring Container** is the core part of the Spring Framework responsible for creating, configuring, and managing the lifecycle of objects called Spring Beans.
---

### 4. What is Dependency Injection in Spring?

Dependency Injection is a design pattern where the **Spring container provides required objects to a class instead of the class creating them itself**. This reduces coupling and improves testability and maintainability.

---

### 5. Difference between @Component, @Service, and @Repository?

All three are **stereotype annotations used for component scanning**.

* `@Component` is a generic Spring bean.
* `@Service` is used for the **service/business layer**.
* `@Repository` is used in the **data access layer** and provides **exception translation for database operations**.

---

### 6. What is ApplicationContext?

ApplicationContext is the **central interface of the Spring container** that manages beans, configuration, and dependency injection. It provides access to application components and manages their lifecycle.
**The Spring Container** is the core part of the Spring Framework responsible for creating, configuring, and managing the lifecycle of objects called Spring Beans.
---

### 7. What is the use of @SpringBootApplication?

`@SpringBootApplication` is a convenience annotation that combines **@Configuration, @EnableAutoConfiguration, and @ComponentScan**, making it easier to configure and run a Spring Boot application.

---

### 8. What is @EnableAutoConfiguration?

`@EnableAutoConfiguration` tells Spring Boot to **automatically configure beans based on the libraries present in the classpath**, reducing the need for manual configuration.

---

### 9. How does Spring Boot decide what to auto-configure?

Spring Boot checks the **classpath and configuration files**, then loads configurations from **spring.factories** using conditional annotations like `@ConditionalOnClass` and `@ConditionalOnMissingBean`.

---

### 10. Role of application.properties or application.yml?

These files store **application configuration values**, such as **server port, database credentials, logging settings, and environment-specific properties**.

---

### 11. What is a Bean in Spring?

A Spring Bean is a **Java object managed by the Spring IoC container**. The container is responsible for creating, configuring, and managing the lifecycle of these objects.

---

### 12. How do you define a Bean?

A bean can be defined using **annotations like @Component, @Service, or @Repository**, or by using the **@Bean annotation inside a @Configuration class**.

---

### 13. What is the lifecycle of a Spring Bean?

The Spring bean lifecycle includes **instantiation, dependency injection, initialization, usage, and destruction**. Initialization and destruction can be customized using annotations like `@PostConstruct` and `@PreDestroy`.

---

### 14. What is @Qualifier used for?

`@Qualifier` is used to **resolve ambiguity when multiple beans of the same type exist**. It allows Spring to inject the correct bean by specifying its name.

---

### 15. Difference between Singleton and Prototype scope?

* **Singleton**: Only **one instance of the bean** is created per Spring container.
* **Prototype**: A **new bean instance is created every time** it is requested.

---

### 16. What is the use of @Value annotation?

`@Value` is used to **inject values from configuration files or environment variables** into Spring beans.

---

### 17. What is Spring AOP?

Spring AOP stands for **Aspect-Oriented Programming**. It allows developers to separate **cross-cutting concerns** like logging, security, and transaction management from the business logic.

---

### 18. What are the key modules in Spring?

Spring includes several modules such as **Core, Context, AOP, JDBC, ORM, Web, and Security**, which provide different functionalities for building enterprise applications.

---

### 19. What is circular dependency and how does Spring handle it?

Circular dependency occurs when **two or more beans depend on each other**. Spring can resolve it using **setter injection or lazy initialization**, but it may fail with constructor injection.

---

### 20. How do you run a Spring Boot application?

A Spring Boot application can be run using the **main method with SpringApplication.run()**, or from the command line using **mvn spring-boot:run** or **gradlew bootRun**.

---


# Spring Boot Annotations & Configurations 

### 1. What is @Configuration?

`@Configuration` marks a class as a **source of Spring bean definitions**. It tells the Spring container that the class contains methods annotated with `@Bean`, which should be managed as beans in the application context.

---

### 2. What is @Bean?

`@Bean` is used inside a `@Configuration` class to **explicitly define a Spring-managed bean**. It allows developers to create and configure objects manually when automatic component scanning is not suitable.

---

### 3. What is @ComponentScan?

`@ComponentScan` tells Spring **where to search for components such as @Component, @Service, @Repository, and @Controller**. Spring automatically detects these classes and registers them as beans in the container.

---

### 4. What is @Component?

`@Component` is a **generic annotation used to declare a class as a Spring-managed bean**. When component scanning is enabled, Spring automatically detects and registers the class in the application context.

---

### 5. Difference between @Component and @Bean?

`@Component` is used on **class level** for automatic detection through component scanning.
`@Bean` is used on **method level inside a configuration class** to manually define a bean.

---

### 6. What is @RestController?

`@RestController` is used to create **RESTful web services**. It combines `@Controller` and `@ResponseBody`, meaning the methods return **JSON or XML responses directly** instead of rendering views.

---

### 7. What is @RequestMapping?

`@RequestMapping` is used to **map HTTP requests to specific handler methods** in a controller. It can define the URL path, HTTP method, and other request conditions.

---

### 8. Difference between @GetMapping and @RequestMapping(method = GET)?

`@GetMapping` is a **shortcut annotation for @RequestMapping(method = RequestMethod.GET)**. It makes the code cleaner and easier to read when handling GET requests.

---

### 9. What is @Autowired?

`@Autowired` is used for **automatic dependency injection**. Spring automatically finds and injects the required bean based on its type.

---

### 10. Constructor-based vs Field-based Injection?

Constructor-based injection is **recommended** because it supports **immutability and easier unit testing**. Field injection is simpler but makes testing and dependency management harder.

---

### 11. What is @ConditionalOnProperty?

`@ConditionalOnProperty` is used to **create or enable a bean only if a specific property exists or has a certain value** in the configuration file.

---

### 12. What is @Profile?

`@Profile` allows beans to be **loaded only for specific environments**, such as *dev, test, or prod*. This helps manage different configurations for different environments.

---

### 13. How do you set active profiles in Spring Boot?

Active profiles can be set using **application.properties**, **command-line arguments**, or **environment variables** like `spring.profiles.active=dev`.

---

### 14. What is @PropertySource?

`@PropertySource` is used to **load external property files into the Spring environment**, allowing the application to access configuration values.

---

### 15. What is @ConfigurationProperties?

`@ConfigurationProperties` binds **configuration values from application.properties or application.yml directly to a Java class (POJO)**, making configuration management cleaner and type-safe.

---

### 16. What is @EnableConfigurationProperties?

`@EnableConfigurationProperties` enables support for **@ConfigurationProperties classes** so Spring can detect and bind configuration values to them.

---

### 17. What is @EnableScheduling?

`@EnableScheduling` enables **scheduled task execution** in a Spring Boot application, allowing methods annotated with `@Scheduled` to run automatically.

---

### 18. What is @Scheduled?

`@Scheduled` is used to **run methods automatically at fixed intervals or based on cron expressions** for background tasks like cleanup or reporting.

---

### 19. What is @EnableAsync?

`@EnableAsync` enables **asynchronous method execution** in a Spring application so that methods annotated with `@Async` run in separate threads.

---

### 20. What is @Async?

`@Async` allows a method to **run asynchronously in a separate thread**, improving performance by allowing tasks to run in parallel.

---

Here are **clear ~30-second interview answers** for the **Spring Data JPA & Transactions** questions.

---

# Spring Data JPA & Transactions

### 1. What is Spring Data JPA?

**Spring Data JPA** is a module of the Spring ecosystem that simplifies database access using **Java Persistence API**. It reduces boilerplate code by automatically implementing repository interfaces and providing built-in CRUD operations, pagination, and query generation.

---

### 2. What is JpaRepository?

**JpaRepository** is a repository interface that provides built-in methods for **CRUD operations, pagination, and sorting**. Developers just extend this interface, and Spring automatically provides the implementation.

---

### 3. Difference between CrudRepository and JpaRepository?

**CrudRepository** provides basic CRUD operations like save, find, and delete.
**JpaRepository** extends it and adds additional features like **pagination, sorting, batch operations, and JPA-specific methods**.

---

### 4. How do you define a custom query in Spring Data JPA?

Custom queries can be defined using the **@Query annotation** in repository interfaces. The query can be written using **JPQL or native SQL**, and parameters can be passed using `@Param`.

---

### 5. What is the use of @Entity?

`@Entity` marks a Java class as a **JPA entity**, meaning it will be mapped to a **database table**. Each instance of the class represents a row in the table.

---

### 6. What is @Id?

`@Id` is used to **define the primary key** of an entity. It uniquely identifies each record in the database table.

---

### 7. What is @GeneratedValue?

`@GeneratedValue` defines **how the primary key value is generated automatically**, such as `AUTO`, `IDENTITY`, `SEQUENCE`, or `TABLE`.

---

### 8. What is the N+1 select problem in JPA?

The **N+1 select problem** occurs when fetching related entities lazily, causing **one query for the main entity and multiple additional queries for each related entity**, which leads to performance issues.

---

### 9. How to solve the N+1 problem?

It can be solved using **fetch joins, @EntityGraph, or batch fetching** so that related entities are loaded efficiently in fewer queries.

---

### 10. Default fetch type for @OneToMany and @ManyToOne?

* `@OneToMany` → **LAZY loading by default**
* `@ManyToOne` → **EAGER loading by default**

This means one-to-many relationships are usually loaded only when needed.

---

### 11. What is @Transactional?

`@Transactional` is used to **define transaction boundaries declaratively**. It ensures that a set of database operations either **complete successfully together or roll back if an error occurs**.

---

### 12. Where should @Transactional be placed?

`@Transactional` is typically placed on **service layer methods**, because the service layer usually handles business logic and database operations.

---

### 13. Difference between @Transactional(readOnly = true) and normal?

`readOnly = true` indicates that the method performs **only read operations**. This allows the database and persistence provider to **optimize performance** and avoid unnecessary write checks.

---

### 14. Can @Transactional be used on private methods?

No. `@Transactional` works only on **public methods** because Spring uses **proxy-based AOP**, which cannot intercept private methods.

---

### 15. How do you handle transactions in nested service calls?

Nested transactions can be handled using **propagation settings in @Transactional**, which control how a transaction behaves when a method is called within another transaction.

---

### 16. What is propagation in Spring transactions?

**Transaction propagation** defines **how a transaction behaves when one transactional method calls another**. Examples include `REQUIRED`, `REQUIRES_NEW`, and `SUPPORTS`.

---

### 17. What is isolation in transactions?

**Isolation level** defines how transactions interact with each other in concurrent environments, controlling issues like **dirty reads, non-repeatable reads, and phantom reads**.

---

### 18. How do you handle optimistic locking in JPA?

Optimistic locking is implemented using the **@Version annotation**. It adds a version column to detect concurrent updates and prevent overwriting changes made by another transaction.

---

### 19. How do you implement pagination in Spring Data JPA?

Pagination is implemented using the **Pageable interface**, and repository methods return a **Page<T>** object that contains both the data and pagination metadata.

---

### 20. How do you audit entities in JPA?

Entity auditing can be implemented using annotations like **@CreatedDate and @LastModifiedDate** along with **@EntityListeners** to automatically track creation and update timestamps.

---
