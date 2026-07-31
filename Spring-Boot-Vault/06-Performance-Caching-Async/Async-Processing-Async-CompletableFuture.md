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

# 📌 Async Processing — @Async & CompletableFuture

## 💡 1-Sentence Elevator Pitch (For Interviews)
> `@Async` offloads a method call to a Spring-managed thread pool so the caller doesn't block — combined with `CompletableFuture`, you can chain async steps, run independent operations in parallel, and join results — all without blocking the HTTP request thread.

## 🧱 Key Concepts & Mechanics

### `@Async` Fundamentals
- Enables by: `@EnableAsync` on a `@Configuration` class
- Returns `void`, `Future<T>`, `CompletableFuture<T>`, or `ListenableFuture<T>`
- Spring wraps the method in an AOP proxy → submits to a configured `TaskExecutor`
- Default executor: `SimpleAsyncTaskExecutor` (creates new threads — **not for production**)
- Always configure a thread pool via `ThreadPoolTaskExecutor`

### CompletableFuture Chaining

| Method | Description |
|--------|-------------|
| `thenApply(fn)` | Transform result synchronously |
| `thenApplyAsync(fn, executor)` | Transform result on another thread |
| `thenCompose(fn)` | Flat-map to another `CompletableFuture` |
| `thenCombine(cf, fn)` | Combine two futures when both complete |
| `allOf(cf1, cf2, ...)` | Wait for all to complete |
| `anyOf(cf1, cf2, ...)` | First completed wins |
| `exceptionally(fn)` | Handle errors inline |
| `handle(fn)` | Handle both result and exception |

### Thread Pool Sizing
- I/O-bound tasks (HTTP calls, DB): large pool (50–200 threads)
- CPU-bound tasks (computation): `Runtime.getRuntime().availableProcessors()` threads
- Name threads for observability: `thread.setName("async-email-")`

## 💻 Production Code Example

```java
// ── Enable async ──────────────────────────────────────────────────────
@Configuration
@EnableAsync
public class AsyncConfig {

    @Bean("emailExecutor")
    public Executor emailExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);
        executor.setMaxPoolSize(20);
        executor.setQueueCapacity(100);
        executor.setThreadNamePrefix("async-email-");
        executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
        executor.initialize();
        return executor;
    }

    @Bean("reportExecutor")
    public Executor reportExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(2);
        executor.setMaxPoolSize(5);
        executor.setQueueCapacity(10);
        executor.setThreadNamePrefix("async-report-");
        executor.initialize();
        return executor;
    }

    // Global uncaught exception handler for @Async methods returning void
    @Bean
    public AsyncUncaughtExceptionHandler asyncUncaughtExceptionHandler() {
        return (ex, method, params) ->
            log.error("Async error in {}: {}", method.getName(), ex.getMessage(), ex);
    }
}

// ── Async service ─────────────────────────────────────────────────────
@Service
@Slf4j
@RequiredArgsConstructor
public class NotificationService {

    private final EmailClient emailClient;
    private final SmsClient smsClient;
    private final PushClient pushClient;

    // Fire-and-forget — void return, caller doesn't wait
    @Async("emailExecutor")
    public void sendWelcomeEmail(User user) {
        try {
            emailClient.send(user.getEmail(), "Welcome!", buildWelcomeBody(user));
        } catch (Exception e) {
            log.error("Failed to send welcome email to {}", user.getEmail(), e);
            // Don't rethrow — caller has already moved on
        }
    }

    // Return CompletableFuture — caller can optionally await
    @Async("emailExecutor")
    public CompletableFuture<EmailResult> sendAsync(EmailRequest req) {
        return CompletableFuture.completedFuture(emailClient.send(req));
    }
}

// ── Parallel execution pattern ─────────────────────────────────────────
@Service
@RequiredArgsConstructor
public class OrderFulfillmentService {

    private final InventoryService inventory;
    private final PaymentService payment;
    private final NotificationService notification;
    private final Executor reportExecutor;

    public FulfillmentResult fulfil(Order order) {
        // Run inventory check and payment in parallel
        CompletableFuture<InventoryResult> invFuture =
            CompletableFuture.supplyAsync(
                () -> inventory.reserve(order.getItems()), reportExecutor);

        CompletableFuture<PaymentResult> payFuture =
            CompletableFuture.supplyAsync(
                () -> payment.charge(order), reportExecutor);

        // Join both — waits for both to complete
        CompletableFuture<FulfillmentResult> combined =
            invFuture.thenCombine(payFuture, (inv, pay) -> {
                if (!inv.isSuccess() || !pay.isSuccess()) {
                    return FulfillmentResult.failed(inv, pay);
                }
                return FulfillmentResult.success(inv, pay);
            });

        // Send notification async — fire and forget after fulfilment
        combined.thenAcceptAsync(result -> {
            if (result.isSuccess()) {
                notification.sendWelcomeEmail(order.getCustomer());
            }
        }, reportExecutor);

        // Error handling
        return combined
            .exceptionally(ex -> {
                log.error("Fulfillment failed for order {}", order.getId(), ex);
                return FulfillmentResult.error(ex.getMessage());
            })
            .join();  // block the HTTP thread only here, at the end
    }

    // Fan-out: process list of items in parallel, collect results
    public List<ProcessResult> processAll(List<OrderItem> items) {
        List<CompletableFuture<ProcessResult>> futures = items.stream()
            .map(item -> CompletableFuture.supplyAsync(
                () -> processItem(item), reportExecutor))
            .toList();

        return CompletableFuture.allOf(futures.toArray(new CompletableFuture[0]))
            .thenApply(v -> futures.stream()
                .map(CompletableFuture::join)
                .toList())
            .join();
    }
}
```

