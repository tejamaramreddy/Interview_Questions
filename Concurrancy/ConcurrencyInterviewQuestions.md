These are **25 foundational interview questions** covering basic concurrency concepts, key keywords, memory model basics, thread pools, and modern Java features.

---

### Section 1: Fundamentals, `synchronized`, and `volatile`

1. **What is the difference between a Process and a Thread in Java?**
2. **How do you create and start a thread in Java using `Thread` vs `Runnable`?**
3. **What are the key states in a Java Thread Lifecycle (e.g., `Thread.State`)?**
4. **What is the `synchronized` keyword in Java, and what problem does it solve?**
5. **What is the difference between synchronizing a static method vs. a non-static method?**
6. **What is intrinsic locking (or monitor locking) in Java?**
7. **What is the `volatile` keyword, and how does it differ from `synchronized`?**
8. **Does the `volatile` keyword guarantee atomicity for compound operations like `i++`? Why or why not?**

---

### Section 2: Java Memory Model (JMM) & Happens-Before

9. **What is the primary purpose of the Java Memory Model (JMM)?**
10. **What is instruction reordering by the compiler or CPU, and why can it cause issues in multithreaded applications?**
11. **What does the term "Happens-Before relationship" mean in the JMM?**
12. **Name two built-in rules that establish a Happens-Before edge in Java (e.g., thread start, volatile write).**

---

### Section 3: Explicit Locks (`java.util.concurrent.locks`)

13. **What is a `ReentrantLock`, and how does it differ from a standard `synchronized` block?**
14. **Why should you always call `lock.unlock()` inside a `finally` block?**
15. **What is the difference between a fair lock and an unfair lock in `ReentrantLock`?**
16. **What is `ReentrantReadWriteLock`, and in what scenarios is it preferred over a standard lock?**

---

### Section 4: ExecutorService, CompletableFuture, and ForkJoinPool

17. **What is the main benefit of using `ExecutorService` over creating raw `Thread` objects manually?**
18. **What is the difference between `executorService.submit()` and `executorService.execute()`?**
19. **What is the difference between `Callable` and `Runnable` in the concurrency framework?**
20. **What is `CompletableFuture`, and how does it improve upon standard `Future` objects?**
21. **What is `ForkJoinPool`, and what core problem is it designed to solve?**
22. **What is "Work-Stealing" in the context of `ForkJoinPool`?**

---

### Section 5: Modern Java (Virtual Threads & Structured Concurrency)

23. **What is a Virtual Thread (introduced in Java 21 via Project Loom), and how does it differ from a platform (OS) thread?**
24. **Why are Virtual Threads particularly well-suited for I/O-bound tasks rather than CPU-bound tasks?**
25. **What is Structured Concurrency, and how does it treat subtasks launched in parallel as a single unit of work?**

---

Here are **25 intermediate and scenario-based interview questions** designed to test your ability to diagnose concurrency bugs, choose the right execution abstractions, optimize performance, and leverage modern Java concurrency primitives.

---

### Section 1: Memory Model, Volatile, & Thread Safety Scenarios

1. **Scenario (Double-Checked Locking):** A developer writes the following singleton implementation:
```java
public class CacheManager {
    private static CacheManager instance;
    public static CacheManager getInstance() {
        if (instance == null) {
            synchronized (CacheManager.class) {
                if (instance == null) {
                    instance = new CacheManager();
                }
            }
        }
        return instance;
    }
}

```


What subtle thread-safety bug exists in this code under the Java Memory Model, and how does declaring `instance` as `volatile` fix it?
2. **Scenario (Visibility vs Atomicity):** You have a high-throughput metrics counter initialized as `private volatile long requestCount = 0;`. Multiple HTTP handler threads increment this variable via `requestCount++`. Under load, the reported count is lower than actual total requests. Why does `volatile` fail here, and what `java.util.concurrent.atomic` class should be used instead?
3. **Scenario (Publishing & Escaping):** A developer initializes an `ArrayList` in a constructor and starts a worker thread, passing `this` reference before the constructor finishes. What is "unsafe publication," and what concurrency hazards does it introduce?
4. **Scenario (Read-Heavy Cache):** You are building an in-memory routing cache that is updated once a day but read thousands of times per second across 100 concurrent threads. Compare using `synchronized HashMap`, `ConcurrentHashMap`, `ReentrantReadWriteLock`, and `CopyOnWriteArrayList/Set`. Which is optimal and why?
5. **Scenario (Lock Stripping):** How does `ConcurrentHashMap` achieve high write concurrency without locking the entire map? Explain the evolution from Lock Stripping (Segments in Java 7) to Synchronized Bins & CAS operations (Java 8+).

---

### Section 2: Explicit Locks & Thread Synchronization

6. **Scenario (Deadlock Prevention):** Thread A acquires `lock1` and attempts to acquire `lock2`. Simultaneously, Thread B acquires `lock2` and attempts to acquire `lock1`, resulting in a deadlock. How can you refactor this code using `ReentrantLock.tryLock()` with a timeout to recover gracefully?
7. **Scenario (Producer-Consumer Coordination):** You are implementing a custom bounded buffer without using standard JDK collections. Explain how to use `ReentrantLock` with two distinct `Condition` instances (`notFull` and `notEmpty`) to signal waiting producer and consumer threads efficiently. Why is this better than standard `wait()`/`notifyAll()` on a single monitor?
8. **Scenario (Reentrancy & Fairness):** What makes a lock "reentrant"? If Thread A holds a `ReentrantLock(true)` (fair lock) and Thread B calls `lock()`, what determines execution order? What is the throughput performance penalty of fair locks vs unfair locks under high contention?
9. **Scenario (ReadWriteLock Downgrading):** Can a thread holding a Write Lock in `ReentrantReadWriteLock` acquire a Read Lock before releasing the Write Lock (lock downgrading)? Can it upgrade from a Read Lock to a Write Lock? What happens if two reading threads attempt to upgrade simultaneously?
10. **Scenario (Optimistic Reading):** How does `StampedLock` improve upon `ReentrantReadWriteLock` for read-heavy workloads using optimistic validation (`tryOptimisticRead()`)? What is the main drawback regarding reentrancy?

