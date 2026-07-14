# Kafka Interview Questions


# Kafka Fundamentals

## 1. What is the difference between Kafka and a traditional message queue like RabbitMQ?

**Difficulty:** 🟢 Must

<details>
<summary><strong>Show Answer</strong></summary>


- Key differences:
- 1.
- Retention — Kafka retains messages on disk for a configurable period (days/weeks); RabbitMQ deletes on ACK.
- 2.
- Consumer model — Kafka consumers pull at their own pace; RabbitMQ pushes.
- 3.
- Replay — Kafka consumers can reset offsets and replay history; RabbitMQ cannot.
- 4.
- Ordering — Kafka guarantees order within a partition; RabbitMQ only within a single queue.
- 5.
- Throughput — Kafka handles millions of msg/sec; RabbitMQ is better for complex routing, lower latency at lower volume.


</details>

## 2. Explain the core components of Kafka: broker, topic, partition, producer, consumer.

**Difficulty:** 🟢 Must

<details>
<summary><strong>Show Answer</strong></summary>


- Broker:
- a Kafka server that stores data and serves clients.
- Cluster = multiple brokers.
- Topic:
- a named category for messages (like a table in a DB).
- Partition:
- a topic is split into partitions for parallelism — each is an ordered, immutable log.
- Producer:
- client that writes messages to topics.
- Consumer:
- client that reads messages from topics.
- ZooKeeper/KRaft:
- manages cluster metadata and leader election (ZooKeeper being replaced by KRaft in newer versions).


</details>

## 3. What is an offset in Kafka?

**Difficulty:** 🟢 Must

<details>
<summary><strong>Show Answer</strong></summary>


- An offset is a unique sequential ID assigned to each message within a partition.
- Offsets are per-partition (not global).
- Consumers track their position by storing the last committed offset.
- On restart, a consumer resumes from the committed offset.
- This is what makes Kafka replay possible — reset the offset to re-consume old messages.


</details>

## 4. What is a Kafka broker and what is a Kafka cluster?

**Difficulty:** 🟢 Must

<details>
<summary><strong>Show Answer</strong></summary>


- A broker is a single Kafka server process — it receives, stores, and serves messages.
- A cluster is a group of brokers working together.
- Data is distributed across brokers via partitions.
- One broker per partition acts as the leader (handles reads/writes); others hold replicas (followers).
- If the leader fails, a follower is elected as the new leader automatically.


</details>

## 5. What is the role of ZooKeeper in Kafka and what is KRaft?

**Difficulty:** 🟡 Important

<details>
<summary><strong>Show Answer</strong></summary>


- ZooKeeper historically managed Kafka cluster metadata:
- broker registration, leader election, topic config storage.
- KRaft (Kafka Raft) is the replacement introduced in Kafka 2.8, made production-ready in 3.3.
- It moves metadata management into Kafka itself using the Raft consensus protocol, eliminating ZooKeeper dependency.
- This simplifies deployment, improves scalability, and reduces operational complexity.


</details>

## 6. What is log compaction in Kafka?

**Difficulty:** 🟡 Important

<details>
<summary><strong>Show Answer</strong></summary>


- Log compaction is a cleanup policy (cleanup.policy=compact) where Kafka retains only the latest message for each key within a partition.
- Earlier messages with the same key are deleted.
- Useful for changelog topics where you only need the latest state per entity (e.g.
- user profile updates).
- Delete markers (tombstones — messages with null value) signal that a key should be fully deleted.


```text
cleanup.policy=compact
# Or both compact and delete:
- cleanup.policy=compact,delete
```


</details>

## 7. What is the difference between Kafka and a database?

**Difficulty:** 🟡 Important

<details>
<summary><strong>Show Answer</strong></summary>


- Kafka is an event log, not a queryable database.
- Key differences:
- Kafka is append-only (no update/delete by key); data is ordered by time not indexed for random access; Kafka retains for a time window not forever (by default); queries require reading the whole partition or using Kafka Streams/ksqlDB.
- Use Kafka for event streaming and DB for current state.
- Often used together:
- Kafka carries events, DB stores current state derived from those events.


</details>

## 8. What is the Kafka Controller and what does it do?

**Difficulty:** 🔴 Advanced

<details>
<summary><strong>Show Answer</strong></summary>


- The Controller is one elected broker in the cluster that is responsible for:
- partition leader election when a broker goes down, managing ISR (In-Sync Replicas) lists, propagating partition state changes to other brokers.
- In KRaft mode, the controller role is separated into dedicated controller nodes.
- Only one active controller at a time; if it fails, another broker is elected.


</details>


# Topics & Partitions

## 9. What is a Kafka topic partition and why does partitioning matter?

**Difficulty:** 🟢 Must

<details>
<summary><strong>Show Answer</strong></summary>


- A partition is an ordered, immutable sequence of records within a topic.
- Partitioning matters because:
- 1.
- Parallelism — multiple consumers in a group each read from different partitions simultaneously.
- 2.
- Throughput — multiple brokers handle different partitions.
- 3.
- Ordering — order is guaranteed only within a partition, not across partitions.
- The number of partitions determines the maximum parallelism of consumption.


```text
# Create topic with 6 partitions, 3 replicas
- kafka-topics.sh --create \\
- --topic orders \\
- --partitions 6 \\
- --replication-factor 3
```


</details>

## 10. How does a producer decide which partition to send a message to?

**Difficulty:** 🟢 Must

<details>
<summary><strong>Show Answer</strong></summary>


- Partition selection order:
- 1.
- Explicit partition specified by the producer — message goes there.
- 2.
- Message has a key — Kafka hashes the key (murmur2) and mods by partition count:
- partition = hash(key) % numPartitions.
- Same key always goes to the same partition (ordering guaranteed for that key).
- 3.
- No key and no explicit partition — RoundRobin or StickyPartitioner (batches to one partition, then switches).
- Key-based routing is critical for ordering guarantees.


```text
// Same key always → same partition
- kafkaTemplate.send("orders", order.getCustomerId(), order);
```


</details>

## 11. What is a replication factor and what is ISR (In-Sync Replicas)?

**Difficulty:** 🟢 Must

<details>
<summary><strong>Show Answer</strong></summary>


- Replication factor is how many copies of each partition exist across brokers (typically 3 in production).
- ISR is the set of replicas that are fully caught up with the leader.
- A replica falls out of ISR if it falls too far behind (replica.lag.time.max.ms).
- Producers can configure acks=all to wait for all ISR replicas to confirm a write before considering it successful — maximum durability.


```text
# Recommended production settings
- replication.factor=3
- min.insync.replicas=2
# Producer
- acks=all
```


</details>

## 12. What happens when you increase the number of partitions on an existing topic?

**Difficulty:** 🟡 Important

<details>
<summary><strong>Show Answer</strong></summary>


