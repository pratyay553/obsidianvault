---
tags: [microservices, springboot, principle, domain]
principle: 1
status: to-review
created: 2026-11-04
---

# 1. Single Responsibility and Bounded Context

> Each microservice owns one business capability and one bounded context (DDD). If you cannot describe the service in one sentence without 'and', it is too big.

Back to [[00 - Index (MOC)]]

## Why it matters
Small, focused services change for one reason, can be owned by one team, and can be deployed independently.

## How to apply in Spring Boot
- Run Event Storming to find bounded contexts before writing code.
- Name services after business capabilities (`order-service`, `payment-service`), not technical layers.
- Keep package-by-feature: `com.acme.order.{api,domain,infra}`.
- Share nothing but contracts (API/event schemas), never entity classes.

## Example
```java
@RestController
@RequestMapping("/orders")
class OrderController {
    private final PlaceOrderUseCase placeOrder;   // only order concerns here

    OrderController(PlaceOrderUseCase placeOrder) { this.placeOrder = placeOrder; }

    @PostMapping
    ResponseEntity<OrderResponse> create(@Valid @RequestBody CreateOrderRequest req) {
        return ResponseEntity.status(201).body(placeOrder.execute(req));
    }
}
```

## Common pitfalls
- Nano-services: one entity per service creates chatty networks.
- Distributed monolith: services that must deploy together.

## Checklist
- [ ] Applied in our current service(s)
- [ ] Covered by automated tests or pipeline checks
- [ ] Documented in an ADR if we deviated

## My notes
-

## Related
- [[02 - Database per Service]]
- [[04 - Loose Coupling and High Cohesion]]

---
Previous: - | Next: [[02 - Database per Service]]