---

### Section 3: Thread Pools & Resource Tuning

11. **Scenario (ThreadPool Saturation & Rejection):** You configure a `ThreadPoolExecutor` with `corePoolSize = 5`, `maximumPoolSize = 10`, and an unbounded `LinkedBlockingQueue`. When 1,000 requests hit the system simultaneously, how many threads will be created? Why will `maximumPoolSize` never be reached?
12. **Scenario (Memory Leak via Unbounded Queues):** An API gateway offloads log processing to an `ExecutorService` using `Executors.newFixedThreadPool(10)`. During an upstream service outage, the application crashes with `java.lang.OutOfMemoryError: Java heap space`. What caused this, and how should the pool and queue be reconfigured?
13. **Scenario (Graceful Shutdown Failure):** An application calls `executorService.shutdown()` during server teardown, but the JVM process hangs indefinitely and fails to exit. What could cause worker threads to remain alive, and how does `awaitTermination` combined with `shutdownNow()` resolve it?
14. **Scenario (Thread Local Leaks):** In a web application running on a thread pool, developers use `ThreadLocal<UserSession>` to hold context. Users occasionally see another user's profile data after logging in. What causes this state bleed across HTTP requests, and where should `.remove()` be invoked?
15. **Scenario (CallerRunsPolicy Backpressure):** You configure a `ThreadPoolExecutor` with a bounded queue and `CallerRunsPolicy`. What happens to the HTTP caller thread when the queue fills up, and how does this serve as a natural rate-limiting (backpressure) mechanism?

---

### Section 4: Asynchronous Pipelines with `CompletableFuture`

16. **Scenario (Exception Recovery in Pipelines):** You have a pipeline: `fetchUserAsync() -> fetchOrdersAsync() -> formatInvoice()`. If `fetchOrdersAsync()` throws an `OrderNotFoundException`, how do you catch it using `.exceptionally()` or `.handle()` to return an empty order list without breaking downstream invoice formatting?
17. **Scenario (Scatter-Gather Fan-Out):** You need to call 5 independent vendor pricing APIs concurrently and aggregate all valid responses within a strict 500ms SLA. How do you implement this using `CompletableFuture.allOf()`, `orTimeout()`, and custom thread pools?
18. **Scenario (Thread Context Switch in Chains):** Given the following code:
```java
CompletableFuture.supplyAsync(() -> fetchFromDb(), dbExecutor)
                 .thenApply(user -> formatUser(user))
                 .thenAcceptAsync(formatted -> sendEmail(formatted), emailExecutor);

```


Which thread executes `formatUser(user)`? What determines whether it runs on the `dbExecutor` thread or the calling main thread?
19. **Scenario (CompletableFuture vs ForkJoinPool):** Why is calling `CompletableFuture.supplyAsync(() -> blockingRestCall())` without passing an explicit `Executor` dangerous in a production web service? How does it affect `ForkJoinPool.commonPool()` and other parallel operations in the JVM?

---

### Section 5: ForkJoinPool, Virtual Threads, & Structured Concurrency

20. **Scenario (ForkJoinPool Join Deadlock):** A developer implements a recursive file search using `ForkJoinPool`. Inside the `compute()` method, subtasks call `.join()` sequentially before calling `.fork()`. Why does this destroy parallelism, and what is the correct order of operations (`fork()` vs `compute()`/`join()`)?
21. **Scenario (Virtual Thread Pinning):** You migrate an application to Virtual Threads (`Executors.newVirtualThreadPerTaskExecutor()`). Under load, throughput drops sharply because platform (carrier) threads are blocked from unmounting. What causes Virtual Thread "pinning" (e.g., `synchronized` blocks or native methods), and how do you replace it with `ReentrantLock`?
22. **Scenario (CPU-Bound vs I/O-Bound Scaling):** You have a microservice that performs heavy AES-256 encryption (CPU-bound) and another that fetches records from Postgres over TCP (I/O-bound). Why are Virtual Threads highly effective for the Postgres service but provide zero performance gain (or slight degradation) for the encryption service?
23. **Scenario (Carrier Thread Pool Exhaustion):** Virtual Threads are mounted onto Carrier Threads (a underlying `ForkJoinPool`). What is the default size of this Carrier Pool? What happens when all carrier threads become pinned by legacy synchronous libraries?
24. **Scenario (Structured Concurrency Failure Handling):** Using Java 21+ `StructuredTaskScope.ShutdownOnFailure`, you spawn two concurrent tasks: `fetchFlight()` and `fetchHotel()`. If `fetchFlight()` throws an exception, what automatically happens to the `fetchHotel()` task, and how does this prevent "thread leaks" or orphan background operations?
25. **Scenario (Structured Concurrency Short-Circuit):** How does `StructuredTaskScope.ShutdownOnSuccess` differ from `ShutdownOnFailure` when querying 3 redundant DNS servers concurrently to get the fastest response?

---