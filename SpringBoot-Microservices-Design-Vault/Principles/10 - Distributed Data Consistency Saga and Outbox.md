---
tags: [microservices, springboot, principle, data]
principle: 10
status: to-review
created: 2026-11-04
---

# 10. Distributed Data Consistency (Saga and Outbox)

> Without distributed ACID transactions, coordinate multi-service workflows with sagas and publish events reliably with the outbox pattern.

Back to [[00 - Index (MOC)]]

## Why it matters
Business operations span services; you need eventual consistency with compensation.

## How to apply in Spring Boot
- Choreography saga for simple flows, orchestration (e.g. Temporal, Camunda) for complex ones.
- Define a compensating action for every step (`CancelPayment`).
- Write the entity and the outbox row in one local transaction; relay with Debezium or a poller.
- Accept eventual consistency in the UX (pending states).

## Example
```java
@Transactional
public void placeOrder(Order order) {
    orders.save(order);
    outbox.save(new OutboxEvent("OrderPlaced", order.id(), toJson(order))); // same TX
}
```

## Common pitfalls
- Dual writes (DB then broker) that lose events.
- Missing compensation for partial failures.

## Checklist
- [ ] Applied in our current service(s)
- [ ] Covered by automated tests or pipeline checks
- [ ] Documented in an ADR if we deviated

## My notes
-

## Related
- [[02 - Database per Service]]
- [[09 - Asynchronous and EventDriven Communication]]

---
Previous: [[09 - Asynchronous and EventDriven Communication]] | Next: [[11 - Observability]]