- You can increase partitions (kafka-topics.sh --alter) but you cannot decrease them (would require data deletion).
- Increasing partitions:
- 1.
- New messages with keys may be routed to different partitions — breaking per-key ordering for keys that hash to new partitions.
- 2.
- Consumer group rebalancing is triggered.
- 3.
- Existing messages stay in their original partitions.
- Plan partition count carefully upfront.
- Rule of thumb:
- target throughput / throughput per partition, with 10x headroom.


```text
kafka-topics.sh --alter \\
- --topic orders \\
- --partitions 12 \\
- --bootstrap-server localhost:9092
```


</details>

## 13. What is a compacted topic used for in microservices?

**Difficulty:** 🔴 Advanced

<details>
<summary><strong>Show Answer</strong></summary>


- Compacted topics act as a shared event-sourced state store.
- Common uses:
- 1.
- DB changelog replication (Debezium publishes every DB change; consumers rebuild state).
- 2.
- Kafka Streams internal changelogs for fault-tolerant stateful processors.
- 3.
- Service configuration broadcast — latest config for each key is always available.
- 4.
- Event sourcing replay — new consumers read the compacted topic to get current state without reading the full history.


</details>


# Producers

## 14. What are the Kafka producer acks settings and what do they mean?

**Difficulty:** 🟢 Must

<details>
<summary><strong>Show Answer</strong></summary>


- acks controls durability guarantees:
- acks=0 — fire and forget, no confirmation (fastest, data loss possible).
- acks=1 — leader confirms write (default, leader loss can lose data).
- acks=all (or -1) — all ISR replicas confirm (strongest guarantee, slowest).
- In production financial/critical systems use acks=all with min.insync.replicas=2.
- For high-throughput analytics where some loss is OK, acks=1 is fine.


```text
spring:
- kafka:
- producer:
- acks:
- all
- properties:
- min.insync.replicas:
- 2
```


</details>

## 15. What is idempotent producer in Kafka?

**Difficulty:** 🟢 Must

<details>
<summary><strong>Show Answer</strong></summary>


- Setting enable.idempotence=true on the producer ensures exactly-once delivery to the broker — duplicate messages from retries are detected and deduplicated using a producer ID (PID) and sequence number per partition.
- The broker rejects duplicates.
- Automatically sets acks=all and retries=Integer.MAX_VALUE.
- Required for Kafka transactions.
- Available since Kafka 0.11.


```text
spring:
- kafka:
- producer:
- properties:
- enable.idempotence:
- true
        # Auto-sets acks=all and max.in.flight=5
```


</details>

## 16. What is the producer batch.size and linger.ms and how do they affect throughput?

**Difficulty:** 🟡 Important

<details>
<summary><strong>Show Answer</strong></summary>


- batch.size (default 16KB):
- max bytes to batch before sending.
- Larger batch = better compression and throughput.
- linger.ms (default 0ms):
- time to wait for more messages before sending a batch even if batch.size not reached.
- Setting linger.ms=5–20ms allows batching at the cost of a small latency increase but dramatically improves throughput.
- Tune together:
- larger batch.size + moderate linger.ms for high-throughput producers.


```text
spring:
- kafka:
- producer:
- batch-size:
- 65536  # 64KB
- properties:
- linger.ms:
- 10
- compression.type:
- snappy
```


</details>

## 17. What is the difference between synchronous and asynchronous Kafka send?

**Difficulty:** 🟡 Important

<details>
<summary><strong>Show Answer</strong></summary>


- Async (default):
- kafkaTemplate.send() returns a CompletableFuture immediately.
- The message is queued in the producer buffer.
- High throughput but you must handle failures in the callback.
- Sync:
- kafkaTemplate.send().get() blocks until the broker acknowledges.
- Lower throughput but simpler error handling.
- In Spring Boot, use the callback pattern for async:
- kafkaTemplate.send(...).thenAccept(result -> ...).exceptionally(ex -> ...)


```text
// Async with callback (preferred)
- kafkaTemplate.send("topic", key, value)
- .thenAccept(r -> log.info("Sent to partition {}", r.getRecordMetadata().partition()))
- .exceptionally(ex -> { log.error("Send failed", ex); return null; });

- // Sync (use sparingly)
- kafkaTemplate.send("topic", key, value).get();
```


</details>

## 18. What is a Kafka transaction and when do you need one?

**Difficulty:** 🔴 Advanced

<details>
<summary><strong>Show Answer</strong></summary>


- Kafka transactions provide atomic writes across multiple partitions/topics — all succeed or all fail.
- Use cases:
- 1.
- Consume-process-produce:
- read from topic A, process, write to topic B atomically (read_committed isolation).
- 2.
- Writing to multiple topics atomically (e.g.
- order-events AND payment-events in one transaction).
- Enable with transactional.id on the producer.
- In Spring Boot use @Transactional with a KafkaTransactionManager.


```text
@Bean
- public ProducerFactory<String, String> pf() {
- Map<String, Object> props = new HashMap<>();
- props.put(ProducerConfig.TRANSACTIONAL_ID_CONFIG, "my-tx-id");
- return new DefaultKafkaProducerFactory<>(props);
- }

- @Transactional("kafkaTransactionManager")
- public void sendAtomic(String a, String b) {
- kafkaTemplate.send("topic-a", a);
- kafkaTemplate.send("topic-b", b);
- }
```


</details>

## 19. What is the max.in.flight.requests.per.connection setting and why does it matter for ordering?

**Difficulty:** 🔴 Advanced

<details>
<summary><strong>Show Answer</strong></summary>


- This controls how many unacknowledged requests the producer can have in flight per broker connection (default 5).
- If retries=true and max.in.flight>1, a failed request followed by a successful later request can cause out-of-order delivery.
- To guarantee ordering with retries:
- set max.in.flight.requests.per.connection=1 (sacrifices throughput) OR enable idempotence (enable.idempotence=true) which handles ordering correctly with up to 5 in-flight requests.


```text
# For strict ordering without idempotence
- max.in.flight.requests.per.connection=1

# Better: use idempotence (handles ordering + dedup)
- enable.idempotence=true
# Max in-flight auto-set to 5 safely
```


</details>


# Consumers & Consumer Groups

## 20. What is a Kafka consumer group and how does partition assignment work?

**Difficulty:** 🟢 Must

<details>
<summary><strong>Show Answer</strong></summary>


- A consumer group is a logical group of consumers that jointly consume a topic.
- Kafka assigns each partition to exactly one consumer in the group.
- If you have 6 partitions and 3 consumers, each consumer gets 2 partitions.
- If 6 consumers, each gets 1.
- If 7 consumers, one is idle.
- This is the parallelism model — you cannot have more active consumers than partitions.
- Adding consumers beyond partition count adds no throughput.


```text
spring:
- kafka:
- consumer:
- group-id:
- order-processing-group

- @KafkaListener(topics = "orders", groupId = "order-processing-group")
- public void consume(OrderEvent event) { ...
- }
```


</details>

## 21. What is a consumer group rebalance and what causes it?

**Difficulty:** 🟢 Must

<details>
<summary><strong>Show Answer</strong></summary>


