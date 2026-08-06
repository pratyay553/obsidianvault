# 60 - ThreadLocal

Tags: #thread-local-storage #section10
Related: [[61 - InheritableThreadLocal]] | [[74 - Spring Boot Threading Model]] | [[75 - Tomcat Thread Pool]]
Back to: [[00 - Java Multithreading Home]]

---

## Objective
Master `ThreadLocal<T>` — a mechanism to give each thread its own independent copy of a variable without synchronization.

---

## Beginner Explanation
A `ThreadLocal` variable looks like one variable but is actually a separate variable per thread. Thread A's value never interferes with Thread B's value. No synchronization needed because there's no sharing.

---

## Analogy
A locker room: one locker per person. Everyone uses "my locker" but each person has their own physical locker. There's no conflict because they're not sharing.

---

## Technical Deep Dive

### Internal structure
Each `Thread` object holds a `ThreadLocalMap` (a custom hash map). When you call `threadLocal.get()`, it looks up the current thread's map with the `ThreadLocal` instance as the key. No cross-thread access ever occurs.

```
Thread A  →  ThreadLocalMap { threadLocal → valueA }
Thread B  →  ThreadLocalMap { threadLocal → valueB }
```

### API
```java
ThreadLocal<T> tl = new ThreadLocal<>();
ThreadLocal<T> tl = ThreadLocal.withInitial(() -> defaultValue);

tl.set(value);    // store value for current thread
tl.get();         // retrieve current thread's value
tl.remove();      // remove current thread's value — IMPORTANT
```

### Classic use cases
1. **Database connection / transaction context** — each request thread holds its own `Connection`
2. **Security context** — current user stored per-thread (Spring Security's `SecurityContextHolder`)
3. **Request context** — HTTP request data per thread (Tomcat, Spring MVC)
4. **SimpleDateFormat** — not thread-safe; store one per thread
5. **Random/UUID generation** — thread-local seeds

### ⚠️ Memory leak warning (Thread Pool hazard)
In thread pools, threads are **reused**. If you call `set()` but forget `remove()`, the value lingers in the thread's map forever — the thread pool thread never dies, so the value is never GC'd.

```java
// DANGER — value stays in pool thread after request ends
threadLocal.set(requestContext);
handleRequest();
// forgot threadLocal.remove() → memory leak + stale data for next request
```

**Always call `remove()` in a `finally` block when using `ThreadLocal` with pooled threads.**

---

## Code

```java
public class ThreadLocalDemo {

    // Per-thread formatter (SimpleDateFormat is not thread-safe)
    static ThreadLocal<java.text.SimpleDateFormat> dateFormat =
        ThreadLocal.withInitial(() -> new java.text.SimpleDateFormat("yyyy-MM-dd"));

    // Per-thread request ID
    static ThreadLocal<String> requestId = new ThreadLocal<>();

    static void handleRequest(String id) {
        requestId.set(id);
        try {
            System.out.println(Thread.currentThread().getName()
                + " processing request: " + requestId.get());
            String formatted = dateFormat.get().format(new java.util.Date());
            System.out.println(Thread.currentThread().getName()
                + " date: " + formatted);
        } finally {
            requestId.remove(); // MUST remove in thread pools
        }
    }

    public static void main(String[] args) throws InterruptedException {
        java.util.concurrent.ExecutorService pool =
            java.util.concurrent.Executors.newFixedThreadPool(3);

        for (int i = 0; i < 6; i++) {
            final String reqId = "REQ-" + i;
            pool.submit(() -> handleRequest(reqId));
        }

        pool.shutdown();
        pool.awaitTermination(5, java.util.concurrent.TimeUnit.SECONDS);
    }
}
```

---

## Interview Questions

**Q: What is `ThreadLocal` and when do you use it?**
> `ThreadLocal` provides per-thread variable copies. Use it when you need thread-specific state without synchronization: database connections, security contexts, per-request logging MDC, non-thread-safe utilities like `SimpleDateFormat`.

**Q: What is the memory leak risk with `ThreadLocal` in thread pools?**
> Thread pool threads are reused; they never die. If `ThreadLocal.remove()` is not called, the value stays in the thread's `ThreadLocalMap` indefinitely. The object is still referenced, so GC cannot collect it.

**Q: How is `ThreadLocal` implemented internally?**
> Each `Thread` object contains a `ThreadLocalMap`. `threadLocal.get()` looks up the current thread's map using the `ThreadLocal` instance as the key. No synchronization needed — only the owning thread ever accesses its map.

**Q: Is `ThreadLocal` compatible with virtual threads (Java 21+)?**
> Technically yes, but it's discouraged for virtual threads. With millions of virtual threads, per-thread copies explode memory. Java 21 introduces **Scoped Values** as the preferred alternative for virtual threads.

---

## Cheat Sheet
| Method | Description |
|---|---|
| `ThreadLocal.withInitial(() -> v)` | Create with default value |
| `set(v)` | Store for current thread |
| `get()` | Retrieve current thread's value |
| `remove()` | ⚠️ Always call in finally with thread pools |
| Synchronization needed | ❌ No — no sharing |
| Memory leak risk | ✅ Yes — in thread pools if not removed |
