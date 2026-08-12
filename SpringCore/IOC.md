# Why the term “Inversion of Control”?

In a traditional application, developers have to manually manage object creation and their dependencies, which can lead to tightly coupled code that's hard to maintain and scale.

Spring IoC inverts this control by handing over the responsibility of object creation and dependency management to the framework itself. This is where the term "Inversion of Control" comes from developers no longer control the flow of their application directly; instead, they rely on the IoC Container to manage the application's components.

# What Spring IOC container does?

The IoC Container is responsible for creating objects, injecting dependencies, and managing the entire lifecycle of these objects. This process, known as Dependency Injection (DI), decouples the application's components, allowing for more flexible and maintainable code.

# From where does the IOC container gets the information of objects?

The IoC Container uses various configuration methods XML, Java annotations, and Java code to understand the objects that make up the application. These objects, known as Beans, are then managed by the container. Whether it's creating a new instance of a class, setting its properties, or handling its destruction, the IoC Container takes care of everything, freeing developers to focus on the application's business logic.

# Benefits of IoC

* **Decoupling:** IoC reduces the dependency between objects, making the application easier to manage and modify.
* **Testability:** By decoupling dependencies, IoC makes it easier to mock and test components in isolation.
* **Modularity:** IoC encourages a modular design, where individual components are focused on specific tasks and are loosely coupled.
* **Maintainability:** Applications built using IoC are generally easier to maintain and extend, as changes in one part of the system are less likely to impact others.

# How IOC Works in Spring

1. **Configuration:** The Spring IoC container is configured using XML, Java annotations, or Java-based configuration classes.
2. **Bean Creation:** The container reads the configuration and creates the required beans.
3. **Dependency Injection:** The container injects the necessary dependencies into the beans.
4. **Bean Management:** The container manages the lifecycle of the beans, including initialization and destruction.

## Example

```java
public class Engine {
    public void start() {
        System.out.println("Engine started");
    }
}

public class Car {
    private Engine engine;

    // Constructor Injection
    public Car(Engine engine) {
        this.engine = engine;
    }

    public void drive() {
        engine.start();
        System.out.println("Car is driving");
    }
}

// Configuration using Java Annotations
@Configuration
public class AppConfig {

    @Bean
    public Engine engine() {
        return new Engine();
    }

    @Bean
    public Car car() {
        return new Car(engine());
    }
}

// Running the Spring Application
public class SpringIoCExample {
    public static void main(String[] args) {
        ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
        Car car = context.getBean(Car.class);
        car.drive();
    }
}
```
