# 1. How do you ensure that Microservices are loosely coupled and highly cohesive?

We keep each microservice focused on a single business capability, like Orders, Payments, or Inventory. That improves cohesion because one service owns one responsibility. To keep services loosely coupled, communication happens through APIs or events instead of direct database access. Each service should have its own database and should not depend on another service’s internal implementation. Small services are easier to deploy, scale, and maintain independently.

---

# 2. How does a Java Microservice and .NET Microservice talk with each other?

Language does not matter in microservices because communication usually happens over HTTP or messaging systems using JSON. A Java Spring Boot service and a .NET service can expose REST APIs and exchange JSON payloads. They can also communicate asynchronously using Kafka or RabbitMQ. Since both understand standard protocols like HTTP and JSON, interoperability becomes easy.

---

# 3. How do you handle cross-cutting concerns such as security?

Cross-cutting concerns like security, logging, tracing, and monitoring are centralized using API gateways, service mesh, or shared libraries. For security, we commonly use OAuth2 and JWT tokens. API Gateway handles authentication and rate limiting, while centralized logging and distributed tracing help with observability. This avoids duplicating the same logic in every service.

---

# 4. Why is debugging tough in Microservice Architecture?

Debugging is difficult because a single request may pass through multiple services. Failures can happen due to network latency, service unavailability, or message delays. Logs are distributed across systems, so tracing the full request flow is harder compared to monoliths. That’s why tools like centralized logging, correlation IDs, Jaeger, Zipkin, and OpenTelemetry are very important.

---

# 5. How do you handle data consistency in Microservices?

In microservices, each service usually owns its own database, so distributed transactions become challenging. Instead of using traditional ACID transactions across services, we use eventual consistency with patterns like Saga. Services communicate through events, and compensating transactions are used when failures occur. This improves scalability and avoids tight coupling between databases.

---

# 6. How do you ensure Microservices are scalable and resilient?

Scalability is achieved by making services stateless so multiple instances can run behind a load balancer. Kubernetes helps with auto-scaling using HPA. Resiliency is handled using retries, circuit breakers, bulkheads, and health checks. If one instance fails, Kubernetes restarts it automatically. Caching and asynchronous communication also improve system stability under heavy load.

---

# 7. How do you handle service discovery and registration?

In dynamic cloud environments, service IPs change frequently, so we use service discovery tools like Eureka, Consul, or Kubernetes Service Discovery. Services register themselves during startup, and other services discover them dynamically. In Kubernetes, DNS-based service discovery is built in, which simplifies communication between services.

---

# 8. How do you handle service communication and data sharing?

Service communication can be synchronous using REST/gRPC or asynchronous using Kafka or RabbitMQ. REST is simple and widely used, while messaging improves decoupling and scalability. For data sharing, we avoid shared databases because that creates tight coupling. Instead, services exchange data through APIs or events.

---

# 9. How do you handle service versioning and backward compatibility?

We maintain backward compatibility by versioning APIs, such as `/v1/orders` and `/v2/orders`. New changes should not break existing clients. We usually follow contract-first API design and avoid removing fields immediately. Deprecation policies are communicated before removing old versions.

---

# 10. How do you monitor and troubleshoot Microservices?

We use centralized logging tools like ELK or Loki, metrics tools like Prometheus and Grafana, and tracing tools like Jaeger. Correlation IDs help track requests across services. Dashboards and alerts help identify CPU spikes, memory leaks, or failed requests quickly.

---

# 11. How do you handle deployments and rollbacks?

We use CI/CD pipelines with Docker and Kubernetes. Kubernetes supports rolling deployments, blue-green deployments, and canary releases. If deployment fails, rollback can be done quickly using previous container images or Kubernetes rollout undo commands. This minimizes downtime.

---

# 12. How do you handle testing and continuous integration?

Each microservice has its own automated unit tests, integration tests, and API tests. CI pipelines run tests automatically during code commits. Services are independently built and deployed, which allows faster releases. Contract testing tools like Pact are also useful to verify API compatibility between services.

---

# 13. How do you handle service governance and lifecycle management?

We maintain standards for logging, security, API naming, monitoring, and deployments across all services. API gateways and service registries help manage services centrally. Documentation using OpenAPI/Swagger is important. Governance ensures consistency while lifecycle management tracks service versions, ownership, and deprecation.

---

# 14. How do you handle security and access control?

We use OAuth2 and JWT-based authentication. API Gateway validates tokens before requests reach services. Role-based access control ensures users only access authorized resources. Secrets and credentials are stored securely using Vault or Kubernetes Secrets. Communication between services can also be encrypted using TLS or mTLS.

---

# 15. How do you handle data integration and migration?

Data integration is usually event-driven using Kafka or ETL pipelines. During migration from monolith to microservices, we gradually split domains and databases. Data synchronization is carefully planned to avoid inconsistencies. CDC (Change Data Capture) tools like Debezium are often used.

---

# 16. How do you handle service composition and orchestration?

For workflows involving multiple services, orchestration tools like Temporal, Camunda, or Saga orchestrators can manage the flow. Sometimes choreography is used where services react to events independently. Choice depends on workflow complexity and business requirements.

---

# 17. How do you deploy your Java Microservices?

We containerize services using Docker and deploy them to Kubernetes on AWS or Azure. Docker packages the application and dependencies into images. Kubernetes handles scaling, load balancing, health checks, and restarts failed containers automatically. CI/CD pipelines automate deployments.

---

# 18. How do you handle service resiliency in case of failures?