- A rebalance is when Kafka redistributes partition assignments among consumers in a group.
- Causes:
- 1.
- New consumer joins the group.
- 2.
- Consumer leaves or crashes.
- 3.
- Partitions are added to the topic.
- 4.
- Consumer does not poll fast enough (max.poll.interval.ms exceeded).
- During rebalance, all consumers stop consuming (stop-the-world in older Kafka).
- Incremental Cooperative Rebalancing (Kafka 2.4+) reduces this by only moving partitions that need to change.


```text
spring:
- kafka:
- consumer:
- properties:
- partition.assignment.strategy:
- org.apache.kafka.clients.consumer.CooperativeStickyAssignor
- max.poll.interval.ms:
- 300000
```


</details>

## 22. What is the difference between auto-commit and manual offset commit?

**Difficulty:** 🟢 Must

<details>
<summary><strong>Show Answer</strong></summary>


- Auto-commit (enable.auto.commit=true, default):
- Kafka commits the offset periodically (auto.commit.interval.ms=5000).
- Risk:
- if the app crashes after auto-commit but before finishing processing, the message is lost (at-most-once).
- Manual commit:
- you control when to commit.
- After processing:
- ack.acknowledge() — at-least-once (may reprocess on crash).
- Before processing:
- at-most-once.
- Kafka transaction:
- exactly-once.
- Spring Boot:
- use AckMode.MANUAL_IMMEDIATE for manual control.


```text
spring:
- kafka:
- consumer:
- enable-auto-commit:
- false
- listener:
- ack-mode:
- MANUAL_IMMEDIATE

- @KafkaListener(topics = "orders")
- public void consume(OrderEvent event, Acknowledgment ack) {
- processOrder(event);
- ack.acknowledge();  // commit only after success
- }
```


</details>

## 23. What is max.poll.records and max.poll.interval.ms and how do they interact?

**Difficulty:** 🟡 Important

<details>
<summary><strong>Show Answer</strong></summary>


- max.poll.records (default 500):
- max messages returned per poll() call.
- max.poll.interval.ms (default 300s):
- max time between poll() calls before the broker considers the consumer dead and triggers a rebalance.
- If your processing is slow (e.g.
- 100ms per message) and max.poll.records=500:
- 500 * 100ms = 50 seconds per batch.
- Fine if under 300s.
- If processing is slower, reduce max.poll.records or increase max.poll.interval.ms.
- Balance between throughput and heartbeat frequency.


```text
spring:
- kafka:
- consumer:
- max-poll-records:
- 50  # reduce if processing is slow
- properties:
- max.poll.interval.ms:
- 600000  # increase if batches take long
```


</details>

## 24. What is the difference between earliest, latest, and none for auto.offset.reset?

**Difficulty:** 🟡 Important

<details>
<summary><strong>Show Answer</strong></summary>


- auto.offset.reset controls what happens when a consumer group has no committed offset (first run or offset expired):
- earliest — start from the beginning of the topic (consume all historical messages).
- latest (default) — start from the newest messages (skip history).
- none — throw exception if no offset found (fail fast).
- Use earliest for new consumers that must process all history.
- Use latest for real-time consumers that only care about new events.


```text
spring:
- kafka:
- consumer:
- auto-offset-reset:
- earliest  # or latest
```


</details>

## 25. What is the difference between assign() and subscribe() in Kafka consumer?

**Difficulty:** 🔴 Advanced

<details>
<summary><strong>Show Answer</strong></summary>


- subscribe(topics):
- consumer group management — Kafka assigns partitions automatically via rebalancing.
- The group coordinator manages membership.
- Use this for normal consumer group scenarios.
- assign(partitions):
- manual partition assignment — no consumer group coordination.
- Consumer directly owns specific partitions.
- Use for:
- admin tools, replay scenarios, exactly-once with external offset storage, low-level control.
- No rebalancing, no group coordinator overhead.


```text
// subscribe - group managed
- consumer.subscribe(Arrays.asList("orders"));

- // assign - manual
- TopicPartition tp = new TopicPartition("orders", 0);
- consumer.assign(Collections.singletonList(tp));
- consumer.seek(tp, specificOffset);
```


</details>

## 26. How does Cooperative Sticky rebalancing differ from Eager rebalancing?

**Difficulty:** 🔴 Advanced

<details>
<summary><strong>Show Answer</strong></summary>


- Eager (default in older Kafka):
- ALL consumers revoke ALL partition assignments at start of rebalance, then get reassigned.
- Causes complete processing stop during rebalance.
- Cooperative Sticky (Kafka 2.4+, CooperativeStickyAssignor):
- only partitions that need to move are revoked and reassigned.
- Consumers keep their non-moved partitions and keep processing.
- Much lower disruption.
- Enable with partition.assignment.strategy=CooperativeStickyAssignor.
- Incremental Cooperative rebalancing requires all consumers in the group to use the same strategy.


```text
properties:
- partition.assignment.strategy:
- org.apache.kafka.clients.consumer.CooperativeStickyAssignor
```


</details>


# Spring Kafka

## 27. What is @KafkaListener and what are its key attributes?

**Difficulty:** 🟢 Must

<details>
<summary><strong>Show Answer</strong></summary>


- @KafkaListener marks a method as a Kafka message consumer.
- Key attributes:
- topics (topic names), groupId (consumer group, overrides default), topicPattern (regex for topic names), containerFactory (custom listener container factory), concurrency (number of consumer threads), id (listener ID for management).
- The method parameters can be the payload, ConsumerRecord, Acknowledgment (for manual ack), @Header values, and @Payload.


```text
@KafkaListener(
- topics = "orders",
- groupId = "order-service",
- concurrency = "3",
- containerFactory = "kafkaListenerContainerFactory"
- )
- public void consume(
- @Payload OrderEvent event,
- @Header(KafkaHeaders.RECEIVED_PARTITION) int partition,
- @Header(KafkaHeaders.OFFSET) long offset,
- Acknowledgment ack
- ) {
- process(event);
- ack.acknowledge();
- }
```


</details>

## 28. How do you configure a KafkaTemplate in Spring Boot?

**Difficulty:** 🟢 Must

<details>
<summary><strong>Show Answer</strong></summary>


- KafkaTemplate is the main Spring Kafka class for producing messages.
- Spring Boot auto-configures it from spring.kafka.producer.* properties.
- For custom config, define a ProducerFactory and KafkaTemplate @Bean.
- Key operations:
- send(topic, value), send(topic, key, value), send(topic, partition, key, value).
- Returns CompletableFuture<SendResult<K,V>>.


```text
# Auto-configured via properties
- spring:
- kafka:
- producer:
- bootstrap-servers:
- localhost:9092
- key-serializer:
- org.apache.kafka.common.serialization.StringSerializer
- value-serializer:
- org.springframework.kafka.support.serializer.JsonSerializer

- // Inject and use
- @Autowired KafkaTemplate<String, OrderEvent> kafkaTemplate;

- kafkaTemplate.send("orders", order.getId(), order);
```


</details>

## 29. How do you configure JSON serialization/deserialization in Spring Kafka?

