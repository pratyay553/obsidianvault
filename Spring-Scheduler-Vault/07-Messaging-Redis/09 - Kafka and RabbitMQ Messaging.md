---
tags: [messaging, kafka, rabbitmq, interview]
---

# Kafka & RabbitMQ Messaging

Related: [[08 - Spring Cloud and Microservices]] · [[10 - Redis Caching]] · [[20 - Interview Cheat Sheet]]

## 🧩 What it is
Both let services communicate **asynchronously** through messages instead of direct calls — but they solve different problems.

## 🆚 Kafka vs RabbitMQ

| | Kafka | RabbitMQ |
|---|---|---|
| Model | Distributed **log** — messages are stored & can be replayed | Traditional **message queue/broker** — messages are consumed & typically removed |
| Best for | High-throughput event streaming, event sourcing, analytics pipelines | Task queues, request/reply, complex routing (exchanges/bindings) |
| Ordering | Guaranteed *within a partition* | Guaranteed within a single queue |
| Message retention | Configurable retention window; consumers can "replay" | Message gone once acknowledged/consumed (by default) |
| In this module | `@KafkaListener`, `KafkaTemplate` | `@RabbitListener`, `RabbitTemplate`, plus **Spring Cloud Bus** transport (see [[08 - Spring Cloud and Microservices]]) |

## 🛠️ Kafka Code

```java
@Service
public class TripEventProducer {
    private final KafkaTemplate<String, TripEvent> kafkaTemplate;

    public void publish(TripEvent event) {
        kafkaTemplate.send("trip-events", event.getTripId(), event);
    }
}

@Component
public class TripEventConsumer {
    @KafkaListener(topics = "trip-events", groupId = "scheduler-group")
    public void onMessage(TripEvent event) {
        // process event
    }
}
```

## 🛠️ RabbitMQ Code

```java
@Service
public class AlertProducer {
    private final RabbitTemplate rabbitTemplate;

    public void sendAlert(Alert alert) {
        rabbitTemplate.convertAndSend("alerts.exchange", "alert.routing.key", alert);
    }
}

@Component
public class AlertConsumer {
    @RabbitListener(queues = "alerts.queue")
    public void onAlert(Alert alert) { ... }
}
```

## ❓ Interview Q&A

**Q: Why might this scheduler module use Kafka for some things and RabbitMQ for others?**
A: Kafka fits high-volume event streams (e.g., continuous GPS/trip data from `LindeClient`/`AmnexClient`) where replay and throughput matter. RabbitMQ fits the Spring Cloud Bus config-refresh mechanism and simpler task/notification queues where complex routing (exchanges, topic/direct/fanout) is more natural than a log.

**Q: What is a Kafka "consumer group" and why does it matter?**
A: Multiple consumer instances sharing the same `groupId` split the partitions of a topic between them — each partition is only processed by one consumer in the group. This gives you horizontal scaling of message processing while Kafka handles the partition assignment.

**Q: What happens if a `@RabbitListener` method throws an exception?**
A: By default, the message is *requeued* (or sent to a dead-letter exchange if configured), so you don't lose it — but a bug can cause infinite requeue loops ("poison message") unless retry limits/dead-lettering are configured.

**Q: How do you guarantee message ordering in Kafka?**
A: Ordering is only guaranteed *within a single partition* — use a consistent partition key (e.g., trip ID) so all events for the same entity go to the same partition and are processed in order.
