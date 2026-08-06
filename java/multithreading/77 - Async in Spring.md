# 77 - Async in Spring

Tags: #frameworks #section13
Related: [[74 - Spring Boot Threading Model]] | [[49 - CompletableFuture]] | [[46 - ExecutorService]]
Back to: [[00 - Java Multithreading Home]]

---

## Objective
Master `@Async` in Spring — how to run methods on background threads, handle return values, propagate exceptions, and configure the executor.

---

## Beginner Explanation
`@Async` tells Spring: "run this method on a different thread from a thread pool, not the caller's thread." The caller gets back immediately (or gets a `Future`). The actual work happens in the background.

---

## Technical Deep Dive

### Setup
```java
@SpringBootApplication
@EnableAsync  // Required — enables @Async processing
public class App { public static void main(String[] args) { SpringApplication.run(App.class, args); } }
```

### Return types for `@Async` methods

| Return type | Behaviour |
|---|---|
| `void` | Fire-and-forget; exceptions go to `AsyncUncaughtExceptionHandler` |
| `Future<T>` | Caller can block with `.get()` |
| `CompletableFuture<T>` | Non-blocking chaining; exception propagation |
| `ListenableFuture<T>` | (Deprecated in Spring 6) |

### Exception handling

```java
// void async — exceptions are silently swallowed by default
// Configure a handler:
@Configuration
public class AsyncExceptionConfig implements AsyncConfigurer {
    @Override
    public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
        return (ex, method, params) ->
            log.error("Async exception in {}: {}", method.getName(), ex.getMessage());
    }
}

// CompletableFuture async — exception is captured in the future
@Async
public CompletableFuture<String> riskyTask() {
    if (errorCondition) throw new RuntimeException("failed");
    return CompletableFuture.completedFuture("ok");
}
// Caller:
future.exceptionally(ex -> { log.error("Failed", ex); return "default"; });
```

### Configuring the executor

```java
@Configuration
@EnableAsync
public class AsyncConfig implements AsyncConfigurer {
    @Bean("taskExecutor")
    public Executor taskExecutor() {
        ThreadPoolTaskExecutor exec = new ThreadPoolTaskExecutor();
        exec.setCorePoolSize(10);
        exec.setMaxPoolSize(50);
        exec.setQueueCapacity(500);
        exec.setThreadNamePrefix("async-exec-");
        exec.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
        exec.initialize();
        return exec;
    }

    @Override
    public Executor getAsyncExecutor() { return taskExecutor(); } // default executor
}
```

### Common pitfalls

**Pitfall 1 — Self-invocation (same bean)**
```java
@Service
public class MyService {
    @Async
    public void asyncMethod() { ... }

    public void callerMethod() {
        asyncMethod(); // WRONG — called directly, not through proxy — @Async ignored!
    }
}
```
Fix: inject `ApplicationContext` and get `self` as a bean reference, or split into two beans.

**Pitfall 2 — `@Async` on private methods**
Spring AOP only intercepts public methods. `@Async` on private methods is silently ignored.

**Pitfall 3 — Transaction context not propagated**
`@Transactional` context is thread-bound. `@Async` methods run on a different thread — they start with no transaction. They need their own `@Transactional` annotation.

---

## Code

```java
@Service
public class ReportService {

    @Async("taskExecutor")
    public CompletableFuture<Report> generateReport(Long reportId) {
        System.out.println("Generating on: " + Thread.currentThread().getName());
        // heavy computation / I/O
        Report report = buildReport(reportId);
        return CompletableFuture.completedFuture(report);
    }

    // Combine multiple async results
    public Report generateCombined(Long id1, Long id2) throws Exception {
        CompletableFuture<Report> f1 = generateReport(id1);
        CompletableFuture<Report> f2 = generateReport(id2);
        CompletableFuture.allOf(f1, f2).join(); // wait for both
        return merge(f1.get(), f2.get());
    }

    Report buildReport(Long id) { return new Report(id); }
    Report merge(Report a, Report b) { return a; }
    record Report(Long id) {}
}

@RestController
public class ReportController {
    @Autowired ReportService service;

    @GetMapping("/reports/{id}")
    public CompletableFuture<Report> getReport(@PathVariable Long id) {
        // Controller returns CompletableFuture → Spring MVC releases the request thread immediately
        return service.generateReport(id);
    }
}
```

---

## Interview Questions

**Q: Why does `@Async` not work when called from the same class?**
> Spring `@Async` works via AOP proxies. When you call a method on `this` directly, you bypass the proxy and the method runs synchronously on the calling thread. Call through the Spring-managed bean reference instead.

**Q: How do you handle exceptions in void `@Async` methods?**
> Implement `AsyncUncaughtExceptionHandler` and register it via `AsyncConfigurer.getAsyncUncaughtExceptionHandler()`. For `CompletableFuture` return types, exceptions are captured in the future and accessible via `.exceptionally()`.

**Q: Does `@Transactional` propagate to an `@Async` method?**
> No. Each thread has its own transaction context via `ThreadLocal`. The `@Async` method starts fresh — add its own `@Transactional` annotation if needed.

---

## Cheat Sheet
| Aspect | Detail |
|---|---|
| Enable | `@EnableAsync` on config class |
| Method scope | Public methods only |
| Self-call | ❌ Bypasses proxy — @Async ignored |
| Return `void` | Fire-and-forget; exceptions need handler |
| Return `CompletableFuture<T>` | Best — composable, exception-aware |
| Transaction | Not inherited — add `@Transactional` separately |
| Executor config | `AsyncConfigurer.getAsyncExecutor()` |