**Difficulty:** 🟢 Must

<details>
<summary><strong>Show Answer</strong></summary>


- Use JsonSerializer (producer) and JsonDeserializer (consumer) from spring-kafka.
- On the consumer side, set the trusted packages to allow deserialization of your classes.
- The type info can be embedded in the header (TYPE_MAPPINGS) or inferred from the target class.


```text
spring:
- kafka:
- producer:
- value-serializer:
- org.springframework.kafka.support.serializer.JsonSerializer
- consumer:
- value-deserializer:
- org.springframework.kafka.support.serializer.JsonDeserializer
- properties:
- spring.json.trusted.packages:
- "com.myapp.events"
- spring.json.type.mapping:
- "order:com.myapp.events.OrderEvent"
```


</details>

## 30. What is ConcurrentKafkaListenerContainerFactory and how does concurrency work?

**Difficulty:** 🟡 Important

<details>
<summary><strong>Show Answer</strong></summary>


- ConcurrentKafkaListenerContainerFactory creates a ConcurrentMessageListenerContainer which spawns N consumer threads (concurrency property).
- Each thread is a separate Kafka consumer in the same group, so it gets assigned its own partitions.
- Setting concurrency=3 on a topic with 6 partitions gives 3 consumer threads each handling 2 partitions.
- concurrency should not exceed partition count — extra threads will be idle.
- Configure via factory bean or spring.kafka.listener.concurrency.


```text
@Bean
- public ConcurrentKafkaListenerContainerFactory<String, OrderEvent> factory(
- ConsumerFactory<String, OrderEvent> cf) {
- var factory = new ConcurrentKafkaListenerContainerFactory<String, OrderEvent>();
- factory.setConsumerFactory(cf);
- factory.setConcurrency(3);
- factory.getContainerProperties().setAckMode(AckMode.MANUAL_IMMEDIATE);
- return factory;
- }
```


</details>

## 31. How do you consume a batch of messages instead of one at a time in Spring Kafka?

**Difficulty:** 🟡 Important

<details>
<summary><strong>Show Answer</strong></summary>


- Enable batch listener on the container factory with setBatchListener(true).
- The method receives List<ConsumerRecord<K,V>> or List<V>.
- Useful for bulk DB inserts, reducing per-message overhead.
- With manual ack, call ack.acknowledge() after processing the full batch — commits all offsets in the batch.
- Tune max.poll.records to control batch size.


```text
@Bean
- public ConcurrentKafkaListenerContainerFactory<String, OrderEvent> batchFactory(
- ConsumerFactory<String, OrderEvent> cf) {
- var factory = new ConcurrentKafkaListenerContainerFactory<String, OrderEvent>();
- factory.setBatchListener(true);
- factory.setConsumerFactory(cf);
- return factory;
- }

- @KafkaListener(topics = "orders", containerFactory = "batchFactory")
- public void consumeBatch(List<OrderEvent> events, Acknowledgment ack) {
- orderRepo.saveAll(events.stream().map(this::toEntity).toList());
- ack.acknowledge();
- }
```


</details>

## 32. How do you filter messages in a @KafkaListener before they reach your consumer method?

**Difficulty:** 🟡 Important

<details>
<summary><strong>Show Answer</strong></summary>


- Use RecordFilterStrategy on the container factory.
- Return true to discard a message (it is skipped and offset committed), false to pass it through.
- Useful for filtering by header, message type, or content without writing filter logic in the listener method.


```text
@Bean
- public ConcurrentKafkaListenerContainerFactory<String, OrderEvent> factory(
- ConsumerFactory<String, OrderEvent> cf) {
- var factory = new ConcurrentKafkaListenerContainerFactory<String, OrderEvent>();
- factory.setConsumerFactory(cf);
- factory.setRecordFilterStrategy(record ->
- record.value().getStatus().equals("CANCELLED") // discard cancelled
- );
- return factory;
- }
```


</details>

## 33. How do you implement request-reply pattern with Kafka in Spring Boot?

**Difficulty:** 🔴 Advanced

<details>
<summary><strong>Show Answer</strong></summary>


- Spring Kafka supports request-reply via ReplyingKafkaTemplate.
- Producer sends to a request topic with a REPLY_TOPIC header.
- Consumer processes the request and sends a reply to the reply topic.
- The sender blocks (with timeout) waiting for the reply correlated by a correlation ID.
- Useful for synchronous-style interactions over Kafka.
- Not recommended for high-throughput — adds latency and complexity.


```text
@Bean
- public ReplyingKafkaTemplate<String, Request, Response> replyingTemplate(
- ProducerFactory<String, Request> pf,
- ConcurrentMessageListenerContainer<String, Response> repliesContainer) {
- return new ReplyingKafkaTemplate<>(pf, repliesContainer);
- }

- // Send and wait
- RequestReplyFuture<String, Request, Response> future =
- replyingTemplate.sendAndReceive(record);
- Response response = future.get(10, TimeUnit.SECONDS).value();
```


</details>

## 34. How do you use @KafkaListener with multiple topics and different message types?

**Difficulty:** 🔴 Advanced

<details>
<summary><strong>Show Answer</strong></summary>


- Option 1:
- separate @KafkaListener methods per topic.
- Option 2:
- one listener with topicPattern regex.
- Option 3:
- use @KafkaHandler in a @KafkaListener class — Spring dispatches to the right method based on payload type (requires type info in headers).
- The class is annotated @KafkaListener(topics=...) and methods are @KafkaHandler.


```text
@KafkaListener(topics = {"orders", "payments"})
- @Component
- public class EventRouter {

- @KafkaHandler
- public void handleOrder(OrderEvent event) { ...
- }

- @KafkaHandler
- public void handlePayment(PaymentEvent event) { ...
- }

- @KafkaHandler(isDefault = true)
- public void handleUnknown(Object unknown) { ...
- }
- }
```


</details>


# Delivery Guarantees

## 35. Explain at-most-once, at-least-once, and exactly-once delivery semantics in Kafka.

**Difficulty:** 🟢 Must

<details>
<summary><strong>Show Answer</strong></summary>


- At-most-once:
- commit offset before processing.
- If processing fails, message is skipped.
- No duplicates but possible data loss.
- At-least-once:
- commit offset after processing.
- If app crashes after processing but before commit, message is reprocessed.
- No data loss but possible duplicates — consumer must be idempotent.
- Exactly-once:
- using Kafka transactions (idempotent producer + transactional consumer) or the Outbox pattern — message is processed exactly once with no loss and no duplicates.
- Most complex but strongest guarantee.


</details>

## 36. How do you achieve exactly-once semantics in a Spring Boot Kafka application?

**Difficulty:** 🟢 Must

<details>
<summary><strong>Show Answer</strong></summary>


- Two approaches:
- 1.
- Kafka EOS transactions:
- set transactional.id on producer, enable.idempotence=true, use isolation.level=read_committed on consumer.
- Use KafkaTransactionManager with @Transactional for consume-process-produce atomicity.
- 2.
- Outbox pattern (recommended for DB writes):
- write to DB and outbox table in one DB transaction, relay outbox to Kafka separately.
- Combine with idempotency key check on consumer side for robustness.


