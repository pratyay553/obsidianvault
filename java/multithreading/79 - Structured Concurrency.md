# 79 - Structured Concurrency

Tags: #frameworks #section13 #java21
Related: [[78 - Virtual Threads (Project Loom)]] | [[49 - CompletableFuture]] | [[07 - Future]]
Back to: [[00 - Java Multithreading Home]]

---

## Objective
Understand Structured Concurrency (Java 21 preview, Java 23 standard) — a programming model that treats a group of concurrent tasks as a single unit of work with clean lifecycle and error handling.

---

## Beginner Explanation
In unstructured concurrency, you `submit()` tasks to a pool and manage `Future`s manually — if one task fails, you must remember to cancel the others. Structured concurrency ties the lifetime of child tasks to the enclosing scope: when the scope exits (normally or by exception), all child tasks are cancelled automatically.

---

## Analogy
A project manager (parent scope) hires two consultants (child tasks). In unstructured concurrency, if Consultant A quits, the manager must manually tell Consultant B to stop. In structured concurrency, when the project scope ends (for any reason), both consultants are automatically let go.

---

## Technical Deep Dive

### The problem with unstructured concurrency

```java
// UNSTRUCTURED — manual, error-prone
Future<User> userFuture = pool.submit(() -> fetchUser(id));
Future<Order> orderFuture = pool.submit(() -> fetchOrder(id));

try {
    User user = userFuture.get();
    Order order = orderFuture.get();
} catch (Exception e) {
    userFuture.cancel(true);   // easy to forget
    orderFuture.cancel(true);  // easy to forget
    throw e;
}
```

### StructuredTaskScope — Java 21+

```java
import java.util.concurrent.StructuredTaskScope;

try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
    Subtask<User>  user  = scope.fork(() -> fetchUser(id));
    Subtask<Order> order = scope.fork(() -> fetchOrder(id));

    scope.join();           // wait for both
    scope.throwIfFailed();  // re-throw if either failed

    return new Response(user.get(), order.get());
} // scope.close() — cancels any unfinished tasks automatically
```

### Built-in policies

**`ShutdownOnFailure`** — if any task fails, all others are cancelled:
```java
try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
    var t1 = scope.fork(task1);
    var t2 = scope.fork(task2);
    scope.join().throwIfFailed(); // cancel all if any fails
    return merge(t1.get(), t2.get());
}
```

**`ShutdownOnSuccess`** — as soon as any task succeeds, cancel the rest:
```java
try (var scope = new StructuredTaskScope.ShutdownOnSuccess<String>()) {
    scope.fork(() -> queryPrimary());
    scope.fork(() -> queryReplica());
    scope.join();
    return scope.result(); // first successful result
}
```

### Custom policies
Extend `StructuredTaskScope` and override `handleComplete(Subtask<? extends T> subtask)` to implement custom logic (e.g., majority quorum, collecting all results).

### Subtask states
- `UNAVAILABLE` — task not yet complete
- `SUCCESS` — completed successfully (call `subtask.get()`)
- `FAILED` — threw an exception (call `subtask.exception()`)

### Structured concurrency + virtual threads
Structured concurrency is designed to work with virtual threads. Each `fork()` creates a virtual thread — cheap, so you can fork thousands.

### Java version status
| Version | Status |
|---|---|
| Java 19–20 | Preview (incubator) |
| Java 21 | Preview |
| Java 23+ | Standard (non-preview) |

Enable preview: `--enable-preview --release 21`

---

## Code

```java
import java.util.concurrent.*;

public class StructuredConcurrencyDemo {
    record User(String name) {}
    record Order(int total) {}
    record Response(User user, Order order) {}

    static User  fetchUser(long id)  throws InterruptedException { Thread.sleep(100); return new User("Alice"); }
    static Order fetchOrder(long id) throws InterruptedException { Thread.sleep(150); return new Order(250); }
    static String queryPrimary() throws InterruptedException { Thread.sleep(200); return "primary result"; }
    static String queryReplica() throws InterruptedException { Thread.sleep(50);  return "replica result"; }

    // ShutdownOnFailure — both must succeed
    static Response fetchBoth(long id) throws Exception {
        try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
            StructuredTaskScope.Subtask<User>  user  = scope.fork(() -> fetchUser(id));
            StructuredTaskScope.Subtask<Order> order = scope.fork(() -> fetchOrder(id));
            scope.join().throwIfFailed();
            return new Response(user.get(), order.get());
        }
    }

    // ShutdownOnSuccess — first to succeed wins
    static String fetchFastest() throws Exception {
        try (var scope = new StructuredTaskScope.ShutdownOnSuccess<String>()) {
            scope.fork(StructuredConcurrencyDemo::queryPrimary);
            scope.fork(StructuredConcurrencyDemo::queryReplica);
            return scope.join().result();
        }
    }

    public static void main(String[] args) throws Exception {
        long start = System.currentTimeMillis();
        Response r = fetchBoth(1L);
        System.out.println("Both: " + r + " in " + (System.currentTimeMillis()-start) + "ms");
        // ~150ms (parallel), not 250ms (sequential)

        start = System.currentTimeMillis();
        String fastest = fetchFastest();
        System.out.println("Fastest: " + fastest + " in " + (System.currentTimeMillis()-start) + "ms");
        // ~50ms (replica wins)
    }
}
```

---

## Interview Questions

**Q: What problem does structured concurrency solve?**
> Manual `Future` management is error-prone — easy to forget to cancel sibling tasks on failure. Structured concurrency ties child task lifetimes to their enclosing scope, ensuring all children are cancelled when the scope exits.

**Q: What is the difference between `ShutdownOnFailure` and `ShutdownOnSuccess`?**
> `ShutdownOnFailure` cancels all tasks if any one fails (fail-fast). `ShutdownOnSuccess` cancels all remaining tasks as soon as any one succeeds (race for fastest result).

**Q: How does structured concurrency relate to virtual threads?**
> They're designed together. Each `scope.fork()` spawns a virtual thread — cheap enough to do thousands of times. Without virtual threads, forking thousands of tasks would exhaust the OS thread limit.

---

## Cheat Sheet
| API | Behaviour |
|---|---|
| `StructuredTaskScope.ShutdownOnFailure` | Cancel all if any fails |
| `StructuredTaskScope.ShutdownOnSuccess<T>` | Cancel rest when first succeeds |
| `scope.fork(callable)` | Start a subtask (virtual thread) |
| `scope.join()` | Wait for all (or policy trigger) |
| `scope.throwIfFailed()` | Re-throw first exception |
| `scope.result()` | Get first successful result (ShutdownOnSuccess) |
| Auto-cancel | ✅ On scope exit (try-with-resources) |
