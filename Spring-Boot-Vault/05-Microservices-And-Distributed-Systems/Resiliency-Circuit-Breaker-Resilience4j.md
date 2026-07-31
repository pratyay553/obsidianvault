---
tags:
  - springboot
  - java
  - backend
  - interview-prep
technology: Spring Boot 3.x
level: Advanced
status: Not Started
last_reviewed: 2026-07-23
---

# 📌 Resiliency — Circuit Breaker with Resilience4j

## 💡 1-Sentence Elevator Pitch (For Interviews)
> Resilience4j's CircuitBreaker monitors downstream call failure rates and automatically stops sending requests to a failing service (OPEN state), allowing it to recover — preventing one slow service from cascading into system-wide failures.

## 🧱 Key Concepts & Mechanics

### Circuit Breaker States
```
CLOSED → (failure threshold exceeded) → OPEN → (wait duration) → HALF_OPEN
   ↑                                                                  |
   └────────── (probe succeeds) ─────────────────────────────────────┘
```
- **CLOSED**: Normal operation — calls pass through; failures are counted
- **OPEN**: Calls fail fast (no downstream call made); fallback is invoked
- **HALF_OPEN**: Limited calls allowed to probe recovery; reverts if they fail

### Resilience4j Modules

| Module | Annotation | Purpose |
|--------|-----------|---------|
| CircuitBreaker | `@CircuitBreaker` | Stop cascading failures |
| Retry | `@Retry` | Retry transient failures |
| RateLimiter | `@RateLimiter` | Throttle call rate |
| Bulkhead | `@Bulkhead` | Limit concurrent calls |
| TimeLimiter | `@TimeLimiter` | Timeout on async calls |

### Annotation Ordering (when stacked)
```
TimeLimiter → CircuitBreaker → Bulkhead → Retry → RateLimiter
```
Resilience4j applies decorators in this order — know it for interviews.

### Sliding Window Types
- `COUNT_BASED` — tracks last N calls (e.g., 10 calls, 50% failure → OPEN)
- `TIME_BASED` — tracks calls in last N seconds

## 💻 Production Code Example

```yaml
# application.yml
resilience4j:
  circuitbreaker:
    instances:
      payment-service:
        slidingWindowType: COUNT_BASED
        slidingWindowSize: 10
        failureRateThreshold: 50          # % failures to OPEN
        slowCallRateThreshold: 80         # % slow calls to also OPEN
        slowCallDurationThreshold: 2s
        waitDurationInOpenState: 30s
        permittedNumberOfCallsInHalfOpenState: 3
        automaticTransitionFromOpenToHalfOpenEnabled: true
        recordExceptions:
          - feign.FeignException
          - java.net.ConnectException
        ignoreExceptions:
          - com.example.exception.BusinessRuleException

  retry:
    instances:
      payment-service:
        maxAttempts: 3
        waitDuration: 500ms
        retryExceptions:
          - feign.RetryableException
          - java.net.SocketTimeoutException

  timelimiter:
    instances:
      payment-service:
        timeoutDuration: 3s

  bulkhead:
    instances:
      payment-service:
        maxConcurrentCalls: 20
        maxWaitDuration: 100ms
```