```text
# Producer - enable transactions
- spring.kafka.producer.transaction-id-prefix=tx-

# Consumer - only read committed
- spring.kafka.consumer.properties.isolation.level=read_committed

- @Transactional("kafkaTransactionManager")
- @KafkaListener(topics = "orders")
- public void process(OrderEvent event) {
- orderService.save(event);      // DB write
- kafkaTemplate.send("processed-orders", event); // Kafka write
- // Both succeed or both roll back
- }
```


</details>

## 37. What is idempotent consumer design and why is it necessary with Kafka?

**Difficulty:** 🟡 Important

<details>
<summary><strong>Show Answer</strong></summary>


- Kafka at-least-once delivery means your consumer may receive the same message more than once (after crashes, rebalances, or retries).
- An idempotent consumer produces the same result regardless of how many times it processes the same message.
- Techniques:
- 1.
- Unique constraint in DB (natural idempotency — insert fails silently on duplicate).
- 2.
- Check-then-act:
- store processed event IDs (in DB or Redis), skip if already processed.
- 3.
- Upsert instead of insert — always safe to re-apply.


```text
@Transactional
- @KafkaListener(topics = "orders")
- public void consume(OrderEvent event) {
- // Idempotent:
- skip if already processed
- if (processedEventRepo.existsByEventId(event.getEventId())) {
- return;
- }
- orderService.createOrder(event);
- processedEventRepo.save(new ProcessedEvent(event.getEventId()));
- }
```


</details>


# Error Handling

## 38. What is a Dead Letter Topic (DLT) in Spring Kafka and how do you configure it?

**Difficulty:** 🟢 Must

<details>
<summary><strong>Show Answer</strong></summary>


- A DLT is a topic where messages that fail all retry attempts are sent instead of being discarded or blocking the consumer.
- In Spring Kafka, configure DefaultErrorHandler with DeadLetterPublishingRecoverer.
- Failed messages are sent to <original-topic>.DLT with headers containing the original exception, topic, partition, and offset for debugging.


```text
@Bean
- public DefaultErrorHandler errorHandler(
- KafkaTemplate<String, Object> template) {
- var recoverer = new DeadLetterPublishingRecoverer(template,
- (record, ex) -> new TopicPartition(record.topic() + ".DLT", -1)
- );
- var backoff = new ExponentialBackOff(1000L, 2.0);
- backoff.setMaxAttempts(3);
- return new DefaultErrorHandler(recoverer, backoff);
- }
```


</details>

## 39. What is the difference between retryable and non-retryable exceptions in Spring Kafka error handling?

**Difficulty:** 🟢 Must

<details>
<summary><strong>Show Answer</strong></summary>


- Retryable exceptions:
- transient failures that may succeed on retry (DB connection timeout, temporary network issue, optimistic locking).
- Spring Kafka retries these with backoff.
- Non-retryable exceptions:
- business logic errors that will never succeed on retry (invalid message format, validation failure, NullPointerException).
- Retrying these wastes time — send directly to DLT.
- Configure in DefaultErrorHandler with addNotRetryableExceptions() and addRetryableExceptions().


```text
@Bean
- public DefaultErrorHandler errorHandler(KafkaTemplate<String, Object> template) {
- var handler = new DefaultErrorHandler(
- new DeadLetterPublishingRecoverer(template),
- new FixedBackOff(1000L, 3L)
- );
- handler.addNotRetryableExceptions(
- ValidationException.class,
- InvalidMessageException.class
- );
- return handler;
- }
```


</details>

## 40. How do you implement exponential backoff retry for Kafka consumers in Spring Boot?

**Difficulty:** 🟡 Important

<details>
<summary><strong>Show Answer</strong></summary>


- Use ExponentialBackOff with DefaultErrorHandler.
- The retry delay doubles on each attempt:
- 1s → 2s → 4s → DLT.
- This prevents hammering a downstream service that is recovering.
- Also consider ExponentialBackOffWithMaxRetries to set a max attempt count.
- For Spring Boot 2.x, SeekToCurrentErrorHandler was used; in 3.x it is replaced by DefaultErrorHandler.


```text
@Bean
- public DefaultErrorHandler errorHandler(
- KafkaTemplate<String, Object> template) {
- var backoff = new ExponentialBackOffWithMaxRetries(5);
- backoff.setInitialInterval(1_000L);  // 1s
- backoff.setMultiplier(2.0);          // doubles each time
- backoff.setMaxInterval(60_000L);     // cap at 60s
- return new DefaultErrorHandler(
- new DeadLetterPublishingRecoverer(template), backoff
- );
- }
```


</details>

## 41. What is a "poison pill" message and how do you handle it?

**Difficulty:** 🟡 Important

<details>
<summary><strong>Show Answer</strong></summary>


- A poison pill is a message that consistently causes the consumer to fail (invalid format, data that triggers a bug).
- Without a DLT, the consumer retries forever and blocks the entire partition — all subsequent messages are stuck.
- Handle with:
- 1.
- DLT after N retries (most important).
- 2.
- Monitor DLT and alert on messages landing there.
- 3.
- Build a DLT replay tool to fix and re-process DLT messages after the bug is fixed.
- 4.
- Add schema validation (Avro + Schema Registry) to reject malformed messages at the producer side.


</details>

## 42. How do you build a DLT replay mechanism to reprocess failed messages after fixing the bug?

**Difficulty:** 🔴 Advanced

<details>
<summary><strong>Show Answer</strong></summary>


- 1.
- Consumer for the DLT topic reads failed messages.
- 2.
- Apply fix/transformation logic to the message (or just re-publish as-is if the consumer bug is now fixed).
- 3.
- Send to the original topic with the original key.
- 4.
- Monitor offset progress to confirm all DLT messages are replayed.
- In Spring Boot:
- create a separate @KafkaListener on the DLT topic, controlled by a feature flag so it only runs when you want to replay.
- The DLT headers contain the original topic/partition/offset for traceability.


```text
@KafkaListener(topics = "orders.DLT", groupId = "dlt-replayer",
- containerFactory = "dltFactory")
- public void replayDlt(
- @Payload OrderEvent event,
- @Header(KafkaHeaders.DLT_ORIGINAL_TOPIC) String origTopic) {
- log.info("Replaying DLT message to {}", origTopic);
- kafkaTemplate.send(origTopic, event.getId(), event);
- }
```


</details>


# Microservice Patterns

## 43. What is the Outbox pattern and how does it solve the dual-write problem?

**Difficulty:** 🟢 Must

<details>
<summary><strong>Show Answer</strong></summary>


- Dual-write problem:
- writing to DB and Kafka in separate operations means one can succeed and the other fail — inconsistent state.
- Outbox pattern solution:
- 1.
- In the same DB transaction, write to your domain table AND an outbox table.
- 2.
- A separate relay process reads the outbox and publishes to Kafka.
- 3.
- The relay marks outbox rows as published after Kafka ACK.
- Result:
- DB write and Kafka publish are effectively atomic.
- Relay options:
- polling scheduler (@Scheduled) or CDC (Debezium reads DB binlog and publishes to Kafka — zero-polling, lower latency).


