# Spring Boot Production-Level Interview Questions (38 Scenarios)

---

## Memory & Performance Issues

1. Your Spring Boot service throws `OutOfMemoryError: Java heap space` in production under load. Walk through your exact diagnosis steps.
2. You see a gradual memory increase (memory leak) over 48 hours before the app crashes. How do you find the leak without restarting prod?
3. Your service is running fine but GC pause times spike to 5 seconds every few minutes, causing timeouts. How do you diagnose and fix?
4. `OutOfMemoryError: Metaspace` in production. What causes this in Spring Boot and how do you fix it?
5. How do you right-size JVM heap for a Spring Boot service running in a Kubernetes pod with a 1GB memory limit?
6. API response time suddenly spikes from 50ms to 5 seconds in production. You have no code changes deployed. What do you investigate first?
7. Your service handles 1000 rps fine but at 1500 rps everything times out. Identify the bottleneck and fix it.
8. A specific API endpoint is slow (2s) but only for some users, not all. How do you debug this?
9. Your Spring Boot app's startup time grew from 10 seconds to 90 seconds after adding new microservice dependencies. How do you diagnose and reduce it?
10. Redis cache hit rate is 30% in production despite caching being in place. Walk through how you would debug and improve it.
11. How do you implement a graceful shutdown that ensures in-flight requests are completed before the pod terminates in Kubernetes?

---

## Database & JPA Production Issues

12. Production database has a deadlock happening every few minutes. Your Spring Boot app is throwing deadlock exceptions. How do you find and fix the root cause?
13. Your HikariCP connection pool is exhausted in production. All requests hang. What are the immediate steps and the root cause investigation?
14. A database migration (Flyway) failed halfway through in production, leaving the schema in a broken state. How do you recover?
15. Hibernate is generating 500 SQL queries for a single API call that should need only 5. How do you detect and fix this N+1 problem?
16. Your `@Transactional` method is not rolling back on a checked exception. Why and how do you fix it?
17. Your production database CPU is at 100%. You need to identify the top offending queries from your Spring Boot app quickly.
18. You need to migrate 50 million rows in a table with zero downtime. How do you approach this with Spring Boot and Flyway?

---

## Kafka & Messaging

19. Kafka consumers are stuck. Lag is growing by millions of messages per minute. Production data processing has stopped. Walk through your response.
20. A Kafka consumer is consuming the same message thousands of times (infinite retry loop). How do you stop the bleeding and fix it permanently?
21. You are seeing duplicate messages being processed by your Kafka consumer after a pod restart. How do you achieve idempotent processing?
22. Kafka producer in your Spring Boot app is getting "Record too large" errors. Messages are being dropped. How do you handle large payloads?
23. How do you implement the Outbox pattern in Spring Boot to guarantee Kafka message delivery without distributed transactions?

---

## Security Incidents

24. You discover your Spring Boot service has been returning 200 OK with sensitive data for unauthenticated requests on some endpoints. How do you investigate and triage?
25. JWT tokens from your system are being used after they expire. Access is not being revoked. How is this happening and how do you fix it?
26. Your API is under a credential stuffing attack—50,000 login attempts per minute from rotating IPs. How do you defend the Spring Boot app?
27. You find SQL injection vulnerabilities in a legacy part of your Spring Boot app that uses native queries. How do you fix them without rewriting everything?
28. Secrets (DB passwords, API keys) were accidentally committed to your Git repository. What is your incident response and how do you prevent it in the future?
29. Your Spring Boot app is vulnerable to SSRF (Server-Side Request Forgery). A user can make your server fetch arbitrary URLs. How do you detect and fix this?

---

## Deployment & Configuration

30. A Spring Boot deployment is failing health checks in Kubernetes after startup. The pod keeps crashing in a CrashLoopBackOff. How do you diagnose?
31. You deployed a Spring Boot service and it is now throwing "No such property" errors because a required environment variable is missing. How do you prevent this pattern?
32. A blue-green deployment caused 2 minutes of downtime because the new (green) version had an incompatible DB schema change. How do you prevent this?
33. Your Spring Boot app has different behavior in prod vs staging despite using the same Docker image. The bug only appears in prod. How do you debug this?
34. Your Spring Cloud Config server is down and all microservices are failing to start. How do you make your Spring Boot services resilient to config server outages?
35. After enabling Spring Boot Actuator in production you discover the `/actuator/env` endpoint is exposing all environment variables including secrets. How do you fix this?
36. How do you implement feature flags in Spring Boot to safely roll out features to a subset of users without deploying new code?
37. Your Spring Boot service needs to handle configuration changes (like updated timeouts/pool sizes) at runtime without requiring a full restart. How do you design this?
38. *[Missing Question from Source]* The document indicates 38 total questions across these categories, concluding immediately after the context on question 37.