## ⚠️ Common Pitfalls & Anti-Patterns

- **Mistake:** Using the default `SimpleAsyncTaskExecutor` — creates a new thread per invocation, causes thread explosion under load.
  **Fix:** Always define a named `ThreadPoolTaskExecutor` bean and reference it in `@Async("beanName")`.

- **Mistake:** `@Async` on a `private` method or called from within the same class — Spring AOP bypasses proxies on self-calls.
  **Fix:** Extract to a separate Spring bean or inject `self`.

- **Mistake:** `@Async` with `void` return and no exception handling — exceptions are silently swallowed.
  **Fix:** Implement `AsyncUncaughtExceptionHandler` and/or wrap the method body in try/catch.

- **Mistake:** Blocking inside `CompletableFuture.supplyAsync()` with `RestTemplate` — blocks the common ForkJoinPool.
  **Fix:** Provide a dedicated `Executor` to `supplyAsync(fn, executor)` sized for I/O-bound blocking work.

- **Mistake:** Calling `.join()` inside a method annotated with `@Async` — nested blocking defeats the purpose.
  **Fix:** Chain `.thenApply()` / `.thenCompose()` to keep the entire pipeline non-blocking.

- **Mistake:** Losing `SecurityContext` in async threads — `SecurityContextHolder` is thread-local.
  **Fix:** Configure `SecurityContextHolder.setStrategyName(MODE_INHERITABLETHREADLOCAL)` or use `DelegatingSecurityContextExecutor` to wrap your executor.

## ❓ Frequently Asked Interview Questions

1. **Q:** What is the difference between `@Async` and `CompletableFuture.runAsync()`?
   - **A:** `@Async` is a Spring AOP annotation — Spring intercepts the method call and submits it to a configured `TaskExecutor`. `CompletableFuture.runAsync()` is a pure Java construct — you explicitly submit a lambda to an executor. Both run async, but `@Async` is more transparent; `CompletableFuture` gives you richer chaining and composition APIs.

2. **Q:** What executor does `@Async` use by default and why is it a problem?
   - **A:** `SimpleAsyncTaskExecutor` — creates a new OS thread per invocation, with no upper bound. Under load this causes thread explosion, context-switch overhead, and eventual OOM. Always configure a bounded `ThreadPoolTaskExecutor`.

3. **Q:** How do you run two async operations in parallel and wait for both?
   - **A:** `CompletableFuture.allOf(cf1, cf2).thenApply(v -> ...)` waits for both to complete. Use `.thenCombine(cf2, (r1, r2) -> ...)` when you need to combine exactly two results. Avoid `.get()` or `.join()` inside async chains — keep the pipeline non-blocking.

4. **Q:** How do you propagate the Spring Security context to `@Async` threads?
   - **A:** Set `SecurityContextHolder.setStrategyName(SecurityContextHolder.MODE_INHERITABLETHREADLOCAL)` — this copies the context to child threads via `InheritableThreadLocal`. Alternatively, wrap the executor with `new DelegatingSecurityContextExecutor(executor)`.

## 🔗 Related Notes
- [[Redis-Caching-Strategies]]
- [[Resiliency-Circuit-Breaker-Resilience4j]]
- [[Unit-And-Integration-Testing-Testcontainers]]
