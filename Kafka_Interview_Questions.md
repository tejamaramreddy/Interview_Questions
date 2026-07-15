# Kafka Interview Questions

## Kafka Fundamentals
1. What is Apache Kafka?
2. Why is Kafka used?
3. What problems does Kafka solve?
4. Explain Kafka architecture.
5. What is a Kafka broker?
6. What is a Kafka cluster?
7. What is a Kafka topic?
8. What is a Kafka partition?
9. Why are partitions important?
10. How are messages distributed across partitions?
11. What is a message key?
12. What is the replication factor?
13. What are leader and follower replicas?
14. What is ISR (In-Sync Replicas)?
15. What happens when ISR shrinks?
16. What is leader election?
17. What happens if a broker fails?
18. How does Kafka ensure fault tolerance?
19. What is Kafka retention policy?
20. What is log compaction?
21. What is the difference between retention and log compaction?
22. When should log compaction be used?
23. How does Kafka achieve high throughput?
24. When should you not use Kafka?

---

## Kafka Producer
25. What is a Kafka producer?
26. How does a producer send messages?
27. How does a producer determine the target partition?
28. What is batching in Kafka?
29. What is `linger.ms`?
30. What is `batch.size`?
31. What is message compression?
32. What compression algorithms does Kafka support?
33. What are producer acknowledgments (`acks`)?
34. What is the difference between `acks=0`, `acks=1`, and `acks=all`?
35. What are producer retries?
36. What is `max.in.flight.requests.per.connection`?
37. What is an idempotent producer?
38. How do you enable idempotence?
39. How does Kafka prevent duplicate message production?
40. How can producer throughput be improved?

---

## Kafka Consumer
41. What is a Kafka consumer?
42. What is a consumer group?
43. Why do we use consumer groups?
44. How are partitions assigned to consumers?
45. What happens when a new consumer joins a group?
46. What is consumer rebalance?
47. What causes a rebalance?
48. How can rebalances be minimized?
49. What is an offset?
50. Where are consumer offsets stored?
51. What is auto offset commit?
52. What is manual offset commit?
53. When should manual commits be used?
54. What happens if offset commit fails?
55. What is `auto.offset.reset`?
56. What is the difference between `earliest` and `latest`?
57. What is consumer lag?
58. How do you monitor consumer lag?
59. How do you reduce consumer lag?
60. Why does Kafka use a pull model instead of push?

---

## Delivery Guarantees
61. What is at-most-once delivery?
62. What is at-least-once delivery?
63. What is exactly-once delivery?
64. Which delivery guarantee does Kafka provide by default?
65. How does Kafka implement Exactly Once Semantics (EOS)?
66. Can duplicate messages still occur?
67. How do you handle duplicate messages?
68. What is an idempotent consumer?
69. How do idempotent consumers achieve effectively-once processing?

---

## Kafka Transactions
70. What are Kafka transactions?
71. Why are Kafka transactions important?
72. What is a `transaction.id`?
73. How do Kafka transactions work across multiple partitions?
74. What is the transaction coordinator?
75. What is `read_committed` isolation?
76. What is the difference between `read_committed` and `read_uncommitted`?

---

## Spring Kafka
77. What is `@KafkaListener`?
78. How does `@KafkaListener` work?
79. How do you configure concurrent Kafka consumers?
80. What is `ConcurrentKafkaListenerContainerFactory`?
81. What is the difference between auto acknowledgment and manual acknowledgment?
82. What are the different `AckMode` options?
83. What is `AckMode.RECORD`?
84. What is `AckMode.BATCH`?
85. What is `AckMode.MANUAL`?
86. What is `AckMode.MANUAL_IMMEDIATE`?
87. What happens when a `@KafkaListener` throws an exception?
88. What is `DefaultErrorHandler`?
89. What is `SeekToCurrentErrorHandler`?
90. Why was `SeekToCurrentErrorHandler` replaced?
91. How do retries work in Spring Kafka?
92. How do you configure retry with exponential backoff?
93. What is `FixedBackOff`?
94. What is a Dead Letter Topic (DLT)?
95. Why is a DLT used?
96. How do you configure a DLT in Spring Kafka?
97. How do you reprocess messages from a DLT?

---

## Kafka Performance
98. How do you improve Kafka producer performance?
99. How do you improve Kafka consumer performance?
100. How do partitions impact performance?
101. How many partitions should a topic have?
102. How do you avoid hot partitions?
103. Which Kafka metrics should be monitored?

---

## RabbitMQ Comparison
104. Kafka vs RabbitMQ?
105. When would you choose Kafka over RabbitMQ?
106. When would you choose RabbitMQ over Kafka?
107. Kafka vs JMS?
108. Kafka vs ActiveMQ?
109. Kafka vs traditional message queues?

---

## Event-Driven Architecture
110. What is event-driven architecture?
111. What are the benefits of event-driven architecture?
112. What are the challenges of event-driven systems?
113. What is event notification?
114. What is event-carried state transfer?
115. How do you maintain event ordering?
116. How do you handle event versioning?
117. What is Schema Registry?

---

## Transactional Outbox Pattern
118. What is the transactional outbox pattern?
119. What problem does the outbox pattern solve?
120. What is the dual-write problem?
121. How does the transactional outbox work?
122. How is the outbox table processed?
123. What are the advantages and disadvantages of the outbox pattern?

---

## CDC (Debezium)
124. What is Change Data Capture (CDC)?
125. What is Debezium?
126. How does Debezium work?
127. Why use CDC instead of publishing events directly?
128. Debezium vs polling?
129. Which databases are supported by Debezium?

---

## Saga Pattern
130. What is the Saga pattern?
131. Why is the Saga pattern needed?
132. What is Saga choreography?
133. What is Saga orchestration?
134. What is the difference between choreography and orchestration?
135. What are compensating transactions?
136. How is Kafka used in Saga implementations?

---

## Event Sourcing
137. What is Event Sourcing?
138. What are the benefits of Event Sourcing?
139. What are the drawbacks of Event Sourcing?
140. Event Sourcing vs CRUD?
141. How is Kafka used for Event Sourcing?
142. What is event replay?

---

## CQRS
143. What is CQRS?
144. Why separate read and write models?
145. How does CQRS work with Kafka?
146. CQRS vs Event Sourcing?
147. When should CQRS be used?

---

## Idempotency & Deduplication
148. What is idempotency?
149. Why is idempotency important?
150. What is an idempotency key?
151. How do you implement idempotency using a database?
152. What is a deduplication table?
153. How do you track processed message IDs?
154. How do you implement Redis-based deduplication?
155. Why can't Kafka completely eliminate duplicates?
156. How do you build an idempotent consumer?
157. Explain effectively-once processing.

---

## Scenario-Based Questions
158. A consumer crashes after processing a message but before committing the offset. What happens?
159. A producer sends duplicate messages. How do you prevent duplicate processing?
160. A topic has 3 partitions and 5 consumers. What happens?
161. A topic has 10 partitions and 2 consumers. How are messages distributed?
162. How do you guarantee message ordering?
163. How do you guarantee ordering for a single customer or account?
164. Consumer lag is continuously increasing. How would you troubleshoot it?
165. A Kafka broker goes down. What happens next?
166. The leader replica crashes. How does Kafka recover?
167. How would you retry failed events without blocking successful ones?
168. How do you handle poison messages?
169. How would you design a payment processing system using Kafka?
170. How would you implement reliable email notifications using Kafka?
171. Explain a Kafka-based architecture you have implemented in production.
