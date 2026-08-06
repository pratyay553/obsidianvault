# 76 - Kafka Consumer Threads

Tags: #frameworks #section13
Related: [[44 - Thread Pool]] | [[74 - Spring Boot Threading Model]] | [[39 - Producer Consumer]]
Back to: [[00 - Java Multithreading Home]]

---

## Objective
Understand how Kafka consumers use threads — the single-threaded consumer rule, partition parallelism, and safe multi-threaded processing patterns.

---

## Beginner Explanation
Kafka gives you messages in order per partition. One consumer thread reads from the broker. You can process messages in parallel using worker threads — but you must be careful about offset commits.

---

## Analogy
A single mail carrier (consumer thread) delivers letters (messages) to a sorting room. Inside the sorting room, many clerks (worker threads) process the letters in parallel. The mail carrier must not mark a letter as "delivered" until the clerk finishes processing it.

---

## Technical Deep Dive

### The cardinal rule: KafkaConsumer is NOT thread-safe
One `KafkaConsumer` instance must be used by exactly one thread. Calling `poll()` from multiple threads throws `ConcurrentModificationException`.

### Parallelism model in Kafka

```
N partitions → N consumer threads (one consumer per thread)
             OR
1 consumer thread + N worker threads (process in parallel, commit carefully)
```

**Option 1 — One consumer per thread (simplest)**
```java
for (int i = 0; i < numPartitions; i++) {
    new Thread(new KafkaConsumerRunnable(topic, groupId)).start();
}
```
Each thread has its own `KafkaConsumer`. Parallelism = number of partitions.

**Option 2 — One consumer + worker pool (more throughput)**
```java
// Consumer thread:
while (running) {
    ConsumerRecords<K, V> records = consumer.poll(Duration.ofMillis(100));
    for (ConsumerRecord<K, V> record : records) {
        workerPool.submit(() -> process(record)); // async processing
    }
    // DANGER: must not commit until workers finish!
}
```
Offset commit must wait for all workers to finish processing → use a `CompletableFuture` barrier or `CountDownLatch`.

### Spring Kafka threading
```java
@KafkaListener(topics = "orders", concurrency = "4")
// concurrency=4 → 4 consumer threads, each on a separate partition
public void onMessage(String message) {
    // Each invocation is on its own thread
    // Spring handles offset commits automatically
}
```

### Offset commit safety
- **Auto-commit** (`enable.auto.commit=true`): commits on a timer — risk of reprocessing or data loss if crash
- **Manual sync commit** (`commitSync()`): safe but blocks consumer thread
- **Manual async commit** (`commitAsync()`): non-blocking but requires careful retry logic

```java
consumer.poll(Duration.ofMillis(100)).forEach(record -> {
    processRecord(record);
    consumer.commitSync(Collections.singletonMap(
        new TopicPartition(record.topic(), record.partition()),
        new OffsetAndMetadata(record.offset() + 1)
    ));
});
```

---

## Code

```java
import org.apache.kafka.clients.consumer.*;
import java.util.*;
import java.util.concurrent.*;

public class KafkaThreadingDemo {
    static final String TOPIC = "orders";
    static final String GROUP = "order-processors";

    // Option 1: one consumer per thread
    static class ConsumerThread implements Runnable {
        private final KafkaConsumer<String, String> consumer;
        private volatile boolean running = true;

        ConsumerThread() {
            Properties props = new Properties();
            props.put("bootstrap.servers", "localhost:9092");
            props.put("group.id", GROUP);
            props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
            props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
            props.put("enable.auto.commit", "false");
            this.consumer = new KafkaConsumer<>(props);
        }

        @Override
        public void run() {
            consumer.subscribe(List.of(TOPIC));
            while (running) {
                ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));
                for (ConsumerRecord<String, String> record : records) {
                    processRecord(record);
                }
                consumer.commitSync(); // commit after processing batch
            }
            consumer.close();
        }

        void processRecord(ConsumerRecord<String, String> record) {
            System.out.printf("Thread=%s partition=%d offset=%d key=%s%n",
                Thread.currentThread().getName(),
                record.partition(), record.offset(), record.key());
        }

        void stop() { running = false; }
    }

    // Spring Kafka equivalent (annotation-based):
    /*
    @KafkaListener(topics = "orders", concurrency = "3")
    public void listen(String message, Acknowledgment ack) {
        processMessage(message);
        ack.acknowledge(); // manual commit
    }
    */
}
```

---

## Interview Questions

**Q: Is `KafkaConsumer` thread-safe?**
> No. A single `KafkaConsumer` instance must only be used by one thread. The standard pattern is one consumer per thread or one consumer with a worker pool for processing.

**Q: How does partition count relate to consumer parallelism?**
> Maximum parallelism = number of partitions. If you have 10 partitions in a topic and a consumer group with 10 consumers, each consumer handles one partition. Extra consumers beyond partition count are idle.

**Q: What is the risk of auto-commit in Kafka?**
> If the consumer crashes after auto-commit but before processing is complete, those messages are "lost" (won't be reprocessed). If the consumer crashes after processing but before auto-commit, messages are reprocessed ("at-least-once"). Manual commit gives explicit control.

---

## Cheat Sheet
| Concept | Detail |
|---|---|
| Thread safety | `KafkaConsumer` — NOT thread-safe |
| Parallelism unit | 1 consumer per partition |
| Max parallelism | = number of partitions |
| Spring concurrency | `@KafkaListener(concurrency = "N")` |
| Auto-commit risk | At-least-once or data loss |
| Safe commit | `commitSync()` after processing |
