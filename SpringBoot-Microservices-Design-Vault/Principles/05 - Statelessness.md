---
tags: [microservices, springboot, principle, scalability]
principle: 5
status: to-review
created: 2026-11-04
---

# 5. Statelessness

> Service instances keep no client session state in memory; any instance can serve any request.

Back to [[00 - Index (MOC)]]

## Why it matters
Enables horizontal scaling, rolling deploys and fast recovery.

## How to apply in Spring Boot
- Use token-based auth (JWT) instead of HTTP sessions.
- Externalise state to Redis, the database or a broker.
- Make operations idempotent so retries are safe.
- Avoid local file storage; use object storage (S3, GCS).

## Example
```java
@PostMapping("/payments")
ResponseEntity<Payment> pay(@RequestHeader("Idempotency-Key") String key,
                            @RequestBody PayRequest req) {
    return ResponseEntity.ok(paymentService.payOnce(key, req)); // dedupe by key in DB/Redis
}
```

## Common pitfalls
- In-memory caches that diverge between instances.
- Sticky sessions hiding state problems.

## Checklist
- [ ] Applied in our current service(s)
- [ ] Covered by automated tests or pipeline checks
- [ ] Documented in an ADR if we deviated

## My notes
-

## Related
- [[14 - Containerization and Elastic Scalability]]
- [[08 - Service Discovery and API Gateway]]

---
Previous: [[04 - Loose Coupling and High Cohesion]] | Next: [[06 - Externalized Configuration]]
