# Microservices Architecture Interview Questions

## Core Concepts & Monolith vs Microservices

1. What are the primary trade-offs when choosing between a monolith and microservices?
2. What are the primary characteristics of a microservices architecture compared to a monolithic one?
3. How does independent scaling of individual services work, and what advantage does it give over scaling a monolith?
4. When would you recommend against using microservices for a new project?
5. What is the fundamental difference between Service-Oriented Architecture (SOA) and the modern microservices architectural style?
6. What are the primary drivers for moving from a monolith to microservices, and conversely, when is a monolith the better choice for a team?
7. How do microservices differ from a serverless (Function-as-a-Service) architecture, and when would you choose one over the other?
8. What does the 'you build it, you run it' ownership model mean, and how does it change how teams operate microservices?
9. Why is debugging a distributed microservices system fundamentally harder than debugging a monolith?
10. What is the 'operational tax' of microservices, and what added complexities arise in testing, deployment, and debugging compared to a monolithic system?
11. What are the fallacies of distributed computing, and how do they apply to microservices?

---

## Service Decomposition & Boundaries

12. In the context of Domain-Driven Design, what is a 'Bounded Context' and how does it relate to microservice boundaries?
13. Explain Conway's Law and how it influences the design and success of a microservices architecture.
14. What are the trade-offs of sharing code between services via a shared library versus duplicating it?
15. How do you decide whether a piece of functionality should be a new microservice or added to an existing one?
16. How do you determine the right size for a service, and what are the dangers of making a service too small (nanoservices)?
17. How do you decide where to draw the boundaries between services, and what is the difference between decomposing by business capability vs. by sub-domain?
18. Explain the relationship between high cohesion and loose coupling in the context of microservices, and how do you measure them?
19. What is the difference between decomposing by business capability versus decomposing by sub-domain?
20. How do you handle shared logic or common code across multiple services without creating tight coupling?
21. What is an Anti-Corruption Layer in Domain-Driven Design, and when would you use one between services?

---

## Inter-Service Communication

22. What are the trade-offs between synchronous (`REST` / `gRPC`) and asynchronous (message-driven) communication, and when would you choose one over the other?
23. When would you choose `gRPC` over `REST` for internal microservices communication?
24. Explain the publish/subscribe pattern and how it enables loose coupling between microservices.
25. What is the difference between point-to-point messaging and a broker-based/pub-sub model in inter-service communication?
26. How is load balancing performed across multiple instances of a service, and how does it interact with service discovery?
27. Explain the difference between service orchestration and service choreography, and which is more scalable and why.

---

## API Gateway & Service Discovery

28. What is the role of an `API Gateway`, and how does it differ from a load balancer?
29. How does containerization fit the microservices model, and why is it such a natural pairing?
30. What is the role of a container orchestrator in running microservices, conceptually?
31. What is the "Backend-for-Frontend" (BFF) pattern, and what problem does it solve for mobile vs. web clients?
32. Explain the "Sidecar" pattern and how it is used to offload cross-cutting concerns.
33. What is "Service Discovery," and why is it needed in dynamic, cloud-native environments?
34. How do microservices find and communicate with each other in a dynamic environment, and what is the difference between client-side and server-side discovery?
35. What is the role of an API Gateway, and how does the 'Backend for Frontend' (BFF) pattern differ from a generic gateway?
36. What is the difference between an API Gateway aggregating requests and a service directly calling multiple downstream services?
37. What is a service registry, and how does registration and health-based deregistration work?
38. What is a Service Mesh (including the concept of a sidecar), and how does its purpose differ from an `API Gateway`?

---

## Resilience & Fault Tolerance

39. What is the difference between a retry and a fallback strategy?
40. Explain the 'Circuit Breaker' pattern: what are its three states and why is it used?
41. What does it mean for a service to be "Idempotent," and why is this critical in an event-driven microservices system?
42. What is 'Graceful Degradation' and can you give an example in a microservices context?
43. How do you implement "Retry with Exponential Backoff and Jitter," and why is jitter important?
44. What is rate limiting and throttling, and why is it an important protective pattern in a microservices system?
45. Why is setting appropriate timeouts critical in inter-service calls, and what happens if you rely on default timeouts?
46. Explain the "Bulkhead" pattern and how it prevents a single service failure from taking down the entire system.
47. What is cascading failure, and how do you prevent it in a distributed system?
48. What is 'Backpressure' and why is it important in an event-driven microservices system?
49. How do you prevent a 'Retry Storm' when a downstream service is struggling?
50. Explain the difference between a 'Retry with Backoff' and a 'Bulkhead' pattern, and when you would use one over the other.
51. How do you prevent a single slow downstream service from taking down your entire system?

