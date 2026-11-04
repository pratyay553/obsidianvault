---
tags: [microservices, springboot, principle, data]
principle: 2
status: to-review
created: 2026-11-04
---

# 2. Database per Service

> Each service owns its private data store. No other service reads or writes its tables directly.

Back to [[00 - Index (MOC)]]

## Why it matters
Shared databases create hidden coupling; schema changes then break other teams. Private data lets each service pick the best store (polyglot persistence).

## How to apply in Spring Boot
- One schema (or database) per service with its own credentials.
- Expose data only through APIs or events.
- Use Flyway or Liquibase for versioned migrations per service.
- Replicate needed data via events into a local read model (CQRS-style).

## Example
```yaml
# application.yml of order-service
spring:
  datasource:
    url: jdbc:postgresql://orders-db:5432/orders
    username: ${ORDERS_DB_USER}
    password: ${ORDERS_DB_PASSWORD}
  flyway:
    enabled: true
    locations: classpath:db/migration
```

## Common pitfalls
- Cross-service JOINs: replace with API composition or a read model.
- Shared 'common' DB user across services.

## Checklist
- [ ] Applied in our current service(s)
- [ ] Covered by automated tests or pipeline checks
- [ ] Documented in an ADR if we deviated

## My notes
-

## Related
- [[01 - Single Responsibility and Bounded Context]]
- [[10 - Distributed Data Consistency Saga and Outbox]]

---
Previous: [[01 - Single Responsibility and Bounded Context]] | Next: [[03 - APIFirst Design]]
