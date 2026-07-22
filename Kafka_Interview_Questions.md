---

# Kafka Fundamentals

1. What is Apache Kafka?
2. What problem does Kafka solve?
3. Why Kafka instead of RabbitMQ or ActiveMQ?
4. Explain Kafka architecture.
5. What are Brokers, Topics, Partitions, Producers, and Consumers?
6. What is a Kafka cluster?
7. What is the difference between a Topic and a Partition?
8. Why are partitions important?
9. What is an Offset?
10. How are messages stored in Kafka?

---

# Producer

11. How does a Kafka producer work?
12. How does Kafka decide which partition to send a message to?
13. What is the purpose of the message key?
14. Explain `acks=0`, `acks=1`, and `acks=all`.
15. What is an idempotent producer?
16. How do retries work in Kafka?
17. What problems can retries cause?
18. What is batching in Kafka producers?
19. What is producer compression?
20. Which producer configurations do you use in production?

---

# Consumer

21. What is a Kafka consumer?
22. What is a Consumer Group?
23. How does Kafka distribute partitions among consumers?
24. What happens if you have more consumers than partitions?
25. Explain Kafka offsets.
26. Auto offset commit vs Manual offset commit.
27. What is `commitSync()` vs `commitAsync()`?
28. How do you replay messages?
29. How do you avoid duplicate processing?
30. How do you handle poison messages (bad messages)?

---

# Reliability & Replication

31. What is a replication factor?
32. Explain Leader and Follower replicas.
33. What is ISR (In-Sync Replicas)?
34. What happens when the leader broker fails?
35. How does Kafka ensure fault tolerance?
36. What is High Watermark?
37. What happens if all replicas fail?
38. How do you prevent data loss?
39. Explain At-most-once, At-least-once, and Exactly-once delivery.
40. What are Kafka Transactions?

---

# Performance & Scaling

41. Why is Kafka so fast?
42. How does Kafka achieve high throughput?
43. What causes Consumer Lag?
44. How do you monitor Consumer Lag?
45. What happens if consumers are slower than producers?
46. How do you tune Kafka for high performance?
47. How many partitions should a topic have?
48. Can Kafka guarantee ordering?
49. What happens if you increase the number of partitions?
50. What metrics do you monitor in production?

---

# Spring Boot Kafka

51. How do you integrate Kafka with Spring Boot?
52. What is `KafkaTemplate`?
53. What is `@KafkaListener`?
54. How do you configure multiple consumers?
55. How do you configure multiple producers?
56. How do you implement retries in Spring Kafka?
57. What is a Dead Letter Topic (DLT)?
58. How do you process failed messages?
59. How do you implement error handling in Kafka consumers?
60. How do you deserialize JSON messages?

---

# Advanced Kafka

61. What is Kafka Rebalancing?
62. What causes rebalancing?
63. How can you reduce rebalancing?
64. What is Cooperative Rebalancing?
65. What is Static Membership?
66. What is Log Compaction?
67. Log Compaction vs Log Retention.
68. Explain Kafka Streams.
69. Explain KTable vs KStream.
70. What is Kafka Connect?

---

# Microservices

71. Why is Kafka used in Microservices?
72. Synchronous vs Asynchronous communication.
73. REST vs Kafka.
74. Event-driven architecture.
75. Choreography vs Orchestration.
76. Saga Pattern with Kafka.
77. Event Sourcing.
78. CQRS with Kafka.
79. How do multiple microservices consume the same event?
80. How do you version Kafka events?

---

# Production Scenario Questions

81. Consumer lag suddenly increases. How do you investigate?
82. Duplicate messages are being processed. What will you do?
83. Messages are arriving out of order. Why?
84. A consumer crashes while processing a message. What happens?
85. One broker goes down. What happens?
86. Kafka producer is very slow. How do you troubleshoot?
87. A topic contains billions of messages. How does Kafka manage it?
88. How do you migrate from REST communication to Kafka?
89. How do you monitor Kafka in production?
90. What Kafka-related issues have you faced in your project?

---

# Coding & Practical

91. Write a Spring Boot Kafka Producer.
92. Write a Kafka Consumer using `@KafkaListener`.
93. How do you send JSON messages?
94. How do you consume Avro messages?
95. How do you retry failed messages?
96. How do you implement DLT in Spring Kafka?
97. How do you manually commit offsets?
98. How do you consume from a specific partition?
99. How do you reset offsets?
100. Explain a Kafka implementation from your project end-to-end.

---

## Most Important Questions (Frequently Asked)

If you're short on time, focus on these 20:

1. Kafka Architecture
2. Topic vs Partition
3. Offset
4. Consumer Group
5. Producer Partitioning
6. Message Key
7. `acks`
8. Idempotent Producer
9. Replication Factor
10. Leader & Follower
11. ISR
12. Consumer Lag
13. Auto vs Manual Commit
14. Rebalancing
15. Exactly-once Semantics
16. Dead Letter Topic (DLT)
17. Kafka Transactions
18. Ordering Guarantees
19. Spring Boot Kafka Integration
20. Explain your Kafka project

Mastering these questions will prepare you well for most **5–8 years Java Backend Developer** interviews involving Kafka.