```text
@Transactional
- public void createOrder(OrderRequest req) {
- Order order = orderRepo.save(mapper.toEntity(req));
- // Same transaction — atomic with order save
- outboxRepo.save(OutboxEvent.builder()
- .topic("orders")
- .key(order.getId().toString())
- .payload(toJson(order))
- .build());
- }
```


</details>

## 44. What is Event Sourcing and how does Kafka support it?

**Difficulty:** 🟢 Must

<details>
<summary><strong>Show Answer</strong></summary>


- Event Sourcing:
- instead of storing current state in a DB, store the full sequence of events.
- Current state is derived by replaying events.
- Kafka supports this naturally:
- events are stored durably in topics, consumers replay from offset 0 to rebuild state.
- Pattern:
- every state change is published as an immutable event.
- A read model (materialized view) is built by consuming the event stream.
- The event log is the source of truth.


```text
// Every action creates an immutable event
- public void placeOrder(PlaceOrderCommand cmd) {
- OrderPlacedEvent event = new OrderPlacedEvent(cmd);
- kafkaTemplate.send("order-events", cmd.getOrderId(), event);
- // DB state is derived by consuming this topic
- }
```


</details>

## 45. What is CQRS and how does Kafka fit into it?

**Difficulty:** 🟢 Must

<details>
<summary><strong>Show Answer</strong></summary>


- CQRS (Command Query Responsibility Segregation) separates write operations (commands) from read operations (queries) using different models.
- Kafka acts as the bridge:
- 1.
- Command side writes events to Kafka.
- 2.
- Multiple read-side consumers subscribe and build denormalized read models optimized for queries.
- 3.
- Each read model (user service view, analytics view, search index) is built by its own consumer.
- Enables independent scaling of read and write sides.


```text
// Write side: publish command result as event
- kafkaTemplate.send("orders", orderId, new OrderCreatedEvent(...));

- // Read side:
- build query-optimized view
- @KafkaListener(topics = "orders")
- public void updateReadModel(OrderCreatedEvent event) {
- orderSummaryRepo.upsert(toSummary(event));
- }
```


</details>

## 46. What is the Saga pattern and how do you implement it with Kafka?

**Difficulty:** 🟡 Important

<details>
<summary><strong>Show Answer</strong></summary>


- Saga manages distributed transactions across microservices without 2PC.
- Two styles:
- Choreography (event-driven):
- each service publishes an event on success; the next service listens and reacts.
- On failure, it publishes a failure event; previous services listen and run compensating transactions.
- Orchestration:
- a central saga orchestrator sends commands to services and listens for responses.
- Kafka is ideal for choreography sagas — each service publishes and subscribes to domain events.


```text
// Order service publishes OrderCreated
- kafkaTemplate.send("order-events", new OrderCreatedEvent(orderId));

- // Payment service listens, processes, publishes result
- @KafkaListener(topics = "order-events")
- public void onOrderCreated(OrderCreatedEvent event) {
- try {
- paymentService.charge(event);
- kafkaTemplate.send("payment-events", new PaymentSucceededEvent(event.getOrderId()));
- } catch (Exception e) {
- kafkaTemplate.send("payment-events", new PaymentFailedEvent(event.getOrderId()));
- }
- }
```


</details>

## 47. How does CDC (Change Data Capture) with Debezium and Kafka work?

**Difficulty:** 🟡 Important

<details>
<summary><strong>Show Answer</strong></summary>


- Debezium reads the database transaction log (MySQL binlog, PostgreSQL WAL) and publishes every insert/update/delete as a Kafka event — without any application code changes.
- This enables:
- 1.
- Outbox pattern relay (Debezium reads the outbox table and publishes to Kafka).
- 2.
- Real-time DB replication to other systems.
- 3.
- Cache invalidation — when a DB row changes, invalidate the cache.
- 4.
- Audit log.
- Debezium runs as a Kafka Connect connector and is highly reliable — it tracks its position in the binlog.


```text
# Debezium MySQL connector config
- {
- "connector.class":
- "io.debezium.connector.mysql.MySqlConnector",
- "database.hostname":
- "mysql",
- "database.include.list":
- "mydb",
- "table.include.list":
- "mydb.outbox_events",
- "topic.prefix":
- "dbz"
- }
```


</details>

## 48. How do you implement the Claim Check pattern in Kafka for large messages?

**Difficulty:** 🔴 Advanced

<details>
<summary><strong>Show Answer</strong></summary>


- Kafka is optimized for small-to-medium messages (default max 1MB, increase is possible but not recommended for very large payloads).
- Claim Check:
- 1.
- Producer stores the large payload in blob storage (S3, Azure Blob).
- 2.
- Producer sends a small Kafka message containing only the reference (S3 URL or ID).
- 3.
- Consumer reads the Kafka message, fetches the payload from blob storage, processes it.
- This keeps Kafka performant and avoids large message overhead on brokers.


```text
// Producer
- String s3Key = s3Client.upload(largePayload);
- kafkaTemplate.send("events", new EventReference(eventId, s3Key));

- // Consumer
- @KafkaListener(topics = "events")
- public void consume(EventReference ref) {
- byte[] payload = s3Client.download(ref.getS3Key());
- process(payload);
- }
```


</details>


# Performance & Scaling

## 49. How do you scale Kafka consumers horizontally in a Spring Boot microservice?

**Difficulty:** 🟢 Must

<details>
<summary><strong>Show Answer</strong></summary>


- Scale by adding more consumer instances in the same consumer group.
- Each instance gets a subset of partitions.
- In Kubernetes:
- increase replicas in the Deployment.
- The max effective parallelism = number of partitions (extra consumers beyond partition count are idle).
- Rule:
- set partition count at topic creation to your expected max consumer count × 2–3 for headroom.
- Do not add more consumers than partitions — they will sit idle.


```text
# Scale in Kubernetes
- kubectl scale deployment order-consumer --replicas=6

# Or concurrency within one pod:
- spring:
- kafka:
- listener:
- concurrency:
- 3  # 3 consumer threads per pod
```


</details>

## 50. What is the effect of compression on Kafka producer performance?

**Difficulty:** 🟡 Important

<details>
<summary><strong>Show Answer</strong></summary>


- Compression (snappy, lz4, gzip, zstd) is applied per batch on the producer.
- Benefits:
- smaller network transfer, less broker disk usage, higher throughput.
- Snappy and lz4 are fast with moderate compression — best for most cases.
- Gzip has higher compression ratio but more CPU cost.
- zstd (Kafka 2.1+) is the best balance of ratio and speed.
- Compression is transparent to consumers — they decompress automatically.
- Enable with compression.type=snappy on the producer.


```text
spring:
- kafka:
- producer:
- properties:
- compression.type:
- snappy
- batch-size:
- 65536
- linger.ms:
- 20
```


