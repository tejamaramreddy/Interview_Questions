---

# Kafka Fundamentals
---

## 1. What is Apache Kafka?

**Answer:**

Apache Kafka is an open-source distributed event streaming platform that can be used either as a message queue or as a stream processing system. Kafka excels in delivering high performance, scalability, and durability. It's engineered to handle vast volumes of data in real-time, and when configured properly (with appropriate replication and acknowledgment settings), it can provide strong guarantees against message loss.

---

## 2. What problem does Kafka solve?

**Answer:**

Kafka solves the problem of reliable and scalable communication between distributed systems. It enables asynchronous communication, so services don't have to wait for each other. This reduces tight coupling between microservices and improves system resilience. Kafka can handle very high volumes of data while ensuring fault tolerance through replication. It also allows multiple consumers to process the same event independently and replay events when needed.

---

## 3. Why Kafka instead of RabbitMQ or ActiveMQ?

**Answer:**

| Kafka                               | RabbitMQ / ActiveMQ                        |
| ----------------------------------- | ------------------------------------------ |
| Event streaming platform            | Message broker                             |
| Very high throughput                | Moderate throughput                        |
| Stores messages for replay          | Deletes after acknowledgment               |
| Horizontal scaling with partitions  | Scaling is more limited                    |
| Best for event-driven microservices | Best for task queues and work distribution |

---

## 4. Explain Kafka architecture.

**Answer:**

A Kafka cluster is made up of multiple brokers. These are just individual servers (they can be physical or virtual). Each broker is responsible for storing data and serving clients. The more brokers you have, the more data you can store and the more clients you can serve.

Each broker has a number of partitions. Each partition is an ordered, immutable sequence of messages that is continually appended to -- think of like a log file. Partitions are the way Kafka scales as they allow for messages to be consumed in parallel.

A topic is just a logical grouping of partitions. Topics are the way you publish and subscribe to data in Kafka. When you publish a message, you publish it to a topic, and when you consume a message, you consume it from a topic. Topics are always multi-producer; that is, a topic can have zero, one, or many producers that write data to it.

Last up we have our producers and consumers. Producers are the ones who write data to topics, and consumers are the ones who read data from topics. While Kafka exposes a simple API for both producers and consumers, the creation and processing of messages is on you, the developer. Kafka doesn't care what the data is, it just stores and serves it.

---

## 5. What are Brokers, Topics, Partitions, Producers, and Consumers?

**Answer:**
*(Look in architecture)*

---

## 6. What is a Kafka cluster?

**Answer:**
*(Look in architecture)*

---

## 7. What is the difference between a Topic and a Partition?

**Answer:**

A topic is a logical grouping of messages. A partition is a physical grouping of messages. A topic can have multiple partitions, and each partition can be on a different broker. Topics are just a way to organize your data, while partitions are a way to scale your data.

---

## 8. Why are partitions important?

**Answer:**

*(Look in architecture)*

---

## 9. What is an Offset?

**Answer:**

Each message in a Kafka partition is assigned a unique offset, which is a sequential identifier indicating the message’s position in the partition. This offset is used by consumers to track their progress in reading messages from the topic.

As consumers read messages, they maintain their current offset and periodically commit this offset back to Kafka. This way, they can resume reading from where they left off in case of failure or restart.

Note that Kafka provides at-least-once delivery by default: if a consumer crashes after processing a message but before committing its offset, the message will be reprocessed after restart. Exactly-once semantics are possible but require additional configuration (idempotent producers + transactional APIs).

---

## 10. How are messages stored in Kafka?

**Answer:**

Kafka stores messages in an append-only log inside topic partitions. Each partition is a sequence of immutable records where every new message is appended to the end of the log and assigned a unique offset.


---

# Producer
## 11. How does a Kafka producer work?

When an event occurs in an application, the Kafka producer creates a message (also called a record) and sends it to a Kafka topic.

A message consists of:

* A value (payload)
* An optional key
* A timestamp
* Optional headers

The producer determines the target partition based on the message key. If no key is provided, Kafka uses its partitioning strategy to distribute messages across partitions.

The producer batches records for efficiency, optionally compresses them, and sends them to the leader broker of the target partition.

Based on the configured acknowledgment (`acks`) setting, the producer waits for confirmation before considering the message successfully written. If the acknowledgment is not received due to a transient failure, the producer can automatically retry sending the message.

---

## 12. How does Kafka decide which partition to send a message to?

