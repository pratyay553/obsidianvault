---
tags: [microservices, springboot, principle, messaging]
principle: 9
status: to-review
created: 2026-11-04
---

# 9. Asynchronous and Event-Driven Communication

> Publish facts (events) about what happened; let interested services react on their own time.

Back to [[00 - Index (MOC)]]

## Why it matters
Decouples services in time and availability, and supports scale and extensibility.

## How to apply in Spring Boot
- Use Kafka or RabbitMQ via Spring Cloud Stream or Spring Kafka.
- Events are immutable, past-tense: `OrderPlaced`.
- Version event schemas (Avro/JSON Schema + Schema Registry).
- Make consumers idempotent and add dead-letter topics.

## Example
```java
@Component
class OrderEvents {
    private final KafkaTemplate<String, OrderPlaced> kafka;
    OrderEvents(KafkaTemplate<String, OrderPlaced> kafka) { this.kafka = kafka; }
    void publish(OrderPlaced e) { kafka.send("orders.placed", e.orderId(), e); }
}

@KafkaListener(topics = "orders.placed", groupId = "inventory")
void on(OrderPlaced e) { inventory.reserve(e.items()); }
```

## Common pitfalls
- Treating events as remote procedure calls.
- No ordering or duplicate-handling strategy.

## Checklist
- [ ] Applied in our current service(s)
- [ ] Covered by automated tests or pipeline checks
- [ ] Documented in an ADR if we deviated

## My notes
-

## Related
- [[04 - Loose Coupling and High Cohesion]]
- [[10 - Distributed Data Consistency Saga and Outbox]]

---
Previous: [[08 - Service Discovery and API Gateway]] | Next: [[10 - Distributed Data Consistency Saga and Outbox]]
