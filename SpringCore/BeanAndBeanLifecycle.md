# What is a Bean?

In simple terms, a bean is an object that is instantiated, configured, and managed by the Spring IoC container.

They are defined either through XML configuration, annotations, or Java-based configuration.

The container is responsible for managing the lifecycle of these beans, including their creation, configuration, and destruction.

# Key Characteristics of Beans

* **Singleton by Default:** By default, Spring beans are singleton, meaning that only one instance of a bean is created per Spring IoC container. This instance is shared across the entire application.
* **Configurable:** Beans can be customized and configured through XML, annotations, or Java code.
* **Managed by the Container:** The entire lifecycle of a bean from instantiation to destruction is managed by the Spring IoC container.

# Defining Beans in Spring

There are three primary ways to define beans in Spring:

## XML-Based Configuration

In XML-based configuration, beans are defined in an XML file (usually named `applicationContext.xml`). Each bean is represented by a `<bean>` tag, where you can specify the bean's class, properties, and dependencies.

```java
@Component
public class MyClass {
    // Class implementation
}
```

To enable component scanning, which automatically detects and registers beans annotated with `@Component`, you need to add the `@ComponentScan` annotation in your configuration class.

```java
@Configuration
@ComponentScan(basePackages = "com.example")
public class AppConfig {
    // Configuration code
}
```

## Java-Based Configuration

In Java-based configuration, beans are defined using `@Bean` methods inside a `@Configuration` class. This approach provides a type-safe way of defining beans and is often used in modern Spring applications.

```java
@Configuration
public class AppConfig {

    @Bean
    public MyClass myBean() {
        return new MyClass();
    }
}
```

# Bean Scopes

Spring provides several scopes that determine the lifecycle of a bean:

* **Singleton:** (Default) A single instance per Spring IoC container.
* **Prototype:** A new instance is created each time the bean is requested.
* **Request:** A new instance is created for each HTTP request. This scope is used in web applications.
* **Session:** A new instance is created for each HTTP session.
* **Global Session:** A new instance is created for each global HTTP session. This is typically used in portlet-based applications.
* **Application:** A single instance for the entire web application lifecycle.

You can specify the scope of a bean using the `@Scope` annotation or by configuring it in XML.

```java
@Component
@Scope("prototype")
public class MyClass {
    // Class implementation
}
```
# Bean Life cycle:

# 1. Loading Bean Definitions

* **What Happens:** Spring first reads and understands the definitions of all the beans in the application. These definitions are like blueprints that tell Spring how to create and configure beans.

* **Where Definitions Come From:** These configurations can be provided by different ways as below:

  * **XML Configuration:** In the old days, beans were defined in XML files.
  * **Java Configuration:** With the rise of Java-based configuration, we can define beans using `@Configuration` and `@Bean` annotations.
  * **Annotations:** With annotations like `@Component`, `@Service`, etc., we can let Spring automatically discover and register beans.

* **What Spring Understands:** In this stage, Spring knows the class of each bean, its properties (like name, rollno, emailid in Student class) and its dependencies (other beans it needs).

# 2. Bean Instantiation

* **What Happens:** Once Spring knows what beans it needs to create, it creates instances of those beans. This is the instantiation phase.

* **How Beans Are Created:** Beans can be created by different ways as below:

  * **Default Constructor:** A no-argument constructor is used.
  * **Static Factory Method:** A static method on a class that returns an instance of the bean.
  * **Instance Factory Method:** A non-static method on a separate factory class.

* **Dependency Injection:** During instantiation, dependencies are injected into the bean. This can be done in three ways:

  1. **Constructor Injection:** Dependencies are provided through the constructor.
  2. **Setter Injection:** Dependencies are set using setter methods.
  3. **Property Injection:** Dependencies are set using XML tags or annotations like `@Value` (in Java-based config).

* **Example:** If we configure a Student bean with name, rollno, and emailid, Spring injects the property values (like name Deepak, rollno 102 and email id as [deepak@gmail.com](mailto:deepak@gmail.com) in this [example](https://smartprogramming.in/tutorials/spring-framework/spring-program-using-java-based-configurations.php)) in this phase (e.g., through setter methods or annotations).

# 3. Bean Initialization

* **What Happens:** After a bean is created, it’s initialized by setting its properties to their configured values. This is where any external dependencies, such as values from a configuration file, are injected into the bean.

* **Different Ways to Initialize:** Beans are initialized by different ways as below:

  * **Using init() method:** We can define a custom initialization method in our bean.
  * **Using afterPropertiesSet():** This method from the `InitializingBean` interface is called to perform any necessary initialization.
  * **Using @PostConstruct:** A special annotation that indicates a method to be run after all properties are set.

* **Example:** Here the bean is prepared for use. This includes setting properties (if not already done in instantiation), performing validation, setting up database connections, initializing caches, configuring logging, and executing any custom initialization logic such as loading configuration files or establishing network connections.

* **Additional Functionality:**

  * **Aware Interfaces:** If the bean implements special interfaces, it can access the container or its environment. For example:

    * **BeanNameAware:** Allows the bean to know its own name.
    * **ApplicationContextAware:** Provides access to the Spring ApplicationContext.
    * **BeanFactoryAware:** Gives access to the BeanFactory (for bean creation).

* **Optional Customizations:** At this point, we can also use `BeanPostProcessor` to add custom logic before and after the bean’s initialization. This is useful for advanced scenarios like modifying beans dynamically or adding cross-cutting concerns (e.g., logging or security).

# 4. Bean Usage

* **What Happens:** Now that the bean is initialized, it’s ready to be used. Other beans or components in the Spring application can access and use this bean. The bean is fully functional and serves its purpose in the application.

# 5. Bean Destruction

* **What Happens:** When the application shuts down or the Spring container is closed, Spring cleans up by destroying the beans.

* **How Beans Are Destroyed:**

  * **Using destroy() method:** We can define a custom destruction method in the bean.
  * **Using DisposableBean interface:** This interface provides a `destroy()` method to clean up resources when the bean is no longer needed.
  * **Using @PreDestroy annotation:** This annotation marks a method to be called before the bean is destroyed.

* **Optional Aspects:** If the bean doesn't require any special clean-up, Spring will skip these methods.

---