---

## Data Management & Consistency

52. What is polyglot persistence, and what are its advantages and disadvantages?
53. How do you explain eventual consistency to a business stakeholder who expects immediate data updates?
54. Explain the concept of eventual consistency. In what business scenarios is it unacceptable?
55. What is Command Query Responsibility Segregation (`CQRS`), and in what scenarios does it become necessary in a microservices environment?
56. How do you query or join data spread across three different microservices with three different databases, comparing `API Composition` versus `CQRS`?
57. Why is the `database-per-service` pattern recommended, and what are the challenges when you need to join or query across data owned by different services?
58. What is event sourcing, and how does it relate to microservices and `CQRS`?
59. How do you keep duplicated data in sync across services when each service owns its own copy?
60. What is the API Composition pattern, and what are its limitations compared to `CQRS` for cross-service queries?
61. How would you approach caching within a microservices architecture, and what are the pitfalls of shared cache state?

---

## Distributed Transactions & Messaging Patterns

62. What is a compensating transaction and how does it differ from a traditional database rollback?
63. Why is two-phase commit (`2PC`) generally avoided in microservices, and what problems does it introduce?
64. What is the 'Transactional Outbox' pattern, and how does it solve the problem of atomically updating a database and sending a message to a broker?
65. Explain the difference between Orchestration and Choreography in a Saga and the trade-offs of each.
66. Explain how the Saga pattern manages distributed transactions and how you handle a failure in the middle of a multi-service workflow with compensating transactions.
67. How do you handle distributed transactions across multiple services without using two-phase commit?
68. How do you handle "Eventual Consistency" in a system where a user expects immediate feedback?
69. What is the inbox pattern, and how does it complement the outbox pattern for reliable message processing?
70. How do you evolve event schemas over time without breaking downstream consumers in an event-driven system?
71. How does the `CAP` theorem force design decisions like eventual consistency and compensating transactions in microservices?

---

## Observability & Monitoring

72. How do you track a single user request as it travels through ten different microservices, and what is a `Correlation ID`?
73. Why is centralized logging more important in microservices than in a monolith?
74. What are health checks, and how do they support self-healing in an orchestrated microservices environment?
75. What is the difference between a liveness check and a readiness check, and why does an orchestrator need both?
76. What are the "Three Pillars of Observability" (Metrics, Logs, Traces) and how do they apply to microservices?
77. What is the difference between "Log Aggregation" and "Distributed Tracing"?
78. Why are logs alone insufficient in microservices, and what are the roles of metrics and distributed tracing in maintaining system health?
79. What metrics would you monitor to understand the health and performance of a microservices system?

---

## Security

80. How do you handle authentication and authorization across services, and how is a `JWT` typically propagated from the gateway to downstream services?
81. What is `mTLS` (Mutual `TLS`), and why is it used for service-to-service communication?
82. What is the zero trust security model in the context of microservices?
83. How do you manage secret management (API keys, DB credentials) in a system with hundreds of services?
84. Why does a microservices architecture increase the attack surface, and how do you mitigate the added security risk?

---

## Deployment & Configuration

85. What is the strangler fig pattern, and how is it used to migrate a monolith to microservices?
86. Why is the shared database considered an anti-pattern in microservices, and what are the risks of ignoring this rule?
87. What is the difference between Blue-Green deployment and Canary deployment in a microservices context?
88. What does it mean for a service to be 'independently deployable,' and what happens to the architecture if this requirement is violated?
89. How do you manage externalized configuration across many services, and what are the trade-offs of centralized versus per-service configuration?
90. What role do feature flags/toggles play in safely deploying and releasing microservices?
91. What is a 'distributed monolith,' how does it happen, and why is it considered an anti-pattern?
92. How do you handle breaking changes in a service's API when multiple other services depend on it, using strategies like semantic versioning or parallel versioning?
93. Which of the 12 factors are most critical for ensuring a microservice is truly cloud-native and independently deployable?

## Testing

94. What is consumer-driven contract (CDC) testing, and why is it preferred over traditional integration testing for microservices?
95. What does the testing pyramid look like for microservices, and why do end-to-end tests become problematic at scale?
96. How do integration and end-to-end testing challenges differ in microservices compared to a monolith?
