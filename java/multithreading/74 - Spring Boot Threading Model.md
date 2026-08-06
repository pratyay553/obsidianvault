# 74 - Spring Boot Threading Model

Tags: #frameworks #section13
Related: [[75 - Tomcat Thread Pool]] | [[77 - Async in Spring]] | [[60 - ThreadLocal]] | [[44 - Thread Pool]]
Back to: [[00 - Java Multithreading Home]]

---

## Objective
Understand how Spring Boot handles threading — from request handling to async tasks and background jobs.

---

## Beginner Explanation
Spring Boot (with Tomcat) assigns one thread per HTTP request. That thread handles the whole request lifecycle: controller, service, repository, response. Spring provides `@Async` and `ThreadPoolTaskExecutor` to move work off the request thread.

---

## Technical Deep Dive

### Default model: one thread per request
```
HTTP Request → Tomcat thread pool → DispatcherServlet → Controller → Service → DB → Response
```
One platform thread handles the entire lifecycle of one HTTP request. Default Tomcat pool: 200 threads → 200 concurrent requests max. → [[75 - Tomcat Thread Pool]]

### Thread-local context in Spring

Spring uses `ThreadLocal` internally for:
- **Security context**: `SecurityContextHolder` — stores the authenticated user per request thread
- **Transaction context**: `TransactionSynchronizationManager` — current transaction bound to thread
- **Request attributes**: `RequestContextHolder` — current `HttpServletRequest`

⚠️ These break when you move work to another thread! The new thread has no security/transaction context.

```java
// BROKEN — SecurityContext not propagated to async thread
@Async
public void asyncTask() {
    SecurityContext ctx = SecurityContextHolder.getContext(); // null or empty!
}

// FIX — use DelegatingSecurityContextRunnable or configure executor
@Bean
public Executor taskExecutor() {
    ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
    executor.setCorePoolSize(10);
    executor.setMaxPoolSize(50);
    executor.setThreadNamePrefix("async-");
    executor.initialize();
    return new DelegatingSecurityContextAsyncTaskExecutor(executor);
}
```

### Spring's threading components

| Component | Purpose |
|---|---|
| `ThreadPoolTaskExecutor` | Spring-managed thread pool for `@Async` |
| `@Async` | Run method in a thread pool → [[77 - Async in Spring]] |
| `@Scheduled` | Scheduled tasks on a configurable scheduler |
| `TaskScheduler` | Spring abstraction for scheduled execution |
| `SimpleAsyncTaskExecutor` | Warning: creates a new thread per task — do not use in prod |

### Configuring `ThreadPoolTaskExecutor`
```java
@Configuration
@EnableAsync
public class AsyncConfig {
    @Bean(name = "taskExecutor")
    public Executor taskExecutor() {
        ThreadPoolTaskExecutor exec = new ThreadPoolTaskExecutor();
        exec.setCorePoolSize(10);
        exec.setMaxPoolSize(50);
        exec.setQueueCapacity(100);
        exec.setKeepAliveSeconds(60);
        exec.setThreadNamePrefix("my-async-");
        exec.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
        exec.initialize();
        return exec;
    }
}
```

### Virtual threads in Spring Boot 3.2+
```yaml
# application.yml — enable virtual threads for Tomcat (Java 21+)
spring:
  threads:
    virtual:
      enabled: true
```
With virtual threads enabled, Tomcat uses one virtual thread per request — no pool size limit needed.

---

## Code

```java
@Service
public class OrderService {

    @Async("taskExecutor")
    public CompletableFuture<String> processOrderAsync(Long orderId) {
        // Runs on taskExecutor pool, not the request thread
        System.out.println("Processing on: " + Thread.currentThread().getName());
        // heavy work
        return CompletableFuture.completedFuture("Order " + orderId + " processed");
    }

    @Transactional
    public void processOrder(Long orderId) {
        // Transaction context bound to this thread via ThreadLocal
        // DO NOT pass this thread's context to another thread manually
        orderRepository.save(new Order(orderId));
        // If you call an @Async method here, the new thread WON'T share this transaction
    }
}

@RestController
public class OrderController {
    @Autowired OrderService orderService;

    @GetMapping("/orders/{id}")
    public DeferredResult<String> getOrder(@PathVariable Long id) throws Exception {
        DeferredResult<String> result = new DeferredResult<>(5000L);
        CompletableFuture<String> future = orderService.processOrderAsync(id);
        future.thenAccept(result::setResult)
              .exceptionally(ex -> { result.setErrorResult(ex); return null; });
        return result; // request thread freed immediately
    }
}
```

---

## Interview Questions

**Q: How does Spring Boot handle concurrent HTTP requests by default?**
> Each request is handled by a dedicated Tomcat thread from its thread pool (default 200). The thread is returned to the pool when the response is sent.

**Q: Why does `SecurityContextHolder` return null in an `@Async` method?**
> `SecurityContextHolder` uses `ThreadLocal` by default. `@Async` runs on a different thread with a fresh, empty `ThreadLocal`. Fix: use `DelegatingSecurityContextAsyncTaskExecutor` or configure the `SecurityContextHolder` strategy.

**Q: What is the risk of using `SimpleAsyncTaskExecutor` in production?**
> It creates a new OS thread for every task submitted — no pooling. Under load this creates thousands of threads, consuming memory and causing excessive context switching.

---

## Cheat Sheet
| Concern | Solution |
|---|---|
| Request threads | Tomcat pool (default 200) |
| Async tasks | `@Async` + `ThreadPoolTaskExecutor` |
| Security in async | `DelegatingSecurityContextAsyncTaskExecutor` |
| Transaction in async | New transaction per async method |
| Virtual threads | `spring.threads.virtual.enabled=true` (Spring Boot 3.2+, Java 21) |
| Scheduled tasks | `@Scheduled` + `TaskScheduler` |
