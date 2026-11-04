---
tags: [microservices, springboot, principle, resilience]
principle: 7
status: to-review
created: 2026-11-04
---

# 7. Resilience and Design for Failure

> Assume every remote call can be slow or fail. Contain failures with timeouts, retries, circuit breakers, bulkheads and fallbacks.

Back to [[00 - Index (MOC)]]

## Why it matters
In a distributed system partial failure is normal; resilience prevents cascading outages.

## How to apply in Spring Boot
- Always set connect and read timeouts.
- Retry only idempotent calls, with exponential backoff and jitter.
- Add a circuit breaker with a meaningful fallback (Resilience4j).
- Use bulkheads (thread-pool or semaphore) to isolate dependencies.
- Test with chaos experiments.

## Example
```java
@CircuitBreaker(name = "payment", fallbackMethod = "pendingPayment")
@Retry(name = "payment")
@TimeLimiter(name = "payment")
public CompletableFuture<PaymentResult> charge(Order o) {
    return CompletableFuture.supplyAsync(() -> paymentPort.charge(o.id(), o.total()));
}

CompletableFuture<PaymentResult> pendingPayment(Order o, Throwable t) {
    return CompletableFuture.completedFuture(PaymentResult.pending(o.id()));
}
```

## Common pitfalls
- Retry storms amplifying an outage.
- Fallbacks that hide errors with no alerting.

## Checklist
- [ ] Applied in our current service(s)
- [ ] Covered by automated tests or pipeline checks
- [ ] Documented in an ADR if we deviated

## My notes
-

## Related
- [[08 - Service Discovery and API Gateway]]
- [[11 - Observability]]

---
Previous: [[06 - Externalized Configuration]] | Next: [[08 - Service Discovery and API Gateway]]
