---
tags: [microservices, springboot, principle, architecture]
principle: 4
status: to-review
created: 2026-11-04
---

# 4. Loose Coupling and High Cohesion

> Services depend on abstractions and contracts, not on each other's internals. Related behaviour lives together.

Back to [[00 - Index (MOC)]]

## Why it matters
Loose coupling limits blast radius of change; high cohesion keeps each service understandable.

## How to apply in Spring Boot
- Prefer events over synchronous calls where the business allows.
- Use hexagonal architecture (ports and adapters) inside the service.
- Wrap external clients behind an interface so they can be swapped or mocked.
- Apply an anti-corruption layer when integrating legacy or third-party models.

## Example
```java
public interface PaymentPort {                 // port (domain)
    PaymentResult charge(OrderId id, Money amount);
}

@Component
class PaymentRestAdapter implements PaymentPort {   // adapter (infra)
    private final RestClient client;
    PaymentRestAdapter(RestClient.Builder b) { this.client = b.baseUrl("http://payment-service").build(); }
    public PaymentResult charge(OrderId id, Money amount) {
        return client.post().uri("/payments").body(new ChargeRequest(id, amount))
                     .retrieve().body(PaymentResult.class);
    }
}
```

## Common pitfalls
- Chains of synchronous calls A -> B -> C -> D.
- Sharing a DTO library that couples release cycles.

## Checklist
- [ ] Applied in our current service(s)
- [ ] Covered by automated tests or pipeline checks
- [ ] Documented in an ADR if we deviated

## My notes
-

## Related
- [[01 - Single Responsibility and Bounded Context]]
- [[09 - Asynchronous and EventDriven Communication]]

---
Previous: [[03 - APIFirst Design]] | Next: [[05 - Statelessness]]