Kubernetes helps by restarting failed containers automatically. We also use retries, circuit breakers, fallback mechanisms, and health probes. If one service fails, it should not bring down the entire system. Resilience patterns help isolate failures and improve availability.

---

# 19. What Java frameworks can be used to create Microservices?

Popular frameworks include:

* Spring Boot
* Quarkus
* Micronaut

Spring Boot is the most widely used because of its ecosystem and cloud support. Quarkus and Micronaut are optimized for faster startup and lower memory usage, especially useful in cloud-native and serverless environments.

---

# 20. How many Microservices do you have in your project? How would you investigate a missing order?

In enterprise systems, there can be dozens of microservices like Order, Payment, Inventory, Notification, and Shipping. To investigate a missing order, first identify which service owns the order data because each service typically has its own database. Then trace the request using correlation IDs, logs, Kafka events, and distributed tracing tools. We check whether the order event failed, message processing stopped, or downstream services had errors.

# 1. Can you explain Event-Driven pattern and how it is used in Microservices?

In the Event-Driven pattern, services communicate through events instead of direct synchronous API calls. When one service performs an action, it publishes an event to a message broker like Kafka or RabbitMQ, and other interested services consume that event.

For example, when an Order Service creates an order, it publishes an `OrderCreated` event. Payment, Inventory, and Notification services react independently.

This improves loose coupling, scalability, and asynchronous processing. It is widely used in microservices for workflows, notifications, analytics, and distributed systems.

---

# 2. Can you explain Service Registry pattern and how it is used?

In microservices, service instances are dynamic and their IP addresses may change frequently. A Service Registry keeps track of available service instances.

When a service starts, it registers itself with the registry. Other services query the registry to discover available instances.

Examples include:

* Eureka
* Consul
* Kubernetes Service Discovery

This enables dynamic service discovery and load balancing without hardcoding service URLs.

---

# 3. Can you explain Sidecar pattern and how it is used?

In the Sidecar pattern, an additional helper container runs alongside the main application container in the same pod or host.

The sidecar handles supporting features like:

* Logging
* Monitoring
* Security
* Traffic proxying

For example, in Kubernetes, Envoy proxy often runs as a sidecar beside the application container.

This keeps the main application lightweight and separates infrastructure concerns from business logic.

---

# 4. Can you explain Service Mesh pattern and how it is used?

A Service Mesh manages communication between microservices at the infrastructure level instead of application code.

It provides:

* Service-to-service security
* mTLS encryption
* Traffic routing
* Retries
* Circuit breaking
* Observability

Examples:

* Istio
* Linkerd

It usually works using sidecar proxies like Envoy. Developers focus on business logic while the mesh handles networking and resilience concerns centrally.

---

# 5. Can you explain Back-end for Front-end (BFF) pattern?

The BFF pattern creates separate backend services for different frontend clients.

For example:

* Mobile app → Mobile BFF
* Web app → Web BFF

Each frontend gets APIs optimized for its needs instead of calling many microservices directly.

Benefits:

* Reduced frontend complexity
* Better performance
* Customized responses
* Independent frontend evolution

This is commonly used in large-scale systems with multiple client applications.

---

# 6. Can you explain Bulkhead pattern and how it is used?

The Bulkhead pattern isolates failures so one failing component does not bring down the entire system.

It is similar to compartments in a ship. If one compartment floods, the whole ship still survives.

In microservices, this can be done using:

* Separate thread pools
* Resource isolation
* Independent service instances

For example, slow payment processing should not consume all application threads and block order processing.

This improves resiliency and fault isolation.

---

# 7. What is Saga pattern? What problem does it solve?

Saga pattern manages distributed transactions across multiple microservices.

Traditional ACID transactions do not work well across distributed services with separate databases. Saga solves this using a sequence of local transactions and compensating actions.

Example:

1. Order Service creates order
2. Payment Service charges payment
3. Inventory reserves stock

If inventory fails:

* Payment is refunded
* Order is cancelled

Saga ensures eventual consistency without using distributed database transactions.

---

# 8. Can you explain Outbox pattern and how it is used?

The Outbox pattern ensures reliable event publishing when updating a database and sending events.

Problem:
A service may save data successfully but fail while publishing the event.

Solution:

* Save both business data and event into the same database transaction
* A background process reads the outbox table and publishes events to Kafka/RabbitMQ

This guarantees no events are lost and maintains consistency between database updates and messaging systems.

---

# 9. What is Self-Containment pattern and how is it used?

The Self-Containment pattern means each microservice should own everything needed to perform its business function independently.

This includes:

* Business logic
* Database
* APIs
* Configurations

Services should avoid depending heavily on other services during runtime.

Benefits:

* Independent deployment
* Better scalability
* Reduced coupling
* Higher resilience

It aligns strongly with the “database per service” principle.

---

# 10. Can you explain External Configuration pattern and how it is used?

External Configuration means application configuration is stored outside the application code.

Examples:

* Kubernetes ConfigMaps
* Spring Cloud Config
* Vault
* Environment variables

This allows:

* Different configs for dev/test/prod
* Secure secret management
* Runtime configuration updates

Applications become portable and easier to manage across environments.

---

# 11. What is Strangler pattern and how is it used?

The Strangler pattern is used for gradually migrating a monolithic application into microservices.

Instead of rewriting the entire system at once:

* New functionality is built as microservices
* Old functionality is slowly replaced piece by piece

Traffic is gradually routed from monolith to new services.

Benefits:

* Lower migration risk
* Incremental modernization
* Easier testing and rollback

This is one of the most common enterprise migration strategies.

