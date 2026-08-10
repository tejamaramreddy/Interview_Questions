# What is Dependency Injection?

Dependency Injection (DI) is a design pattern in which an object’s dependencies are provided to it from an external source, rather than the object creating them itself. In other words, instead of a class being responsible for instantiating its dependencies, those dependencies are injected into the class by an external entity, such as a framework or container.

In Spring, DI can be implemented in three primary ways:

## 1. Constructor Injection

In constructor injection, dependencies are provided through a class’s constructor. This approach ensures that the class is always initialized with its required dependencies.

```java
@Component
public class A {
    private final B b;

    @Autowired
    public A(B b) {
        this.b = b;
    }

    public void doSomething() {
        b.performTask();
    }
}
```

## 2. Setter Injection

In setter injection, dependencies are provided through setter methods. This approach allows dependencies to be set or changed after the object is created.

```java
@Component
public class A {
    private B b;

    @Autowired
    public void setB(B b) {
        this.b = b;
    }

    public void doSomething() {
        b.performTask();
    }
}
```

## 3. Field Injection

In field injection, dependencies are injected directly into the fields of a class using annotations. While this approach is concise, it is generally considered less flexible than constructor or setter injection.

```java
@Component
public class A {
    @Autowired
    private B b;

    public void doSomething() {
        b.performTask();
    }
}
```

# Benefits of Dependency Injection

## 1. Loose Coupling

With DI, classes are not responsible for creating their dependencies. This leads to loose coupling, making the code more modular and easier to maintain.

## 2. Improved Testability

DI allows for easier testing because dependencies can be easily mocked or replaced with stubs during testing, without changing the code of the class being tested.

## 3. Enhanced Flexibility

Because dependencies are injected externally, it’s easy to swap out implementations. For example, you can inject a different implementation of a service or repository without changing the class that uses it.

## 4. Simplified Object Management

The Spring IoC container takes care of creating and managing the lifecycle of beans (objects), reducing the complexity of managing object graphs in large applications.