```java
// ── Service with stacked resilience decorators ────────────────────────
@Service
@RequiredArgsConstructor
@Slf4j
public class PaymentServiceClient {

    private final PaymentFeignClient feignClient;

    @CircuitBreaker(name = "payment-service", fallbackMethod = "chargeFailback")
    @Retry(name = "payment-service")
    @TimeLimiter(name = "payment-service")
    @Bulkhead(name = "payment-service", type = Bulkhead.Type.SEMAPHORE)
    public CompletableFuture<PaymentResult> charge(ChargeRequest req) {
        return CompletableFuture.supplyAsync(() -> feignClient.charge(req));
    }

    // Fallback — same signature + Throwable parameter
    public CompletableFuture<PaymentResult> chargeFailback(
            ChargeRequest req, Throwable ex) {
        log.warn("Payment service unavailable ({}), using fallback", ex.getMessage());
        // Publish to outbox for async retry, or return cached/default response
        return CompletableFuture.completedFuture(
            PaymentResult.pending(req.orderId(), "Queued for processing")
        );
    }
}

// ── Expose CB metrics to Actuator / Prometheus ────────────────────────
// application.yml — already wired when resilience4j-micrometer is on classpath:
// management.health.circuitbreakers.enabled: true
// management.endpoints.web.exposure.include: health,metrics,circuitbreakers

// ── Programmatic CircuitBreaker (for fine control) ────────────────────
@Service
public class ManualCbService {

    private final CircuitBreaker cb;

    public ManualCbService(CircuitBreakerRegistry registry) {
        this.cb = registry.circuitBreaker("inventory-service");
    }

    public InventoryResult checkStock(UUID productId) {
        return cb.executeSupplier(() -> inventoryClient.check(productId));
    }

    // Listen to state transitions for alerting
    @PostConstruct
    public void registerListeners() {
        cb.getEventPublisher()
            .onStateTransition(e ->
                log.warn("Circuit breaker {} transitioned {} → {}",
                    e.getCircuitBreakerName(),
                    e.getStateTransition().getFromState(),
                    e.getStateTransition().getToState()));
    }
}
```

## ⚠️ Common Pitfalls & Anti-Patterns

- **Mistake:** No fallback defined — when the circuit opens, exceptions propagate and cause cascading failures anyway.
  **Fix:** Always implement a `fallbackMethod`. Return a default, queue for async retry, or return a cached result.

- **Mistake:** Retrying on non-transient errors (e.g., `400 Bad Request`, `404 Not Found`) — wastes time and adds load.
  **Fix:** Configure `retryExceptions` and `ignoreExceptions` carefully. Only retry `5xx`, timeouts, and connection errors.

- **Mistake:** Using `@Retry` without `@CircuitBreaker` — retries keep hammering an already-failing service.
  **Fix:** Stack them: `@CircuitBreaker` + `@Retry`. The circuit breaker will eventually OPEN and stop retries altogether.

- **Mistake:** Annotated method calling itself within the same class — Resilience4j decorators (like Spring AOP) are bypassed.
  **Fix:** Call through the Spring proxy (inject `self`, or extract to another bean).

- **Mistake:** Forgetting to configure the `TimeLimiter` when using `CompletableFuture` — `@TimeLimiter` only applies to `CompletableFuture` return types.
  **Fix:** Methods with `@TimeLimiter` must return `CompletableFuture<T>`.

## ❓ Frequently Asked Interview Questions

1. **Q:** What problem does a circuit breaker solve?
   - **A:** Cascading failures. If Service A calls Service B, and B is slow/down, A's threads block waiting. Eventually A's thread pool exhausts, A starts failing too, and the failure cascades upstream. The circuit breaker detects B's failure rate and short-circuits (OPEN state), failing fast so A's threads are freed immediately.

2. **Q:** What is the difference between circuit breaker and retry?
   - **A:** Retry is for transient failures — the service is temporarily unavailable and trying again quickly might succeed. Circuit breaker is for persistent failures — after many retries, stop trying and give the service time to recover. They're complementary: retry handles blips, circuit breaker handles outages.

3. **Q:** What is a Bulkhead and what does it protect against?
   - **A:** A bulkhead limits the number of concurrent calls to a downstream service. Without it, a slow dependency could exhaust all threads in your service. The bulkhead isolates the blast radius — only the limited pool dedicated to that service suffers, not the whole application.

4. **Q:** How does `HALF_OPEN` state work?
   - **A:** After the wait duration in OPEN state, the circuit moves to HALF_OPEN and allows a limited number of probe calls (`permittedNumberOfCallsInHalfOpenState`). If those succeed above the threshold, the circuit closes (CLOSED). If they fail, it returns to OPEN.

## 🔗 Related Notes
- [[Service-Discovery-Eureka]]
- [[Inter-Service-Communication-Feign-gRPC]]
- [[API-Gateway-Spring-Cloud-Gateway]]
- [[Async-Processing-Async-CompletableFuture]]