</details>

## 51. How do you monitor Kafka consumer performance in a Spring Boot app?

**Difficulty:** 🟡 Important

<details>
<summary><strong>Show Answer</strong></summary>


- Key metrics to monitor:
- 1.
- kafka.consumer.records-lag (consumer lag per partition — most important).
- 2.
- kafka.consumer.fetch-rate (how often the consumer polls).
- 3.
- kafka.consumer.records-consumed-rate (messages per second).
- 4.
- Processing time per message (add a Timer metric).
- 5.
- kafka.producer.record-send-rate.
- In Spring Boot:
- Micrometer auto-instruments Kafka consumer and producer metrics via spring-kafka.
- Export to Prometheus/Grafana.
- Alert on lag > threshold and processing time > SLA.


```text
# Actuator Kafka metrics
- GET /actuator/metrics/kafka.consumer.records-lag
- GET /actuator/metrics/kafka.consumer.records-consumed-rate

# Custom processing timer
- @Timed("order.processing.time")
- @KafkaListener(topics = "orders")
- public void consume(OrderEvent event) { ...
- }
```


</details>

## 52. What is the fetch.min.bytes and fetch.max.wait.ms consumer configuration and how does it affect throughput?

**Difficulty:** 🔴 Advanced

<details>
<summary><strong>Show Answer</strong></summary>


- fetch.min.bytes (default 1 byte):
- minimum data the broker waits to accumulate before responding to a fetch request.
- Increasing to 1MB makes the broker wait until it has 1MB of data, reducing the number of fetch requests and improving throughput at the cost of latency.
- fetch.max.wait.ms (default 500ms):
- max time broker waits if fetch.min.bytes is not met.
- Together they control the throughput/latency tradeoff on the consumer side.
- For batch processing consumers, increase both.


```text
spring:
- kafka:
- consumer:
- fetch-min-size:
- 1048576  # 1MB
- fetch-max-wait:
- 500
- properties:
- max.partition.fetch.bytes:
- 10485760  # 10MB per partition
```


</details>


# Security

## 53. What are the security mechanisms available in Kafka?

**Difficulty:** 🟢 Must

<details>
<summary><strong>Show Answer</strong></summary>


- 1.
- Encryption (TLS/SSL):
- encrypts data in transit between clients and brokers.
- Configure ssl.* properties.
- 2.
- Authentication:
- SASL/PLAIN (username/password), SASL/SCRAM-SHA-256/512 (hashed credentials), SASL/OAUTHBEARER (OAuth2 tokens), SASL/GSSAPI (Kerberos for enterprise).
- 3.
- Authorization:
- Kafka ACLs (Access Control Lists) — control which principals can read/write to which topics.
- 4.
- Audit logging:
- track who accessed what.
- In Spring Boot, configure security.protocol and sasl.* properties.


```text
spring:
- kafka:
- properties:
- security.protocol:
- SASL_SSL
- sasl.mechanism:
- SCRAM-SHA-256
- sasl.jaas.config:
- >
- org.apache.kafka.common.security.scram.ScramLoginModule required
- username="app-user" password="${KAFKA_PASSWORD}";
```


</details>

## 54. How do you configure SSL/TLS for Kafka in Spring Boot?

**Difficulty:** 🟡 Important

<details>
<summary><strong>Show Answer</strong></summary>


- Configure the SSL truststore (CA certificate) and optionally keystore (for mutual TLS).
- In production:
- use cert files mounted from Kubernetes secrets, not hardcoded paths.
- Use security.protocol=SSL for encryption only, SASL_SSL for both encryption and authentication.


```text
spring:
- kafka:
- ssl:
- trust-store-location:
- classpath:kafka.truststore.jks
- trust-store-password:
- ${KAFKA_TRUSTSTORE_PASSWORD}
- key-store-location:
- classpath:kafka.keystore.jks
- key-store-password:
- ${KAFKA_KEYSTORE_PASSWORD}
- properties:
- security.protocol:
- SSL
```


</details>

## 55. What are Kafka ACLs and how do you apply the principle of least privilege?

**Difficulty:** 🔴 Advanced

<details>
<summary><strong>Show Answer</strong></summary>


- Kafka ACLs (Access Control Lists) define what principals (users/service accounts) can do on which resources (topics, consumer groups, cluster).
- Operations:
- READ, WRITE, CREATE, DELETE, DESCRIBE.
- Best practices:
- 1.
- Each microservice has its own service account.
- 2.
- Producer service:
- WRITE on its topic(s) only.
- 3.
- Consumer service:
- READ on its topic(s) + READ on its consumer group.
- 4.
- No service has CLUSTER ADMIN.
- 5.
- Automate ACL management via Terraform or Helm.


```text
# Grant write to producer service account
- kafka-acls.sh --add \\
- --allow-principal User:order-service \\
- --operation WRITE \\
- --topic orders

# Grant read to consumer
- kafka-acls.sh --add \\
- --allow-principal User:payment-service \\
- --operation READ \\
- --topic orders \\
- --group payment-consumer-group
```


</details>


# Kafka Streams

## 56. What is Kafka Streams and how is it different from a regular Kafka consumer?

**Difficulty:** 🟡 Important

<details>
<summary><strong>Show Answer</strong></summary>


- Kafka Streams is a client library for stream processing — transforming, filtering, joining, and aggregating data directly on Kafka topics.
- Differences from regular consumer:
- 1.
- Stateful operations (counts, joins, windowed aggregations) with local state stores.
- 2.
- Fault-tolerant state:
- state is backed up to a changelog Kafka topic, rebuilt on restart.
- 3.
- Exactly-once processing semantics with Kafka transactions.
- 4.
- Time windowing (tumbling, hopping, session windows).
- A regular consumer just reads and processes; Kafka Streams provides a full DSL for stream transformations.


```text
@Bean
- public KStream<String, OrderEvent> orderStream(StreamsBuilder builder) {
- return builder.stream("orders")
- .filter((k, v) -> v.getStatus() == CONFIRMED)
- .mapValues(this::enrich);
- }
```


</details>

## 57. What is a KTable in Kafka Streams?

**Difficulty:** 🟡 Important

<details>
<summary><strong>Show Answer</strong></summary>


- A KTable represents a changelog stream — a stream of updates where each new record for a key replaces the previous value.
- It is like a materialized view of the latest state per key.
- KStream = append log (every event matters).
- KTable = current state per key (latest value wins).
- KTable is backed by a compacted topic and a local RocksDB state store.
- You can join KStream and KTable to enrich events with current state.


```text
// KTable: current user preferences (latest per userId)
- KTable<String, UserProfile> profiles = builder.table("user-profiles");

- // KStream:
- incoming order events
- KStream<String, OrderEvent> orders = builder.stream("orders");

- // Join order event with user profile
- orders.join(profiles,
- (order, profile) -> enrich(order, profile))
- .to("enriched-orders");
```


</details>

## 58. What is a windowed aggregation in Kafka Streams and what window types are available?