When a message is published to a Kafka topic, Kafka first determines the appropriate partition for the message. This partition selection is critical because it influences the distribution of data across the cluster.

This is a two-step process:

### Partition Determination

Kafka uses a partitioning algorithm that hashes the message key to assign the message to a specific partition.

If the message does not have a key, Kafka can either round-robin the message to partitions or follow another partitioning logic defined in the producer configuration.

This ensures that messages with the same key always go to the same partition, preserving order at the partition level.

### Broker Assignment

Once the partition is determined, Kafka then identifies which broker holds that particular partition.

The mapping of partitions to specific brokers is managed by the Kafka cluster metadata, which is maintained by the Kafka controller (a role within the broker cluster).

The producer uses this metadata to send the message directly to the broker that hosts the target partition.

---

## 13. What is the purpose of the message key?

---

## 14. Explain `acks=0`, `acks=1`, and `acks=all`.

The `acks` (acknowledgment) configuration determines how many Kafka brokers must acknowledge a message before the producer considers the send operation successful. It directly affects the trade-off between performance and reliability.

### `acks=0`

The producer does not wait for any acknowledgment from the broker.

This provides the highest throughput and lowest latency, but messages can be lost if the broker is unavailable or the request fails.

### `acks=1`

The producer waits for an acknowledgment only from the leader broker after it writes the message to its local log.

This offers a balance between performance and reliability, but if the leader fails before followers replicate the message, the data may be lost.

### `acks=all` (or `acks=-1`)

The producer waits until the leader and all in-sync replicas (ISR) acknowledge the message.

This provides the highest durability and minimizes the risk of data loss, although it introduces slightly higher latency.

---

## 15. What is an idempotent producer?

To avoid duplicate messages when retries are enabled. This just ensures that messages are only sent once in the case we incorrectly think they weren't sent.

---

## 16. How do retries work in Kafka?

Kafka producers automatically retry sending a message if a send operation fails due to temporary issues such as network failures, broker unavailability, or request timeouts.

The producer retries based on its retry configuration until the message is successfully acknowledged or the retry limit is reached.

During a retry, the producer resends the same message to the leader broker.

Without idempotence enabled, these retries can result in duplicate messages if the broker had already stored the original message but the acknowledgment was lost.

To prevent this, we enable idempotent producers (`enable.idempotence=true`), which ensure that duplicate retries are ignored.

---

## 17. What problems can retries cause?

Retries improve reliability by handling temporary failures, but they can introduce some challenges.

The main problem is duplicate messages. If the producer sends a message successfully to the broker but the acknowledgment is lost due to a network issue, the producer may retry and the same message can be stored multiple times.

Another issue is message ordering. If retries are configured with multiple in-flight requests, an earlier message might fail and be retried after a later message succeeds, causing messages to arrive out of order.

To avoid these issues, we enable idempotent producers, configure appropriate retry settings, and limit the number of in-flight requests when ordering is critical.

---

## 18. What is batching in Kafka producers?

Batching is a Kafka producer optimization where multiple messages are grouped together and sent to the broker in a single request instead of sending each message individually.

This reduces network overhead and improves producer throughput.

Kafka automatically batches messages based on configurations like `batch.size` and `linger.ms`.

A larger batch size improves throughput, while a smaller batch size reduces latency.

---

## 19. What is producer compression?

Producer compression is a feature where Kafka compresses messages before sending them to the broker.

Compression reduces network bandwidth usage and disk storage requirements.

Kafka compresses messages in batches rather than individually, making compression more efficient.

Common compression algorithms supported by Kafka are:

* gzip
* snappy
* lz4
* zstd

The choice depends on the trade-off between compression ratio and CPU usage.

---

## 20. Which producer configurations do you use in production?

Some common producer configurations we use are:

* `acks=all` to ensure messages are replicated successfully.
* `enable.idempotence=true` to prevent duplicate messages during retries.
* `retries` to handle temporary failures.
* `compression.type=lz4` to reduce network and storage usage.
* `batch.size` and `linger.ms` to improve throughput.
* `delivery.timeout.ms` to control message delivery time.
* `key.serializer` and `value.serializer` to convert objects into Kafka records.

### Example Production Configuration

```properties
acks=all

enable.idempotence=true

retries=10

compression.type=lz4

batch.size=32768

linger.ms=10

delivery.timeout.ms=120000

key.serializer=org.apache.kafka.common.serialization.StringSerializer

value.serializer=org.springframework.kafka.support.serializer.JsonSerializer
```


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
