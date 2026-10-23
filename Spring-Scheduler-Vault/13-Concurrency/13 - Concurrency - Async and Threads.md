---
tags: [concurrency, async, interview]
---

# Concurrency — @Async, Threads, Thread Pools

Related: [[02 - Virtual Threads and SimpleAsyncTaskScheduler|02 - Virtual Threads and SimpleAsyncTaskScheduler]] · [[01 - Spring Scheduling Basics]] · [[20 - Interview Cheat Sheet]]

## 🧩 What it is
Beyond scheduling *when* something runs, `@Async` lets a method run **on a separate thread**, so the caller doesn't block waiting for it — useful for fire-and-forget work triggered from inside a scheduled job (e.g., sending a notification while continuing other processing).

## 🔑 Core Pieces

| Component | Role |
|---|---|
| `@Async` | Marks a method to run on a separate thread instead of the caller's thread |
| `@EnableAsync` | Turns on `@Async` processing for the app (parallel to `@EnableScheduling`) |
| `Thread pools` (`ThreadPoolTaskExecutor`) | Bounds how many concurrent async tasks can run, reusing threads instead of creating a new OS thread per call |
| Virtual Threads | Modern alternative to thread pools for I/O-bound async work — see [[02 - Virtual Threads and SimpleAsyncTaskScheduler]] |

## 🛠️ Code

```java
@Configuration
@EnableAsync
public class AsyncConfig {
    @Bean
    public Executor taskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);
        executor.setMaxPoolSize(10);
        executor.setQueueCapacity(50);
        executor.setThreadNamePrefix("async-");
        executor.initialize();
        return executor;
    }
}

@Service
public class NotificationService {
    @Async
    public CompletableFuture<Void> notifyAsync(String message) {
        // runs on a pooled thread, not the caller's
        sendPushNotification(message);
        return CompletableFuture.completedFuture(null);
    }
}
```

## ⚠️ Common Gotchas (great interview material)
- **Self-invocation**: calling `this.notifyAsync()` from within the *same class* bypasses the proxy — it runs synchronously, same as the [[04 - AOP - Aspect Oriented Programming]] self-invocation issue, because both `@Async` and AOP advice rely on Spring's proxy mechanism.
- **Return type**: `@Async` methods should return `void`, `Future<T>`, or `CompletableFuture<T>` — never a plain `T`, since the method returns immediately and the real result isn't ready yet.
- **Exceptions in `void @Async` methods are swallowed** unless you configure an `AsyncUncaughtExceptionHandler` — they won't propagate to the caller since the caller already moved on.

## ❓ Interview Q&A

**Q: What happens if you call an `@Async` method and don't configure a custom `Executor`?**
A: Spring falls back to a `SimpleAsyncTaskExecutor`, which creates a **new thread for every single call** — no pooling, no reuse, no bound on concurrency. This is fine for light usage but dangerous under load (unbounded thread creation). Always define a bounded `ThreadPoolTaskExecutor` (or virtual threads) for production.

**Q: How do you handle an exception thrown inside a `void`-returning `@Async` method?**
A: Implement `AsyncConfigurer` and override `getAsyncUncaughtExceptionHandler()` to supply a custom `AsyncUncaughtExceptionHandler` — otherwise the exception is just logged by Spring's default handler and effectively lost to the caller.

**Q: `@Async` vs a manually submitted `ExecutorService.submit()`?**
A: `@Async` is declarative — Spring wires the proxy/executor for you, keeping business code clean. Manual `ExecutorService` usage gives more explicit control (e.g., waiting on multiple futures, custom shutdown handling) but requires more boilerplate.

**Q: Why might virtual threads make `ThreadPoolTaskExecutor` tuning less critical?**
A: Because virtual threads are cheap to create, you often don't need a bounded pool sized to hardware threads at all for I/O-bound tasks — you can use a virtual-thread-per-task executor and let the JVM handle scheduling, avoiding the classic "pick the right pool size" tuning problem.