**Difficulty:** 🔴 Advanced

<details>
<summary><strong>Show Answer</strong></summary>


- Windowed aggregations group events within a time window and compute aggregates (count, sum).
- Window types:
- 1.
- Tumbling window:
- fixed non-overlapping intervals (e.g.
- count events per 1-minute window).
- 2.
- Hopping window:
- fixed size but overlapping (e.g.
- count in the last 5 minutes, updated every 1 minute).
- 3.
- Session window:
- dynamic size grouped by inactivity gap (events within N seconds of each other form one session).
- Results are emitted per window per key.


```text
KStream<String, OrderEvent> orders = builder.stream("orders");

- orders.groupByKey()
- .windowedBy(TimeWindows.ofSizeWithNoGrace(Duration.ofMinutes(1)))
- .count()
- .toStream()
- .to("order-counts-per-minute");
```


</details>


# Production Scenarios

## 59. Kafka consumer lag is growing. What are the possible causes and how do you fix each?

**Difficulty:** 🟢 Must

<details>
<summary><strong>Show Answer</strong></summary>


- Causes and fixes:
- 1.
- Processing is too slow → reduce max.poll.records, optimize the processing logic, add DB indexes, cache external lookups.
- 2.
- Not enough consumers → scale up pods or increase concurrency (up to partition count).
- 3.
- Message spikes → producers are publishing faster than consumers can handle — scale consumers ahead of spikes.
- 4.
- Downstream bottleneck → DB/API the consumer calls is slow — add circuit breaker, bulk-process, async writes.
- 5.
- Rebalancing loop → consumer crashing and rebalancing repeatedly — fix the consumer exception causing the crash.


```text
# Monitor lag
- kafka-consumer-groups.sh --describe --group my-group

# Quick scale fix
- kubectl scale deployment consumer --replicas=12
```


</details>

## 60. How do you handle a scenario where you need to replay all Kafka events from the beginning for a new service?

**Difficulty:** 🟢 Must

<details>
<summary><strong>Show Answer</strong></summary>


- 1.
- Set auto.offset.reset=earliest for the new consumer group — it will read from offset 0 of each partition.
- 2.
- If the topic retention period has passed and old messages are gone — you need an alternative:
- replay from a snapshot, or use a compacted topic that retains the latest state per key.
- 3.
- For a new service reading historical data:
- create a dedicated consumer group with a unique group-id — it gets its own offset pointer, independent of other groups.
- 4.
- If the topic is long (millions of messages), use concurrency to parallelize the catch-up.


```text
# New consumer group reads from beginning
- spring:
- kafka:
- consumer:
- group-id:
- new-analytics-service-v1  # brand new group
- auto-offset-reset:
- earliest
```


</details>

## 61. How do you safely reset a Kafka consumer group offset in production?

**Difficulty:** 🟡 Important

<details>
<summary><strong>Show Answer</strong></summary>


- Resetting offsets means consumers will re-process (shift earlier) or skip (shift later) messages.
- Steps:
- 1.
- Stop all consumers in the group first (offset reset only works on inactive groups).
- 2.
- Use kafka-consumer-groups.sh --reset-offsets.
- Options:
- --to-earliest (replay all), --to-latest (skip all), --to-offset <N> (specific offset), --to-datetime (messages after a timestamp), --shift-by -100 (go back 100 messages).
- 3.
- Restart consumers.
- 4.
- Monitor lag to confirm expected behavior.


```text
# Reset to specific timestamp (e.g. re-process last 1 hour)
- kafka-consumer-groups.sh \\
- --bootstrap-server localhost:9092 \\
- --group order-service \\
- --topic orders \\
- --reset-offsets \\
- --to-datetime 2025-07-14T10:00:00.000 \\
- --execute
```


</details>

## 62. A Kafka broker goes down. What happens to producers and consumers and how does Kafka recover?

**Difficulty:** 🟡 Important

<details>
<summary><strong>Show Answer</strong></summary>


- When a broker goes down:
- 1.
- Partitions it was leader for need a new leader — the Controller detects the failure via ZooKeeper/KRaft heartbeat and elects a new leader from ISR.
- 2.
- Producers retrying:
- if acks=all and the broker was in ISR, producer retries until the new leader is elected (usually seconds).
- 3.
- Consumers:
- brief pause while metadata is refreshed and the new leader is discovered.
- 4.
- Recovery time depends on unclean.leader.election.enable (false recommended for durability — only elect from ISR).
- With replication-factor=3 and min.insync.replicas=2, cluster can tolerate 1 broker failure with no data loss.


```text
# Verify topic replication health
- kafka-topics.sh --describe --topic orders \\
- --bootstrap-server localhost:9092
# Look for: Leader, Replicas, Isr columns
```


</details>

## 63. How do you perform a zero-downtime Kafka topic partition increase and consumer migration?

**Difficulty:** 🔴 Advanced

<details>
<summary><strong>Show Answer</strong></summary>


- 1.
- Increase partitions on the topic (cannot be decreased).
- 2.
- Be aware:
- existing key-to-partition mapping may change for some keys — new messages with those keys go to different partitions, breaking per-key ordering temporarily.
- 3.
- If strict ordering is critical:
- use a migration period — produce to old topic, gradually shift to new.
- 4.
- Trigger a consumer rebalance by restarting consumers after the partition increase — new partitions are assigned.
- 5.
- Monitor lag on new partitions to confirm they are being consumed.
- 6.
- For topic rename or restructure:
- create a new topic and use a migration consumer to copy old → new.


```text
# Increase partitions
- kafka-topics.sh --alter \\
- --topic orders \\
- --partitions 12 \\
- --bootstrap-server localhost:9092

# Verify
- kafka-topics.sh --describe --topic orders
```


</details>

## 64. How do you design a Kafka-based system for a high-stakes financial event (e.g. payment processing) where no message can be lost or duplicated?

**Difficulty:** 🔴 Advanced

<details>
<summary><strong>Show Answer</strong></summary>


- Layered approach:
- 1.
- Producer:
- enable.idempotence=true, acks=all, retries=MAX_VALUE, transactional.id set.
- 2.
- Topic:
- replication-factor=3, min.insync.replicas=2.
- 3.
- Consumer:
- isolation.level=read_committed, manual offset commit after DB write.
- 4.
- Idempotency key:
- store payment event ID in DB with unique constraint — duplicate processing fails silently.
- 5.
- Outbox pattern:
- DB write + outbox insert in one transaction, Debezium relays outbox to Kafka.
- 6.
- DLT with alerting:
- any DLT message triggers immediate on-call alert.
- 7.
- Reconciliation job:
- daily job comparing source system with processed events to detect any gaps.


```text
# Full producer config for financial system
- spring:
- kafka:
- producer:
- acks:
- all
- retries:
- 2147483647
- properties:
- enable.idempotence:
- true
- transactional.id:
- payment-producer
- max.in.flight.requests.per.connection:
- 5
- consumer:
- enable-auto-commit:
- false
- properties:
- isolation.level:
- read_committed
```


</details>
